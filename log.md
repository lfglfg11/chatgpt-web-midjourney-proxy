# 开发变更记录

- 2025-12-26
  - 启用 pnpm workspace：新增 pnpm-workspace.yaml，使根目录一次 pnpm install 覆盖 service。
  - 根目录新增脚本：dev:all / dev:service / build:service / build:all，并调整 start.cmd 调用 dev:all。
  - 修复前端依赖解析报错：调整 @openai/realtime-api-beta 与 vue-waterfall-plugin-next 的导入路径/类型导入。
  - 后端启动健壮性：dotenv 指定读取 service/.env；对 *_SERVER 空字符串做防护，避免 express-http-proxy Host empty。
  - UI 精简：隐藏绘图页 MidJourney、IdeoGram 标签；隐藏侧边栏“舞蹈”入口（注释保留）。

- 2025-12-27
  - 修复 DALL 绘图尺寸：切换模型不再强制复位 size=1:1；发送 /v1/images/generations 与 /v1/images/edits 前统一规范化 size（支持把 16:9 等比例映射为 2560x1440），确保 banana/jimeng 传参一致。
  - 增加尺寸调试日志：后端终端打印 /v1/images/generations 的 model/size 入参；复用 OPENAI_API_DISABLE_DEBUG=true 关闭调试输出。
  - UI 精简：注释隐藏 PC 侧边栏“音乐/视频”入口；注释隐藏手机端底部“音乐”入口。

- 2025-12-28
  - 模型选择分组：服务端 /v1/models 拉取的模型列表新增“即梦绘图/谷歌绘图/视频生成”三组，将 jimeng-*/nano-banana*/jimeng-video* 从 Other 分离归类。
  - 默认模型：将首页/默认路由的 model 从 gpt-3.5-turbo 调整为 jimeng-4.1。
