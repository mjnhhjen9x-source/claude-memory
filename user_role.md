---
name: user_role
description: User profile - non-coder game farmer using AI to build all technical infrastructure
type: user
originSessionId: 8be1cda3-5aab-4ab4-89fc-97600b06a50a
---
# User Profile

## Background
- **Khong code** — khong biet Python, JavaScript, hay bat ky language nao
- **Game farmer** — chuyen lam bot game de farm wcoin/items/currency
- **Tat ca code Python/JS/Frida/IDA trong workspace nay deu do AI viet**, user chi describe yeu cau

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
