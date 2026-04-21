---
name: M4VN Progress - Full Auto Bot + Headless Client
description: M4VN auto bot full pipeline: login + tanthu + sell + delete + multi-instance + headless TCP client
type: project
originSessionId: 05a683ce-7fc5-430d-b086-61b4424e86f8
---
## Trang thai: HOAN THANH - Full auto bot multi-instance

## Binary version
Dung `Main_unpacked_v2.exe` (IDA db). ImageBase 0x400000.
Cac dia chi o memory cu (0x8B2FC0 CreateChar etc) la **cua version cu, khong dung nua**.

## Dia chi function (v2 unpacked binary)
| Function | VA | RVA | Args | Packet |
|---|---|---|---|---|
| CreateChar | 0x8B18D0 | 0x4B18D0 | (name, class_hi, class_lo) | F3 0C |
| DeleteChar | 0x8B3000 | 0x4B3000 | (name, data20_zeros_ptr) | F3 0F |
| Logout | 0x8BA710 | 0x4BA710 | (byte) | F1 0D |
| RequestCharList | 0x8B0FD0 | 0x4B0FD0 | () | F3 0D |
| EnterGame (ham)| 0x8B9990 | 0x4B9990 | (char* name) | F3 0E |
| SelectServer | 0x8C0840 | 0x4C0840 | (uint16 server_id) | F4 03 |
| Login | 0x8B9F60 | 0x4B9F60 | (user, pass, flag_byte) | F1 0E |
| SendSell | 0x7FD5E0 | 0x3FD5E0 | (item_code, tab_id) stdcall | |
| NpcClick | 0x8C1CA0 | 0x4C1CA0 | () | |
| Interact | 0x8ADD30 | 0x4ADD30 | (a1=122, a2=0/3/4) | |
| chatFunc | 0x8ABA30 | 0x4ABA30 | (6 uint32 = wstring) | 0D 00 |
| GetCharBySlot | 0x50D0D0 | 0x10D0D0 | (slot) → char struct ptr; name @ +118 | |
| SendPacket (low) | 0x8AD160 | 0x4AD160 | (src, size, enc, a4) | |

## State vars (global)
- dword_11BA330 = slot index (selected char)
- dword_11BA320 = scene ID (2=login, 4=char select, 5=in-game)
- dword_63C72C8 = scene state (20=list request, 61=in-game enter, 56=delete, ...)
- byte_A395D73 = scene init flag (reset = 0 to force re-init)

## Flow chinh (da giai quyet van de scene transition)
**EnterGame KHONG goi sub_8B9990 truc tiep**. Chi set state:
```
dword_11BA330 = slot
byte_A395D73 = 0      // force init
dword_63C72C8 = 61    // in-game state
dword_11BA320 = 5     // in-game scene
```
Game thread next frame tu chay sub_A577C0 → goi sub_8B9990. Goi truc tiep bi conflict thread.

**Multi-instance bypass**: bypass_v5.js patch sub_7E66E0 (RVA 0x3E66E0) → return 1. Cho phep spawn nhieu game cung luc.

## Cac buoc test da qua
1. ✅ CreateChar - F3 0C packet (15 byte)
2. ✅ EnterGame passive scene switch
3. ✅ /tanthu + sell (chatFunc + NpcClick + Interact + SendSell)
4. ✅ Logout - F1 0D packet
5. ✅ DeleteChar - F3 0F packet (34 byte, data20 = zeros, khong can password)
6. ✅ Login - SelectServer + Login with user/pass
7. ✅ Multi-instance: N account chay song song, stagger 3s

## File chinh (HIEN TAI)
- `D:\Code Tools\PC_GAMES\M4VN\bot_ui.py` - UI multi-instance (PRIMARY)
- `D:\Code Tools\PC_GAMES\M4VN\bot_loop.js` - Frida RPC (all functions)
- `D:\Code Tools\PC_GAMES\M4VN\bypass_v5.js` - bypass PhoenixCheat (da them multi-instance patch)
- `D:\Code Tools\PC_GAMES\M4VN\dist\M4VN_Bot.exe` - PyInstaller bundle

## Tool features
- Load accounts tu file .txt (hoac nhap vao textarea)
- Password chung tat ca account
- Server ID configurable (default 4)
- Class selector (DW/DK/FE/MG/DL)
- Prefix ten bot (default "Bot") - safety: KHONG xoa char khong bat dau bang prefix
- Vong/login = 0 = vo han
- Stagger 3s giua moi spawn
- Auto-recovery: game crash → detect → respawn + relogin + resume
- Slot-full: tu xoa cac char co prefix → create lai

