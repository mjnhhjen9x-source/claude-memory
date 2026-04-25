---
name: Workspace map and key reference files
description: Pointers to surviving context files after Windows reinstall — CLAUDE.md and the D:\Code Tools structure plan
type: reference
originSessionId: 739a471f-c622-43c3-b869-927876ec61e8
---
Sau khi cài lại Windows (2026-04-25), memory bị xóa nhưng các file context sau vẫn sống:

- **`D:\Code Tools\CLAUDE.md`** — workspace authorization + scope RE + Frida rules + môi trường (LDPlayer, ADB, frida-server paths). Auto-load mỗi session.
- **`C:\Users\ThanTai\.claude\plans\d-code-tools-b-n-c-cuddly-fox.md`** — bản đồ chi tiết toàn bộ `D:\Code Tools\` (cấp 1-2, kích thước, mô tả từng project con). Đọc khi cần overview workspace.
- **`D:\Code Tools\PC_GAMES\`** — chứa các report markdown: `RECON_REPORT.md`, `IDA_FINDINGS.md`, `PROTOCOL_CATALOG.md` (knowledge tích lũy về PC games đã RE).
- **`D:\Code Tools\IDA_DATABASES\`** — IDA `.i64/.idb` cho Gunbound, jx1shxt, KyNguyenIcarus, VL_MULTI, VNKU. Dùng khi cần tra offset/struct.

Frida rules (từ CLAUDE.md, lặp lại để dễ nhớ):
- API: `Process.getModuleByName('libc.so').getExportByName('send')` — KHÔNG dùng `Module.findExportByName` (deprecated)
- Game networking: hook BOTH `send()` + `recv()` + `recvfrom()`
- Console output: ASCII only (Windows cp1252)
- Memory values: Double 64-bit trước, rồi Float, rồi Int32 (cho Flash/AIR games)
- Frida JS: dùng `var` không dùng `let/const`
