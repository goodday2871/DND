# 遊戲數據管理系統

## 遊戲數據管理系統

### 問題背景

在之前的遊戲運作中,由於缺乏數據持久化機制,導致以下問題:
- 同一個任務每次讀檔後賞金不同
- 同一種怪物的屬性和掉落物會變化
- 商人的收購價格浮動不定
- 住宿、餐飲等固定服務的價格改變

這會破壞遊戲的一致性和沉浸感。

### 解決方案:JSON 數據庫

每個角色存檔資料夾下新增 `data/` 子資料夾,使用 JSON 格式持久化所有需要保持一致的數據。

**完整資料夾結構:**
```
saves/
└── [角色名]-[職業]-[日期]/
    ├── character.md      # 角色屬性與技能
    ├── equipment.md      # 裝備與財產
    ├── location.md       # 當前位置
    ├── npcs.md           # NPC 關係
    ├── quests.md         # 任務進度
    ├── journal.md        # 冒險日誌
    ├── notes.md          # GM 備註
    ├── meta.md           # 存檔元資料
    └── data/             # 遊戲數據 (新增)
        ├── quests.json      # 任務數據
        ├── monsters.json    # 怪物數據
        ├── items.json       # 道具數據
        ├── prices.json      # 價格數據
        └── locations.json   # 地點數據
```

### 數據文件說明

#### 1. `quests.json` - 任務數據庫

**用途:** 記錄所有任務的固定賞金、獎勵、進度

**結構:**
```json
{
  "quests": [
    {
      "id": "quest_001",
      "name": "任務名稱",
      "giver": "發布者名稱",
      "location": "發布地點",
      "description": "任務描述",
      "reward_gold": 50,
      "reward_items": ["道具1", "道具2"],
      "objectives": {},
      "status": "completed",
      "accepted_at": "第1天",
      "completed_at": "第2天"
    }
  ]
}
```

**重要:** `reward_gold` 和 `reward_items` 一旦生成就永久固定

#### 2. `monsters.json` - 怪物數據庫

**用途:** 記錄所有怪物種類的固定屬性和戰利品規則

**結構:**
```json
{
  "monster_types": [
    {
      "id": "slime_common",
      "name": "史萊姆",
      "hp": 8,
      "ac": 10,
      "attack_bonus": 2,
      "damage": "1d4+1",
      "xp": 25,
      "loot_table": {
        "slime_gel": {
          "chance": 1.0,
          "quantity": "1d3"
        }
      },
      "first_encounter": "第2天",
      "total_killed": 3
    }
  ],
  "encounters": [
    {
      "id": "encounter_001",
      "date": "第2天下午",
      "monster_type": "slime_common",
      "quantity": 3,
      "result": "victory",
      "loot_obtained": {
        "slime_gel": 5
      }
    }
  ]
}
```

**重要:** 同種怪物的 HP、AC、攻擊力、戰利品規則固定

#### 3. `items.json` - 道具數據庫

**用途:** 記錄所有道具的基礎屬性和庫存追蹤

**結構:**
```json
{
  "items": [
    {
      "id": "slime_gel",
      "name": "史萊姆膠質",
      "type": "material",
      "base_price": 5,
      "quality": "common",
      "weight": 0.1,
      "total_obtained": 5,
      "total_sold": 0,
      "total_used": 0,
      "current_stock": 5
    }
  ]
}
```

**重要:** 追蹤數量流向,確保 `current_stock = obtained - sold - used`

#### 4. `prices.json` - 價格數據庫

**用途:** 記錄固定價格和商人交易倍率

**結構:**
```json
{
  "fixed_prices": {
    "services": {
      "inn_room": 20,
      "inn_meal": 5,
      "ale": 1
    },
    "common_goods": {
      "bread": 2,
      "rope_50ft": 100
    }
  },
  "merchants": {
    "alchemist_lilian": {
      "name": "煉金術師莉莉安",
      "location": "綠野村",
      "buy_multiplier": 0.5,
      "sell_multiplier": 1.5,
      "specialties": ["materials", "potions"]
    }
  },
  "transaction_history": [
    {
      "id": "trans_001",
      "date": "第2天",
      "type": "sell",
      "merchant": "alchemist_lilian",
      "item": "slime_gel",
      "quantity": 5,
      "unit_price": 2.5,
      "total": 12.5
    }
  ]
}
```

**重要:**
- `fixed_prices` 永久固定
- 商人 `buy_multiplier` 和 `sell_multiplier` 固定
- 所有交易必須記錄到 `transaction_history`

**價格計算公式:**
- 商人收購價 = `items.json` 的 `base_price` × `buy_multiplier`
- 商人販售價 = `items.json` 的 `base_price` × `sell_multiplier`

#### 5. `locations.json` - 地點數據庫

**用途:** 記錄所有地點的固定屬性和服務

**結構:**
```json
{
  "locations": [
    {
      "id": "greenfield_village",
      "name": "綠野村",
      "type": "village",
      "population": 200,
      "first_visit": "第1天",
      "services": {
        "inn": {
          "name": "綠野旅店",
          "room_price": 20,
          "meal_price": 5
        },
        "blacksmith": {
          "name": "湯姆鐵匠鋪",
          "owner": "鐵匠湯姆"
        }
      }
    }
  ]
}
```