## Luu y khi dev tiep
- Prefix MUST NOT match user's main char name (else se bi xoa)
- Server ID = 4 la "Sub-19" cua 1 user test, co the khac voi server khac
- Data20 trong DeleteChar = all zeros duoc (server nay khong require password)

---
## Headless Login Progress (2026-04-18) - F4 HANDSHAKE DONE

### SERVER ENDPOINTS (xac nhan tu sniff WS2_32.send/recv/connect):
- **ConnectServer**: `14.225.209.79:44415`
- **GameServer Sub-1 (ID 0)**: port chua capture (port = CS resp)
- **GameServer Sub-2 (ID 1)**: `14.225.209.79:55903`
- **GameServer Sub-19 (ID 4)**: `14.225.209.79:55909`

### CS Handshake (da work trong headless_v3.py):
```
1. Connect CS (44415)
2. Send: c1 28 f4 04 [HWID_UUID 36 ASCII] 00    (40B)
3. Recv: c1 04 00 01                              (4B hello)
4. Send: c1 04 f4 06                              (4B request list) x2
5. Recv: c2 00 73 f4 06 [list 3 servers] + c2 00 06 f4 0a 00   (121B mix)
6. Send: c1 06 f4 03 [id_lo] [id_hi]              (6B select)
7. Recv: c1 16 f4 03 [16B IP ASCII] [2B port LE]  (22B response!)
```

### List response format (115 bytes body):
- Header: `c2 00 73 f4 06 00 03`
- Count: `00 03` (3 servers)
- Each entry: `[id_lo] [id_hi] [count] cc [name 20B] 00 ...metadata`

### F4/03 response parse:
- `c1 16 f4 03` = header
- `31 34 2e 32 32 35 2e 32 30 39 2e 37 39 00 00 00` = "14.225.209.79" (null-padded 16B)
- `5f da` = port LE (Sub-2=55903), `65 da` = 55909 (Sub-19)

### PHAT HIEN QUAN TRONG: F1/0E LA 134 BYTES (KHONG PHAI 96!)
- Memory cu ghi 96B C1 60 F1 0E ... la SAI
- Thuc te game gui 134B bat dau `dd 82` = wire-encrypted
- Wire-decrypt: `c3 86 [cipher_body 132B]` (C3 = encrypted packet marker, 86 = 134 len)
- Cipher body chua F1 0E + session data, KEY32 XOR

### REVERSE ENGINEERING DONE (IDA Pro MCP):
Login function fully reversed tai RVA 0x4B9F60:
- Plaintext 96 bytes: `C1 [size] F1 0E [user 10 XOR-encoded] [pass 20 XOR] [session 36 XOR] [tick 4] [const 5] [const 16] [flag 1]`
- KEY32 cipher chain: `buf[i+2] ^= buf[i+1] ^ KEY32[i%32]` applied to bytes 4..95
- SendPacket (0x4AD160) voi enc=1:
  1. Copy plain, append rand byte at [96]
  2. Inject counter at [1] (overwrites size byte)
  3. Call MU_Encrypt(0x401080, &buf[1], 95) -> 132 bytes
  4. Prepend C3 [134] header -> 134 bytes
- MU SimpleModulus (0x401170): 8->11 block cipher
  - Keys load tu `Data\Enc1.dat` (54 bytes, magic 0x1112)
  - XOR key tai 0x11BA4B4: `9b a7 08 3f 87 c2 5c e2 b9 7a d2 93 bf a7 de 20`
  - Keys decoded: ModKey, MulKey, XorKey (moi cai 4x uint32)
  - Algorithm: modular multiply + reverse XOR chain + bit pack (sub_4013E0) + checksum byte
- Wire encrypt o socket layer: `enc[i] = ((plain[i] + 0x0B) ^ 0x13) & 0xFF`

### IDA functions renamed:
- 0x8B9F60 = Login
- 0x8AD160 = SendPacket  
- 0x401080 = MU_Encrypt
- 0x401170 = MU_EncryptBlock_8to11

### STATUS 2026-04-18: ✅ HEADLESS LOGIN F1/0E HOAT DONG HOAN TOAN!
Server response: `c1 05 f1 01 01` = SUCCESS

### KEY FINDINGS:
1. ✅ F4 handshake CS -> GS hoat dong (`headless_v3.py`)
2. ✅ Enc1.dat keys decoded: XOR key tai 0x11BA4B4
3. ✅ MU SimpleModulus encrypt 100% match game output (verified vs captured pair)
4. ✅ Template plaintext bytes 0..69 = fixed per credentials+machine
5. ✅ Only bytes 70..95 vary per session (GTC + KEY32 chain)

