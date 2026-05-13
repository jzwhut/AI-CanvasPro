# AGENTS.md

## 角色定位

你是本项目的长期工程协作者，不是一次性脚本生成器。

本项目是 Electron + AI 画布 + 多厂商模型 / 工作流平台。写代码时不能只追求当前需求跑通，必须优先让代码持续向产品化、配置化、可扩展方向演进。

## 最高优先级规则

- 不要编造文件、路径、函数、配置、接口、运行结果或测试结果。
- 修改前必须先确认仓库真实文件、调用链与可复用实现。
- 所有新增和修改的文本文件必须保持 UTF-8 编码。
- 禁止使用 GBK、ANSI、UTF-16 保存文件。
- 修改包含中文的文件前，必须先确认原文件编码；修改后不得出现中文乱码。
- 优先小步收敛修改，禁止无关重构。
- 若把握不足，不要直接改代码；先说明已知事实、不确定点、拟修改文件和风险点。
- 除非用户明确要求，否则不要改 `README.md`。
- 修改后必须运行与改动范围匹配的测试或检查；不能运行时必须说明原因和剩余风险。
- 不要声称通过了未实际运行的命令。

## AI 实际工作流程

每次接到开发、修复、重构、接入模型、调整交互或修改 Electron 能力的需求时，必须按以下顺序执行。

1. 判断任务类型：
   - 一次性修复
   - 产品能力
   - 平台能力
   - 架构收敛

2. 判断是否触发专项规则：
   - 涉及模型、工作流、生成任务、输入槽、参数面板、模型菜单、订阅 / VIP、结果展示时，先走 `registry` / `manifest` / `schema` / `adapter` / `runtime` 判断。
   - 涉及 Electron、IPC、文件、路径、下载、外链、系统能力、打包环境时，先判断落点是 `main` / `preload` / `renderer` / `IPC` / 打包配置，并查阅 `docs/electron-rules.md`。
   - 涉及模块职责、状态流、渲染、交互分层、坐标换算、网络收口时，先查阅 `docs/architecture.md`。
   - 涉及样式时，先查 `styles/` 中是否已有对应样式、变量、主题 token 和组件结构。

3. 修改前必须先搜索真实实现：
   - 搜索现有文件、函数、调用链、测试和相似实现。
   - 优先复用现有 `registry`、`manifest`、`adapter`、`runtime`、`store`、UI schema、样式 token。
   - 不允许凭记忆新增路径、模块、函数、配置或运行结果。

4. 决定修改落点：
   - 状态变更放 `store` / `appStore`。
   - 纯展示放 `renderer` / component。
   - 高频拖拽、缩放、框选等交互放 `interaction`。
   - 坐标换算放 `math`。
   - 网络请求放 `api/`。
   - Electron 系统能力放 `main`，通过 `preload` 暴露最小 IPC API。
   - 模型能力优先新增或修改 manifest，不优先写硬编码分支。

5. 修改前输出说明：
   - 本次是一次性修复、产品能力、平台能力还是架构收敛。
   - 拟改文件 / 函数 / 模块。
   - 是否新增或修改 `registry` / `manifest` / `schema`。
   - 前端是否可由 schema 自动生成。
   - 后端是否可由 adapter 自动映射。
   - 是否涉及 Electron 分层或 IPC。
   - 是否存在旧逻辑、硬编码分支或 fallback；若存在，说明处理计划。
   - 风险点和验证方式。

6. 执行修改：
   - 小步修改，避免无关重构。
   - 不改 `README.md`，除非用户明确要求。
   - 不放宽 Electron 安全配置。
   - 不新增绕过 `api/` 的网络请求。
   - 不新增散落的 `modelId` / `workflowId` / `provider` 判断。
   - 不新增非 UTF-8 文本文件。

7. 验证：
   - 运行与改动范围匹配的测试或检查。
   - 涉及中文、文档或编码时，优先运行编码检查。
   - 涉及架构边界时，运行架构检查。
   - 涉及 Electron 分层、安全、IPC、路径或打包时，运行对应单元测试或最小启动验证。

8. 最终回复：
   - 说明实际修改了什么。
   - 说明实际运行了哪些验证。
   - 说明未覆盖的风险或后续建议。

