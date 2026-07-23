---
name: country-knowledge-poster
description: >
  Given a humancehr.com article URL or topic, fetch the article content,
  generate an editable poster (.html) + screenshot (.jpg) + share copy text.
  Triggered when user asks to create/share a poster/knowledge-card/image
  from an HR/policy article.
agent_created: true
disable: false
---

# Country Knowledge Poster Skill

Generate a social-media-ready knowledge card poster for HR/policy country articles.
Outputs an editable `.html` file, a rendered `.jpg` image, and a share copy text.
Suitable for WeChat groups / Feishu / social sharing.

## When to use

- User says "做一张海报/知识卡片" or provides a humancehr.com article URL.
- User provides a text topic or summary and wants a poster + share copy.
- User wants to turn a policy update / country guide into shareable content.

## Workflow

### Step 0: 重复链接检查（必须先做）

Before generating any poster, check if the article URL has been used this month:

1. Grep the project directory for the article URL slug: `Grep` for the slug in `*.md` and poster files
2. 搜索当月所有慧思出海早报（`慧思出海早报_YYYYMM*.md`）和海报目录（`poster/*`）
3. **如果发现重复**：告诉用户具体在哪天哪个渠道用过，先确认再继续
4. **如果未发现重复**：正常进入 Step 1

> ⚠️ 不得跳过此步骤。同一个月不应重复推送同一篇文章。

### Step 1: 获取文章内容

If the user provides a URL:
1. Use `WebFetch` to fetch the full article content from the URL
2. 提取标题、正文、关键数据（日期/百分比/金额/政策变化点）
3. 判断文章所属地区（欧洲/东南亚/拉美/中东/北美/全球）

If the user provides only a topic/title (no URL):
1. 直接根据用户提供的信息判断地区
2. 根据常识填充数据结构

### Step 2: 生成分享文案

基于已获取的内容，生成分享文案，格式如下：

```
【知识卡片】🇽🇽 [标题关键词]

[2-3句文章核心内容摘要]

📌 企业需关注：
• [行动项1]
• [行动项2]
• [行动项3]

👉 查看全文：[URL]
```

- 如果有URL，放在最后；如果用户只给了主题没有URL，则省略"查看全文"
- 行动项来源于文章中的合规建议/风险提示，每条≤25字
- 摘要控制在80字以内

### Step 3: 确定配色方案

