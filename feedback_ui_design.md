---
name: Phong cach thiet ke UI tools cua user
description: User thich UI Tkinter fixed size, grid aligned, chia tab
type: feedback
originSessionId: 8be1cda3-5aab-4ab4-89fc-97600b06a50a
---
Phong cach UI cho tools Python Tkinter:

**Why**: User sua nhieu lan yeu cau "gon", "thang hang", "co dinh size", "chia tab" → pattern ro rang.

**How to apply**: Khi build UI moi, ap dung ngay tu dau:

1. **Window fixed size, khong resize**:
```python
root.geometry("680x620")
root.resizable(False, False)
```

2. **Config grid 4 cot aligned**:
- Labels right-align (anchor="e"), width uniform (vd 14)
- Inputs left-align (sticky="w"), width uniform (vd 12)
- 2 cap label+input moi hang → cot 0,1,2,3
```python
ttk.Label(parent, text="Label:", width=14, anchor="e")\
    .grid(row=R, column=0, padx=2, pady=3)
ttk.Entry(parent, width=14).grid(row=R, column=1, padx=2, pady=3, sticky="w")
```

3. **Chia tab Notebook** khi co nhieu noi dung:
- Tab chinh: config + controls + status
- Tab Log rieng (scrolledtext)

4. **Status gon**:
- Chi show so quan trong (vd WC total, WC/phut)
- Bo cycles/uptime/char name khi user yeu cau gon

**Reference**: `D:\Code Tools\PC_GAMES\M4VN\headless_ui_multi.py`
