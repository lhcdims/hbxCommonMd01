A. uni-app x 技術知識（vapor 模式策略對齊 — 2026-09-03 user 拍板）



A1 核心決策邏輯
（補充資料）
- 為什麼選 vapor 模式 = 不只是「最快」，是「方向正確 + 生態最大 + DX 最好」
- 5 個決策維度（依重要性排序）：
  1. 戰略對齊：DCloud 未來主推 vapor，不主推原生（A4 T1：admin 公開場合說過）
  2. 生態規模：JS 生態 ≫ UTS 生態（npm + Vue 3 生態直接用）
  3. 開發效率：純 JS 比 UTS 學習成本低（招人 / 維護都容易）
  4. 維護成本：原生模式會變 legacy，plugin / 文件 / 範例長線跟進慢
  5. 性能：跨平台框架中最強，接近原生（T1 + T0 多源複證）
- 不選 vapor 的後果：跟 DCloud 主線脫鉤、新人學習成本高、長線維護成本上升
- 對 專案 的實用意義：
  - 是否用 vapor 模式？✅ 必須用（已用）
  - 業務代碼是否改純 JS？✅ 可以（vapor 模式支持）— 但不是必須
  - 是否關閉原生編譯備援？⚠️ 看你業務場景



A2 vapor 模式 = Vue 3 子項目（不是 DCloud 發明）
（補充資料）
- T0 一手來源：`https://blog.vuejs.org/posts/2022-year-in-review`（Evan You 2022 年終回顧）
- 核心原理：編譯期靜態分析，跳過 VDOM，直接生成 DOM/原生指令（更高效 JS / bytecode）
- 靈感來源：Solid.js
- DCloud 只是把它**整合進 uni-app x**（中文叫「蒸汽模式」），不是發明者
- ⚠️ PM 違規記錄：之前說「DCloud bytecode VM」「DCloud 自家 AOT」是純腦補（E5 段）



A3 UTS 編譯矩陣（vapor vs non-vapor）
（補充資料）
- T0 來源：`https://doc.dcloud.net.cn/uni-app-x/uts/`
- 矩陣（重點：vapor 模式所有平台頁面組件都編譯為 JS）：

| 平台 | 非 vapor 模式 | vapor 模式 |
|---|---|---|
| Web / 小程序 | JS | JS |
| Android | Kotlin（原生）| JS |
| iOS | Swift（目前大部分仍是 JS 兼容模式）| JS |
| HarmonyOS | uts 插件 → ArkTS；頁面組件 → JS | JS |

- 架構模式：DCloud 推「業務 JS + 插件 UTS」分層
- 插件層（uni_modules）必須 UTS（要調原生 API）；業務層 JS/UTS 都行（vapor 編譯為 JS）



A4 DCloud admin 5 條策略口頭訊息（T1：admin 公開場合說）
（補充資料）
- 來源類型：admin 在 DCloud 公開聊天室（QQ / Discord 等）公開說，**多人可見可證**
- 注意：T1 ≠ T0，「公開說過」≠「官方承諾 / 公告」
- 5 條訊息：
  1. vapor 模式最好用 JS 編程
  2. 未來只推 vapor，不主推原生編譯
  3. 純 JS 生態良好（npm + Vue 3 生態可直接用）
  4. 插件只能用 UTS 編寫（要調原生）；但純 JS uni-app-x 項目可調 UTS 插件
  5. vapor 運行速度比原生編譯快（在跨平台框架中）