## 任务类型判断

每次写代码前，必须先判断本次需求属于哪一类。

- 一次性修复：修一个明确 bug，不扩展产品能力，不改架构边界。
- 产品能力：新增或完善用户可感知功能，但不引入新的平台抽象。
- 平台能力：新增厂商、模型、工作流、输入槽、生成任务、订阅权限、上传下载、任务生命周期或可复用执行能力。
- 架构收敛：调整模块边界、状态流、交互分层、网络收口、Electron 分层或迁移旧硬编码。

只要涉及以下任一项，默认按平台能力处理：

- 新厂商
- 新模型
- 新 RunningHub / APIMart / PPIO / GRSAI / Dreamina 工作流或模型 API
- 新模型参数
- 新输入能力：图片、视频、音频、文本，以及图片输入上的遮罩编辑扩展
- 新生成任务、轮询、取消、恢复、保存结果
- 新模型菜单、参数面板、生成按钮、结果展示
- 新订阅 / VIP / 权限判断
- 新上传、转存、下载、外链、本地保存能力

## 平台化主线

凡是涉及模型或工作流，优先使用以下链路：

`Model Registry -> Execution Manifest -> UI Schema Renderer -> Payload Normalizer -> Provider Adapter -> Task Runtime -> Result Renderer`

禁止让前端、后端、任务编排、订阅判断各写一遍同一个 `modelId` / `workflowId`。

新增普通模型 / 工作流时，主要新增或修改 manifest，而不是重新写一套前端、后端和任务编排：

- 前端通过 schema 自动显示。
- 后端通过 adapter 自动映射。
- 任务生命周期由统一 runtime 管。
- 特殊能力通过受控 extension point 插入。

## 强产品化主线规则

本项目后续以新项目、新模型、新工作流的产品化主线为优先目标，不默认兼容旧 RunningHub ID、旧模型 ID、旧 workflowId、旧硬编码分支或旧项目保存数据。

除非用户明确要求兼容历史项目，否则禁止为了旧项目添加 alias、legacy fallback、旧 ID 迁移、旧 payload 猜测或“找不到 manifest 就尝试旧逻辑”的兜底。

缺少 `model manifest` / `execution manifest` 时，应直接报错并暴露清晰原因，推动新能力补齐 registry / manifest，而不是在 adapter、UI、任务编排层继续堆特殊分支。

产品化的默认目标是：

- 新功能必须走 `registry` / `manifest` / `schema` / `adapter` / `runtime`。
- 旧模型分支、旧 workflow 分支和旧硬编码，在新链路验证后应逐步删除。
- 测试应覆盖“缺 manifest 直接失败”的行为，而不是鼓励 fallback。
- 只有当前菜单、当前新项目流程、当前 manifest 注册能力需要被维护。
- 历史项目兼容属于显式需求，不属于默认工程目标。

## 执行类型分层

RunningHub 工作流和厂商模型 API 是两类执行协议，禁止强行混成同一种 payload。

RunningHub 工作流走 `workflow manifest` + `workflow adapter`，核心是：

- `workflowId` / `appId`
- `nodeInfoList`
- `instanceType`
- query mode
- result extractor

厂商模型 API 走 `model API manifest` + `provider adapter`，核心是：

- endpoint
- method
- model
- body mapping
- headers
- response mapping

上层必须统一：

- 模型菜单
- 参数 UI schema
- 输入槽策略
- 订阅 / VIP 判断
- 任务生命周期
- 加载动画 / 按钮状态 / 错误展示
- 结果回填

下层必须按 `adapterType` 分流：

- `workflow`：RunningHub / 其他节点式工作流平台。
- `modelApi`：RunningHub Model API、APIMart、PPIO、GRSAI、Dreamina 等模型接口。
- `localRuntime`：本地处理、转码、文件任务等非远程模型能力。

`model manifest` 只负责声明“用户选择的能力”；`execution manifest` 负责声明“怎么执行”。两者通过稳定的 `executionId` 关联，禁止把 UI 菜单项直接绑定到某段硬编码请求逻辑。

## 硬编码分支规则

禁止默认在多个文件里散落硬写：

- `if (model === "xxx")`
- `if (workflowId === "xxx")`
- `if (provider === "xxx")`
- `switch(model)`
- `switch(workflowId)`

