# React 通用质量规范

> 本文件为 react-coding-standard 的**版本通用**质量规范，同时适用于函数组件与 class 组件遗留代码。写法规约见分册：现行体系（函数组件 + Hooks）见 `react-modern.md`，class 组件遗留见 `react-legacy.md`。来源：react.dev Rules of React 与官方文档惯用法。完整示例见 `style-examples.md`（同目录）。

## 目录

- 一、核心原则 —— 纯渲染、单向数据流与单一数据源、组合优于继承
- 二、组件设计与状态 —— 状态就近与提升、受控组件、命名约定、props 只读
- 三、列表与条件渲染 —— key、&& 数字陷阱
- 四、安全 —— dangerouslySetInnerHTML、URL 注入

## 一、核心原则

1. **渲染应为纯计算**：同样输入（props/state）产出同样 UI；副作用放在事件处理器或副作用机制中（函数组件为 Effect、class 为生命周期，形态见对应分册）
2. **单向数据流与单一数据源**：数据自上而下经 props 流动，变更自下而上经回调请求；同一数据只存一处，其余派生

```tsx
// ❌ props 同步进 state 的反模式：两份数据源易失步
const [list, setList] = useState(props.items);
useEffect(() => setList(props.items), [props.items]);

// ✅ 用 key 重置，或直接派生
<ModelList key={props.version} items={props.items} />
```

3. **组合优于继承**：用 `children`、组合组件表达复用；单一职责，组件过大拆分

## 二、组件设计与状态

- **状态就近放置**：放实际使用的最低层级；多组件共享时提升至最近公共父级
- **受控组件为默认**；`<input type="file">` 等简单场景可用非受控（`defaultValue` + ref）
- **命名约定**：组件 PascalCase（`ModelCard.tsx`）；回调 props `onXxx`（`onSelect`），内部处理器 `handleXxx`（`handleSelect`）；布尔 props `is` / `has` / `can` 前缀（`isCompact`）；自定义 Hook `useXxx`（Hooks 形态见 `react-modern.md`）
- **Props 只读**：不得变异 props（含深层）；函数组件默认值用参数默认值（`{ page = 1 }`；`defaultProps` 已废弃）

```tsx
interface ModelCardProps {
  model: ModelInfo;
  isCompact?: boolean;
  onSelect?: (id: number) => void;
}

function ModelCard({ model, isCompact = false, onSelect }: ModelCardProps) {
}
```

## 三、列表与条件渲染

- 列表 `key` 用稳定业务 id，禁 index（重排/过滤时状态错乱，class 与函数组件同理）：

```tsx
// ❌
{models.map((m, i) => <ModelRow key={i} model={m} />)}
// ✅
{models.map((m) => <ModelRow key={m.id} model={m} />)}
```

- 条件渲染 `&&` 数字陷阱（两版本同理）：

```tsx
// ❌ count 为 0 时渲染出字面量 "0"
{count && <ModelList models={models} />}
// ✅
{count > 0 && <ModelList models={models} />}
```

## 四、安全

- `dangerouslySetInnerHTML` 等同 Vue 的 `v-html`：仅限完全可信内容，用户输入必须消毒（DOMPurify 类库）
- 防止 `javascript:` URL 注入：动态 href 校验协议白名单
