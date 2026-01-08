# 柱の会 OpenID Login Demo - 5種核心狀態完整流程

## 核心邏輯簡化
**關鍵判斷點**：
1. `test_isLoggedIn` 決定是否跳過登入頁
2. `test_hasAccount` 決定是否需要建立帳號或使用現有帳號
3. `test_hasPaid` 決定是否需要付費流程

**當 hasAccount=OFF 時**：
- 無論登入狀態如何，都要走**會員登録流程**（直接進 personal-info.html）
- 不需要登入頁或認證頁

## 五種實際狀態組合

| 狀態 | isLoggedIn | hasAccount | hasPaid | 流程分類 | 優先度 |
|------|-----------|-----------|--------|---------|--------|
| A | ON | ON | ON | 快速通道 + 已付費 | 🔴 高 |
| B | ON | ON | OFF | 快速通道 + 未付費 | 🔴 高 |
| C | OFF | ON | ON | 正常登入 + 已付費 | 🔴 高 |
| D | OFF | ON | OFF | 正常登入 + 未付費 | 🔴 高 |
| E | (任意) | OFF | (任意) | 會員登録流程 | 🟡 中 |

---

## 狀態 A: isLoggedIn=ON, hasAccount=ON, hasPaid=ON
**場景**：已登入 + 有帳號 + 已付費（最快通道）

```
首頁 (index.html)
  ↓ 點擊「柱の会 ログイン」
  ↓ (檢查 isLoggedIn = ON，直接跳過登入)
  ↓
OpenID 認證頁 (openid-auth.html)
  ↓ 點擊「確認」
  ↓ (檢查 hasPaid = ON，直接進 mypage)
  ↓
會員專用頁 (mypage.html)
  ↓ 查看會員內容、個人資訊
  ↓ 點擊「ログアウト」回首頁
```

**操作數**：2步
**流程**：index → openid-auth → mypage
**耗時**：⚡ 最短（無需登入、已付費）✅最快

---

## 狀態 B: isLoggedIn=ON, hasAccount=ON, hasPaid=OFF
**場景**：已登入 + 有帳號 + 未付費（快速通道但需付費）

```
首頁 (index.html)
  ↓ 點擊「柱の会 ログイン」
  ↓ (檢查 isLoggedIn = ON，直接跳過登入)
  ↓
OpenID 認證頁 (openid-auth.html)
  ↓ 點擊「確認」
  ↓ (檢查 hasPaid = OFF，進入成功頁)
  ↓
成功頁 (success.html)
  ↓ 顯示「使用信用卡付費」按鈕
  ↓ 點擊「使用信用卡付費」
  ↓ (設定 localStorage['test_hasPaid'] = true)
  ↓
會員專用頁 (mypage.html)
  ↓ 查看會員內容、個人資訊
  ↓ 點擊「ログアウト」回首頁
```

**操作數**：3步
**流程**：index → openid-auth → success → mypage
**耗時**：🔶 短（無需登入，但需付費）❌缺少付費

---

## 狀態 C: isLoggedIn=OFF, hasAccount=ON, hasPaid=ON
**場景**：未登入 + 有帳號 + 已付費（正常登入但已付費）

```
首頁 (index.html)
  ↓ 點擊「柱の会 ログイン」
  ↓ (檢查 isLoggedIn = OFF，進入登入頁)
  ↓
登入頁 (login.html)
  ↓ 輸入 Email/密碼
  ↓ 點擊「ログイン」
  ↓ (檢查 hasAccount = ON ✓)
  ↓
OpenID 認證頁 (openid-auth.html)
  ↓ 點擊「確認」
  ↓ (檢查 hasPaid = ON，直接進 mypage)
  ↓
會員專用頁 (mypage.html)
  ↓ 查看會員內容、個人資訊
  ↓ 點擊「ログアウト」回首頁
```

**操作數**：3步
**流程**：index → login → openid-auth → mypage
**耗時**：🔶 中等（需要登入，但已付費）✅已付費

---

## 狀態 D: isLoggedIn=OFF, hasAccount=ON, hasPaid=OFF
**場景**：未登入 + 有帳號 + 未付費（完整標準流程）

```
首頁 (index.html)
  ↓ 點擊「柱の会 ログイン」
  ↓ (檢查 isLoggedIn = OFF，進入登入頁)
  ↓
登入頁 (login.html)
  ↓ 輸入 Email/密碼
  ↓ 點擊「ログイン」
  ↓ (檢查 hasAccount = ON ✓)
  ↓
OpenID 認證頁 (openid-auth.html)
  ↓ 點擊「確認」
  ↓ (檢查 hasPaid = OFF，進入成功頁)
  ↓
成功頁 (success.html)
  ↓ 顯示「使用信用卡付費」按鈕
  ↓ 點擊「使用信用卡付費」
  ↓ (設定 localStorage['test_hasPaid'] = true)
  ↓
會員專用頁 (mypage.html)
  ↓ 查看會員內容、個人資訊
  ↓ 點擊「ログアウト」回首頁
```

**操作數**：4步
**流程**：index → login → openid-auth → success → mypage
**耗時**：⏱️ 最長（經過所有認證和付費）❌缺少付費

---

## 狀態 E: hasAccount=OFF（任意 isLoggedIn, hasPaid）
**場景**：無帳號（會員登録流程）