**重要:** 地點的服務價格和設施固定

---

### GM 操作規範

#### 數據查詢規則 (強制執行)

**在回應以下內容前,必須先讀取對應 JSON:**

| 內容類型 | 必須讀取的文件 | 禁止行為 |
|---------|--------------|---------|
| 任務賞金 | `quests.json` | ❌ 不查詢就說出賞金 |
| 怪物屬性/掉落 | `monsters.json` | ❌ 不查詢就隨機生成 |
| 道具價格 | `items.json` + `prices.json` | ❌ 不查詢就決定價格 |
| 商人收購/販售 | `prices.json` | ❌ 不查詢就報價 |
| 住宿/服務 | `prices.json` (fixed_prices) | ❌ 不查詢就收費 |
| 地點描述 | `locations.json` | ❌ 不查詢就隨意設定 |

#### 數據更新規則 (強制執行)

**以下事件發生後,必須立即更新 JSON:**

| 事件類型 | 更新文件 | 更新內容 |
|---------|---------|---------|
| 接取/完成任務 | `quests.json` | 任務狀態、完成時間 |
| 遭遇新種類怪物 | `monsters.json` | 新增怪物類型數據 |
| 戰鬥結束 | `monsters.json` | 新增遭遇記錄、戰利品 |
| 獲得新道具 | `items.json` | 新增道具、更新數量 |
| 使用/販售道具 | `items.json` | 更新 used/sold/stock |
| 任何交易 | `prices.json` | 新增 transaction_history |
| 首次使用服務 | `prices.json` | 新增 fixed_prices |
| 首次到達地點 | `locations.json` | 新增地點數據 |

#### 數據生成時機

**只在「第一次遭遇」時隨機生成,之後永久固定:**

- ✅ 第一次聽說某任務 → 生成固定賞金
- ✅ 第一次遭遇某種怪物 → 生成固定屬性
- ✅ 第一次獲得某道具 → 生成固定 base_price
- ✅ 第一次使用某服務 → 生成固定價格
- ✅ 第一次與某商人交易 → 生成固定倍率

**之後都使用已記錄的數據**

---

### 存檔操作整合

#### 「存檔」指令整合

執行步驟:
1. 更新所有 `.md` 檔案 (character, equipment, journal 等)
2. **同時更新** `data/` 資料夾下的所有 JSON
3. 驗證數據一致性:
   - `items.json` 的庫存 = `equipment.md` 的道具
   - `prices.json` 的交易總額 = `equipment.md` 的財產變化
4. 顯示: `✅ 已將記錄儲存完成`

#### 「讀檔」指令整合

執行步驟:
1. 讀取所有 `.md` 檔案
2. **同時讀取** `data/` 資料夾下的所有 JSON
3. 將 JSON 數據載入記憶
4. 繼續遊戲時,所有數值查詢都參照 JSON

#### 「上傳」指令整合

Git commit 會自動包含:
- 所有 `.md` 檔案
- `data/` 資料夾下的所有 JSON 檔案

**Commit message 範例:**
```
記錄存檔 - 第2天傍晚 - 初次狩獵與成長

- 完成史萊姆討伐任務
- 獲得戰利品並販售
- 更新任務/怪物/道具/價格數據
```

---

### 操作流程範例

#### 範例 1: 玩家接取新任務

**情境:** 玩家去找村長接任務

**GM 操作:**
```
步驟 1: Read data/quests.json
步驟 2: 檢查是否有 "quest_greenfield_slime"
步驟 3: 不存在,生成新任務數據:
  {
    "id": "quest_greenfield_slime",
    "name": "清理史萊姆巢穴",
    "reward_gold": 50,  // 隨機 30-70,這次是 50
    "reward_items": ["治療藥水 x2"],
    "status": "pending"
  }
步驟 4: Write data/quests.json (立即寫入!)
步驟 5: 回應玩家: "村長說賞金是 50 金幣和 2 瓶治療藥水"
```

**下次玩家讀檔後再問:**
```
步驟 1: Read data/quests.json
步驟 2: 找到 "quest_greenfield_slime"
步驟 3: 讀取 reward_gold: 50
步驟 4: 回應玩家: "村長說賞金是 50 金幣和 2 瓶治療藥水" (相同!)
```

#### 範例 2: 遭遇新種類怪物

**情境:** 玩家首次遭遇史萊姆

**GM 操作:**
```
步驟 1: Read data/monsters.json
步驟 2: 檢查 monster_types 中是否有 "slime_common"
步驟 3: 不存在,生成新怪物類型:
  {
    "id": "slime_common",
    "name": "史萊姆",
    "hp": 8,
    "ac": 10,
    "attack_bonus": 2,
    "damage": "1d4+1",
    "loot_table": {
      "slime_gel": { "chance": 1.0, "quantity": "1d3" }
    }
  }
步驟 4: Write data/monsters.json (立即寫入!)
步驟 5: 使用這些屬性進行戰鬥
步驟 6: 戰鬥結束後,新增遭遇記錄到 encounters
```

