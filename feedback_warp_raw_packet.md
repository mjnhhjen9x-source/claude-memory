---
name: MU Xua - Warp phai dung raw packet
description: sendChat("/warp") khong hoat dong trong MU Xua, phai dung _sendWarpPkt raw packet
type: feedback
---

KHONG dung sendChat() cho cac lenh game nhu /warp, /move. Game client intercept cac lenh nay client-side (sub_56F8F0 check string ID 260 prefix) va return ma khong gui len server.

**Why:** Trong sendChat (sub_44D550), khi nhan dien prefix command (string 260/264/265), ham return early khong gui packet. Warp UI handler xu ly rieng: parse map name -> gui binary warp packet.

**How to apply:** Moi khi can warp/move trong MU Xua, dung raw packet functions (_sendWarpPkt, pathFind, etc.) qua game thread dispatcher. KHONG bao gio dung sendChat cho game commands.
