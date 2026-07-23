# Country Knowledge Poster — WorkBuddy Skill

将 humancehr.com 的出海HR/政策文章一键转为社交媒体知识海报。

## 能力

给定一篇出海HR政策文章链接或主题，自动生成：

- **可编辑海报 HTML**（420px 宽，手机优化）
- **高清截图 JPG**（2x 渲染，可用于微信群/飞书分享）
- **分享文案**（带行动项的知识卡片文案）

## 快速开始

### 前置条件

1. WorkBuddy（腾讯代码助手）已安装
2. Playwright 已安装（用于截图渲染）：
   ```bash
   cd ~/.workbuddy/binaries/node/workspace
   /Users/yoyo/.workbuddy/binaries/node/versions/22.22.2/bin/npm install playwright
   npx playwright install chromium
   ```

### 安装

```bash
# 克隆仓库
git clone https://github.com/YOUR_USERNAME/country-knowledge-poster-skill.git

# 复制到 WorkBuddy 技能目录
cp -r country-knowledge-poster-skill ~/.workbuddy/skills/country-knowledge-poster
```

或在 WorkBuddy 中直接引用 `Skill` 工具加载 `country-knowledge-poster`。

### 使用

```
做一个知识海报，文章：https://humancehr.com/xxx
```

或：

```
帮我做一张知识卡片，主题：泰国2025年最低工资调整
```

## 工作流

1. **重复链接检查** — 自动检查当月是否已用同一篇文章
2. **文章获取** — WebFetch 提取全文内容
3. **文案生成** — 输出分享文案（摘要 + 行动项）
4. **配色匹配** — 按地区匹配配色方案（欧洲深蓝/东南亚绿/拉美绿/中东金/北美靛蓝）
5. **海报渲染** — 基于模板生成 HTML + Playwright 截图

## 项目结构

```
country-knowledge-poster-skill/
├── SKILL.md                      # 技能核心指令（WorkBuddy 驱动文件）
├── README.md                     # 本文件
├── assets/
│   ├── poster_template.html      # 海报 HTML 模板
│   ├── logo.png                  # 品牌 Logo
│   └── qr_code.jpg               # 公众号二维码
└── scripts/
    └── screenshot_poster.js      # Playwright 截图脚本
```

## 配色方案

| 地区 | 主色 | 渐变 |
|------|------|------|
| 欧洲 | 海军蓝 | #1E3A5F → #2B5797 |
| 东南亚 | 绿 | #006633 → #00994D |
| 拉美 | 绿黄 | #009c3b → #00c853 |
| 中东 | 沙漠金 | #8B4513 → #D2691E |
| 北美 | 深靛蓝 | #1a237e → #283593 |

## License

MIT
