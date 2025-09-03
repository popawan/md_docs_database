協作SOP（貼在專案README頂部也可）
Phase 1｜想好再做（只討論，不動工）

目標：把需求與邊界講清楚。
我產出：一頁「決策草案」供你審：

功能範圍（必做/可延後）

OOP-KB 結構與命名空間（避免污染）

多語策略（default / supported / fallback）

介面與指令的 i18n Key 清單

風險與待決策清單
你提供（可口述、可先粗略）：

目的／使用情境（1–3 句）

最小功能（先會什麼、不會什麼）

KB 檔型（JSON/YAML/MD/PY/API/MCP）與可讀範圍

語系清單（例如 zh-TW、en-US）

粗略流程圖（mmd 或 SVG，先草稿也可）

沒有你「確認」我不會開工。

Phase 2｜邊想邊做（可迭代的小步快跑）

目標：用最小骨架驗證架構。
我產出：

模組骨架（如 ChatModule/PhotoProModule）

Router 串接（只掛你允許的 KB 命名空間）

i18n 檔案雛形（ICU key，不寫死字串）

Demo 指令（/lang、/remember…）
你做：跑一次 Demo，回饋「哪裡要改」。

每一輪改動都只動「骨架與字庫」，避免一次做太多。

Phase 3｜做完了再想（驗收與找洞）

目標：驗收 + 清單化缺口。
我產出：

驗收清單（功能、i18n、隔離、錯誤處理）

問題／改良提案（含取捨）
你做：逐條打勾或留言（這時才「想哪裡沒做好」也OK，因為架構已定、改動成本低）。

Phase 4｜想完了就做（改寫與二次驗收）

目標：針對清單完成修補、穩定版本。
我產出：

修正版程式與 KB

最終 README（含 OOP-KB 說明、指令、i18n、檔案結構）

Mermaid v11 原檔（.mmd）＋SVG（做視覺真相）
你做：最後驗收；之後版本再開新循環。

共通原則（全程適用）

停止/恢復：你說「停止」/「/stop」→ 我全停；「繼續」/「/go」→ 才動。

不搶做：未經你 Phase 1「確認」，我不寫任何程式。

OOP-KB：命令模組 → 路由 → KB 命名空間 → 子程式/字庫/提示詞 → 路由編排；嚴格隔離。

多語：預設 default=zh-TW、fallback=en-US、supported=[zh-TW,en-US,ja-JP,ko-KR]（可再改）。

SVG 對齊：你給 mmd+SVG，我以 SVG 為唯一視覺真相。

Phase 1 小表單（你回我這幾行就夠）

目的／情境（1–3 句）：

最小功能（必做）：

延後功能（可下一版）：

KB 命名空間可讀（例：kb/common、kb/pro）：

檔型（勾選）：JSON / YAML / MD / PY / API JSON / MCP JSON

語系（default / supported / fallback）：

流程圖（若有）：mmd 文字或 SVG
