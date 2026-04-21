---
name: M4VN Server ID Mapping
description: SelectServer(idx) cua M4VN khong phai so UI - dung internal server ID
type: project
originSessionId: 8be1cda3-5aab-4ab4-89fc-97600b06a50a
---
## Finding (2026-04-18)
Server ID truyen vao `SelectServer(0x4C0840)` KHONG PHAI so sub hien thi tren UI.
Packet F4/03 gui di: `C1 06 F4 03 [ID_LO] [ID_HI]` (uint16 little-endian).

## Mapping xac nhan
| Sub (UI) | ID thuc |
|----------|---------|
| Sub-1 | 0 |
| Sub-2 | 1 |
| Sub-19 | 4 |

## How to apply
Khi config tool `bot_ui.py` muon chon server:
- Sub-1: server = 0
- Sub-2: server = 1
- Sub-19: server = 4

Khong sua code, chi doi so trong o "Server" cua UI.

## Cach tim server ID cho server khac
1. Chay `sniff_sv_watch.py` (hook WS2_32.send)
2. Click server trong UI game
3. Tim packet `c1 06 f4 03 XX 00` -> XX = server ID
