---
name: vnku_project
description: VNKU game RE project - encryption CRACKED, full c2s+s2c protocol mapped via IDA, KNpc/KPlayer/KItem structs found, quest dialog flow complete
type: project
---

## VNKU (KHTD Mobile variant) - Game Reverse Engineering

**Workspace:** `D:\Code Tools\VNKU\` (scripts) + `D:\Code Tools\VNKU_Recon\` (extracted APK)
**Package:** `com.vennguyenkyuc.jxmobi`
**Engine:** Cocos2d-x + Lua, codebase giong KHT (~70-80% shared)
**Main lib:** `libMyGame.so` (84MB ARM 32-bit, loaded via houdini at base 0x04000000)
**IDA v30:** `D:/Code Tools/VNKU/libMyGame_v30.so` (83.2MB) — IDA 9.1 + DWARF debug info, decompile near-source quality

### Emulator
- LDPlayer 3.130, Android 5.1.1, x86 32-bit
- ADB serial: `emulator-5554` (thay doi moi khi restart)
- Frida 17.7.3 (python + server match)

### Houdini Limitation (CRITICAL)
- Frida `Interceptor.attach()` tren ARM address: hook install OK nhung **NEVER triggers**
- Houdini bypass x86 libc — game syscalls khong di qua x86 libc
- **Workaround:** Frida doc memory truc tiep (OK) + tcpdump bat packet + Python decrypt

### Encryption (CRACKED)
- **KSG_DecodeEncode**: XOR 4-byte blocks voi key, update key via LCG
- `new_key = key * 31 + 0x08088405`
- Offset: `0x004c99bc` (source: KSG_EncodeDecode.cpp)

### Key Addresses (runtime, base=0x04000000)
| Name | Offset | Note |
|------|--------|------|
| g_pClient | 0x00b1cf90 | Global CGameClient* |
| obj+0x48 | | Recv cipher key |
| obj+0x4C | | Send cipher key |
| obj+0x74 | | Socket fd |
| obj+0x88 | | Server port (0x1A0A=6666) |
| obj+0xA2 | | Connected flag (1=yes) |
| g_pGatewayStream | 0x00B1C640 | CGameClient* (IDA v30=0xB1C63C, runtime +4) |
| g_pGatewayStream GOT | 0x00B02960 | GOT entry pointing to g_pGatewayStream |
| CGameClient+0x48 | | Recv cipher key |
| CGameClient+0x4C | | Send cipher key |
| CGameClient+0x74 | | Socket fd |
| CGameClient+0x88 | | Server port (0x1A0A=6666) |
| CGameClient+0xA2 | | Connected flag (1=yes) |
| SendMsg | 0x004c906c | CGameClient::SendMsg(pBuf, nSize) |
| SendPackToServer | 0x004c90c4 | Encrypt + send via socket |
| ProcessDataStream | 0x004c838c | Packet receiver |
| KSG_DecodeEncode | 0x004c99bc | Encrypt/decrypt |
| g_SetClient | 0x004e0870 | Setter for g_pClient |
| WaitAndVerifyCipher | 0x004ca430 | Cipher handshake |

### Packet Format & Injection (VERIFIED WORKING)
- 2 bytes LE = total length (incl header)
- Remaining = encrypted payload (XOR+LCG)
- Decrypted: byte 0 = command byte
- **Inject method**: Frida read sendKey from CGameClient+0x4C, encrypt, call x86 libc send() with socket fd
- x86 libc send() at `Process.getModuleByName("libc.so").getExportByName("send")`
- After encrypt: update sendKey = key * 31 + 0x08088405, write back to CGameClient+0x4C
- **Houdini note**: Module.getExportByName("libc.so","send") returns null, must use Process.getModuleByName
- **g_pGatewayStream offset**: IDA v30 = 0xB1C63C, runtime = 0xB1C640 (diff +4, binary version mismatch)

### Protocol Commands (IDA v30 DECOMPILE + RUNTIME CAPTURE)

**Client -> Server (IDA decompile - 19 functions):**
| Cmd | Hex | Size | Function | Params |
|-----|-----|------|----------|--------|
| 0x49 | 73 | 5B | SendClientCmdRequestNpc | nID |
| 0x4B | 75 | 9B | SendClientCmdWalk | nX, nY |
| 0x4C | 76 | 9B | SendClientCmdRun | nX, nY |
| 0x4D | 77 | 13B | SendClientCmdSkill | nSkillID, nX, nY |
| 0x4E | 78 | 9B | SendClientCmdJump | nMpsX, nMpsY |
| 0x63 | 99 | 7B | SendClientCmdMoveItem | pDownPos[3B], pUpPos[3B] (low bytes of 3 DWORDs each) |
| 0x64 | 100 | 5B | SendClientCmdSell | nId |
| 0x65 | 101 | 4B | SendClientCmdBuy | nBuyIdx, nCount, nShop |
| 0x7A | 122 | 2B | SendClientCmdSit | nSitFlag |
| 0x7C | 124 | 6B | SendClientCmdStoreMoney | nDir, nMoney |
| 0x7D | 125 | 1B | SendClientCmdRevive | (none) |
| 0x82 | 130 | 5B | SendClientCmdQueryLadder | dwLadderID\|(dwCurVersion<<16), max 27 ladders |
| 0x83 | 131 | 5B | SendClientCmdRepair | dwID |
| 0x84 | 132 | 1B | SendClientCmdHorse | (none) |
| 0x86 | 134 | 9B | SendClientCmdPriced | dwID, nPrice |
| 0x88 | 136 | 17B | SendClientCmdClickShop | nDestId, nX, nY, nMoney, dwItemID, nCount |
| 0x8C | 140 | 10B | SendClientCmdSplitItem | nNum, bAuto, nX, nY |
| 0x8F | 143 | 9B | SendClientCmdMarket | nIndex, nPrice |
| 0x92 | 146 | 2B | SendClientCmdReqData | nParam |

**Client -> Server (IDA - generic senders):**
| Cmd | Size | Function | Format |
|-----|------|----------|--------|
| 0x52 | 10B | Send2Dword2Server | [0x52][bType][val1:4B][val2:4B] |
| 0x5C | 35B | GetInputToServer | [0x5C][bOk:2B][szString:32B] — text input |
| 0x67 | 3B | OnSelectFromUI | [0x67][nSelectIndex][bPutItem] — **DIALOG SELECTION** |
| 0x6A | 34B | Send3Dword/4Dword2Server | [0x6A][bType][val1-5:4B each] |

**Server -> Client (IDA v30 — full ProcessFunc table, cmd 0x43-0xFD):**
| Cmd | Handler | Bot-relevant |
|-----|---------|-------------|
| 0x43 | SyncEnd | |
| 0x44 | SyncCurPlayer | **login init, attributes** |
| 0x45 | s2cSyncAllSkill | skill list |
| 0x46 | SyncCurNormalData | **HP/MP/Stamina update** |
| 0x47 | s2cSyncByte | misc sync (VIP, state) |
| 0x48 | s2cSync2Dword | attrib modify, item update |
| 0x49 | SyncWorld | map change |
| 0x4A | SyncPlayer | player enter view (name) |
| 0x4B | SyncPlayerMin | player min sync |
| 0x4C | SyncNpc | **NPC enter view (HP/HPmax)** |
| 0x4D | SyncNpcMin | NPC min sync |
| 0x4E | SyncNpcMinPlayer | |
| 0x4F | SyncObjectAdd | |
| 0x50 | SyncObjectState | |
| 0x54 | NetCommandRemoveNpc | NPC leave |
| 0x55 | NetCommandWalk | |
| 0x56 | NetCommandRun | |
| 0x59 | NetCommandJump | |
| 0x5B | NetCommandHurt | **damage event** |
| 0x5C | NetCommandDeath | **death event** |
| 0x5F | NetCommandSkill | skill cast |
| 0x60 | GetBigPack | large packet |
| 0x61 | s2cPlayerExp | **EXP update** |
| 0x63 | s2cUpdataSelfTeamInfo | team info |
| 0x65 | s2cCreateTeam | |
| 0x6E | s2cBottleSync | |
| 0x71 | s2cLevelUp | **level up** |
| 0x73 | s2cGetCurAttribute | **STR/DEX/VIT/ENE update** |
| 0x74 | s2cGetSkillLevel | skill level |
| 0x75 | s2cSyncItem | **item sync (inventory)** |
| 0x76 | s2cRemoveItem | item remove |
| 0x77 | s2cSyncMoney | **money sync** |
| 0x78 | s2cMoveItem | item move |
| 0x79 | SyncScriptAction | **QUEST DIALOG (OnScriptAction)** |
| 0x8C | NetCommandSit | sit/stand |
| 0x8F | s2cShowMsg | message display |
| 0x92 | PlayerRevive | **revive event** |
| 0xA8 | s2cOpenWarehouse | warehouse |
| 0xAA | s2cUpdatePoint | point update |
| 0xAE | NetCommandStand | stand |
| 0xB4 | s2cItemDataArrive | |
| 0xB5 | s2cUpdatePos | position update |
| 0xFA | s2cExtend | extended protocol |
| 0xFB | s2cExtendChat | chat |
| 0xFC | s2cExtendFriend | friend |
| 0xFD | s2cExtendTong | guild |

**Runtime capture resolved:** 0x75,0x79,0xA9 = s2c handlers (NOT c2s). 0x09,0x29,0xC9 = invalid range/obsolete.

**Quest Dialog Flow:**
1. Client: `RequestNpc(0x49, npcID)` — request NPC dialog
2. Server: `SyncScriptAction(s2c 0x79)` — PLAYER_SCRIPTACTION_SYNC (2014B)
3. Client: `OnSelectFromUI(0x67, optionIndex, 0)` — select dialog option (3B!)
4. Server: next dialog or quest result
5. Text input: `GetInputToServer(0x5C, ok, text)` — 35B

### CGameClient Object (runtime)
- g_pClient tai `0x04b1cf90` -> doc ra CGameClient* (tren heap, vd `0x923f0660`)
- +0x00: vtable (`0x04abcbc8`, trong libMyGame rodata)
- +0x48: recv cipher key (thay doi moi packet)
- +0x4C: send cipher key (thay doi moi packet)
- +0x50: `0x00200022` (handshake size = 0x22 = 34 bytes)
- +0x74: socket fd (thay doi moi lan connect)
- +0x88: server port `0x1A0A` (6666)
- +0xA2: connected flag (1=yes)

### Encryption Details
- Algorithm: KSG_DecodeEncode (XOR + LCG)
- **QUAN TRONG**: XOR TOAN BO buffer voi CUNG 1 key (4 bytes lap lai), advance key 1 LAN duy nhat cuoi cung
- KHONG advance key giua cac 4-byte block! (bug cu da fix)
- Key update: `new_key = (key * 31 + 0x08088405) & 0xFFFFFFFF`
- Key khoi tao qua WaitAndVerifyCipher: server gui 34 bytes, client NOT 4 bytes lam key
- Send va Recv dung key rieng biet, moi key advance sau moi packet

### Packet Injection (DA THANH CONG)
- Doc sendKey tu g_pClient+0x4C, encrypt payload, gui qua x86 libc send()
- Re-read sendKey truoc MOI packet (game gui heartbeat thay doi key)
- Ghi newKey lai vao g_pClient+0x4C sau khi gui
- x86 libc send/recv hooks KHONG bat duoc game traffic (Houdini bypass confirmed)
- NPC interaction: SELECT(0x75, sid) + setTimeout 1s + INTERACT(0x79) = mo dialog OK
- Script: `interact_npc.js` + `run_interact.py`

### Working Scripts
- `packet_sniffer.py` — capture + decrypt live (tcpdump + Frida key read) **[MAIN TOOL]**
- `capture_npc_dialog.py` — NPC dialog capture (tcpdump background + Frida keys)
- `run_all_names.py` + `read_all_names.js` — doc ten NPC tu memory (offset 0x0BA8)
- `run_dump.py` + `dump_npc_fields.js` — dump NPC struct fields
- `search_dialog_mem.js` — search dialog text in memory
- `find_client.py` + `find_by_fd.js` + `find_client2.js` — memory scanner
- `run_dialog_block.py` — **doc + decode dialog tu heap** (TCVN3_MAP, pattern search) **[DIALOG READER]**
- `find_specific_dialog.js` — Frida script tim dialog bang keyword pattern
- `read_live_dialog3.js` — search "Da Tau" pattern in heap
- `all_giupta_dialogs.json` — 34 quest dialog chains da decode
- `game_map.json` — static analysis output
- `core/adb_finder.py` — auto-detect ADB device
- `interact_npc.js` + `run_interact.py` — **inject SELECT+INTERACT packet** (da fix encryption) **[NPC INTERACT]**
- `hook_send_recv.js` + `run_hook.py` — test x86 libc hooks (confirmed: game bypass x86 libc)
- `capture_npc_interact.py` — tcpdump + Frida key capture + decode
- `read_warehouse.js` + `read_warehouse.py` — **doc tat ca items** (equipped/bag/ruong) voi OTP
- `filter_sell.js` + `filter_sell.py` — **loc + ban do tu dong** (giu trang suc/consumable/lock, ban con lai)
- `store_warehouse.js` + `store_warehouse.py` — **cat trang suc vao ruong** (KHONG can NPC, packet truc tiep)
- `auto_train.js` + `auto_train.py` — **auto train** (revive, potion, boss detect, packet inject)
- `find_town_state.js` + `find_town_state.py` — debug tool tim trang thai trong/ngoai thanh

### Server Info
- IP: `14.225.251.148` (static.vnpt.vn), port **6666 hoac 6667** (TCP, thay doi)
- tcpdump filter: `host 14.225.251.148` (KHONG filter port vi port thay doi)
- Game: "Ven Nguyen Ky Uc" (VNKU), KHTD Mobile variant

### KNpc Struct Layout (12629 DWORDs = 0xC554 bytes, stride in Npc[] global array)
**IDA v30 DWORD offsets (multiply by 4 for byte offset):**
| DWORD | Byte | Field | Source |
|-------|------|-------|--------|
| 0 | 0x0000 | Server ID | SyncNpc |
| 7 | 0x001C | NPC Kind (0=NPC, 1=player) | SyncNpc |
| 8 | 0x0020 | Type | SyncPlayer/SyncNpc |
| 9 | 0x0024 | Level | s2cGetCurAttribute |
| 646 | 0x0A18 | SubWorld index (map ID) | GetMpsPos |
| 647 | 0x0A1C | Region index | GetMpsPos |
| **651** | **0x0A2C** | **Current HP (attrib_life)** | **Cost, SyncCurNormalData** |
| **652** | **0x0A30** | **Max HP** | **SyncNpc** |
| **654** | **0x0A38** | **Current MP (attrib_mana)** | **Cost, SyncCurNormalData** |
| **655** | **0x0A3C** | **Max MP** | **RestoreNpcBaseInfo** |
| **657** | **0x0A44** | **Current Stamina** | **Cost** |
| **658** | **0x0A48** | **Max Stamina** | **RestoreNpcBaseInfo** |
| 661 | 0x0A54 | LifeMax ratio | SyncCurPlayer |
| 728 | 0x0B60 | Region X | GetMpsPos |
| 729 | 0x0B64 | Region Y | GetMpsPos |
| 731 | 0x0B6C | Pixel offset X | GetMpsPos |
| 732 | 0x0B70 | Pixel offset Y | GetMpsPos |
| **759** | **0x0BDC** | **Target NPC index (LockSomeoneAction)** | **KCoreShell::LockSomeoneAction** |
| 764 | 0x0BF0 | Base HP value | RestoreNpcBaseInfo |
| 766 | 0x0BF8 | Base MP value | RestoreNpcBaseInfo |
| 768 | 0x0C00 | Base Stamina value | RestoreNpcBaseInfo |
| 770 | 0x0C08 | BaseAttackRating | s2cGetCurAttribute |
| 771 | 0x0C0C | BaseDefence | s2cGetCurAttribute |
| **693** | **0x0AD4** | **NPC Category (6=normal, 8=blue boss)** | **SyncNpc pMsg[36]** |
| **694** | **0x0AD8** | **NPC Category2 (always 6?)** | **SyncNpc pMsg[37]** |
| 800 | 0x0C80 | PK flag | SyncCurNormalData |
| 814 | 0x0CB8 | Last update time | multiple |
| 903 | 0x0E1C | ??? | s2cSyncMoney |
| 0x0BA8(byte) | | NPC Name (VN encoding) | runtime |

### KPlayer Struct Layout (global at dword_DE8700 = 0xDE8700)
**DWORD offsets from KPlayer base:**
| DWORD | Field | Source |
|-------|-------|--------|
| 440 | NPC index in Npc[] array | SyncCurPlayer |
| 439 | File ID | SyncCurPlayer |
| 1464 | ??? (from SyncCurPlayer pMsg[9-10]) | SyncCurPlayer |
| 1465 | ??? (from SyncCurPlayer pMsg[11-12]) | SyncCurPlayer |
| **1466** | **Strength** | **s2cGetCurAttribute case 0/4** |
| **1467** | **Dexterity** | **s2cGetCurAttribute case 1/4** |
| **1468** | **Vitality** | **s2cGetCurAttribute case 2/4** |
| **1469** | **Energy** | **s2cGetCurAttribute case 3/4** |
| 1471-1474 | Base attributes (STR/DEX/VIT/ENE) | s2cGetCurAttribute case 4 |
| 1480 | Lead EXP | SyncCurPlayer |
| 1481 | Lead Level | SyncCurPlayer |

### KItem Struct (1404 bytes, global Item[] array)
| Offset | Size | Field | Bot-relevant |
|--------|------|-------|-------------|
| 0 | 496B | m_CommonAttrib (KItemCommonAttrib) | |
| +0 | 2B | .nItemGenre | item type |
| +4 | 4B | .nDetailType | detail type |
| +8 | 2B | .nParticularType | particular |
| +16 | 4B | .nPrice | sell price |
| +20 | 1B | .nLevel | required level |
| +22 | 80B | .szItemName | **item name** |
| +490 | 2B | .nCount | **stack count** |
| 996 | 4B | m_Priced | listed price |
| 1000 | 1B | m_bLock | locked flag |
| 1378 | 2B | m_BagCount | bag count |
| 1396 | 4B | m_dwID | **item unique ID** |
| 1400 | 2B | m_nCurrentDur | durability |

### PLAYER_SCRIPTACTION_SYNC Struct (2014 bytes, s2c quest dialog)
| Offset | Size | Field |
|--------|------|-------|
| 0 | 1B | ProtocolType (0x79) |
| 1 | 2B | m_wProtocolLong |
| 3 | 1B | m_bUIId (0=dialog+options, 1=simple, 2=item UI, ...) |
| 4 | 1B | m_bOptionNum (number of options) |
| 5 | 1B | m_bParam1 |
| 6 | 4B | m_nParam (NPC param) |
| 10 | 4B | m_nBufferLen |
| 14 | 2000B | m_pContent (dialog text + option strings) |

### Key Global Addresses (IDA v30)
| Symbol | Address | Note |
|--------|---------|------|
| Player (KPlayer global) | 0xDE8700 (dword_DE8700) | current player |
| Npc[] (KNpc array) | global, stride 12629 DWORDs | all NPCs |
| Item[] (KItem array) | global | all items |
| NpcSet | global | NPC management |
| ItemSet | global (KItemSet) | item management |
| KItemList | unk_DE8DE8 | inventory list |
| off_DE8DE0 | | player NPC index |
| SubWorld (KSubWorld*) | GOT 0xB02994 | world/map data (2360 bytes) |
| SubWorldSet (KSubWorldSet) | GOT 0xB02B9C -> 0xD5F46C | 49416 bytes, contains KMapMusic |
| Npc_ptr | GOT 0xB02984 | pointer to Npc[] array |
| g_SkillManager | global | skill data |
| PlayerSet | global (KPlayerSet) | player config |

### Key Function Addresses (IDA v30)
| Function | Address | Size |
|----------|---------|------|
| CGameClient::SendPackToServer | 0x4c90c4 | 0x158 |
| KCoreShell::OperationRequest | 0x4cbe3c | 0xAE10 (44KB) |
| KProtocolProcess::ProcessNetMsg | 0x4ee358 | 0x7C |
| KProtocolProcess::KProtocolProcess | 0x4e2f78 | 0x7D4 (constructor) |
| KPlayer::SyncCurPlayer | 0x564fd4 | 0x288 |
| KPlayer::s2cGetCurAttribute | 0x564304 | 0x218 |
| KPlayer::s2cSyncMoney | 0x5646fc | 0x1AC |
| KPlayer::OnScriptAction | 0x5613a4 | 0x2004 (8KB) |
| KPlayer::OnSelectFromUI | 0x56130c | 0x80 |
| KNpc::GetMpsPos | 0x592d34 | 0x6C |
| KNpc::Cost | 0x593c18 | 0x130 |
| KNpc::SendCommand | 0x595358 | 0x44 |
| KNpc::RestoreNpcBaseInfo | 0x59a040 | 0x358 |
| KCoreShell::LockSomeoneAction | 0x4de64c | 0x138 |
| KPlayer::ApplyUseItem | 0x55dda4 | 0x2E0 |
| NetCommandDeath handler | 0x4e6768 | |
| PlayerRevive handler | 0x4e9a50 | |
| KItemList::Init | 0x5a6a6c | 0x138 |
| KItemList::GetFirstItem | 0x5aa6f0 | 0x3C |
| KItemList::GetNextItem | 0x5aa72c | 0x44 |
| KItemList::GetItemCount | 0x5aa7f4 | 0x1DC |
| KItemList::SearchID | 0x5a9204 | 0x94 |
| KItemList::SearchPosition | 0x5a9144 | 0xB8 |
| KItemList::UseItem | 0x5a8f0c | 0x1D4 |

### Vietnamese Text Encoding (TCVN3-like, confirmed)
- Custom single-byte, giong TCVN3 nhung KHONG chinh xac 100%
- ASCII chars giu nguyen (0x20-0x7E)
- High bytes (0x80+) = Vietnamese diacritics
- **Confirmed mapping (tu context):**
  - 0xA7=Đ, 0xA8=ă, 0xAA=ê, 0xAB=ô, 0xAC=ơ, 0xAD=ư, 0xAE=đ
  - 0xB5=à, 0xB6=ả, 0xB7=ã, 0xB8=á, 0xB9=ạ
  - 0xBE=ắ, 0xC7=ầ, 0xC8=ẩ, 0xCA=ấ, 0xCB=ậ
  - 0xD2=ề, 0xD3=ể, 0xD5=ế, 0xD6=ệ, 0xD7=ì, 0xDD=í, 0xDE=ị
  - 0xE1=ỏ, 0xE3=ó, 0xE5=ồ, 0xE9=ộ, 0xED=ớ, 0xEE=ợ, 0xEF=ờ
  - 0xF1=ủ, 0xF3=ú, 0xF4=ụ, 0xF5=ổ, 0xF6=ử, 0xF7=ữ, 0xF8=ứ, 0xFC=ỹ
- Full decode table in `run_dialog_block.py` (TCVN3_MAP dict)
- 0xFD = 'ý' (da xac nhan), 0xDF = 'ò' (phat hien moi, encoding.py thieu)
- 0xA4 va 0x85: hiem gap, co the la uppercase Vietnamese — bot hien thi [A4]/[85]
- **DUNG encoding.py tu KHT (AutoDaTau2.0/core/encoding.py) + bo sung 0xDF='ò'**

### NPC List (current map)
| Slot | SID | Name (raw) | Name (guess) | MPS |
|------|-----|------------|--------------|-----|
| 7 | 3298 | Ph.ng Nhi / Y Ni | Phung Nhi Y Ni | (10,4) |
| 9 | 3270 | R..ng chua.. | Ruong chua... | (6,18) |
| 21 | 3284 | ..ng Mon Th. V. | ...ng Mon Tho V... | (4,8) |
| **31** | **3275** | **D. T.u / C.m Y Ni** | **Da Tau / Cam Y Ni** | **(14,24)** |
| 33 | 3290 | V. .ang ..o Nh.n | Ve Dang Dao Nhan | (10,3) |
| 47 | 3288 | ..to C.i Bang | ...to Cai Bang | (15,12) |
| 58 | 3276 | Long Ng. | Long Ngo... | (11,27) |
| 76 | 3286 | Nga My C.m Y Ni | Nga My Cam Y Ni | (11,20) |

### NPC Da Tau Quest System (QUAN TRONG - dung nhieu)
- NPC name: Da Tau (slot 31, sid=3275, mps=(14,24))
- Quest strings found in libMyGame.so at 0x42eff49-0x4307010
- AutoIngame code ref: "DT16_%d" format string at 0x43084bb
- **Dialog text trong heap** (~0x8d2fe000 area, thay doi moi session)
- Tim dialog bang pattern search: `44 B7 20 54 C8 75` (Da Tau) hoac `67 69 F3 70 20 74 61` (giup ta)
- **Quest format (tu dialog addr 0x82da5e04):**
  - Header 12 bytes: color codes `[01][1b][16][ff]@[17][12][ff]0[12][0e][ff]`
  - Body: "Day la nhiem vu thu N: Nguoi co the tim giup ta: [ITEM], [STAT]. To: X, Toi: Y."
  - Vi du: "Nhiem vu thu 4: Nhan, tang the luc. To: 41, Toi: 80"
- **Dialog Da Tau (confirmed, ~6KB block):**
  - "Da Tau: Nguoi da tim duoc vat pham ma ta yeu cau chua?" — hoi khi click
  - "Vi dai hiep! Nguoi bo nhieu do..." — nop qua nhieu
  - "Nhung viec nguoi lam chua dung yeu cau..." — nop sai
  - "Da Tau: Vat pham khong dung yeu cau. Chi can 1 vien Thuy Tinh!" — sai item
  - "Da Tau: Can cu vao cap do kho cua nhiem vu..." — goi y hoan thanh nhanh
  - "Hom nay nguoi da lam #s nhiem vu..." — het luot
  - "Huy bo nhiem vu can Thuy Tinh hoac 100 manh Son Ha Xa Tac"
  - "#s placeholder = so nhiem vu (server fill)

### Dialog Buffer (heap, ~0x8d300000 area)
- Chua TOAN BO dialog script cua game (tat ca NPC)
- Tim bang Memory.scanSync voi pattern cua tu khoa
- 34 quest chains da doc duoc (luu tai `all_giupta_dialogs.json`)
- Bao gom: Da Tau, Thieu Lam, Mac Sau, Tong Kim, Tieu Cuc, Vo Dang, Con Lon, v.v.

### KItemList Struct (global at unk_DE8DE8, inventory management)
**Layout:**
| Offset | Size | Field |
|--------|------|-------|
| 0 | 4B | nPlayerIdx |
| 4 | 4B | flag (warehouse mode) |
| 72 | 3728B | PlayerItem[233] array (16B each) |
| 3800 | 8B | KLinkArray FREE list (DO NOT use for reading items!) |
| 3808 | 8B | KLinkArray USED list (dung de duyet items) |
| 3816 | 4B | DWORD 954: iteration cursor |
| 3820 | 16B | KInventory bag room 0 (5x6 grid) |
| 3836 | 16B | KInventory bag room 1 (5x6 grid) |
| 3852-3948 | | KInventory rooms 2-8 (bag/equip) |

**PlayerItem (16 bytes at this+72+16*index):**
| Offset | Field | Note |
|--------|-------|------|
| 0 | Item[] index | index into global Item[] array |
| 4 | nPlace | room ID (see below) |
| 8 | nPosX | grid X position |
| 12 | nPosY | grid Y position |

**Place values:** 1=equip special, **2=equipped (dang mac)**, **3=bag (tui do)**, **4=warehouse (ruong do, doc duoc tu xa KHONG can mo NPC!)**, 5-10=unknown, 100=unused

**KLinkArray traversal (USED list, offset 3808):**
- pNode = *(KItemList+3808) — pointer to KLinkNode array
- first = *(pNode+4) — first nodeIdx
- next = *(pNode+8*nodeIdx+4) — next nodeIdx
- Stop when nodeIdx == 0

### Item Reading System (read_warehouse.js + read_warehouse.py)
**GOT addresses:**
| GOT | Symbol | Usage |
|-----|--------|-------|
| 0xB02BA0 | Item_ptr | Pointer to Item[] global array |
| 0xB02C38 | ItemGen_ptr | KItemGenerator instance |

**KItem attrib arrays (KMagicAttrib = 16B: type(4B) + value[3](12B)):**
| Offset | Count | Field | Description |
|--------|-------|-------|-------------|
| +496 | 7 | m_aryBaseAttrib | Base stats (DmgMin, Def, etc) |
| +608 | 6 | m_aryRequireAttrib | Requirements (ReqStr, ReqLv, etc) |
| +704 | 6 | m_aryMagicAttrib | **OTP** (random magic options) |
| +800 | 2 | m_aryExpandAttrib | Expand |
| +832 | 2 | m_arySmeltAttrib | Smelt/refine |
| +864 | 8 | m_aryMagicAttribExtra | Extra magic |

**Attrib name resolution:** KItemGenerator(0xB02C38)+23116 = KBPT_MagicAttrib_TF table
- KBasicPropertyTable: +1204=m_pBuf, +1208=numEntries (295 records, 424B each)
- KMAGICATTRIB_TABFILE: +4=m_szName(80B), +348=nPropKind (attrib type ID)

**Equipment detailType mapping (genre=0, tu IDA GetEquipmentCommonAttrib 0x52b534):**
| detailType | Name | Jewelry? |
|------------|------|----------|
| 0 | MeleeWeapon | |
| 1 | RangeWeapon | |
| 2 | Armor | |
| **3** | **Ring (Nhan)** | **YES** |
| **4** | **Amulet/Necklace (Day Chuyen)** | **YES** |
| 5 | Boot | |
| 6 | Belt | |
| 7 | Helm | |
| 8 | Cuff | |
| **9** | **Pendant (Huong Nang/Ngoc Boi)** | **YES** |
| 10 | Horse | |
| 11 | Mask | |
| 12 | Gown | |
| 13 | Signet | |
| 14 | Jewelry (general) | |

**Jewelry filter rule:** JEWELRY_TYPES = {3, 4, 9} — giu lai TAT CA trang suc bat ke OTP

### Item Filter + Sell Tool (DA HOAN THANH)
- Script: `filter_sell.js` (Frida) + `filter_sell.py` (Python runner)
- Chay: `python -u filter_sell.py` (xem) hoac `python -u filter_sell.py --sell` (ban)
- **Sell packet**: `[0x64][dwID:4B LE]` = 5 bytes, gui qua x86 libc send()
- **Ban trong thanh KHONG can NPC shop** — packet sell hoat dong o moi noi trong thanh
- Logic loc:
  - GIU: trang suc (detailType 3,4,9) + LOCK + consumable (genre!=0)
  - BAN: tat ca trang bi con lai trong tui (place=3)

### Trang thai trong/ngoai thanh
- **pkFlag**: KNpc offset 800*4 (0xC80) — doc tu `npcBase + playerIdx * 0xC554 + 0xC80`
- **pkFlag=0** → trong thanh (ban do OK, khong danh duoc)
- **pkFlag=1** → ngoai thanh (danh duoc, ban do KHONG duoc)
- Ghi pkFlag=0 khi o ngoai thanh: packet gui OK nhung CHUA CONFIRM server co chap nhan khong
- SubWorld dw1 cung thay doi: 56 (ngoai) vs 80 (trong) — co the server validate bang map ID

### Warehouse Store (DA THANH CONG)
- **Packet**: `Send2Dword2Server(bType=10, m_dwID, 4)` — dw2=4 la place=warehouse
- **KHONG can dung truoc NPC** — server chap nhan o bat ky vi tri TRONG THANH (pkFlag=0)
- **Ngoai thanh (pkFlag=1)**: packet gui OK nhung server REJECT — item van trong bag
- Script: `store_warehouse.js` + `store_warehouse.py`
- Chay: `python -u store_warehouse.py` (xem) hoac `python -u store_warehouse.py --store` (cat do)
- Logic loc: cat trang suc (Ring/Amulet/Pendant) KHONG lock vao ruong, giu lai con lai
- Live decrypt: key formula `key*31+0x08088405` DUNG (IDA confirmed tai 0x4ca90c)

### Buy Items (DA THANH CONG)
- **Packet**: `SendClientCmdBuy(nBuyIdx, nCount, nShop)` = `[0x65][buyIdx:1B][count:1B][shop:1B]` (4 bytes)
- **Shop mo bang**: `Cmd75(npcSID)` = `[0x75][sid:4B LE][0x00 x4]` (9 bytes) — can NPC LOAD (dung gan)
- **Tab/select**: `Cmd67(selectIdx, putItem)` = `[0x67][idx:1B][put:1B]` (3 bytes)
- **shopID global**: `dword_DE9D68` (base+0xDE9D68) — thay doi khi OpenSale
- **BuySell global**: offset `0xB9FDE8` — KBuySell struct (m_MaxItem+16, m_Item+20, m_Price+0)
- **KBuySell::GetItem**: `m_Item[nIndex]` — buyIdx = index trong mang KItem cua shop
- **KBuySell::OpenSale** (0x523818): set shopID, alloc item array, prices

**Shop mapping (DA XAC NHAN bang doc BuySell memory):**
| Item | NPC | shopID | buyIdx | Gia |
|------|-----|--------|--------|-----|
| Kim Sang Duoc (dai) | Ong chu Duoc Diem | 52 | 2 | 250 |
| Tho Dia Phu | Tap Hoa | 4 | 0 | 500 |

**Dieu kien mua:**
- NPC phai **LOAD** (client render) → can dung gan NPC
- Cmd75(npcSID) mo shop server-side → shopID memory update
- Buy packet gui voi shopID hien tai
- **KHONG mua tu xa duoc** — khac voi sell/store warehouse (hoat dong o moi noi trong thanh)
- NPC SID **thay doi moi session** — phai scan NPC array dong (findNPC by name)

**NPC scan**: Npc_ptr (GOT 0xB02984), stride 0xC554, name tai +0x0BA8
- Tim "d-ie" (Duoc Diem) va "ta.p" (Tap Hoa) trong ten VN transliterated
- Scan toi da 300 slots (>300 co the access violation)

**Scripts:**
- `buy_items.js` + `buy_items.py` — tu dong tim NPC + mo shop + mua
- `read_buysell2.js` — doc tat ca items trong shop dang mo (buyIdx + ten + gia)
- `find_shop_npc.js` + `find_shop_npc.py` — scan NPC slots
- `watch_shop.py` — monitor shopID thay doi
- Chay: `python -u buy_items.py` (preview) hoac `python -u buy_items.py --buy` (mua)
- Tim starting key: dung prev_key (modular inverse 31^-1 mod 2^32 = 0xBDEF7BDF) tu KEY_AFTER

### Packet Injection tren Houdini (VERIFIED)
- Khong goi duoc ARM function (illegal instruction)
- Khong hook duoc ARM function (hook OK nhung never trigger)
- **NHUNG**: tu encrypt payload (XOR+LCG) + gui qua x86 libc send() = **HOAT DONG**
- Flow: doc sendKey(+0x4C) -> encrypt -> x86 send(socketFd, packet) -> ghi newKey lai
- Script mau: `auto_train.js` function `encryptAndSend()`
- Mo UI ruong tu xa: KHONG DUOC (can goi CoreDataChanged = ARM function)

**Linked list iteration (for Frida memory reader):**
1. Read `this[952]` = KLinkArray head pointer
2. Read `*(head + 4)` = first node index
3. PlayerItem at `this + 72 + 16*index`
4. Next: `*(head + 8*index + 4)` = next index (0=end)

**Genre bitmask routing:**
- `(1<<genre) & 0x52` → genres 1,4,6 = consumable → EatMedicine → ApplyUseItem packet
- `(1<<genre) & 0x81` → genres 0,7 = equipment → CanEquip

**GetItemCount(detailType, genre, parti, level, nPos, checkLock):**
- Iterates linked list, filters by genre + detailType + particularType + level + room
- Returns sum of nCount for matching items
- checkLock: 0=any, 1=locked(m_bLock&2), 2=unlocked

### Boss Detection (runtime verified)
- **Field:** `KNpc + 693` (DWORD offset, byte 0x0AD4) = NPC Category from SyncNpc `pMsg[36]`
- **Value 6** = quai thuong (ten trang)
- **Value 8** = **boss xanh** (ten xanh duong)
- Tested across multiple maps: map thap (Nhim/Heo rung HP:200) va map cao (Giang Chay HP:32800) — consistent
- Frida check: `npc.add(693 * 4).readS32() !== 6` → is boss
- Houdini note: doc memory OK, dung `/proc/self/maps` parse base thay vi `getModuleByName`
- Script: `D:\Code Tools\VNKU\scan_npc.js` + `scan_npc.py`

### Combat & Target System (IDA v30)
- **LockSomeoneAction** (0x4de64c): store target at `Npc[playerIdx*12629 + 759]`, auto-run if distance > 420
- **Use Item packet**: `[0x61][0x03][posX][posY][flags]` (5 bytes) — ApplyUseItem (0x55dda4)
- Item genre check: genre 1/4/6 = consumable (calls EatMedicine), genre 0/7 = equipment
- **Death**: s2c 0x5C → `KNpc::ProcNetCommand(do_death)`, client requests revive with c2s 0x7D
- **Revive**: s2c 0x92 → `KNpc::ProcNetCommand(do_revive)` for player NPCs

### RestoreNpcBaseInfo Attribute Map (0x59a040)
- `KNpc+764` → base HP → copies to +651 (curHP) and +652 (maxHP)
- `KNpc+766` → base MP → copies to +654 (curMP) and +655 (maxMP)
- `KNpc+768` → base Stamina → copies to +657 (curStamina) and +658 (maxStamina)

### OnScriptAction Dialog Parsing (case 0/8/A/B - quest dialog)
- **Buffer layout**: `260 * m_bOptionNum + 2408` bytes allocated
  - `v5[0..2399]` = main dialog text (2400B max)
  - `*(DWORD*)(v5+2400)` = TEncodeText handle
  - `*(DWORD*)(v5+2404)` = actual option count
  - `v5[2408 + 260*i]` = option string i (256B text + 4B sentinel=-1)
- **m_bParam1=0** (raw text mode): `m_pContent` = `[main_text]\x1E[opt1]\x1E[opt2]...`
  - Separator: byte `0x1E` (ASCII Record Separator, decimal 30)
- **m_bParam1!=0** (string resource mode): `m_pContent[0:4]` = string resource ID → `g_GetStringRes()`
  - `m_pContent[4:4+v90]` = substitution string (replaces `#s` placeholders in resolved text)
  - Options follow after substitution data, same `0x1E` separator