### WORKING FILE: `headless_v5.py`
Day du flow: F4 handshake -> build plain (template + dynamic GTC) -> SendPacket wrap (counter + MU encrypt) -> wire encrypt -> send -> recv login result

### mu_crypto.py - CORRECT implementation
- Load Enc1.dat (54 bytes, magic 0x1112, XOR key 0x11BA4B4)
- Keys: Mod=[0x1f44f, 0x28386, 0x1125b, 0x1a192]
        Mul=[0x5bc1, 0x2e87, 0x4d68, 0x354f]
        Xor=[0xbd1d, 0xb455, 0x3b43, 0x9239]
- Algorithm:
  - Phase 1: `r[i] = MulKey[i] * (prev_low16 XOR (input[i] XOR XorKey[i])) % ModKey[i]`
  - Phase 2: Reverse chain: `r[i] = (orig_r[i+1] & 0xFFFF) XOR XorKey[i] XOR r[i]` for i=2,1,0
  - Phase 3: Bit-pack 4 values (18 bits each = 16 low + 2 bits at position 22) MSB-first
  - Phase 4: Checksum `chk = 0xF8 XOR (XOR all 8 input bytes)`, `size_byte = chk XOR a4 XOR 0x3D`

### IDA functions renamed (Main_unpacked_v2.i64):
- 0x8B9F60 = Login
- 0x8AD160 = SendPacket
- 0x401080 = MU_Encrypt
- 0x401170 = MU_EncryptBlock_8to11

### PROGRESS 2026-04-18 session 2:
1. ✅ Login HEADLESS SUCCESS (F1/01 = 0x01)
2. ✅ F3/0D charlist request works - server returns 564B C4 encrypted + 5B DE/00
3. ✅ F3/0C CreateChar works - server returns F3/01 with result 0x02 + char data
4. ❌ F3/0E EnterGame - server CLOSES connection silently after send
5. ⏳ Chua impl MU_Decrypt de parse charlist

### EnterGame format (RVA 0x4B9990):
- 14 bytes PLAINTEXT (enc=0)
- C1 [0x0e] F3 0E [name 10B KEY32-chained]
- KEY32 chain formula: buf[i] ^= buf[i-1] ^ KEY32[i%32] for i=4..13
- Captured game packet: `df 0a ed 0a 79 5b b3 9b ca 8e 4d b9 17 ba`
- My headless builds identical structure, server rejects

### CreateChar format (RVA 0x4B18D0):
- 15 bytes PLAIN
- C1 [0x0f] F3 0C [name 10B] [class_byte = class_hi<<4 | class_lo] all KEY32-chained
- Headless WORKS, server accepts

### Post-login missing packets (from captured game flow):
Game also sends (before F3/0D):
- 24B C3 encrypted packet `dd 30 6c 89 ...` = 16B plaintext unknown opcode
And after F3/0E:
- 13B C3 encrypted packet `dd 0b 61 aa ...` = 8B plaintext unknown

These MAY be required for EnterGame to succeed. Cần MU decrypt Dec2 keys + capture fresh from game.

### NEXT SESSION:
1. MU_Decrypt using Dec2.dat keys (mirror of Enc1)
2. Capture 24B mystery packet via Frida hook when game enters game
3. Decrypt charlist 564B response
4. Figure out F3/0E rejection reason
5. Full bot pipeline (login → char select → enter → farm)

### 2026-04-18 session 4 findings (batman1 capture):
Captured post-login plaintext packets from game (Frida SP hook):
```
[SP-0E/F0] sz=16 enc=1  ← mystery packet 1 = 0E/F0 ACK (MU encrypted)
[SP-F1/0E] sz=96 enc=1  ← login
[SP-0E/F0] sz=16 enc=1  ← ACK after login response
[SP-F3/0D] sz=4 enc=0   ← charlist request (plain)
[SP-F3/0C] sz=15 enc=0  ← create char (plain)
[SP-F3/0E] sz=14 enc=0  ← enter game (plain)
[SP-0E/F0] sz=16 enc=1  ← ACK
[SP-F3/3D] sz=34 enc=0  ← UNKNOWN 34B plain - maybe client state request?
[SP-F1/0D] sz=5 enc=1   ← F1/0D 0x17 - maybe heartbeat
```

