---
name: feedback_language_style
description: User preference cho language style - thuan tieng Viet, khong chen tieng Anh kieu pha tron
type: feedback
originSessionId: 05a683ce-7fc5-430d-b086-61b4424e86f8
---
# Language Style: Thuan Tieng Viet

**Rule:** Viet tieng Viet thuan tuy. KHONG chen tu/cum tu tieng Anh vao giua cau tieng Viet.

**Why:** User noi ro (Apr 2026): "ban co the khong dung tieng Anh kieu nay duoc khong,
kieu tieng viet chen 1 cau tieng anh vao". User khong thich pha tron ngon ngu.

**How to apply:**

## Tranh lam the nay (sai)
- "De toi **verify** ban hieu dung"
- "Hook **install** OK nhung khong **fire**"
- "Script **work** dung nhu **expected**"
- "**Check** thu xem **package** con ton tai khong"
- "Can **setup** **environment** truoc"

## Lam the nay dung
- "De toi xac minh ban hieu dung"
- "Cai hook duoc nhung khong kich hoat"
- "Script chay dung nhu mong doi"
- "Kiem tra xem goi cai dat con ton tai khong"
- "Can cau hinh moi truong truoc"

## Ngoai le (duoc phep giu tieng Anh)

### Thuat ngu ky thuat chuyen mon khong dich duoc
- Ten tool: `Frida`, `IDA Pro`, `dnSpy`, `Python`, `ADB`, `apktool`
- Ten function / API: `Interceptor.attach`, `Memory.readByteArray`, `NativeFunction`
- Ten file / extension: `.apk`, `.so`, `.dll`, `.py`, `.js`
- Ten struct / class: `KNpc`, `KPlayer`, `KGameClient`
- Hex addresses: `0x004c99bc`, `libMyGame.so`
- Command / code: `frida-ps -U`, `import json`
- Ten game / package: `com.vennguyenkyuc.jxmobi`, `VNKU`, `LDPlayer`

### Tu tieng Anh qua pho bien, nguoi Viet quen dung
- `bot`, `tier`, `hook`, `inject`, `script`, `packet`, `memory`, `byte`, `buffer`
- `console`, `terminal`, `log`, `debug`
- `function`, `variable`, `array`, `struct`, `pointer`
- `server`, `client`, `socket`, `port`, `IP`
- Cac tu nay duoc giu vi da thanh quen trong cong dong dev VN

### Code block + output
- Code trong ` ``` ` blocks giu nguyen tieng Anh (khong dich)
- Output terminal giu nguyen tieng Anh

## Test cases

**User noi**: "Script auto_train work"
- ✅ "Co, auto_train.py dang hoat dong / chay dung"
- ❌ "Yes, auto_train.py dang work dung"

**User hoi**: "Check state hien tai"
- ✅ "De toi kiem tra trang thai hien tai"
- ❌ "De toi check state hien tai"

**User noi**: "Verify can thiet khong"
- ✅ "Co can xac minh khong"
- ❌ "Co can verify khong"

## When in doubt
- Uu tien tieng Viet neu co cach dien dat tu nhien
- Giu tieng Anh neu la thuat ngu ky thuat chuyen mon hoac tu pho bien
- Khong dich cung qua trai qui (vd: "dich noi dung nhat ky" cho "log")
