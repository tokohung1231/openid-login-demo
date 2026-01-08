# 柱の会 OpenID Login Demo - Flow Documentation

## 1. 測試選單 UI/UX
- **改為 Slide 切換**
  - 使用 有/無 toggle switch 樣式（iOS風格）
  - CSS 優化，確保視覺效果清晰專業
  - 切換時立即更新 localStorage，無需額外點擊「適用」按鈕
  - 只在首頁（index.html）允許修改
  - 其他頁面顯示 read-only 狀態

## 2. 跨頁面狀態顯示
- **在所有頁面右側顯示測試選單狀態**
  - index.html: 可編輯的 toggle switches（有/無）
  - login.html, personal-info.html, openid-auth.html, success.html, mypage.html: 顯示唯讀狀態
  - 狀態從 localStorage 讀取，無法在非首頁修改
  - 顯示三個測試項目：
    1. Zaikoアカウント (有/無)
    2. ログイン状態 (有/無)
    3. 柱の会付費資格 (有/無)

## 3. 登入狀態 ON - 快速通道流程
**條件**：`localStorage['test_isLoggedIn'] === true`

```
首頁 (index.html)
    ↓ 點擊「柱の会 ログイン」
    ↓ (不進入 login.html)
    ↓
OpenID 認證頁 (openid-auth.html)
    ↓ 點擊「確認」
    ↓
成功頁 (success.html) 或 個人情報頁 (personal-info.html)
    ↓ 取決於已付費狀態
```

## 4. 登入狀態 OFF - 正常登入流程
**條件**：`localStorage['test_isLoggedIn'] === false`

```
首頁 (index.html)
    ↓ 點擊「柱の会 ログイン」
    ↓
登入頁 (login.html)
    ↓ 輸入認證資訊並點擊「ログイン」
    ↓ 檢查 test_hasAccount flag
    ├─ hasAccount = false → 顯示錯誤訊息「あなたはアカウントがありません」
    │
    └─ hasAccount = true → 進行 OpenID 認證
        ↓
        OpenID 認證頁 (openid-auth.html)
            ↓ 點擊「確認」
            ↓
            成功頁 (success.html) 或 個人情報頁 (personal-info.html)
```

## 5. 付費狀態路由邏輯
**條件**：在 success.html 或 openid-auth.html 檢查 `localStorage['test_hasPaid']`

```
OpenID 認證完成
    ↓ 檢查 test_hasPaid flag
    ├─ hasPaid = true → 直接跳轉至個人情報頁 (personal-info.html)
    │
    └─ hasPaid = false → 進入成功頁 (success.html)
        ↓ 顯示「使用信用卡付費」按鈕
        ↓ 點擊後設定 localStorage['test_hasPaid'] = true
        ↓ 導向個人情報頁 (personal-info.html)
```

## 技術實現細節

### localStorage Keys
- `test_hasAccount` (boolean) - Zaikoアカウント有無
- `test_isLoggedIn` (boolean) - ログイン状態
- `test_hasPaid` (boolean) - 柱の会付費資格

### 頁面導航規則

| 頁面 | 進入點 | 邏輯判斷 | 流向 |
|------|--------|---------|------|
| index.html | 首頁 | test_isLoggedIn | ON → openid-auth.html / OFF → login.html |
| login.html | 來自 index (isLoggedIn=OFF) | test_hasAccount | ON → openid-auth.html / OFF → 顯示錯誤 |
| openid-auth.html | 來自 login 或 index | test_hasPaid | ON → personal-info.html / OFF → success.html |
| success.html | 來自 openid-auth (hasPaid=OFF) | 使用者操作 | 付費按鈕 → personal-info.html |
| personal-info.html | 來自 openid-auth 或 success | - | 次へ → 完成流程 |
| mypage.html | 會員專用頁面 | - | logout → index.html |

### 測試選單顯示方式

#### index.html （可編輯）
```
🧪 テスト
┌─────────────────────┐
│ Zaikoアカウント：  [✓有] [無]  │
│ ログイン状態：     [✓有] [無]  │
│ 柱の会付費資格：   [✓有] [無]  │
└─────────────────────┘
```
*即時更新 localStorage，無需按鈕*

#### 其他頁面 （唯讀）
```
🧪 テスト（表示のみ）
┌─────────────────────┐
│ Zaikoアカウント：有   │
│ ログイン状態：    無   │
│ 柱の会付費資格：有    │
└─────────────────────┘
```
*灰色顯示，無法互動*

## 優先實施順序
1. ✅ 測試選單 UI 改為 toggle switch（slide 效果）
2. ✅ CSS 優化調整
3. ⏳ 在所有頁面加入 read-only 測試選單顯示
4. ⏳ 實現 isLoggedIn 快速通道邏輯
5. ⏳ 實現 hasPaid 路由邏輯
6. ⏳ 完整測試所有流程組合

## 測試矩陣
```
test_isLoggedIn | test_hasAccount | test_hasPaid | 預期流程
    ON          |      ON/OFF     |   ON/OFF     | 首頁 → openid-auth → (personal-info or success)
    OFF         |      ON         |   ON         | 首頁 → login → openid-auth → personal-info
    OFF         |      ON         |   OFF        | 首頁 → login → openid-auth → success → personal-info
    OFF         |      OFF        |   ON/OFF     | 首頁 → login → 錯誤訊息 (無法繼續)
```