### 0E/F0 ACK structure (reverse KEY32 chain):
- [0]: C1, [1]: 0x10 (size=16), [2]: 0x0E, [3]: 0xF0
- [4..5]: varies (possibly GetTickCount low 16?)
- [6..7]: uint16 counter (observed 4056, 4056, 4057, 4057)
- [8..15]: ALL ZEROS (verified across 2 samples)
- ENC=1: MU encrypted

### F3/3D packet (34B plain enc=0):
Example: `c1 22 f3 3d da 06 3c 60 55 99 a7 94 0d 95 4b 47 8a 67 8d 47 91 91 90 72 49 59 6b d5 eb 28 66 2b 80 91`
- Unknown purpose. Appears AFTER F3/0E EnterGame.
- Possibly client version/info request for game entry

### F1/0D 5B packet (enc=1):
`c1 05 f1 0d 17` - F1/0D with 1 byte data = 0x17 (23)
Unknown purpose. Possibly keepalive/heartbeat.

### STATUS: F3/0E EnterGame still rejected
Even with 0E/F0 ACK sent after each recv, server closes connection
immediately after F3/0E. Likely requires F3/3D send BEFORE or
specific timing/sequencing we don't yet understand.

### Files session nay:
- `headless_v8.py` - with 0E/F0 ACK implementation (still fails F3/0E)
- `mu_decrypt.py` - partial MU_Decrypt port (checksum bug)

### NEXT STEPS:
1. Decrypt captured F3/3D to understand its purpose
2. Capture in-game exchanges with timing info
3. Try sending F3/3D before F3/0E
4. Full MU_Decrypt to read server responses (charlist, game state)

---
## 2026-04-18 SESSION 5 (MAJOR PROGRESS)

### Flow mới capture duoc via `sniff_sv_watch.py` + `capture_full.py`:
```
1. F1/0E login (134B enc=1)
2. recv login OK (5B)
3. 0E/F0 ACK (24B enc=1)
4. F3/0D charlist (4B plain)
5. recv charlist (124B C4 encrypted - batman1 acc, 569B cho batmanlk acc)
6. recv 5B: c1 05 de 00 0f  ← MYSTERY OP 0xDE, luon theo sau charlist!
7. F3/0E EnterGame (14B plain)   ← ~885ms after DE recv
8. F1/0D (13B enc=1)             ← ~24ms after F3/0E
9. recv IN-GAME data (massive 3525B + 939B)
```

**F3/3D KHONG xuat hien trong flow login-to-ingame!** F3/3D co the chi gui mid-game.

### F3/3D body decoded (30 bytes after reverse KEY32):
```
FF x20 7F FF FF FF 18 FF 00 x4
```
Static fingerprint, khong dung de enter game.

### F1/0D plain body = 0x02 (pre-XOR-chain)
Wire = c1 05 f1 0d 17 (post XOR chain). Heartbeat hoac client-ready.

### MYSTERY OP 0xDE (C1 05 DE 00 0F):
- Server gui 5B sau charlist response, body byte = 0x0F (variable?)
- Real game khong response truc tiep, 885ms sau gui F3/0E
- Possible: client state validation / key rotation / challenge

### F3/0E rejection (HEADLESS):
- Voi char ton tai (e.g., batmanlk): server gui 57B kick message
  - Decoded: `c1 39 0d 01 00 00 00 00 00 00 00 00 00 [body]`
  - Body UTF-8: **"Bạn sẽ thoát trò chơi sau 4 giây"** (Vietnamese)
  - Nghia: server kick client, 0D/01 = force disconnect with reason
- Voi char moi tao (Bot*): server dong im lang (0 bytes)
- Wait 10-15s giua sessions khong giup
- Flow timing + pkt bytes match game exactly

### Giai thuyet con lai:
1. Server co per-session seed tu hello (bytes 5-6 `30 XX`), dung de validate F3/0E
2. DE packet body = session key, client must incorporate
3. Anti-cheat detection tren IP/HWID/Frida presence
4. Server-side char lookup with specific slot-index requirement

### Files:
- `sniff_sv.js` - updated: full 1024B capture, timestamp, SelectServer hook DISABLED (crash)
- `capture_full.py` - launcher: bypass_v5 + sniff_sv (no optimize for UI)
- `headless_v8.py` - flow khop 100% voi game capture nhung F3/0E van fail
- Captures tasks/brz7k7cw2.output - full login-to-ingame successful flow

### NEXT (uu tien):
1. Fix MU_Decrypt (algorithm bug bit-ordering) de doc charlist + DE response
2. Hook F3/0E Login function (sub_8B9F60) + sub_8B9990 EnterGame trong Frida, BREAKPOINT va dump EXACT bytes de compare
3. Search IDA for 0xDE opcode handler - tim xem client co xu ly no khong
4. Try UUID/HWID spoofing de reset session state

