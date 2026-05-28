# 国产算力动态周报 — 项目说明

## 项目概览

纯静态网站，按周归档国产算力产业动态（华为昇腾、寒武纪、海光信息等）。无后端，无框架，部署在 Vercel。

仓库：`https://github.com/Winsaney/news.git`

## 文件结构

```
index.html          — 主页面（侧边栏导航 + iframe 嵌入报告）
reports.json        — 报告列表数据
reports/
  YYYY-WXX.html     — 每期周报（自包含单文件 HTML），如 2026-W21.html
vercel.json         — Vercel 配置（reports.json 缓存 60s + must-revalidate）
package.json        — 唯一依赖：@vercel/analytics
AGENTS.md           — 本文件，项目说明供 AI Agent 阅读
```

## index.html 架构

- **左侧固定侧边栏**：品牌区 + 历史报告列表（按年份降序、周数降序排列）
- **右侧主区域**：当前报告标题/日期 + iframe 嵌入对应 HTML
- **数据加载**：`fetch('reports.json')` → 失败时回退到 `<script id="fallbackReports">` 内嵌数据
- **URL hash 路由**：`#2026-W21` 直接定位某期，优先级：hash > latest > 第一条
- **响应式**：900px 以下侧边栏变抽屉式，560px 以下隐藏外链按钮

## reports.json 字段

| 字段 | 说明 | 示例 |
|------|------|------|
| id | 唯一标识 | `"2026-W21"` |
| year | 年份 | `2026` |
| week | 周标签 | `"W21"` |
| dateRange | 日期范围 | `"2026年5月22日 - 5月28日"` |
| title | 标题 | `"国产算力动态周报"` |
| summary | 一句话摘要 | `"国测认证历史性突破：..."` |
| href | 报告路径 | `"reports/2026-W21.html"` |
| tags | 标签数组 | `["国测认证", "韬定律", "昇腾950"]` |
| latest | 是否最新 | `true` / `false` |

## 周报 HTML 特点

每期是自包含的单文件 HTML，特点：
- 幻灯片式布局（slide），支持左右翻页、移动端滚动
- 字体：Playfair Display、Source Serif 4、JetBrains Mono
- 暖色调设计（cream `#F6F1EA`、paper `#FAFAF7`、accent `#C4653A`）
- 支持导出 PNG（html2canvas）
- 9–10 页：封面 → 概览 → 公司数据 → 行业趋势 → IPO 追踪 → 趋势研判 → 下周关注

## 每周新增期数流程

1. 将新 HTML 文件放入 `reports/` 目录（命名格式 `YYYY-WXX.html`）
2. 在 `reports.json` 中**最前面**新增条目，将上一期的 `"latest": true` 改为 `false`
3. 在 `index.html` 中同步以下两处：
   - `<script id="fallbackReports">` 内嵌数据：同样在最前面插入新条目，上一期 `latest` 改为 `false`
   - 顶栏 `<a id="openReport" href="...">` 的 `href` 改为新期路径
4. 提交并推送到 main 分支，Vercel 自动部署：
   ```bash
   git add reports.json index.html reports/YYYY-WXX.html
   git commit -m "feat: add YYYY-WXX weekly report"
   git push
   ```

**注意事项**：
- JSON 字符串中禁止使用中文引号 `""`，用 `「」` 替代，否则 JSON 解析失败
- 三处数据必须保持一致：`reports.json`、`fallbackReports`、`reports/` 目录下的实际文件
- 新建的 HTML 文件是 untracked 状态，**必须显式 `git add reports/YYYY-WXX.html`**，不要依赖 `git add .` 自动追踪
- 每期条目在 `reports.json` 和 `fallbackReports` 中都应保持**按周数降序**排列（最新在最前）