如果确实必须写特殊分支，必须满足：

- 先说明为什么不能配置化。
- 特殊逻辑只能收口到单一 `resolver` / `adapter` / `extension point`。
- 必须保留后续迁移到 `registry` / `manifest` 的入口。
- 不得复制同一判断到多个层。

## 模型注册规则

新增模型时，优先抽象为 `model manifest`，至少考虑：

- `schemaVersion`
- `modelId`
- `provider`
- `kind`: `image` / `video` / `audio` / `text`
- `adapterType`: `workflow` / `modelApi` / `localRuntime`
- `executionId`
- `displayName`
- `icon`
- `capabilities`
- `uiSchema`
- `inputSlots`
- `vip`
- `async`
- `cancellable`
- `outputType`

模型菜单、参数显示、任务提交、订阅判断、结果展示，都应优先从 `model manifest` / `registry` 获取信息。

`model manifest` 不应直接描述厂商请求细节；厂商 endpoint、RunningHub 节点映射、结果字段路径等执行细节必须放到对应的 `execution manifest`。

## 工作流配置规则

新增工作流时，优先抽象为 `workflow execution manifest`，至少考虑：

- `schemaVersion`
- `id`
- `provider`
- `kind`
- `adapterType`: `workflow`
- `label`
- `description`
- `workflowId` / `appId`
- `submitMode`
- `queryMode`
- `inputs`
- `params`
- `mapping`
- `result`
- `capabilities`
- `validation`
- `extensions`

RunningHub 工作流新增时，不要优先往 `RunningHubAdapter.js` 堆大段 `workflowId` 分支。优先扩展 workflow manifest，由通用 adapter 生成：

- `nodeInfoList`
- `workflowId` / `appId`
- `instanceType`
- query mode
- result extractor

## 厂商模型 API 配置规则

新增厂商模型 API 时，优先抽象为 `model API execution manifest`，至少考虑：

- `schemaVersion`
- `id`
- `provider`
- `kind`
- `adapterType`: `modelApi`
- `endpoint`
- `method`
- `model`
- `headers`
- `bodyMapping`
- `responseMapping`
- `result`
- `capabilities`
- `validation`
- `extensions`

厂商 API 新增时，不要优先往 `aiImageApi.js`、`aiVideoApi.js`、`aiAudioApi.js` 或具体节点组件里堆模型分支。优先扩展 model API manifest，由 provider adapter 生成请求。

如果某个厂商存在特殊鉴权、上传、轮询或结果解析，必须收口到该厂商 adapter 的单一 extension point，不得复制到 UI、任务编排和结果渲染层。

## 插件化 / 自动导入预留规则

后期模型、工作流、厂商能力要支持插件化和自动导入，因此新增 registry / manifest / schema 时必须预留：

- 稳定 `schemaVersion`，并提供向后兼容策略。
- 稳定唯一 ID：`modelId`、`executionId`、`workflowId` 不得混用。
- manifest 校验入口：导入前必须校验必填字段、类型、能力声明、输入槽、输出类型。
- provider / adapter 白名单：插件只能声明已注册 adapter，不能绕过 `api/` 直接发请求。
- extension point 白名单：插件只能挂载允许的复杂 UI 或 payload resolver，不能直接修改核心状态流。
- migration 入口：manifest schema 升级时必须能迁移旧版本配置。
- sandbox 边界：插件导入的配置默认视为不可信，不得暴露 Node/Electron 高权限 API。

自动导入流程必须是：

`discover plugin manifests -> validate schema -> register model/execution manifests -> render UI schema -> execute through adapter/runtime`

禁止插件直接注入任意 JS 到 renderer 执行业务逻辑。确需自定义逻辑时，只能通过受控 extension point，并说明权限边界、输入输出和失败兜底。

## 前端 UI 规则

前端不要为每个工作流单独写一套参数 UI。普通参数必须优先由 UI schema 自动渲染：

- `segmented`
- `select`
- `stepper`
- `slider`
- `toggle`
- `text`
- `textarea`
- `image input`
- `video input`
- `audio input`