---
## 2026-04-18 SESSION 6 - BREAKTHROUGH: HEADLESS INTO GAME

### MU_Decrypt BUG FIX (mu_decrypt.py):
**Root cause**: Bit-unpacking dung MSB-first nhung uint32 LE → byte order swap sai.

**FIX sai**:
```python
lo16 = read_bits(cipher, bit_off, 16)
write_bits_to_value(values[i], 0, lo16, 16)  # Swap bytes do MSB-first
val_u32 = struct.unpack('<I', values[i])[0]  # Sai! u32 byte-swap vs original
```

**FIX dung**:
```python
byte0 = read_bits(cipher, bit_off, 8)      # byte[0] value (low byte of lo16)
byte1 = read_bits(cipher, bit_off + 8, 8)  # byte[1] value (high byte of lo16)
hi2 = read_bits(cipher, bit_off + 16, 2)   # 2 high bits cua u32 bit 16-17
val_u32 = byte0 | (byte1 << 8) | (hi2 << 16)
```

**Test roundtrip**: encrypt voi Enc1 keys + decrypt voi (ModKey, ModularInverse(MulKey), XorKey) PASS!
```python
def modinv(a, m): # Extended Euclidean
    ...
mul_inv = tuple(modinv(m, mod_k[i]) for i, m in enumerate(mul_k))
```

### CHAR NAME DISCOVERY:
Decrypt charlist response (Dec2 keys) → byte[8..17] = char name padded 10 bytes.
**batmanlk account char name = "IAmKyo"** (khong phai "batmanlk"!)

Root cause cua F3/0E fail suot session 4-5: **da gui ten char sai**.

### HEADLESS ENTER GAME HOAT DONG:
```
headless_v8.py IAmKyo 4 → server phan hoi 9822B in-game data + follow-up packets
+ 1024B, +789B, +784B, +854B, +732B (map, monsters, char state, etc)
```

Packet flow khop 100% voi game capture:
1. F1/0E login (enc=1)
2. 0E/F0 ACK (enc=1)
3. F3/0D charlist (plain) → resp 564B + 5B DE mystery
4. Wait 1s
5. F3/0E EnterGame (plain, with CORRECT char name) + F1/0D heartbeat (enc=1)
6. Server flush in-game data (~15KB)

### CORE FLOW (verified):
```python
from mu_decrypt import decrypt, load_dec_file

def get_char_names_from_charlist(wire_resp):
    # wire_resp: full C4 charlist response bytes
    plain_hdr = wire_decrypt(wire_resp)  # bytes + 0x0B) ^ 0x13
    cipher = plain_hdr[3:-5]  # strip C4 header + tail 5B DE
    plain = decrypt(cipher, dec2_keys)
    names = []
    # plain format: [counter][F3][sub][body...]
    # body after [3:] has slot entries
    # Each slot entry starts at specific offset
    # TODO: reverse exact slot layout
    ...
```

### Files:
- `mu_decrypt.py` - FIXED, working
- `headless_v8.py IAmKyo 4` - WORKS (full login + charlist + EnterGame into map)
- Tasks output: b1rpnml5n.output - success record

### NEXT:
1. Build full bot pipeline headless: farm loop, NPC interact, sell, chat
2. Parse in-game packets (HP, pos, monsters)
3. Auto CreateChar + auto EnterGame for new Bot* accounts
4. Multi-instance headless (100 bots on VPS)
5. Capture Enc2 keys for SERVER-to-CLIENT MU encryption testing

---
## SESSION 7 - FULL BOT PIPELINE HEADLESS (2026-04-18)

### HOAN THANH: `headless_v9.py` chay toan bo bot cycle khong can game client
```
[*] Connect CS (44415) → Sub-19 (55909)
[>] F1/0E login
[>] F3/0D charlist → decrypt → 'IAmKyo'
[>] F3/0E EnterGame → 9195B in-game data
[>] F1/0D post-enter
[>] chat /tanthu → 322B teleport resp
[>] NpcClick → 143B NPC dialog
[>] Interact(122, 0) + Interact(122, 4) → 139B menu
[>] Sell 120 items (all inventory slots 0-127 except EQUIP={0,2,3,4,5,6,64,88})
[>] Interact(122, 3) close dialog
[>] F1/0D logout
[<] Final 1019B disconnect
```

