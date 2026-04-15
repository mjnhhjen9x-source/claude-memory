---
name: Technique - Tim function bang packet capture + return address
description: Thay vi guess function prototype, hook low-level SendPacket + capture caller = tim ham xay dung packet
type: reference
originSessionId: 05a683ce-7fc5-430d-b086-61b4424e86f8
---
## Van de
Can tim function game X (vd CreateChar, Login, DeleteChar). Khong biet ten, khong biet address, khong biet prototype.

## Cach lam
1. Tim **ham send packet thap nhat** (send tren socket): hook WSASend hoac WS2_32 send, stalker lui lai tim wrapper.
2. Hook ham wrapper + log (caller_rva = returnAddress - module.base):
```js
Interceptor.attach(sendAddr, {
    onEnter: function(args) {
        var size = args[1].toInt32();
        var bytes = args[0].readByteArray(Math.min(size, 32));
        var caller = this.returnAddress.sub(base);
        console.log("[SEND size=" + size + " caller=0x" + caller.toString(16) + "] " + hex);
    }
});
```
3. Thao tac UI game (click Create, Login, ...) → capture:
   - Packet bytes (de biet opcode + format)
   - Caller RVA → day la function xay dung packet
4. Tra caller vao IDA → decompile → co signature + args

## Vi du M4VN
- Click "Tao Nhan Vat" → packet F3 0C + caller 0x4B1C08 → inside sub_8B18D0 → CreateChar(name, class_hi, class_lo)
- Click "Login" → packet F1 0E + caller 0x4BA525 → inside sub_8B9F60 → Login(user, pass, flag)

## Tai sao hieu qua
- Khong can doan prototype
- Khong can doan calling convention
- IDA chi can giai ma 1 ham cu the (caller) thay vi scan toan bo binary
- Lam duoc cho **bat ky game nao** co low-level send

## Khi dung
- Can tim ham tu UI action (button click → network request)
- Da co bypass + co the attach Frida