```
首頁 (index.html)
  ↓ 點擊「柱の会 ログイン」或「新規會員登録」
  ↓ (檢查 hasAccount = OFF，進入登入頁)
  ↓
登入頁 (login.html)
  ↓ 點擊「ログイン」
  ↓ (檢查 hasAccount = OFF ✗)
  ↓ 顯示錯誤訊息：「あなたはアカウントがありません」
  ↓ 點擊「會員登録へ」
  ↓
個人情報頁 (personal-info.html)
  ↓ 輸入電話號碼等地址資訊
  ↓ 點擊「次へ」
  ↓
OpenID 認證頁 (openid-auth.html)
  ↓ 點擊「確認」
  ↓ (檢查 hasPaid = OFF，進入成功頁進行付費)
  ↓
成功頁 (success.html)
  ↓ 顯示「使用信用卡付費」按鈕
  ↓ 點擊「使用信用卡付費」
  ↓ (設定 localStorage['test_hasPaid'] = true)
  ↓
會員專用頁 (mypage.html)
  ↓ 查看會員內容、個人資訊
  ↓ 點擊「ログアウト」回首頁
```

**操作數**：5-6步
**流程**：index → login(error) → personal-info → openid-auth → success → mypage
**耗時**：🔶 中等（完整會員註冊流程）👶新會員
**備註**：新用戶註冊流程

---

## 流程邏輯樹

```
首頁 (index.html)
  ↓ 點擊「ログイン」
  ↓
  ├─ isLoggedIn = ON?
  │  ├─ YES → 直接進 openid-auth.html (快速通道)
  │  └─ NO  → 進 login.html (正常登入)
  │
OpenID 認證頁 (openid-auth.html) 或 登入頁 (login.html)
  ↓
  ├─ hasAccount = ON?
  │  ├─ YES → openid-auth.html (已有帳號)
  │  │       ↓ 檢查 hasPaid?
  │  │       ├─ YES → mypage.html (直接進會員頁)
  │  │       └─ NO  → success.html (付費) → mypage.html
  │  │
  │  └─ NO → personal-info.html (會員登録)
  │          ↓ 填完資訊
  │          ↓ openid-auth.html
  │          ↓ 檢查 hasPaid?
  │          ├─ YES → mypage.html
  │          └─ NO  → success.html (付費) → mypage.html
```

---

## 流程分類總結

### 🔴 優先級別（必須支援）

#### 新用戶流程（狀態 D / E）
- **狀態 D**：最完整標準流程（5頁面）
- **狀態 E**：會員登録流程（快速建帳號）

#### 既有用戶流程（狀態 A / C）
- **狀態 A**：已登入已付費（3頁面，最快）
- **狀態 C**：登入後已付費（4頁面，常見）

#### 付費流程（狀態 B）
- **狀態 B**：已登入但需付費（4頁面）

---

## 詳細决策樹

| 檢查項 | 條件 | 動作 | 下一步 |
|--------|------|------|--------|
| **hasAccount** | OFF | ✓ 直接進會員登録 | personal-info.html |
| | ON | ✓ 檢查登入狀態 | ↓ |
| **isLoggedIn** | ON | ✓ 略過登入頁 | openid-auth.html |
| | OFF | ✓ 進登入頁 | login.html → openid-auth.html |
| **hasPaid** | ON | ✓ 略過成功頁 | personal-info.html |
| | OFF | ✓ 進成功頁付費 | success.html → personal-info.html |

---

## 測試優先順序

### 🔴 第一優先（核心流程）
1. **狀態 D**（OFF + ON + OFF）- 新用戶完整註冊流程
2. **狀態 C**（OFF + ON + ON）- 已付費用戶登入
3. **狀態 E**（任意 + OFF + 任意）- 會員登録

### 🟡 第二優先（加速流程）
4. **狀態 B**（ON + ON + OFF）- 快速登入加付費
5. **狀態 A**（ON + ON + ON）- 最快速通道

---

## 實裝檢查清單

### 需要修改的文件及邏輯

- [ ] **index.html**
  - [ ] 檢查 `test_hasAccount` flag
  - [ ] 若 OFF：直接導向 personal-info.html（會員登録）
  - [ ] 若 ON + isLoggedIn=ON：直接導向 openid-auth.html
  - [ ] 若 ON + isLoggedIn=OFF：導向 login.html
  - [ ] toggle switch 已實裝 ✅

- [ ] **login.html**
  - [ ] 此頁只在 hasAccount=ON 且 isLoggedIn=OFF 時出現
  - [ ] 點擊「ログイン」直接導向 openid-auth.html
  - [ ] 保留原有錯誤訊息處理（如需要）
  - [ ] 錯誤訊息已實裝 ✅

- [ ] **openid-auth.html**
  - [ ] 檢查 `test_hasPaid` flag
  - [ ] 若 ON：直接導向 personal-info.html
  - [ ] 若 OFF：導向 success.html
  - [ ] 需實裝路由邏輯 ⏳

- [ ] **success.html**
  - [ ] 付費按鈕已實裝 ✅
  - [ ] 付費按鈕設定 `test_hasPaid = true` 已實裝 ✅
  - [ ] 需加入 read-only 測試選單 ⏳

- [ ] **personal-info.html**
  - [ ] 回首頁按鈕已實裝 ✅
  - [ ] 需加入 read-only 測試選單 ⏳
  - [ ] 支持來自多個入口的流程（會員登録 / 登入 / 快速通道）

- [ ] **mypage.html**
  - [ ] 需加入 read-only 測試選單 ⏳

- [ ] **所有頁面**
  - [ ] 需加入 read-only 測試選單顯示 ⏳
  - [ ] 測試選單顯示當前 localStorage 狀態 ⏳