### Packet formats (tat ca tu IDA decompile):
- **NpcClick** (sub_8C1CA0, enc=1): `C1 05 30 [hi^0xCE] [lo XOR-chain]`
- **Interact** (sub_8ADD30, enc=0): `C1 05 18 [a2^0xE6] [a1 XOR-chain]`
  - Interact(122, 0) = open NPC menu
  - Interact(122, 4) = select sell option
  - Interact(122, 3) = close dialog
- **SendSell** (sub_7FD5E0, enc=0): `C1 08 FB 0D [item] [tab] [00] [00]` (all XOR-chained from byte 4)
- **chatFunc** (sub_8ABA30, enc=0): `C1 [size] 00 [name 10B] [text]` XOR-chain bytes 3+
  - op=0x00 (khong phai 0x0D standard MU)
- **Logout** (sub_8BA710, enc=1): `C1 05 F1 0D [reason]`
  - reason=1 logout, reason=2 post-enter "ready" signal

### Files:
- `headless_v9.py` - Full bot pipeline WORKING
- `mu_decrypt.py` - FIXED, decrypt charlist to find char name
- `mu_crypto.py` - encrypt (unchanged, verified byte-perfect)

### KEY INSIGHT:
XOR obfuscation pattern: constants XORed to hide op bytes from casual packet sniffer:
- NpcClick: 0xCE XOR
- Interact: 0xE6 XOR
- SendSell: 0x0DFB constant

### NEXT STEPS:
1. Multi-instance headless (VPS deploy, 100+ bots parallel)
2. Parse charlist slots (current: only slot 0 name extracted)
3. Auto CreateChar for new accounts
4. Decrypt in-game packets to verify wcoin increase
5. Make template per-account (not just batmanlk) by reverse-chaining KEY32

---
## SESSION 8 - WCOIN EARNING VERIFIED (2026-04-18)

### Kinh nghiem: /tanthu bi intermittent fail do TIMING
**Van de**: char vua EnterGame (~8KB in-game data) → chat /tanthu ngay → server chua ready → reject command, char ket o Lorencia khong co do.

**Fix**: Sau EnterGame, wait 8 GIAY voi periodic ACK/drain moi gui /tanthu:
```python
for _ in range(8):
    time.sleep(1)
    send_ack(gs)
    recv_nonblock(gs, 0.3)
```

Va retry /tanthu 3 lan, check response size > 5KB = map change success.

### KET QUA: WCOIN TANG THAT +1041/cycle CONFIRMED
User test: 148,485 -> 149,526 wcoin sau 1 cycle (Botnrvnw char).

### Flow hoan thien headless_v9.py:
```
Connect → Login → Cleanup Bot* cu → CreateChar → EnterGame
→ Wait 8s (periodic ACK/drain) ← KEY FIX
→ /tanthu retry 3x until >5KB response
→ NpcClick → Interact(122,0) → Interact(122,4)
→ Sell all slots 0..127 (skip EQUIP)
→ Interact(122,3) close
→ Disconnect (no logout packet needed)
```

### Cycle timing:
- Original (broken): ~22s per cycle, /tanthu fail silently
- Fixed: ~30-35s per cycle, /tanthu OK every time
- Rate: ~100-120 cycles/h = **~100-120K wcoin/h per bot**

### UI tool san sang:
`headless_ui.py` chay vong lap CreateChar + tanthu + sell + Delete.
Hien thi: Cycles, Rate, Char name, WC baseline.

### NEXT:
1. Multi-account parallel (mỗi account 1 template, chạy song song)
2. Template generator từ captured login (anh capture batman1/batman2 template)
3. VPS deploy + background daemon mode

---
## SESSION 9 - WCOIN READ CHINH XAC (F3/EA packet)

### Phat hien wcoin packet structure:
```
C1 10 F3 EA [wcoin u32 LE] [12 bytes zeros]
```
- magic: C1
- size: 0x10 = 16 bytes
- op/sub: F3/EA
- body[0..3]: wcoin as uint32 LE
- body[4..15]: zeros

### Vi tri xuat hien:
- Server send F3/EA sau EnterGame (trong in-game stream initial data)
- Server send F3/EA LAI sau khi sell (update)
- Nam plain trong stream (C1 packet, khong encrypted)

### Verify:
- User cung cap wc = 161,089
- Scan in-game stream: offset 9062 found 161089
- Parsed packet @9058: C1 10 F3 EA 41 75 02 00 ...
  - 41 75 02 00 = 0x00027541 = 161089 ✓

