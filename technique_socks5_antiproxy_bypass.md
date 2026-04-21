---
name: technique_socks5_antiproxy_bypass
description: Multi-layer bypass khi game server ban IP/subnet + client-side anti-proxy check
type: reference
originSessionId: 8be1cda3-5aab-4ab4-89fc-97600b06a50a
---
# SOCKS5 Tunnel + getpeername Spoof (Multi-layer Anti-Proxy Bypass)

## Khi nao dung
Game server ban IP/subnet ket hop client-side anti-proxy check. Trieu chung:
- TCP connect OK (accept), nhung server KHONG gui hello packet (silent drop)
- Doi IP cung ISP van bi (ban /16 hoac ASN)
- Khi tunnel qua proxy thong thuong, nhan duoc server list, nhung sau `connect` GS,
  nhan hello -> gui ACK -> **client tu close connection**

Da gap o M4VN SEASON UPGRADE (2026-04).

## Co che chan 3 tang

### Tang 1: Server-side IP/subnet blacklist (app layer)
- Server TCP accept nhung kiem tra src IP -> neu blacklist -> silent drop (khong gui hello)
- Khong phai firewall-level block (TCP handshake OK)
- Ban /24 hoac ASN levels phat hien qua: test proxy khac ISP -> co hello

### Tang 2: Client-side `getpeername()` anti-proxy check
- Game client goi `getpeername(socket)` sau connect
- So sanh voi expected game IP (hardcoded tu CS F4/03 response)
- Neu khac -> disconnect im lang (close socket sau 1-2 packet)
- Common pattern o MMO clients co anti-VPN/anti-proxy

### Tang 3: HWID/Hardware fingerprint (da biet)
- Game gui HWID tu `sub_467B50` trong CS handshake (`c1 28 f4 04 [HWID 36B] 00`)
- Khong check HWID tai tang CS (random HWID van OK)
- GS co the check nhung ta khong thay evidence

## Cach bypass (bypass_v5.js)

### Hook `ws2_32!connect` -> redirect + SOCKS5 handshake
```javascript
Interceptor.attach(connectFn, {
    onEnter: function(args) {
        // Neu dst ip match game server -> rewrite sang proxy
        var ipB = [sa.add(4).readU8(), ..., sa.add(7).readU8()];
        if (matches GAME_SERVER_IP) {
            this.origIp = ipB; this.origPort = port;
            // Overwrite sockaddr voi proxy IP/port
            sa.add(4..7).writeU8(PROXY_IP_BYTES);
            sa.add(2..3).writeU8(PROXY_PORT hi/lo);
        }
    },
    onLeave: function(retval) {
        if (retval.toInt32() === 0 && this.redirect) {
            // SOCKS5 handshake sync trong onLeave:
            // 1. send [05 02 00 02] (methods)
            // 2. recv [05 method]
            // 3. if auth: send [01 ulen user plen pass], recv [01 00]
            // 4. send [05 01 00 01 ip:4B port:2B] (CONNECT target)
            // 5. recv [05 00 00 01 bnd:4B port:2B]
            tunneledSockets[sockHandle] = {ip: this.origIp, port: this.origPort};
        }
    }
});
```

### Hook `ws2_32!getpeername` -> spoof sockaddr
```javascript
Interceptor.attach(getPeerFn, {
    onEnter: function(args) { this.sock = args[0].toInt32(); this.sa = args[1]; },
    onLeave: function(retval) {
        if (retval.toInt32() !== 0) return;
        var info = tunneledSockets[this.sock];
        if (!info) return;
        // Overwrite sockaddr_in voi real game IP/port thay vi proxy IP
        this.sa.add(2..3).writeU8(info.port hi/lo);
        this.sa.add(4..7).writeU8(info.ip);  // 4 bytes IP
    }
});
```

### Cleanup `closesocket` -> xoa tracking
```javascript
Interceptor.attach(closeFn, {
    onEnter: function(args) { delete tunneledSockets[args[0].toInt32()]; }
});
```

## Quy trinh chan doan

1. **Test TCP direct** tu IP user -> `exec 3<>/dev/tcp/SERVER/PORT` OK?
2. **Test app layer direct** -> send HWID packet + wait hello. Neu KHONG hello = silent drop
3. **Test app layer via proxy** -> neu co hello = IP ban confirmed tang 1
4. **Chay game qua proxy** -> neu game van disconnect sau ACK = client-side check tang 2
5. Fix getpeername spoof -> test lai -> login thanh cong

## File va config

- `bypass_v5.js` PROXY_CFG + GAME_SERVER_IP hardcode o dau file
- Hook duoc dat trong `earlyHooks()` IIFE de active TRUOC connect dau tien
- SOCKS5 handshake sync trong `onLeave` (blocking socket). Neu non-blocking
  co the can hook ioctlsocket/WSAEventSelect them

## Key insights

1. **IP ban ≠ firewall block**: server accept TCP but silent drop = app layer decision
2. **Client-side anti-proxy**: `getpeername` trivial check nhung hieu qua vs basic SOCKS
3. **Combined = fort**: chi SOCKS thoi khong du. Spoof getpeername la key
4. Test pattern: direct fail -> proxy hello OK -> proxy full flow -> identify which layer stuck
