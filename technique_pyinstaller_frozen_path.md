---
name: PyInstaller onefile path trap - dung sys.executable cho file persist
description: Khi build --onefile, __file__ tro temp _MEIxxx bi xoa khi thoat, config/log/db phai luu canh sys.executable
type: feedback
originSessionId: 8be1cda3-5aab-4ab4-89fc-97600b06a50a
---
Rule: Python script luu config/log/database → khi build PyInstaller `--onefile`, PHAI check `sys.frozen` va dung `sys.executable` de lay duong dan persist, KHONG dung `__file__`.

**Why**: `--onefile` giai nen bundle vao thu muc temp `C:\Users\...\AppData\Local\Temp\_MEIxxxxxx\` va xoa sach khi exit. `__file__` tro vao do → config luu OK trong session nhung mat sach khi tat app. Dung M4VN bot da dinh: user build exe, chinh config, tat, mo lai → mat het.

**How to apply**:
```python
import sys
from pathlib import Path

if getattr(sys, 'frozen', False):
    BASE_DIR = Path(sys.executable).parent  # canh file .exe
else:
    BASE_DIR = Path(__file__).parent  # dev mode

CONFIG_FILE = BASE_DIR / "config.json"
LOG_FILE = BASE_DIR / "bot.log"
```

Ap dung moi project PyInstaller --onefile. Neu chi `--onedir` thi `__file__` van OK, nhung cu dung pattern nay cho chac.
