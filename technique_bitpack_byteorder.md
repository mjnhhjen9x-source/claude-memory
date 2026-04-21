---
name: Byte-order cho bit-packed custom cipher
description: Cipher bit-packed (18-bit, 20-bit values) phai tach byte rieng roi ghep LE, khong doc 1 lan
type: feedback
originSessionId: 8be1cda3-5aab-4ab4-89fc-97600b06a50a
---
Rule: Decode custom cipher co bit-packed values (vd MU SimpleModulus 18-bit, 20-bit) → doc 8-bit tung byte rieng roi assemble little-endian, KHONG doc full value trong 1 call bit-reader.

**Why**: MU SimpleModulus 8→11 block cipher pack 4 uint32 values (18 bits each) vao 11 bytes. Ban dau viet `read_bits(cipher, offset, 18)` doc thang 18-bit ra → decrypt sai vi bit order trong byte khac voi bit order qua bytes. Phai tach:
```python
byte0 = read_bits(cipher, off, 8)
byte1 = read_bits(cipher, off + 8, 8)
hi2   = read_bits(cipher, off + 16, 2)
val = byte0 | (byte1 << 8) | (hi2 << 16)
```

Bug nay kho debug vi decrypt ra "gan dung" (1-2 byte dau OK) → tuong code OK nhung bytes sau sai het. Chi phat hien khi parse packet failed sau ~10 bytes.

**How to apply**:
- Gap custom cipher voi bit-packing → luon test roundtrip (encrypt → decrypt == original) TRUOC khi dung
- Verify voi cipher tool chuan cua game (vd MuEmu source) neu co
- Neu decrypt ra junk giua packet → nghi endianness truoc tien
- Test case don gian: encrypt 0x12345678, decrypt lai, check == 0x12345678