Use these header gradients based on region (overrides the template's default):

| Region | Gradient | CSS vars |
|--------|----------|----------|
| Europe | `linear-gradient(135deg, #1E3A5F, #2B5797)` (navy blue) | `--header-start: #1E3A5F; --header-end: #2B5797; --header-dark: #142946; --accent: #2B5797; --accent-light: #E8F0FA; --accent-border: #BBDEFB; --accent-text: #1565C0` |
| Southeast Asia | `linear-gradient(135deg, #006633, #00994D)` (green) | `--header-start: #006633; --header-end: #00994D; --header-dark: #004D26; --accent: #00994D; --accent-light: #E8F5E9; --accent-border: #A5D6A7; --accent-text: #2E7D32` |
| Latin America | `linear-gradient(135deg, #009c3b, #00c853)` (green-yellow) | `--header-start: #009c3b; --header-end: #00c853; --header-dark: #006B28; --accent: #00c853; --accent-light: #E8F5E9; --accent-border: #A5D6A7; --accent-text: #2E7D32` |
| Middle East | `linear-gradient(135deg, #8B4513, #D2691E)` (desert gold) | `--header-start: #8B4513; --header-end: #D2691E; --header-dark: #5C2E0D; --accent: #D2691E; --accent-light: #FFF3E0; --accent-border: #FFCC80; --accent-text: #E65100` |
| North America | `linear-gradient(135deg, #1a237e, #283593)` (deep indigo) | `--header-start: #1a237e; --header-end: #283593; --header-dark: #0F1552; --accent: #283593; --accent-light: #E8EAF6; --accent-border: #C5CAE9; --accent-text: #283593` |
| Default/General | `linear-gradient(135deg, #1E3A5F, #2B5797)` (navy blue) | Same as Europe |

### Step 4: 填充海报内容

Read the user-provided article text or fetched content. Extract:

- **标题标签** (`.title-tag`): `🇽🇽 地区+类别` (e.g., "🇸🇬 新加坡用工政策")
- **主标题** (`.title-main`): 2行，第1行重磅信息，第2行关键词
- **高亮数据条** (`.highlight-bar`): 5项关键数据（金额/日期/百分比/政策要点），每项含数字+标签
- **Q&A问题** (`.qa-head .q-text`): 从企业决策者视角出发的痛点问题，自然直接
- **三大要点** (`.step-card` ×3): 纵向3行，左蓝色圆形编号，每行含标题+描述（≤35字）
- **核心数据** (`.fact-grid`): 2×2网格，含fact-title + fact-desc
- **底部双栏** (`.bottom-row`): 左绿（合规底线）+ 右橙（合规风险），描述≤40字
- **行动清单** (`.risk-tip`): 5项，用①②③④⑤编号

### Step 5: 生成海报HTML

1. Copy assets to poster directory:
```bash
cp /Users/yoyo/.workbuddy/skills/country-knowledge-poster/assets/logo.png ./poster/
cp /Users/yoyo/.workbuddy/skills/country-knowledge-poster/assets/qr_code.jpg ./poster/
```

2. Read the template at `assets/poster_template.html` (already have it in context)

3. Generate a complete HTML using the template structure with **all matching CSS variables replaced** for the chosen region's color scheme

4. Save to `poster/{topic_keyword}.html`

### Step 6: 截图渲染

```bash
cd poster && \
NODE_PATH=/Users/yoyo/.workbuddy/binaries/node/workspace/node_modules \
/Users/yoyo/.workbuddy/binaries/node/versions/22.22.2/bin/node \
/Users/yoyo/.workbuddy/skills/country-knowledge-poster/scripts/screenshot_poster.js \
{topic_keyword}.html {topic_keyword}.jpg
```

The script uses 2× viewport scaling with JPEG quality 98.

### Step 7: 提交结果

Call `present_files` with both the `.html` and `.jpg` files.

## Layout Rules（排版标准）

所有海报必须遵守以下排版规则，确保视觉统一和谐。这些规则来自多轮迭代验证，不得随意更改。

### 基础尺寸
- **宽度固定 420px**（手机阅读优化），**不设固定高度**，内容自然撑开
- 布局使用 `display: flex; flex-direction: column` 纵向堆叠
- 所有区块需 `flex-shrink: 0` 防止被挤压

### 间距规则（必须遵守）
- **所有模块底部间距统一 10px**，包括：`.qa-head`, `.section-title`, `.steps-col`, `.fact-grid`, `.bottom-row`
- **风险提示到底签间距 14px**（`.risk-tip` 的 `margin-bottom: 14px`）
- `.content` 内边距：`padding: 14px 18px 0`（上14px 左右18px 下0）
- `.header` 内边距：`padding: 14px 18px 12px`
- `.footer` 内边距：`padding: 12px 18px`
- 步骤卡间距：`gap: 6px`
- 数据网格间距：`gap: 6px`
- 底部双栏间距：`gap: 8px`

### 字号体系
| 元素 | 字号 | 字重 |
|------|------|------|
| 主标题 `.title-main` | 24px | 800 |
| Q&A 文本 `.q-text` | 13px | 600 |
| 段落标题 `.section-title` | 12px | 700 |
| 步骤标题 `.step-card-title` | 12px | 700 |
| 步骤描述 `.step-card-desc` | 10px | 400 |
| 数据标题 `.fact-title` | 11px | 700 |
| 数据描述 `.fact-desc` | 9px | 400 |
| 标签 `.highlight-num` | 14px | 800 |
| 标签说明 `.highlight-label` | 8px | 400 |
| 风险提示 `.risk-tip` | 9px | 400 |
| 底部品牌名 `.footer-name` | 13px | 700 |
| CTA `.cta-main` | 11px | 700 |

### 高亮数据条（5项等高对齐）
- `.highlight-item` 使用 `flex-direction: column; align-items: center; justify-content: center` 实现垂直居中
- 设置 `min-height: 44px` 确保5项高度一致
- `.highlight-label` 设置 `min-height: 21px; display: flex; align-items: center`，短文本和长文本底部对齐
- 每项之间用 `border-right: 1px solid rgba(255,255,255,0.1)` 分隔，最后一项无右边框

### 步骤卡（纵向3行）
- 使用 `.steps-col` 纵向排列，不用横向排列
- 卡片样式：`display: flex; gap: 8px; background: #F9FAFB; border-radius: 8px; padding: 7px 10px; border-left: 3px solid var(--accent)`（左边界强调）
- 步骤编号：`20px` 蓝色圆形，白色数字

### 数据网格（2×2）
- 使用 `.fact-grid` 配合 `flex-wrap: wrap`
- 每项宽度 `calc(50% - 3px)`，正好2列
- 背景 `#F0F7FF`，边框 `1px solid var(--accent-border)`

### 底部双栏
- 等宽 `flex: 1`
- 左栏绿色系（`#E8F5E9` 背景 + `#A5D6A7` 边框） — 表示合规底线/正面信息
- 右栏橙色系（`#FFF8F0` 背景 + `#FFE0B2` 边框） — 表示合规风险/警示信息

### 颜色体系
| 变量 | 用途 | 色值 |
|------|------|------|
| `--header-start` | 头部渐变起点 | `#1E3A5F` |
| `--header-end` | 头部渐变终点 | `#2B5797` |
| `--header-dark` | 数据条背景 | `#142946` |
| `--accent` | 主题强调色 | `#2B5797` |
| `--accent-light` | 浅色强调背景 | `#E8F0FA` |
| `--accent-border` | 浅色边框 | `#BBDEFB` |
| `--accent-text` | 强调文字色 | `#1565C0` |
| `--gold` | 金色数据条数字 | `#FFD700` |
| 底部背景 | Footer 背景 | `#1a1a2e` |

### 内容填充规范
- 步骤卡描述：每行控制在 **35字以内**，简洁完整
- 数据条标签：控制在 **10字以内**，短词优先
- Q&A 问题：从**企业决策者视角**提出，自然直接
- 风险提示清单：**5项为上限**，用①②③④⑤编号
- 底部双栏描述：控制在 **40字以内**

### 输出格式
- HTML 文件：可编辑，保存到 `poster/{topic}_{theme}.html`
- JPG 文件：通过 Playwright 截图 2x 渲染，截图脚本在 `scripts/screenshot_poster.js`，先复制 logo.png 和 qr_code.jpg 到同一输出目录

### Bundled resources

- `assets/poster_template.html` — Base poster HTML template with all layout sections
- `assets/logo.png` — Humance brand logo
- `assets/qr_code.jpg` — QR code for WeChat public account
- `scripts/screenshot_poster.js` — Playwright screenshot script (update filenames before running)
