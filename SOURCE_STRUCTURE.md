# 源码结构总览

本仓库为 PPanel Web 的单仓（monorepo），包含管理端与用户端两套 Next.js 应用，以及共享的 UI 组件与配置包。下文按目录逐层梳理每一个文件的放置位置、主要职责、可修改点及修改联动。

## 仓库根目录
- `README.md` / `README.zh-CN.md` / `CHANGELOG.md`：项目介绍、使用说明与更新记录，修改后便于向读者传递信息，但不会影响运行逻辑。
- `LICENSE`、`CODE_OF_CONDUCT.md`：许可证与贡献准则，仅文档作用。
- `package.json`：定义工作区（apps/*、packages/*）、脚本（build/dev/lint/openapi 等）与依赖，调整脚本或依赖会影响所有工作区的构建和开发流程。
- `bun.lock`：Bun 的锁文件，记录精确依赖版本；手动修改需谨慎，通常由包管理器生成。
- `turbo.json`：Turbo 任务流水线配置，决定 build/lint/dev/openapi 等任务的依赖关系和缓存规则。
- `tsconfig.json`：根 TS 配置，为工作区继承的基础设置，调整会影响所有 TypeScript 工程的编译行为。
- `eslint.config.js`：顶层 ESLint 配置（继承工作区包），影响统一的代码风格检查。
- `.vscode/`：编辑器推荐配置。
- `.github/`：CI 工作流与 Issue 模板，修改会影响自动化流程。
- `docker/`：容器化相关文件，详见后文。
- `scripts/`：开发辅助脚本（清理、依赖更新、发布等），可按需扩展。
- `apps/`：包含 `admin` 与 `user` 两个 Next.js 应用，是主要业务代码所在。
- `packages/`：共享包（UI 组件与 Lint/TS/Prettier/Commitlint 配置）。

## Docker 与脚本
- `docker/docker-compose.yml`：编排管理端与用户端镜像及运行参数，新增服务或端口需同步在此声明。
- `docker/ppanel-admin-web/Dockerfile`、`docker/ppanel-user-web/Dockerfile`：分别描述两端的构建步骤，调整 Node/Bun 版本或构建命令需与应用配置保持一致。
- `scripts/clean.sh`：清理依赖与构建产物；`prepare.sh`：安装 Husky 钩子；`update-deps.sh`、`update-shadcn-ui.sh`：批量升级依赖或 UI 组件；`publish.sh`：发布辅助脚本。修改脚本会改变团队常用的开发/发布流程。

## 公共包（packages/）
### 配置类包
- `packages/eslint-config`、`packages/prettier-config`、`packages/typescript-config`、`packages/commitlint-config`：导出统一的 ESLint/Prettier/TS/Commitlint 规则。调整这里会波及所有工作区的校验与编译行为。

### UI 组件库（packages/ui）
- `src/styles/globals.css`：Tailwind 全局样式与主题变量，可在此调整全局样式基线。
- `src/lib/utils.ts`：通用工具函数（类名合并等），被多数组件依赖。
- `src/components/`：基础 UI 组件（Button/Input/Dialog/Navigation 等），基于 Radix UI & Tailwind。修改组件将直接影响两端应用的 UI 行为。
- `src/custom-components/`：业务增强组件（ProTable/ProList、表单控件、Markdown/Editor、上传、空状态等），在应用层大量复用，改动需关注 API 兼容性。
- `src/hooks/`：`use-outside-click`、`use-lang-dir`、`use-toast`、`use-mobile` 等通用 Hook，被应用与组件共用。
- `src/utils/`：国家区号、单位换算与格式化工具函数，供业务和组件调用。
- `src/lotties/`：Lottie 动画 JSON 资源，UI 动效素材，可替换或新增。

## 应用：管理端（apps/admin）
- `package.json`、`tsconfig.json`、`next.config.ts`、`tailwind.config.ts`、`postcss.config.mjs`、`eslint.config.js`：应用自身的依赖、TS/Next/Tailwind 配置。修改需确保与共享配置兼容。
- `README.md` / `README.zh-CN.md`：管理端使用说明。
- `.env.template`：环境变量示例，新增后端接口或三方服务时需同步此文件。
- `components.json`：shadcn/ui 生成配置。
- `public/`：静态资源（favicon、PWA 图标、manifest），替换会影响浏览器展示。
- `components/`：布局与全局组件（侧栏、头部、用户菜单、主题/语言切换、ProTable 包装、空态、展示组件等），改动会影响整体 UI 框架。
- `components/dashboard/`：仪表盘卡片与对话框组件，驱动首页数据展示。
- `config/`：`constants.ts`（常量）、`navs.ts`（导航定义）、`use-global.tsx`（全局配置加载），调整会影响菜单、路由及全局设定。
- `store/`：`server.ts`、`subscribe.ts`、`stats.ts`、`node.ts` 等 Zustand 或 Zustand-like 状态存储，修改字段需同步使用处。
- `services/`：`common/` 与 `admin/` API 客户端（OpenAPI 生成），统一封装后台接口，改动类型或路径需与后端契合并更新调用方。
- `utils/`：`request.ts`（基于 axios/fetch 的请求封装）、`common.ts`（工具函数），影响所有接口调用的拦截与错误处理。
- `locales/`：多语言 JSON 文案，按语言代码分目录（如 `en-US`、`zh-CN` 等），每个目录包含 `common.json`、`auth.json`、`payment.json` 等主题文案及 `index.json` 入口。新增/调整字段需同步各语言以避免缺失。
- `locales/utils.ts`、`locales/request.ts`：处理语言与请求错误码的辅助函数。
- `app/`：Next.js App Router 入口。根 `layout.tsx` 定义全局布局；`(auth)/` 目录包含登录页、验证码发送等；`dashboard/` 下的 `layout.tsx` 与 `page.tsx` 定义仪表盘路由。修改会影响路由结构与页面渲染。
- `netlify.toml`：Netlify 部署配置。

## 应用：用户端（apps/user）
- 基础配置与公共文件结构与管理端一致（`package.json`、`tsconfig.json`、`next.config.ts` 等）。
- `components/`：页眉页脚、语言/主题切换、用户菜单、ProList、空态、加载等组件，支撑前台 UI 体验。
- `components/subscribe`、`components/main`、`components/affiliate`、`components/auth`、`components/payment`、`components/announcement`：按业务模块拆分的子组件集合，修改会直接影响对应页面模块。
- `config/`：常量、导航定义与全局配置加载，与管理端类似。
- `services/`：`common/` 与 `user/` API 客户端（订阅、订单、工单、公告、支付等），保持与后端接口的一致性。
- `utils/`：`request.ts` 请求封装、`common.ts` 常用工具、`tutorial.ts` 引导与示例相关逻辑。
- `locales/`：与管理端相同的语言与主题切分，每个语言目录提供用户端对应的文案文件；修改字段需同步所有语言。`locales/utils.ts`、`locales/request.ts` 负责语言与错误码处理。
- `app/`：根 `layout.tsx` 管理全局 Provider；`(main)/` 路由包含主页布局与页面；`auth/` 目录提供登录与验证码发送；`oauth/`、`bind/` 目录处理第三方授权绑定流程。
- `public/`、`netlify.toml` 等与管理端类似，分别服务于用户端静态资源与部署。

## 其他顶层文件
- `apps/admin/openapi2ts.config.ts` 与 `apps/user/openapi2ts.config.ts`：OpenAPI 代码生成配置，更新接口时需同步调整。
- `apps/admin/components.json`、`apps/user/components.json`：shadcn/ui 组件生成配置，调整影响 UI 代码的生成路径与别名。
- `bun.lock`、`tsconfig.json`、`package.json` 等根文件为两端共享依赖与构建的基础，修改需评估整体影响。

## 修改联动建议
- **接口调整**：更新 `services/` 下的接口定义时，需同步修改调用处的类型与字段映射，必要时更新 `locales/request.ts` 的错误码提示。
- **多语言文案**：新增文案键值时，应在所有语言目录内补充相同键，避免界面出现占位或缺失。
- **UI 组件**：对 `packages/ui` 的基础或业务组件进行变更，会影响两端应用；建议遵循语义化版本并在引用处验证兼容性。
- **全局配置/导航**：调整 `config/navs.ts` 或 `config/constants.ts` 时，注意与路由、权限和显示文案保持一致。
- **构建与脚本**：修改 `package.json` 脚本、`turbo.json`、`Dockerfile` 等，会改变 CI/CD 与本地开发体验，需在 README 中同步说明。