### Implementation trong headless_v9.py:
```python
def extract_wcoin_from_stream(stream_wire):
    plain = wire_decrypt(stream_wire)
    i = 0
    last_wcoin = None
    while i < len(plain) - 7:
        if plain[i] == 0xC1 and plain[i+1] == 16 and \
           plain[i+2] == 0xF3 and plain[i+3] == 0xEA:
            last_wcoin = struct.unpack_from('<I', plain, i+4)[0]
            i += 16; continue
        # ... skip other packets
        i += 1
    return last_wcoin
```

Ket qua test:
- Pre-tanthu: 161,089 (match user)
- Post-sell: 162,245 (+1,156 wcoin/cycle)
- Real-time accurate

### Files:
- headless_v9.py - extract_wcoin_from_stream added
- headless_ui.py - display WC real from server F3/EA
- locate_wcoin_pkt.py - utility to find wcoin packet
- scan_raw.py - scan raw in-game stream

### NEXT:
1. Multi-account (batman1/batman2 templates)
2. VPS deploy daemon mode

### 2026-04-18 session 3 findings:
- MU_Decrypt port attempted (mu_decrypt.py) but CHECKSUM fails → algorithm bug
- Captured game packets post-login show `[SP] size=16 enc=1` appears REGULARLY
  Pattern: after most recv from server, client sends 16B encrypted packet
  → likely ACK/KEEPALIVE packet
- IDA rename: 0x401280 = MU_DecryptBlock_11to8
- Dec2.dat keys (verified loaded):
  - Block a4 (mod): 0x11e6e, 0x1ada5, 0x1821b, 0x29c32
  - Block a6 (mul_inv): 0x4673, 0x7684, 0x607d, 0x2b85
  - Block a7 (xor): 0xf234, 0xfb99, 0x8a2e, 0xfc57

### ISSUE: Game khong spawn duoc khi headless session con mo
Server seems to reject game's login if batmanlk account has active headless session.
Van de: de capture 24B mystery packet can spawn game, nhung my headless test
da tao session o server -> server reject game login -> game crash during load.

Workaround: CHUA TEST:
- Logout properly tu headless (send F1/0D) before closing
- Or wait 30-60s for session timeout between tests
- Or use SECOND ACCOUNT for game testing (create another account)

### Files session nay:
- `headless_v5.py` - Login only WORKING
- `headless_v6.py` - + charlist observe
- `headless_v7.py` - + CreateChar + EnterGame attempt (stuck)
- `mu_crypto.py` - MU Encrypt verified byte-perfect
- `test_mu.py` - ground truth test

### Files:
- `D:\Code Tools\PC_GAMES\M4VN\headless_v3.py` - full F4 handshake OK, F1/0E van fail
- `D:\Code Tools\PC_GAMES\M4VN\sniff_sv.js` - WS2_32 hook, da update capture 256B
- `D:\Code Tools\PC_GAMES\M4VN\sniff_sv_watch.py` - spawn + sniff UI manual

---
## F1/0E Headless Login - FULLY REVERSED (2025-04) [OUTDATED - 96B format]

### Cau truc packet F1/0E (96 bytes, pre-wire-encrypt):
```
in[0..3]  = C1 60 F1 0E  (header)
in[4..69] = fixed66 (hardcoded per credentials - 66 bytes)
in[70..73] = GetTickCount() little-endian 32-bit
in[74..95] = hardcoded 22 bytes: 31 30 34 30 35 54 62 59 65 68 52 32 68 46 55 37 35 39 67 5A 6A 01
```

### Cipher: KEY32 stream (forward)
```
out[i] = in[i] ^ KEY32[i%32] ^ out[i-1]  (i=4..95)
KEY32 = AB 11 CD FE 18 23 C5 A3 CA 33 C1 CC 66 67 21 F3 32 12 15 35 29 FF FE 1D 44 EF CD 41 26 3C 4E 4D
```

### Wire encryption (ca 2 chieu):
```
enc[i] = ((plain[i] + 0x0B) ^ 0x13) & 0xFF
dec[i] = ((enc[i] ^ 0x13) - 0x0B) & 0xFF
```

### Key insights:
- **Seed tu F1/00 (hello packet) KHONG dung trong F1/0E** - client gui F1/0E TRUOC khi nhan hello
- in[70..73] = GetTickCount() xac nhan 100%: IDA+Frida hook (KERNEL32.DLL+0x22640) + math verify
- fixed66 = out[4..69] hardcoded theo credentials (batmanlk/ongnoi12):
  `88 05 1f 2d 49 bf ee 86 4b d0 62 30 ce 4e fb 0c e8 ea bf 5e d5 91 a0 2e a3 63 e2 04 53 8d 85 8d 09 e3 aa 9a ee 2a 6d 6a 85 7a 94 9e 46 9e 00 b3 20 24 44 e7 2d 5e 5b 93 33 cb 0b d9 be 24 7a 3d dd 55`