- **Auto-quest bot key**: hook SyncScriptAction(0x79), parse m_pContent by `0x1E`, match option text, send `[0x67][index][0x00]`

### KSubWorld Struct (2360 bytes, GOT SubWorld_ptr = 0xB02994)
| Offset | Size | Field | Note |
|--------|------|-------|------|
| 0 | 4B | m_nIndex | -1 at runtime |
| 4 | 4B | m_SubWorldID | map ID (e.g. 56) |
| 8 | 4B | m_Region | KRegion* pointer |
| 12 | 18B | m_ClientRegionIdx | short[9] — loaded region indices |
| 30 | 80B | m_szMapPath | e.g. "\maps\..." |
| 112 | 4B | m_nJustLoadRID | |
| 116 | 4B | m_nWorldRegionWidth | region grid width (e.g. 19) |
| 120 | 4B | m_nWorldRegionHeight | region grid height (e.g. 17) |
| 124 | 4B | m_nTotalRegion | loaded regions (e.g. 9 = 3x3) |
| 128 | 4B | m_nRegionWidth | cells per region X (e.g. 16) |
| 132 | 4B | m_nRegionHeight | cells per region Y (e.g. 32) |
| 136 | 4B | m_nCellWidth | MPS per cell X (e.g. 32) |
| 140 | 4B | m_nCellHeight | MPS per cell Y (e.g. 32) |
| 144 | 4B | m_nRegionBeginX | first region column (e.g. 88) |
| 148 | 4B | m_nRegionBeginY | first region row (e.g. 96) |
| 152 | 4B | m_nRegionEndX | |
| 156 | 4B | m_nRegionEndY | |
| 160 | 4B | m_dwCurrentTime | game time |

