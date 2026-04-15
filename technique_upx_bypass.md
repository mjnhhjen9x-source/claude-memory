---
name: UPX Packed Binary Bypass Technique
description: Ky thuat Frida bypass cho game UPX packed - doi giai nen xong moi patch, giu file goc cho hash check
type: reference
originSessionId: 05a683ce-7fc5-430d-b086-61b4424e86f8
---
## Ky thuat bypass game UPX packed voi Frida

Khi game dung UPX pack va server kiem tra hash file:
1. **KHONG thay file** - giu file packed goc de server hash check dung
2. **Frida spawn** (suspended) -> attach -> load script -> resume
3. **Doi UPX giai nen xong** trong memory truoc khi patch:
   - Chon 1 dia chi RVA da biet (tu IDA phan tich ban unpacked)
   - Doc byte dau tien cua ham -> truoc UPX la du lieu nen, sau UPX la code that
   - Poll moi 50ms cho den khi byte thay doi thanh gia tri mong doi
4. **Patch sau khi UPX xong** - luc nay code that da nam trong memory

### Cach tim byte kiem tra:
```python
import pefile
pe = pefile.PE('file_unpacked.exe')
offset = pe.get_offset_from_rva(TARGET_RVA)
# Doc byte dau tien tai offset do trong file unpacked
# So sanh voi byte tai cung RVA trong file packed
```

### Luu y:
- UPX giai nen rat nhanh (<100ms) nen poll 50ms la du
- Byte truoc/sau UPX hoan toan khac nhau nen de phat hien
- Ky thuat nay ap dung cho bat ky packer nao, khong rieng UPX

**How to apply:** Khi gap game PC dung UPX hoac packer khac, va server co hash check file
