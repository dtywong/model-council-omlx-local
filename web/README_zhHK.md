# 本機模型議會前端

[English](README.md)

呢個資料夾包含 oMLX 議會嘅瀏覽器介面，毋須安裝前端套件。可以經本機 Python 代理運行，亦可以部署到 GitHub Pages 等靜態網站，由瀏覽器直連可存取嘅 oMLX 伺服器。

- `index.html` 定義三席工作區、由使用者決定下一步嘅位置，以及互評同主席整合嘅控制項。
- `styles.css` 定義響應式版面同主題。
- `preferences.js` 管理英文、香港廣東話同外觀偏好。
- `app.js` 負責搵模型，同埋喺 localStorage 儲存草稿、設定同記錄。經代理會呼叫 `/api/council`、`/api/council/review` 同 `/api/council/synthesize`。設定咗伺服器網址，就會直連 oMLX 兼容 OpenAI 嘅 `/v1` 端點。

撳頂欄嘅 **oMLX** 可以設定伺服器網址同選填嘅 API 金鑰。設定會留喺目前瀏覽器嘅 localStorage。HTTPS 靜態網站需要用 HTTPS 呼叫 oMLX，伺服器亦要透過 CORS 允許 Pages 嘅來源。冇身份驗證嘅推理伺服器應該留喺受保護嘅網絡入面。安裝、Windows 或 GitHub Pages 存取方法、私隱同操作指引，見[本機 oMLX 議會](../docs/omlx-local-council_zhHK.md)。