**下次再遇到史萊姆:**
```
步驟 1: Read data/monsters.json
步驟 2: 找到 "slime_common"
步驟 3: 使用已記錄的 HP: 8, AC: 10 等屬性 (相同!)
步驟 4: 戰鬥結束後,新增該次遭遇記錄
```

#### 範例 3: 販售戰利品

**情境:** 玩家要賣 5 個史萊姆膠質給煉金術師

**GM 操作:**
```
步驟 1: Read data/items.json
  → slime_gel.base_price = 5
步驟 2: Read data/prices.json
  → alchemist_lilian.buy_multiplier = 0.5
步驟 3: 計算收購價 = 5 × 0.5 = 2.5 金/個
步驟 4: 總價 = 2.5 × 5 = 12.5 金 (12金5銀)
步驟 5: 玩家確認交易
步驟 6: Edit data/items.json
  → total_sold: 5, current_stock: 0
步驟 7: Edit data/prices.json
  → 新增 transaction_history 記錄
步驟 8: Edit equipment.md
  → 財產 +12金5銀, 移除史萊姆膠質 x5
步驟 9: 回應玩家: "莉莉安以每個 2 金 5 銀收購你的史萊姆膠質..."
```

**下次再賣同樣道具給同一個商人:**
```
價格計算完全相同: 5 × 0.5 = 2.5 金/個 (一致!)
```

#### 範例 4: 使用固定價格服務

**情境:** 玩家第一次在旅店住宿

**GM 操作:**
```
步驟 1: Read data/prices.json
步驟 2: 檢查 fixed_prices.services.inn_room
步驟 3: 不存在,生成固定價格:
  "inn_room": 20  // 隨機 15-25,這次是 20
步驟 4: Write data/prices.json (立即寫入!)
步驟 5: Edit equipment.md → 財產 -20 銀
步驟 6: 回應玩家: "住一晚要 20 銀幣"
```

**之後每次住宿:**
```
步驟 1: Read data/prices.json
步驟 2: 讀取 fixed_prices.services.inn_room = 20
步驟 3: Edit equipment.md → 財產 -20 銀
步驟 4: 回應玩家: "住一晚要 20 銀幣" (永遠相同!)
```

---

### 數據一致性驗證

#### 定期檢查項目

**道具庫存一致性:**
```
items.json 的 current_stock
= total_obtained - total_sold - total_used

與 equipment.md 的道具清單必須一致
```

**財產一致性:**
```
equipment.md 的總財產
= 初始財產 + 所有收入 - 所有支出

收入/支出必須能從 transaction_history 追溯
```

**任務完成度一致性:**
```
quests.json 的 status
與 quests.md 的任務進度必須一致
```

#### 錯誤處理

若發現數據不一致:
1. 優先信任 JSON 數據 (因為有完整歷史記錄)
2. 根據 JSON 修正 `.md` 檔案
3. 在 `notes.md` 中記錄修正原因

---

### 創建新角色時的初始化

**步驟:**
1. 創建角色資料夾: `saves/[角色名]-[職業]-[日期]/`
2. 創建 8 個 `.md` 檔案
3. **創建 `data/` 資料夾**
4. **創建 5 個初始 JSON 檔案:**

```bash
mkdir -p "saves/[角色名]-[職業]-[日期]/data"
cd "saves/[角色名]-[職業]-[日期]/data"

# 創建空的 JSON 文件
echo '{"quests":[]}' > quests.json
echo '{"monster_types":[],"encounters":[]}' > monsters.json
echo '{"items":[]}' > items.json
echo '{"fixed_prices":{"services":{},"common_goods":{}},"merchants":{},"transaction_history":[]}' > prices.json
echo '{"locations":[]}' > locations.json
```

---

### 現有存檔的數據遷移

對於已存在的角色 (如帕拉梅拉),需要進行數據遷移:

**步驟:**
1. 創建 `data/` 資料夾
2. 從 `journal.md` 和其他檔案提取歷史數據:
   - 已完成的任務和實際賞金
   - 遭遇過的怪物種類
   - 獲得的道具和實際交易價格
   - 歷史交易記錄
3. 創建初始 JSON 檔案並填入提取的數據
4. 驗證數據一致性
5. 未來所有操作遵循新規則

**遷移優先級:**
- 高優先級: `prices.json` (避免價格浮動)
- 中優先級: `quests.json`, `items.json`
- 低優先級: `monsters.json`, `locations.json`

---

### 總結

**核心原則:**
1. **先讀取,再回應** - 所有數值查詢必須先讀 JSON
2. **即時更新** - 事件發生後立即寫入 JSON
3. **數據固定** - 第一次生成後永久不變
4. **追蹤一致性** - 所有數量變化都要記錄

**效益:**
- ✅ 遊戲世界一致性
- ✅ 玩家體驗合理性
- ✅ 數據可追溯性
- ✅ 防止數值浮動

---
