# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 项目概述

吴嘉好的个人作品集网站 — 单页静态 HTML 站点，展示 AI 编程项目、自媒体创作、校园大使经历，并带有 Supabase 驱动的留言板功能。

## 开发命令

没有构建系统、包管理器或测试套件。直接通过浏览器打开 `index.html` 即可预览。

```bash
# 启动本地 Supabase 环境（用于留言板后端）
.\supabase.exe start

# 停止 Supabase
.\supabase.exe stop

# 查看 Supabase 状态
.\supabase.exe status

# 推送数据库迁移到远程
.\supabase.exe db push

# 打开 Supabase Studio（本地仪表板）
.\supabase.exe studio
```

本地 Supabase 启动后，API 运行在 `http://127.0.0.1:54321`。如需将留言板指向本地后端，需修改 `index.html` 中第 1051-1052 行的 `SUPABASE_URL` 和 `SUPABASE_KEY` 常量。

## 架构

### 前端

整个网站在单个 `index.html` 文件中，包含内联 `<style>` 和 `<script>`。包含三个主要内容板块，以及三个全屏沉浸式遮罩层：

- **首页** — 个人简介、教育背景、横向滚动功能卡片入口
- **AI 实战列表** — 悬停图片预览 + 点击灯箱查看大图
- **校园履历 & 奖项** — 静态信息卡片
- **留言板** — 通过 Supabase REST API 进行 CRUD 操作

**沉浸式遮罩层（`openLab()` / `closeLab()`）：**

- `#aiLabOverlay` — 代码雨 Canvas 动画 + 项目可滚动展示
- `#mediaLabOverlay` — 模糊文字 Canvas 效果（`FuzzyText` 类）+ 自媒体账号卡片
- `#pptLabOverlay` — 可变字体压力效果（`TextPressure`）+ 堆叠相册（倾斜/抛出交互）+ 云雾 Canvas 动画（`CloudMist` 类）

**关键 JS 对象：**
- `FuzzyText`（第 821 行）— 基于 Canvas 的故障/模糊文字渲染
- `pressureState` + `animatePressure()`（第 882 行）— `Compressa VF` 可变字体的鼠标驱动字体变形
- `CloudMist`（第 930 行）— 平移径向渐变粒子的环境背景
- `hScroll`（第 1013 行）— 基于滚动位置驱动的横向滚动
- 叠纸相册逻辑 `rotateAlbumCards()` / `handleCardClick()`（第 758 行）

### 后端（Supabase）

留言板使用 Supabase REST API 配合公开发布的 anon key（无需 SDK）。数据库详细信息：

- **项目 ID：** `tdwaeekscqntyaehjhoq`
- **表：** `guestbook`（列：`id`、`name`、`message`、`created_at`）
- **迁移文件：** `supabase/migrations/20260515094755_create_guestbook_table.sql`
- **RLS 策略：** 公开 SELECT 和 INSERT，无 DELETE 策略（但前端通过 REST 实现了基于昵称的删除）

Supabase 配置位于 `supabase/config.toml`，使用本地开发的默认端口（API：54321，数据库：54322，Studio：54323）。

### 凭证

`.env` 包含 Supabase URL 和公开发布的 anon key — 两者均为公开值，用于客户端 API 调用。这些不是机密信息，属于 Supabase 的标准公开配置。
