---
name: user_role
description: User profile - non-coder game farmer using AI to build all technical infrastructure
type: user
originSessionId: 8be1cda3-5aab-4ab4-89fc-97600b06a50a
---
# User Profile

## Background
- **Game farmer 5-6 nam** (chuyen nghiep, kiem du song; game ngon thi kha)
- **YouTube channel: "Thanh Cay Thue"** (public identity)
- **Truoc co team:** user lead phan cung/may moc/setup, co 1 nguoi viet tools
- **Hieu sau ve cac level cua tools** (T1->T5 framework) qua nhieu nam noi chuyen
  voi coder cu — biet gioi han tung tier, biet khi nao escalate
- **IT background phan cung** (hardware) — hieu CPU/memory/register concepts -> RE intuition tot
- **Khong code** — khong biet Python, JavaScript, hay language nao (do tu nguoi cu lo phan code)
- AI thay the role coder cu — nay user co the execute vision ma truoc khong tu lam duoc
- **Tat ca code Python/JS/Frida/IDA trong workspace nay deu do AI viet**, user chi describe yeu cau
- **Da dung AI ~3 thang** (tinh den 2026-04) -> biet ro gioi han model
- Team cu chi lam **T1-T2 tools** (anti-cheat khong cang); voi AI nay user da level len T3
- **Workflow "khong on la dung"** — khong co dam, pivot/stop nhanh khi feature/game fail

## Role split (rat quan trong cho session sau)
- **User:** game scout + y tuong kiem tien + domain knowledge + business judgment
  - Tim game co economy ngon, AC bypass duoc, server lifecycle du dai
  - Doan game **"an ngan han"** (server lifecycle ngan): vao truoc, ra truoc — game profile chu yeu
  - Hieu market: gia wcoin/item, where to sell
  - Risk management: khi nao dung de tranh chet acc
- **AI:** code execution + technical infrastructure + iteration
  - Build bypass, hook, bot logic, UI tools, exe packaging
  - RE protocol, find offset, debug runtime
  - Backup/memory/git workflow
- **AI khong scout game ho user** — chi build khi user da chon game va co plan

## Strengths (RE intuition rat tot)
- Biet chinh xac **what** can lam (vision/direction)
- Mo ta trieu chung dung dan (e.g., "select sub > khong vao login", "doi IP van bi")
- Hieu game flow MU Online (CS handshake, GS connect, char select, tanthu, sell NPC, wcoin)
- Phan tich layered protection (3 tang block: IP + getpeername + HWID)
- Biet user/test pattern: thu acc moi, doi IP, test cross-LAN

## Communication style for AI
- **Giao tiep Tieng Viet** thuan, concise
- **Focus on outcome/symptom** thay vi technical detail
- **Build click-and-use tools** (Tkinter UI, exe) thay vi CLI scripts
- AI nen explain **what** thay vi **how** code work (uoc tinh, chi giai thich technical khi user hoi)
- AI tu lam end-to-end: design + code + build + test, user chi xac nhan + dieu chinh

## Don't do
- Khong giai thich line-by-line code unless asked
- Khong tu giai dinh user biet command line / pip / git advanced
- Khong dung jargon ma khong giai thich (RE, hook, regex, etc. — neu can dung thi giai thich ngan)

## Examples thuc te trong session 2026-04-21
- User "gop game co chan IP" -> AI debug ra 3-layer + build SOCKS5 hook
- User "thoi proxy hoi bat tien, lam tab UI" -> AI write spec + plan + implement + build exe
- User noi "vai chuyen don gian" -> AI van apply spec/plan/review process tao production-ready code