### KRegion Struct (184 bytes, stride 46 DWORDs)
| Offset | Size | Field | Note |
|--------|------|-------|------|
| 0 | 2B | m_nIndex | short |
| 4 | 4B | m_RegionID | packed (X<<16 | Y) |
| 8 | 24B | m_NpcList | KList |
| 32 | 24B | m_ObjList | KList |
| 56 | 24B | m_MissleList | KList |
| 80 | 24B | m_PlayerList | KList |
| 104 | 16B | m_nConnectRegion | short[8] |
| 120 | 4B | m_nRegionX | **MPS origin X** |
| 124 | 4B | m_nRegionY | **MPS origin Y** |
| 128 | 2B | m_nWidth | cells wide |
| 130 | 2B | m_nHeight | cells tall |
| 168 | 4B | m_nActive | |

### Map Coordinate System
- **MPS** = world coordinate system (Map Position System)
- KNpc+728/729 = **cell position** within region (NOT MPS!)
- **MPS = regionOrigin + cell * cellSize**
- Map bounds: `beginMPS = regionBegin * regionWidth * cellWidth`
- Server loads 3x3 region grid around player (9 regions)
- Move packet (0x4C) uses MPS coordinates
- Map ID 56 example: MPS (45056,98304)-(54784,115712), size 9728x17408
- Scripts: `read_map.js` + `read_map.py`

