# 国产算力动态周报 — 项目说明

## 项目概览

纯静态网站，按周归档国产算力产业动态（华为昇腾、寒武纪、海光信息等）。无后端，无框架，部署在 Vercel。

仓库：`https://github.com/Winsaney/news.git`

## 文件结构

```
index.html          — 主页面（侧边栏导航 + iframe 嵌入报告）
reports.json        — 报告列表数据
reports/
  2026-W16.html     — 每期周报（自包含单文件 HTML）
  2026-W17.html
  2026-W18.html
vercel.json         — Vercel 配置（reports.json 缓存 60s + must-revalidate）
package.json        — 唯一依赖：@vercel/analytics
```

## index.html 架构

- **左侧固定侧边栏**：品牌区 + 历史报告列表（按年份降序、周数降序排列）
- **右侧主区域**：当前报告标题/日期 + iframe 嵌入对应 HTML
- **数据加载**：`fetch('reports.json')` → 失败时回退到 `<script id="fallbackReports">` 内嵌数据
- **URL hash 路由**：`#2026-W18` 直接定位某期，优先级：hash > latest > 第一条
- **响应式**：900px 以下侧边栏变抽屉式，560px 以下隐藏外链按钮

## reports.json 字段

| 字段 | 说明 | 示例 |
|------|------|------|
| id | 唯一标识 | `"2026-W18"` |
| year | 年份 | `2026` |
| week | 周标签 | `"W18"` |
| dateRange | 日期范围 | `"2026年5月1日 - 5月7日"` |
| title | 标题 | `"国产算力动态周报"` |
| summary | 一句话摘要 | `"业绩兑现期到来：..."` |
| href | 报告路径 | `"reports/2026-W18.html"` |
| tags | 标签数组 | `["兑现期", "业绩", "IPO"]` |
| latest | 是否最新 | `true` / `false` |

## 周报 HTML 特点

每期是自包含的单文件 HTML，特点：
- 幻灯片式布局（slide），支持左右翻页、移动端滚动
- 字体：Playfair Display、Source Serif 4、JetBrains Mono
- 暖色调设计（cream `#F6F1EA`、paper `#FAFAF7`、accent `#C4653A`）
- 支持导出 PNG（html2canvas）
- 10 页左右：封面 → 公司数据 → 行业趋势 → 总结

## 每周新增期数流程

1. 将新 HTML 文件放入 `reports/` 目录（命名格式 `2026-WXX.html`）
2. 在 `reports.json` 中新增条目，将上一期的 `"latest": true` 改为 `false`
3. 在 `index.html` 的 `<script id="fallbackReports">` 中同步更新内嵌数据
4. 提交并推送到 main 分支，Vercel 自动部署

**注意事项**：
- JSON 字符串中禁止使用中文引号 `""`，用 `「」` 替代，否则 JSON 解析失败
- 三处数据必须保持一致：reports.json、fallbackReports、reports/ 目录下的实际文件