### sub_464F20 (RVA 0x464F20) = username/password/session encoder:
- XOR moi byte voi XOR_TABLE[i%3], table tai 0x11b9750 = [0xFC, 0xCF, 0xAB]
- Args: (buf_ptr, len) - in-place XOR

### sub_467B50 (RVA 0x467B50) = session key generator:
- Goi GetVolumeInformationA("C:\") → DWORD volume serial
- Goi UuidCreateSequential() → UUID (sequential, thuc te tra ve fixed UUID string 35 chars + null)
- Goi GetSystemInfo()
- XOR voi constants: 0xb542d8e1, 0xf45bc123, 0x5d78a569, 0x12b586fe
- Output: 36 bytes UUID string "B0458164-7F336A6D-F45BA74F-8A40D8E7\0" (VD)
- KHONG doi theo session - fixed per machine!

### fixed66 structure (in[] = plain, truoc KEY32 cipher):
- in[4..13]  = encode_xor(username padded 10 bytes) → [FC CF AB FC CF AB FC CF AB FC...] repeating
- in[14..33] = encode_xor(password padded 20 bytes)
- in[34..69] = encode_xor(sub_467B50 output 36 bytes) → session key (UUID-based, per-machine)
- out[4..69] = KEY32 cipher(in[4..69]) → FIXED66 (66 bytes)

### FIXED66 compute (Python):
```python
XOR_TABLE = [0xFC, 0xCF, 0xAB]
encode = lambda data: bytes(b ^ XOR_TABLE[i%3] for i, b in enumerate(data))
# username "batmanlk" (8 chars) padded to 10 with zeros, XOR encoded:
in4_33 = encode(b"batmanlk\x00\x00") + encode(b"ongnoi12\x00"*12[:20])  # 30 bytes
# session key from sub_467B50 via Frida NativeFunction call, then encode:
# enc36 = encode(sub467b50_output_36_bytes)
# Then KEY32 cipher from i=4..69 to get out[4..69] = FIXED66
```

### KEY FINDING (2025-04-17) - Server close behavior:
- Khi KHONG gui gi: server gui hello + CHO (khong close) → server active
- Khi gui F4/03 (SelectServer): server KHONG close, van cho (server accept F4/03)
- Khi gui F1/0E: server CLOSE ngay (0 bytes response)
- Dieu nay = server REJECT F1/0E SILENT khong gui error packet
- Test voi fake account (zzzz/zzzz), zeros, random, correct key → tat ca deu bi close
- Con nguyen nhan: co the la "gate key" tu ConnectServer, hoac sub_467B50 output cuoi cung dung ham khac

### PROBLEM VAN CHUA GIAI QUYET (2025-04-17):
- Da verify: packet format dung, cipher dung, session key dung
- Server van close F1/0E silent (0 bytes response) - confirmed KHONG phai timing, KHONG phai account online
- Root cause chua biet: co the trong sub_467B50 offset +0xa0: call 0x430520 voi arg = 0x01063dc4 (global var?)
  - Neu 0x01063dc4 chua gate key tu ConnectServer → session key phu thuoc gate key
  - Ham 0x430520 co the la XOR/hash session key voi gate key nay
- De-bug tiep theo: restart game → sniff ALL send/recv tren ca 55909 va 44415
  - Xem co packet nao truoc F1/0E tren 55909 khong
  - Xem ConnectServer (44415) gui gi, co gate key khong
  - Dump function 0x430520 tu live process
- File test: `D:\Code Tools\PC_GAMES\M4VN\headless_v2.py` - test moi nhat (doc tat ca data stream)

### File:
- `D:\Code Tools\PC_GAMES\M4VN\mu_login_client.py` - headless client goc
- `D:\Code Tools\PC_GAMES\M4VN\headless_v2.py` - version moi nhat, doc all stream
- `D:\Code Tools\PC_GAMES\M4VN\get_session_key.py` - lay session key tu Frida
- `D:\Code Tools\PC_GAMES\M4VN\capture_login_direct.py` - capture + replay exact bytes
- `D:\Code Tools\PC_GAMES\M4VN\sub_464F20.bin`, `sub_467B50_full.bin` - live dumps
- Can chay sniff de lay IP server tu "[CYCLE #X] connect X.X.X.X:55909"
