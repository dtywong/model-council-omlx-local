# 模型議會：用多個模型討論難以判斷嘅問題

[English](README.md)

議會將同一條問題交畀幾個 AI 模型獨立作答，再篩選同衡量各份答案入面嘅證據，整合成一個更有用嘅結果。

原作者用呢套方法處理只靠單一模型未夠把握嘅真實決策。呢個儲存庫記錄方法同提供參考實作，可以按需要調整。

## 點解用議會？

單一模型通常只畀一個相當肯定嘅答案，即使問題有幾個合理立場。遇到**冇標準答案可以核對嘅判斷題**，例如係咪應該繼續、研究方法有冇漏洞、策略上點取捨，議會可以畀你睇清真正分歧，同判斷最終結論有冇足夠理據。

**幾個模型獨立作答後得出同一個結論，本身係一項證據。** 三個階段係並行分派、選做匿名互評、主席整合。呢個做法可以保留獨立視角，並提供機會搵出有漏洞嘅答案。

呢套做法有 Mixture-of-Agents、多代理辯論等研究背景，同 Perplexity Model Council 亦有相似之處。點解有啲問題值得召集議會、席位組合有咩影響，以及同模型路由有咩分別，見[議會模式](docs/pattern_zhHK.md)。三種實作嘅比較見[操作流程同圖表](docs/workflow_zhHK.md)。

## 儲存庫結構

```text
docs/
  pattern.md             — 議會模式、使用時機同三個階段
  workflow.md            — 操作流程同流程圖
  member_registry.md     — 席位名冊範本
  scorecard_schema.md    — 評分表格式

council/
  bedrock_member.py      — AWS Bedrock 席位（Converse API）
  run_council.py         — 串連三個階段嘅協調程式
  omlx_server.py         — 本機瀏覽器介面伺服器同 oMLX 代理

web/
  index.html / styles.css / app.js / preferences.js — 瀏覽器介面同偏好設定
```

## 本機 oMLX 議會介面

呢個分支提供三個並行獨立答案，由使用者揀下一步，可以做匿名互評，之後手動請主席整合。佢用 [oMLX](https://omlx.ai/) 嘅模型清單同兼容 OpenAI 嘅 API。已安裝而且兼容嘅 Qwen、LLaMA、Mistral、Gemma、DeepSeek、MiniMax、GLM 同視覺語言模型等，都可以用模型 ID 選取。

oMLX 喺預設連接埠運行時，用以下指令。

```bash
python -m council.omlx_server
```

打開 `http://127.0.0.1:8765`。草稿、設定同近期簡報留喺瀏覽器。本機伺服器唔會儲存議會輸出。有推理欄位或 `<think>` 內容時可以展開睇。附圖會傳畀支援視覺輸入嘅席位。安裝、用法、私隱同疑難排解見[本機 oMLX 議會](docs/omlx-local-council_zhHK.md)。

### GitHub Pages 同 Windows 瀏覽器模式

GitHub Actions 可以發佈毋須額外前端套件嘅 `web/`。喺 Windows 瀏覽器打開已部署嘅網站，再撳 **oMLX** 輸入可連線嘅 oMLX 網址同選填 API 金鑰。瀏覽器會直接呼叫 oMLX 嘅 `/v1` API，同樣可以並行作答、選做匿名互評同手動請主席整合。伺服器要連得通，CORS 要容許 Pages 網站嘅來源，而 HTTPS Pages 網頁要用 HTTPS 呼叫 oMLX。見 [GitHub Pages 同 Windows 存取](docs/omlx-local-council_zhHK.md#github-pages-同-windows-存取)。

## 快速開始

### 所需環境

- **Python 3.10 或以上**
- **boto3**（`pip install boto3`）
- **AWS 憑證**設成名為 `council-bedrock` 嘅設定檔。IAM 使用者或角色需要有 `us-east-1` 嘅 Bedrock 呼叫權限。

  ```bash
  aws configure --profile council-bedrock
  ```

### 呼叫單一席位

```bash
python -m council.bedrock_member --ask "What are the trade-offs of event sourcing vs CRUD?"
python -m council.bedrock_member --ask "..." --model deepseek-v3.2 --show-thinking
```

### 執行完整議會

```bash
# 兩階段：分派同整合，略過互評
python -m council.run_council \
  --ask "Evaluate this architecture proposal: ..." \
  --seats nova-pro,nova-lite,deepseek-v3.2 \
  --no-review

# 三階段：包含匿名互評
python -m council.run_council \
  --ask "..." \
  --seats nova-pro,deepseek-v3.2,kimi-k2 \
  --reviewers nova-pro,deepseek-v3.2
```

### 當程式庫使用

協調程式接受任何符合 `(prompt: str) -> str` 介面嘅可呼叫函數做席位。可以封裝其他 API、本機模型或者測試替身。

```python
from council.run_council import run_council

def my_custom_seat(prompt: str) -> str:
    # 喺呢度呼叫你嘅模型
    return "the answer"

result = run_council(
    question="your question",
    seats=[my_custom_seat, another_seat],
    chair=strongest_seat,
    reviewers=[reviewer_a, reviewer_b],  # 兩階段流程可以省略
)

print(result.synthesis)
print(result.summary())  # 完整 Markdown 報告
```

## 隨附 Bedrock 模型

以下係 `bedrock_member.py` 支援嘅別名。供應商提供嘅版本會更新，用 `list_foundation_models` 核對模型 ID。

| 別名 | 模型 ID | 備註 |
|---|---|---|
| `nova-pro` | `amazon.nova-pro-v1:0` | 預設席位 |
| `nova-lite` | `amazon.nova-lite-v1:0` | 成本較低 |
| `nova2-lite` | `us.amazon.nova-2-lite-v1:0` | 較新嘅低成本級別 |
| `deepseek-v3.2` | `deepseek.v3.2` | 推理模型 |
| `llama4-maverick` | `us.meta.llama4-maverick-17b-instruct-v1:0` | |
| `llama4-scout` | `us.meta.llama4-scout-17b-instruct-v1:0` | |
| `devstral` | `mistral.devstral-2-123b` | 專門處理程式碼 |
| `gpt-oss-120b` | `openai.gpt-oss-120b-1:0` | 推理模型 |
| `qwen3-coder` | `qwen.qwen3-coder-next` | 專門處理程式碼 |
| `kimi-k2` | `moonshot.kimi-k2-thinking` | 推理模型 |
| `kimi-k2.5` | `moonshotai.kimi-k2.5` | 旗艦推理模型 |

## 記低表現

用[評分表格式](docs/scorecard_schema_zhHK.md)記低每次執行邊啲席位有貢獻。實際記錄應該留喺自己保管嘅副本。

## 延伸閱讀

- [議會模式](docs/pattern_zhHK.md)
- [操作流程同流程圖](docs/workflow_zhHK.md)
- [本機 oMLX 議會](docs/omlx-local-council_zhHK.md)
- [成員名冊範本](docs/member_registry_zhHK.md)
- [評分表格式](docs/scorecard_schema_zhHK.md)

## 授權

MIT，見 [LICENSE](LICENSE)。
