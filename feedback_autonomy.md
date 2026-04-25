---
name: Autonomy granted for install/delete in personal RE workspace
description: User granted blanket authorization to install/download/delete without asking, scoped to local personal game RE workspace
type: feedback
originSessionId: 739a471f-c622-43c3-b869-927876ec61e8
---
User cho phép tự ý cài, tải, xóa file mà KHÔNG cần hỏi xác nhận, trong phạm vi workspace cá nhân của họ.

**Why:** Workspace `D:\Code Tools\` là môi trường RE cá nhân của user, mọi thứ là local + có thể tái tạo. Hỏi xác nhận cho từng `pip install` / xóa file extract / tải frida-server làm chậm việc và phiền user. User chủ động grant "quyền cao nhất" sau khi cài lại Windows.

**How to apply:**
- Tự làm không hỏi: cài/gỡ tool RE (frida, IDA plugins, apktool, BepInEx, Il2CppDumper), pip/npm install, tải APK/binaries/samples, xóa file tạm + build artifacts + extract folders, xóa/ghi đè trong `D:\Code Tools\` và các project con.
- VẪN hỏi trước (vì rủi ro mất dữ liệu hoặc ngoài scope):
  - Xóa thứ ngoài `D:\Code Tools\` (system folder Windows, ổ khác)
  - Xóa IDA database `.i64/.idb` đã phân tích sâu (mất công sức RE)
  - Xóa source code bot đang chạy production (`MOBILE_GAMES\main.py`, `core/`, `scripts/`, ListAcc.txt)
  - `rm -rf` ở drive root, format, registry edits
  - Push code lên remote, share data ra ngoài, gửi tin nhắn
  - Thao tác LDPlayer có thể làm mất account đang login
- Authorization scope = workspace local cá nhân. Không tự suy ra cho production / shared systems / accounts của người khác.
