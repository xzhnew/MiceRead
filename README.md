[MiceRead-README.md](https://github.com/user-attachments/files/31138026/MiceRead-README.md)
# 灵鼠释文 (MiceRead) 子～字～之～

> 一群致力于建立贯通古今理解之桥的小鼠！ (^▽^)
> *Bridges of understanding, built by mice, across the ages.*

这是一个文言文翻译器，致力于宣传还原中国古代文化。

本项目由两位开发者联合维护，目标是打造一个**人人都能轻松读懂古文**的现代化工具。

---

## 一、项目愿景（Vision）

文言文是中华文明的思想载体，但对现代读者存在天然门槛：生僻字词、省略句式、虚词用法、古今异义……我们希望用技术手段拆掉这堵墙。

**灵鼠释文**的核心理念：

- **降低门槛**：让没有古文基础的现代人，也能逐字读懂《论语》《史记》《桃花源记》等经典。
- **释文而非硬译**：先“释”后“译”——为每一个字/词提供白话释义、读音、词性、语法说明，再组装成通顺的现代汉语。
- **贯通古今**：既保留原文的韵味，又给出现代人能理解的表达。
- **开放协作**：以 Apache-2.0 开源，欢迎社区共建词库与功能。

品牌意象取自“灵巧的小鼠帮你解释古文”（灵鼠释文），亲和、有记忆点。

---

## 二、技术路线（Technical Route）

成品采用清晰的三层架构，兼顾“本地可跑、易于协作、可平滑升级”：

| 层 | 选型 | 说明 |
|---|---|---|
| 前端 | React + TypeScript + **MUI v6（Material Design 3）** | MD3 在 Web 上最成熟的官方实现 |
| 接口 | **Python FastAPI** | 异步、自带 Swagger 文档、与 SQLite 原生契合 |
| 本地数据库 | **SQLite**（`miceread.db`，单文件） | 零配置、易分发，符合“本地数据库”诉求 |
| 可选增强 | LLM 意译（DeepSeek / 通义 / 本地模型） | 第二阶段做“读着像人话”的流畅白话与赏析 |

> 前端框架为关键决策点；其余按上表默认走最稳路径。

---

## 三、系统架构（Architecture）

```
┌─────────────────────────────┐
│   前端 (React + MUI / MD3)   │  翻译页 / 词库检索页 / 品牌页
│   - App Bar、卡片、Chip、输入 │
└──────────────┬──────────────┘
               │  HTTPS / JSON
               ▼
┌─────────────────────────────┐
│   接口层 (FastAPI)           │  /api/translate  /api/word  /api/search
│   - 路由、校验、CORS          │  /api/history   /api/import
└──────────────┬──────────────┘
               │
       ┌───────┴────────┐
       ▼                ▼
┌─────────────┐   ┌────────────────┐
│ 本地 SQLite │   │ 可选 LLM 意译   │  (断网降级到本地库)
│ words /     │   │ DeepSeek 等     │
│ examples /  │   │ 结果写 cache    │
│ history /   │   │                │
│ cache       │   └────────────────┘
└─────────────┘
```

**数据流**：用户输入文言文 → 接口层调用分词与释文服务 → 查本地 SQLite 词库 → 返回「逐字标注 / 直译」；若开启 LLM，则基于逐字释文生成流畅白话与赏析（结果写入 `cache`，相同输入不再重复调用，节省开销）。

---

## 四、项目结构（Project Structure）

```
MiceRead/
├── README.md                 # 项目介绍 + 架构 + 启动步骤（本文件）
├── LICENSE                   # Apache-2.0
├── docker-compose.yml        # 一键起前后端 + 库（可选）
├── backend/
│   ├── app/
│   │   ├── main.py           # FastAPI 入口、CORS、挂载路由
│   │   ├── config.py         # 配置（DB 路径、LLM key 开关）
│   │   ├── db.py             # SQLite 引擎 + 会话
│   │   ├── models.py         # ORM 模型
│   │   ├── schemas.py        # Pydantic 出入参
│   │   ├── routers/
│   │   │   ├── translate.py  # /translate 翻译主接口
│   │   │   ├── lexicon.py    # /word 查词、/search 搜词、/word POST 录入
│   │   │   └── history.py    # 历史记录
│   │   ├── services/
│   │   │   ├── segmenter.py  # 文言文分词（最长匹配）
│   │   │   ├── translator.py # 逐字释文 + 直译组装
│   │   │   ├── llm.py        # 可选 LLM 意译客户端
│   │   │   └── seed.py       # 初始词库导入
│   │   └── data/
│   │       ├── lexicon.csv   # 启动词库（文言字词→释义）
│   │       └── examples.csv  # 例句库
│   ├── requirements.txt
│   └── tests/
├── frontend/
│   ├── package.json  vite.config.ts  index.html
│   └── src/
│       ├── main.tsx
│       ├── theme.ts          # MD3 主题 token（古籍配色）
│       ├── api/client.ts     # 调 /api 的封装
│       ├── pages/
│       │   ├── Translator.tsx # 翻译页（主功能）
│       │   ├── Lexicon.tsx    # 词库浏览/检索页
│       │   └── About.tsx      # 灵鼠释文品牌页
│       └── components/
│           ├── AppBar.tsx  ResultCard.tsx  WordChip.tsx
└── data/
    └── miceread.db           # 运行时生成（gitignore）
```

---

## 五、本地数据库 Schema（SQLite）

```sql
words(id, word TEXT, gloss TEXT, pinyin TEXT, pos TEXT,
      is_function_word INT, notes TEXT, source TEXT)
examples(id, word_id FK, classical TEXT, modern TEXT, source TEXT)
history(id, input_text TEXT, output_json TEXT, mode TEXT, created_at)
cache(id, key TEXT, response TEXT, created_at)   -- LLM 结果缓存
```

`words` 是核心：**文言字词 → 现代汉语释义 + 读音 + 词性 + 是否虚词 + 注释**。
建议先手搓 300–500 条高频字（之乎者也、常见实词、常用成语/句式），后续经 `/word` 接口持续补充。

---

## 六、API 设计（FastAPI）

| 方法 | 路径 | 作用 |
|---|---|---|
| POST | `/api/translate` | 入参 `{text, mode:"gloss"\|"literal"\|"free"}` → 逐字标注 / 直译 / 意译 |
| GET  | `/api/word/{w}` | 查单个字词：释义、读音、例句 |
| GET  | `/api/search?q=` | 词库模糊检索 |
| POST | `/api/word` | 录入/更新词条（建库用） |
| GET  | `/api/history` | 最近翻译记录 |
| POST | `/api/import` | 批量导入 CSV 词库 |

---

## 七、开发路线（Roadmap）

- **阶段 0 · 接手仓库**：基于当前空仓库，分支开发，保留 Apache-2.0。
- **阶段 1 · 数据地基（杠杆最高）**：编 `lexicon.csv` + `examples.csv`，写 `seed.py` 导入 SQLite。词库覆盖度直接决定体验。
- **阶段 2 · 后端 MVP**：FastAPI + SQLite + `segmenter` / `translator`，打通 `/translate`、`/word`。
- **阶段 3 · 前端 MVP**：React + MUI（MD3），Translator 页调接口、渲染标注结果。
- **阶段 4 · 增强**：历史记录、词库浏览页、LLM 意译 + 缓存、断网降级。
- **阶段 5 · 打磨交付**：MD3 古籍风主题、docker-compose、文档与 Demo。

> MVP 先做「逐字释文 + 直译」保证可用；「读着像人话」的意译作为第二阶段，依赖 LLM。

---

## 八、协作与许可

- 欢迎提交 Issue / PR 共建词库与功能。
- 词库数据请优先使用自采或明确开源的字典数据，注意许可证。
- 许可证：**Apache License 2.0**。

---

## 九、维护与联系

- 仓库维护者：xzhnew 与协作开发者。
- 项目讨论请使用 GitHub Issues。