遮罩不是独立生成类型。遮罩编辑属于图片输入的复杂交互扩展，应通过 `image input` + 受控 `extension point` 接入；只有 registry / UI schema renderer 明确支持时，才允许新增独立 `mask` 控件类型。

只有复杂交互才允许写自定义组件，例如：

- 遮罩涂抹
- 时间轴选帧
- 框选区域
- 多点标注
- 复杂预览编辑器

自定义组件也必须挂到 manifest 的 `extension point`，不得散落在模型判断里。

## 架构规则

涉及模块职责、状态流、交互分层、坐标换算、网络收口时，先查阅 `docs/architecture.md`。

- `store.js` / `appStore`：唯一状态数据源；只负责状态存取与变更。
- `renderer.js`：纯渲染层，只读 Store；禁止写状态、禁止绑定业务交互。
- `interaction.js`：只处理高频交互；禁止写业务决策；拖拽中的高频临时数据不要持续写入 Store。
- `math.js`：统一处理坐标换算；禁止其他模块重复实现。
- `api/`：统一发起网络请求；禁止其他文件直接请求。
- `main.js`：只做初始化、订阅、事件代理、模块装配；禁止承载具体业务细节。

## Electron 规则

本项目是 Electron 应用，优先遵守 Electron 分层，不按普通前端项目处理。

涉及 Electron 分层、安全、IPC、路径、打包规则时，先查阅 `docs/electron-rules.md`。

- 涉及 Electron 的改动，必须先判断落点：`main` / `preload` / `renderer` / `IPC` / 打包配置。
- `renderer` 视为不可信 UI 层，禁止直接使用 Node/Electron 高权限 API。
- `preload` 只通过 `contextBridge` 暴露最小必要 API，禁止直接暴露 `ipcRenderer`。
- 所有系统能力、文件读写、路径、下载、外链、原生能力优先放在 `main`。
- 实现时必须同时考虑开发环境与打包环境差异，禁止硬编码路径。
- 禁止为了跑通功能放松 `contextIsolation`、`nodeIntegration`、`sandbox` 等安全配置，除非用户明确要求并说明风险。

## 任务运行规则

生成类任务优先收敛到统一 Task Runtime：

- `submitTask`
- `pollTask`
- `cancelTask`
- `resumeTask`
- `saveOutput`
- `reportProgress`
- `parseError`
- `normalizeResult`

Provider Adapter 只处理厂商差异，不承载完整业务流程。

## 样式规则

- 业务样式放在 `styles/` 对应文件，不要直接堆到 `style.css`。
- 禁止新增 `!important`、硬编码颜色、`style.cssText`。
- 视觉状态优先通过 class 切换，不要用内联样式承载业务规则。
- 新增 UI 时优先复用现有样式变量、主题 token、组件结构和交互习惯。

## 渐进迁移规则

不要为了产品化一次性重写全项目。必须渐进：

- 先搜索现有实现和可复用模块。
- 不默认保留旧逻辑兜底；只有用户明确要求兼容历史项目时，才允许临时保留，并必须标记删除条件。
- 新增 `registry` / `manifest` / `schema` 入口。
- 先迁移一个最简单用例验证。
- 新功能优先走新体系。
- 旧硬编码只允许作为短期迁移对象存在，新链路验证后应删除。
- 删除旧逻辑前必须有测试或明确验证路径。

## 验证规则

- 修改后必须运行与改动范围匹配的测试或检查。
- 涉及编码、中文文件、文档变更时，优先运行 `npm run check:encoding`。
- 涉及架构边界时，优先运行 `npm run check:architecture`。
- 涉及核心 Store / math / API / preload 契约时，优先运行对应 `node --test` 或 `npm run test:critical`。
- 涉及模型 registry / manifest 时，优先运行 `node --test tests/manifestRegistry.test.js`。
- 涉及 Electron preload 或 IPC 契约时，优先运行对应 preload / IPC contract 测试。
- 若无法运行测试，必须明确说明原因和剩余风险。
- 不要声称通过了未实际运行的命令。

## 最终目标

新增普通模型 / 工作流时，主要新增或修改 manifest，而不是重新写一套前端、后端和任务编排。

- 前端通过 schema 自动显示。
- 后端通过 adapter 自动映射。
- 任务生命周期由统一 runtime 管。
- 特殊能力通过 extension point 插入。

