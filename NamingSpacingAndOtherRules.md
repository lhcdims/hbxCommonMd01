A. 本文档的隔行规矩和其他规矩



（本行为 Dummy，请不要删除）





A1 隔行规矩





A1.1 每主段落用 A/B/C 英文字母命名段落。第二层以 Ax 命名。第三层以 Ax.y 命名。第 4 层以 Ax.y.z 命名。





A1.2 A 段落和 B 段落之间，隔 5 行。





A1.3 A 和 A1 之间，隔 3 行





A1.4 A1 和 A2 之间，隔 3 行





A1.5 A2 和 A2.1 之间，隔 2 行。





A1.6 A2.1 和 A2.2 之间，隔 2 行。





A1.7 A2.2 和 A2.2.1 之间，隔 1 行





A1.8 A2.2.1 和 A2.2.2（如有）之间，隔 1 行





A1.9 各层下面如需【补充资料】，不需隔行。比方说这一行下面的（本行为 Dummy，用来说明什么情况下不需要隔行，请不要删除），就不需要隔行。但如果【补充资料】要分开几点，最好就用 A1.9.1 / A1.9.2，这种情况下，就要按规矩隔行了：

（本行为 Dummy，用来说明什么情况下不需要隔行，请不要删除）







A2 其他规矩





A2.1 请【尽量】让每一个段落，都有【命名】。这种命名方式的好处是，比方说人类想和 ai 说，请看某文档的第 5 点，但是第 5 点可能出现过 N 次。如果说 C.3.5 那么就是唯一了！





A2.2 本文档是 workspace 通用 OpenClaw（PM）记忆。P004 项目的 OpenClaw 记忆应放 `<P004项目根目录>\\openclaw\\MEMORY\_share.md`（user 2026-08-24 拍板；2026-08-26 rename 為 MEMORY\_share.md 以避免命名混淆；2026-08-28 强调：\*\*完整絕對路徑 = 辦公室 `C:\\Users\\lichiukenneth\\Documents\\HBuilderProjects\\Learn\\xLearnAiP004\\openclaw\\MEMORY\_share.md` / 老家 `D:\\uniapp\\xLearnAiP004\\openclaw\\MEMORY\_share.md` / 機房 `D:\\uniapp\\uniappx\\xLearnAiP004\\openclaw\\MEMORY\_share.md`\*\*；三路徑詳見 P004 MEMORY\_share.md D1。⚠️ \*\*`workspace\\P004\\openclaw\\` 是錯位置\*\* — 其他 OpenClaw 龍蝦看不見，只能 sync 通過 git 在項目 source tree 內；2026-08-28 PM 親犯過，已 merge 修正）





A2.3 记忆不重复，使用 reference 取代 copy/paste（2026-08-26 11:14 user 拍板；2026-08-28 補強調路徑規則）

（補充資料）

\- 背景（user 原话）：「写进 p004 的 MEMORY.md，需要在你自己的 MEMORY.md 重复吗？这样会不会浪费 token？」

\- 規則：跨層記憶不重複內容，用 reference 取代

\- 範例：`<P004项目根目录>\\openclaw\\MEMORY\_share.md` 引用 workspace MEMORY.md D3.x 時，寫「見 workspace/MEMORY.md D3.9」而非整段複製

\- ⚠️ 路徑規則：所有 P004-related reference 一律用 `<P004项目根目录>` 完整絕對路徑（per A2.2 三路徑），不准寫 `P004/openclaw/...` 模糊 reference — 因為這個路徑在 workspace 內根本看不見（其他龍蝦 sync 不到，2026-08-28 user 拍板強調）

\- 理由：❌ 重複會浪費 token（每次 session load 都付費）❌ 兩邊容易不同步 ❌ 路徑不明確會導致 PM 寫到錯位置（2026-08-28 PM 親犯過）

\- ✅ single source of truth：workspace MEMORY.md 是通用 PM 紀律的唯一出處

\- ✅ project MEMORY\_share.md 只放 project 專屬內容 + reference 到 workspace

\- 應用場景：P004、未來其他項目（每個項目的 openclaw/MEMORY.md 都應遵守）









B. 本文档介绍







B1. 本文档是这台电脑 workspace 通用 OpenClaw（PM）记忆文档，跨会话保留





B2. 内容範圍：OwnerTgHk 信息、角色定位、PM 鐵律、OpenClaw 运维、minimax 配额、本机配置 / user 偏好、調試歷史、通用 skill 引用





B3. ⚠️ P004 项目的 OpenClaw 记忆不在这里！去 `<P004项目根目录>\\openclaw\\MEMORY\_share.md` 找（per A2.2 辦公室 `C:\\Users\\lichiukenneth\\Documents\\HBuilderProjects\\Learn\\xLearnAiP004\\openclaw\\MEMORY\_share.md`）





B4. ⚠️ 更新 P004 相关长期记忆，必须改 `<P004项目根目录>\\openclaw\\MEMORY\_share.md`，不要改这里（user 2026-08-24 拍板；2026-08-28 强调：❌ 禁止创建 `workspace\\P004\\openclaw\\` mirror — 那个位置其他 OpenClaw 龍蝦 sync 不到，是 local artifact）





B5. 本文档不是给 uni-agent 看的。uni-agent 看各自项目的 AGENTS.md





B6. 从下面 C 段落开始，由龙虾主人保留；D 段开始由各 OpenClaw 自己维护













C. 龙虾主人需要提供的其他资料（如有）







（user 可加備註）













