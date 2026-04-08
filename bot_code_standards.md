---
name: bot_code_standards
description: Quy tac code bot game chung - kien truc 5 tang, code style, debug, error handling, ap dung moi game
type: feedback
---

# Bot Game Code Standards

## Kien truc 5 tang

```
Tang 5: UI/Manager     - Dashboard, multi-account, log viewer
Tang 4: Bot Logic       - State machine, dieu kien, uu tien
Tang 3: Game Actions    - Di chuyen, danh, nhat do, quest
Tang 2: Game State      - Doc HP, vi tri, inventory, quai
Tang 1: Game Bridge     - Frida/BepInEx/ADB ket noi game
```

### Tang 1-3: VIET LAI per game (offset, function khac nhau)
### Tang 4-5: DUNG LAI (state machine, UI chung)

## Cau truc folder

```
game_bot/
├── config/             # Cau hinh (accounts, train spots, settings)
├── bridge/             # Tang 1: ket noi game (Frida/ADB/socket)
│   └── hooks/          # Frida JS scripts
├── state/              # Tang 2: doc trang thai game
├── actions/            # Tang 3: thuc hien hanh dong
├── logic/              # Tang 4: state machine, dieu kien
├── ui/                 # Tang 5: dashboard
├── utils/              # Logger, timer, error handler
├── tests/              # Test tung tang
├── memory_map.py       # TAT CA offset 1 file
├── main.py             # Entry point
└── README.md
```

## Quy tac code

### Functions
- 1 function = 1 viec duy nhat
- Moi function co docstring (Args, Returns)
- Return True/False cho actions (thanh cong/that bai)
- Dat ten ro rang: attack_target(), move_to_spot(), check_hp()

### Error handling
- Tang 1-3: KHONG catch exception, throw len
- Tang 4: Catch + log + chuyen state an toan
- Tang 5: Catch + hien thi UI + auto-restart
- Dung decorator @safe_tick cho state machine

### Logging
- Format: [TIME] [ACCOUNT] [MODULE] LEVEL: message
- Log MOI hanh dong va state change
- File rieng per account (logs/account_name.log)
- Console + file dong thoi

### Memory map
- TAT CA offset trong 1 file memory_map.py
- Comment: # IDA function name hoac cach tim
- Game update → sua 1 file → bot chay lai

### State Machine
- Moi state co: enter_XXX() (1 lan) + update_XXX() (moi tick)
- change_state(new, reason) - log ly do chuyen
- Timeout per state (mac dinh 5 phut) → reset IDLE
- Critical check MOI tick: chet, disconnect, HP thap

### Priority xu ly
1. Song sot (HP, respawn, disconnect)
2. Nhiem vu (quest, reset level)
3. Toi uu (stats, gear, potion)
4. Farm (danh quai, nhat do)

### Config
- accounts.json: tai khoan (KHONG commit git)
- settings.py: cau hinh chung
- train_spots.py: bai train per level
- Doi account/spot KHONG sua code

### Multi-account
- Moi account = 1 BotWorker thread rieng
- Log rieng per account
- Crash 1 account khong anh huong account khac
- Auto-restart per account

### Frida JS
- var (KHONG let/const)
- ASCII only console output
- Game function goi tu game thread (runOnGameThread queue)
- RPC exports cho Python goi

### Testing
- Test tung tang doc lap
- test_state.py: doc HP, position, verify range
- test_actions.py: goi 1 action, verify ket qua
- test_logic.py: simulate state transitions

### Debug
- Log du de debug KHONG can breakpoint
- Moi state change co reason
- Moi action co log truoc/sau
- Error co full traceback trong log file
- UI hien thi state hien tai + last error

## Ap dung game moi

1. Copy template folder
2. Viet memory_map.py (offset tu IDA)
3. Viet bridge/ (Frida hooks)
4. Viet state/ (doc game data)
5. Viet actions/ (goi game function)
6. Tuy chinh logic/ (them/bot state)
7. Tang 5 UI giu nguyen
