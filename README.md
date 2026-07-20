# 六科目课题进度看板

这是一个无构建步骤的静态网站，用于维护六个科目的课题进度和共享排期。

## 页面

- `index.html`：课题进度主看板，展示各科目课题组阶段、KPI 统计和「亮点汇总」并行进度。
- `schedule.html`：共享排期日历，支持新增、编辑和删除日程。

## 本地预览

可以直接打开 HTML 文件，也可以用本地静态服务器预览：

```sh
python3 -m http.server 8000
```

然后访问：

- `http://localhost:8000/index.html`
- `http://localhost:8000/schedule.html`

## 外部依赖

页面运行时依赖以下外部服务：

- Supabase：保存和实时同步看板、排期数据。
- Tabler Icons CDN：页面图标字体。
- esm.sh：浏览器端加载 `@supabase/supabase-js@2`。

如果页面无法加载数据，优先检查浏览器控制台、Supabase 项目状态、网络访问和表权限。

## Supabase 数据依赖

前端公开配置位于两个页面的脚本区域：

- `SUPABASE_URL`
- `SUPABASE_KEY`

当前页面会访问：

- `groups` 表：主看板课题组数据。
- `events` 表：排期日历数据。
- `update_group_progress` RPC：更新课题组主进度。
- `update_achievement_review` RPC：更新亮点汇总进度。
- `groups-changes` realtime channel：同步主看板变更。
- `events-changes` realtime channel：同步排期变更。

维护 Supabase 时，请确认：

- `groups` 和 `events` 已开启符合预期的 Row Level Security。
- 前端公开 key 只能执行必要的读取和更新操作。
- RPC 函数的权限、参数名和前端调用保持一致。
- Realtime 已对相关表开启。

## 常见维护点

### 修改主看板科目或顺序

在 `index.html` 中维护：

- `SUBJECT_ICON`
- `SUBJECTS`
- `GROUP_LABEL_OVERRIDES`
- Supabase `groups.sort_order`

### 修改进度阶段文案

在 `index.html` 中维护：

- `STAGE_LABEL`
- `badgeText`
- 页面图例文案

如需新增阶段，需要同步调整 `trackHTML`、KPI 统计、编辑表单和数据库字段约束。

### 修改排期行为

在 `schedule.html` 中维护：

- `renderCalendar`
- `showAddInput`
- `showEditInput`
- 删除事件处理逻辑

### 修改顶部链接

当前 `index.html` 顶部有三个入口：

- 建设目录与可研需求明细库
- 亮点成果
- 排期

## 发布

本仓库没有构建步骤。若通过 GitHub Pages 或其他静态托管发布，直接发布仓库根目录即可。

发布前建议检查：

- `index.html` 能成功加载数据。
- KPI 卡片点击后能展开明细。
- 主进度保存后其他浏览器窗口能实时同步。
- `schedule.html` 能新增、编辑、删除日程。
- 排期变更能实时同步到其他浏览器窗口。

## 维护注意事项

- 不要把 Supabase service role key、数据库密码或其他私密凭据提交到仓库。
- 前端里的 publishable key 是公开信息，真正的访问控制必须放在 Supabase RLS 和 RPC 权限里。
- 如果多人同时编辑同一条记录，当前实现以 Supabase 最新推送的数据为准。
- 此项目没有自动测试，较大改动后请至少用两个浏览器窗口做一次实时同步验证。