### Pending
- ~~Phan tich sau attack packet~~ → Attack = Skill (cmd 0x4D, SendClientCmdSkill)
- ~~Map quest accept/complete packet~~ → OnSelectFromUI (cmd 0x67, 3B)
- ~~Tim player data trong memory~~ → KPlayer+1466-1469 (STR/DEX/VIT/ENE), KNpc+651/652/654 (HP/HPmax/MP)
- ~~Tim MP max offset~~ → KNpc+655 (maxMP), +658 (maxStamina) — from RestoreNpcBaseInfo
- Patrol bot: di chuyen zigzag qua map, scan boss moi region
- ~~Death/Revive handlers~~ → Death s2c 0x5C, Revive s2c 0x92, client revive c2s 0x7D
- ~~LockSomeoneAction + Use Item~~ → target at KNpc+759, UseItem=[0x61][0x03][posX][posY][flags]
- ~~Parse OnScriptAction case 0~~ → dialog text + options separated by 0x1E, option buffer at +2408
- ~~Hoan thien bang ma VN~~ → dung encoding.py + 0xDF='ò', 0xA4/0x85 hiem gap
- ~~KItemList inventory~~ → PlayerItem[233] linked list, 16B/slot, iterate via GetFirst/GetNext
- **Build auto bot Da Tau** (flow: RequestNpc(0x49) → read SyncScriptAction(0x79) → OnSelectFromUI(0x67) → loop)
- Frida memory reader cho HP/MP/EXP/Money (doc tu global addresses)
- Hook SyncScriptAction de tu dong parse quest dialog
- Build packet injector cho full quest automation
