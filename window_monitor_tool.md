---
name: Window Monitor Tool
description: Tool theo doi nhieu cua so game PC cung luc bang DWM Thumbnail API - live preview, click to focus, cascade arrange
type: reference
---

## Window Monitor - D:\Code Tools\WindowMonitor\monitor.py

Tool Python theo doi nhieu cua so game PC tren cung 1 man hinh.

**Cong nghe:** DWM Thumbnail API (Windows compositor truyen truc tiep hinh anh game, khong can screenshot)
- Live update, GPU-accelerated, hoat dong voi DirectX/DirectDraw/OpenGL
- Click vao thumbnail de goi cua so game len foreground
- Chi hien thi client area (khong title bar/vien)
- Rat nhe: 0% CPU, ~15MB RAM

**Tinh nang:**
- Add EXE: chon file .exe game can theo doi
- Manage: them/xoa target exe
- Show TARGETS / ALL: chuyen che do xem
- Cascade: sap xep cua so game kieu bac thang ngang, tuy chinh so game/hang (per row), tu fit man hinh, hang duoi trong len hang tren
- Config luu tai `D:\Code Tools\WindowMonitor\config.json` (targets + cascade_per_row)
- Auto detect khi game moi mo / game dong

**Dependencies:** pywin32, psutil, Pillow, tkinter (built-in)

**Chay:** `python "D:\Code Tools\WindowMonitor\monitor.py"`

**Luu y:**
- Game phai o windowed mode (khong fullscreen exclusive)
- Khi game minimize, DWM giu frame cuoi cung (DX game co the hien den)
- DWM Thumbnail chi hoat dong tren Windows Vista+ voi DWM enabled (Win10 luon bat)
- Cascade giu nguyen kich thuoc goc cua game (khong resize)

**How to apply:** Khi user can theo doi nhieu cua so game PC cung luc, dung tool nay. Co the add bat ky game exe nao.
