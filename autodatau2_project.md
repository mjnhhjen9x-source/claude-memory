---
name: AutoDaTau 2.0 Project
description: Python + Frida automation project for KHTD Mobile — full quest flow, multi-tab, 3000 acc
type: project
---

## AutoDaTau 2.0
- **Path**: `D:\Code Tools\AutoDaTau2.0\`
- **Stack**: Python 3.14 + Frida 17.7.3
- **Emulator**: LDPlayer 5.0.13, Android 7.1.2, 600x360, DPI 100
- **Game**: `com.growx.kht1` (Cocos2d-x)
- **Acc**: 2999 acc (`haisanlk1`→`haisanlk1000`+), pass chung `ongnoi12`, file `ListAcc.txt`

**Why:** Thay thế hoàn toàn C# image matching (AutoVL1). Python là host tự nhiên cho Frida.

**How to apply:** Khi user nhắc AutoDaTau hoặc auto quest, tham chiếu project này.

## Login Flow (ĐÃ TEST THÀNH CÔNG 2026-03-13)
1. ADB connect → fallback `emulator-5554` (TCP `127.0.0.1:5555` offline trên LDPlayer)
2. Frida USB attach (`frida.get_usb_device()`)
3. `detectScreen` → "login" (NPC array fallback, không dùng heap scan)
4. Login: Frida RPC nếu hooks đã capture KuiLogin, else **ADB tap fallback** (combined shell command)
5. `detectScreen` → "select_char" (via cached KuiSelPlayer hook)
6. Enter game: Frida RPC `enterGame()` via `runOnMainThread()`
7. `detectScreen` → "in_game" (coords > 0)

### Kỹ thuật quan trọng
- **Heap scan gây CRASH game** → đã bỏ khỏi detectScreen và doLogin/doEnterGame
- **ADB `input tap` treo 30s+ trên LDPlayer** → dùng `Popen` fire-and-forget
- **ADB login**: gộp tất cả commands thành 1 shell call (`&&` chain) để chạy tuần tự, wait 60s
- **Hooks capture UI**: 20 hooks KuiLogin + 20 hooks KuiSelPlayer, tự capture `this` pointer khi method gọi
- **Frida camelCase→snake_case**: `FridaManager._to_snake_case()` converter

### ADB Coordinates (600x360 DPI 100)
- input_id: (250, 147), input_pass: (250, 193)
- btn_login: (372, 244), close_kb: (25, 65)
- btn_enter: (530, 321)

## Flow 1 acc
1. Login (ADB tap fallback) hoặc skip nếu manual mode
2. Inject Frida → load agent.js
3. Di chuyển đến NPC Dã Tẩu (typeId=10205, map Dương Châu=80)
4. Loop đến 15/15:
   - Dialog → đọc NV text
   - "nâng thêm ... kinh nghiệm" → do_exp()
   - "tìm {Cá ..." → do_fish()
   - "... khoáng ..." → do_mine()
   - Khác → HỦY (tag=3) → lặp (không tính lượt)
   - NV xong → quay lại Dã Tẩu → count++
5. 15/15 → DONE → logout → đổi acc

## 2 Mode
- **Auto full**: Login từ list → làm NV → mark_done → logout → đổi acc
- **Manual**: Lần đầu skip login (acc sẵn, KHÔNG mark_done) → logout → lần 2+ chạy như Auto full

## Quản lý acc
- `ListAcc.txt`: mỗi dòng 1 ID, pass chung `ongnoi12`
- `accounts_status.json`: lưu trạng thái (done/failed/pending), tự resume khi restart
- Thread-safe: multi-tab gọi next() không trùng acc
- mark_done CHỈ khi tool tự login acc đó

## Cấu trúc
```
AutoDaTau2.0/
├── main.py, config.json, ListAcc.txt, requirements.txt
├── core/ — adb (fire-and-forget), frida_manager (USB+snake_case), encoding, logger
├── game/ — state, player, items, mining, quest, detector, constants
├── account/ — manager (ListAcc.txt + status), login (Frida+ADB fallback)
├── tasks/ — session, quest_datau, stats, do_exp, do_fish, do_mine
├── scripts/ — agent.js (Frida JS rpc.exports, NO heap scan)
├── test_login.py, test_debug_screen.py
└── logs/
```

## UseItem Packet (CẬP NHẬT 2026-03-14)
- Packet format: `[0x5A, id0, id1, id2, 0x00, pos, 0x00, byte7, byte8]`
- Bytes 1-3: item genre/ID từ offset +0x4E0 trong KItem struct
- Byte 5: UseItemPos = 3 (bag)
- **byte7/byte8: vị trí item trong grid UI, 5 cột × 6 hàng**

### KPlayer::UseItem — Disassembly Findings
- **UseItemPos KHÔNG phải int — là struct 16 bytes (4 int32)**
  - Struct layout: `{row, 0, col, type}` (qua pshufb mask analysis)
  - pshufb mask: `[0x0c, 0x04, 0x08, 0x00, ...]`
    - packet byte5 = struct[12] (type), byte6 = struct[4] (0), byte7 = struct[8] (col), byte8 = struct[0] (row)
  - **Game UI (caller) truyền grid position vào struct, UseItem KHÔNG tự tính**
  - Gọi từ Frida NativeFunction chỉ pass int=3 → byte7/byte8 là rác từ stack
- **Computed formula KHÔNG ĐÁNG TIN** — test 3+ acc, formula sai và **dùng nhầm item khác**!

### Giải pháp: Inject Frida TRƯỚC khi vào game (VERIFIED 2026-03-14)
- **`KItemList::Add(slot, tab, col, row, 0)`** hook capture grid positions
- Hook chỉ fire khi items được load (lúc enter game) hoặc nhận item mới
- **Flow bắt buộc:** Inject Frida → Login → Enter game → KItemList::Add fires → `_gridPositions` có đủ
- **Test:** 15/16 items captured thành công, grid positions đều đúng
- **Items mới** (Cần câu nhận trong game): KItemList::Add cũng fire → tự capture
- `useItem` RPC đã sửa: dùng `sendPacket(0x5A)` + `_gridPositions` (fallback computed)
- Flow: useItem(slot) → SyncScriptAction (dialog) → selectOption(tag) → nhận item

## Quest Flow (SKELETON 2026-03-14)
### Quest types
- **EXP** ("nâng thêm...kinh nghiệm"): Dùng cần câu cá → câu = tăng exp. **Chỉ 1 lần/loop**, hủy nếu đã làm hoặc không có cần.
- **FISH** ("tìm {Cá..."): Cá đã nằm trong Giỏ cá → UseItem(Giỏ cá) → chọn ngăn → check qty >= 3 → lấy ra → nộp Dã Tẩu
- **MINE** ("khoáng"): Check túi có khoáng >= 3 → nộp luôn. Nếu không: mua Thổ Địa Phù → Xa Phu → Tương Dương → đào → phù về → nộp

### Submit flow
- Quay lại Dã Tẩu → dialog → TAG_SUBMIT (1) → game tự kiểm tra item

### Files
- `tasks/do_exp.py`, `tasks/do_fish.py`, `tasks/do_mine.py`
- `tasks/session.py` — quest loop với `exp_done` flag, `_execute_quest()`

## Cần làm tiếp
1. ~~Login flow~~ ✅ DONE
2. ~~UseItem + selectOption~~ ✅ DONE
3. ~~Grid position mapping (KItemList::Add hook)~~ ✅ DONE
4. ~~Quest skeleton (do_exp/fish/mine + session integration)~~ ✅ DONE
5. Cập nhật tọa độ: fishing spot, NPC shop, Xa Phu, map Tương Dương
6. Implement chi tiết: Giỏ cá dialog, lấy cá ra, đào khoáng flow
7. Nộp quest (verify TAG_SUBMIT flow)
8. Logout → đổi acc (logout đã có, cần test flow đổi acc)
9. Multi-tab support
