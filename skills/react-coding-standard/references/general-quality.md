# React 通用质量规范

> 本文件为 react-coding-standard 的通用质量规范，基线 **函数组件 + Hooks（React 18/19）**。一～二章来自 React 官方 Rules of React（“rules not guidelines”，违反即 bug），三～六章为官方文档惯用法。完整示例见 `style-examples.md`（同目录）。

## 目录

- 一、Rules of React —— 纯粹性（幂等、副作用出渲染期、props/state 不可变）、由 React 调用、Rules of Hooks
- 二、Hooks 使用约定 —— 派生不建 state、Effect 正确场景与清理、依赖数组、状态选型
- 三、组件设计与状态 —— 组合优于继承、状态就近与提升、受控组件、命名约定
- 四、渲染性能 —— key、条件渲染 && 陷阱、memo 三件套、虚拟滚动
- 五、数据获取原则 —— Effect 获取与清理、请求库原则层
- 六、安全 —— dangerouslySetInnerHTML、URL 注入

## 一、Rules of React

### 1. 组件与 Hooks 必须纯粹（官方 Rules of React）

- **幂等**：同样输入（props/state/context）永远产出同样 JSX——渲染期禁随机数、时间戳、未受控的可变读取

```tsx
// ❌ 渲染期副作用 + 不纯
function ModelCard({ model }: Props) {
  items.push(model.id);                       // 修改外部状态
  return <div>{Math.random()}</div>;
}

// ✅
function ModelCard({ model }: Props) {
  return <div>{model.name}</div>;
}
```

- **副作用在渲染期之外**：事件处理器或 Effect 中执行（请求、订阅、DOM 操作、日志）
- **props 与 state 不可变**：

```tsx
// ❌ 直接变异
function handleAdd(tag: string) {
  model.tags.push(tag);          // 变异 props
  setModels((m) => { m.push(newModel); return m; });  // 变异 state
}

// ✅ 不可变更新
function handleAdd(tag: string) {
  setModel({ ...model, tags: [...model.tags, tag] });
  setModels((prev) => [...prev, newModel]);
}
```

- 传给 Hook 的参数/返回值、已写入 JSX 的值，之后不得修改

### 2. 组件与 Hooks 由 React 调用（官方 Rules of React）

```tsx
// ❌ 直接调用组件函数——只是创建元素，不走 React 生命周期
const card = ModelCard({ model });

// ✅
const card = <ModelCard model={model} />;

// ❌ Hook 作为普通值传递/在普通函数中调用
function useThingFactory() {
  return useState(0);   // Hook 被转手
}
```

### 3. Rules of Hooks（官方 Rules of React）

```tsx
// ❌ 条件/循环/嵌套中调用 Hook
if (expanded) {
  const [open, setOpen] = useState(false);
}

// ✅ 顶层调用，早于任何 return
const [open, setOpen] = useState(false);
if (!expanded) return null;
```

## 二、Hooks 使用约定

### 1. 派生数据不建 state（官方“你可能不需要 Effect”）

```tsx
// ❌ state 冗余 + Effect 同步，易失步
const [fullName, setFullName] = useState('');
useEffect(() => {
  setFullName(`${first} ${last}`);
}, [first, last]);

// ✅ 渲染期派生
const fullName = `${first} ${last}`;
```

昂贵派生用 `useMemo`：

```tsx
const sorted = useMemo(
  () => [...models].sort(compareFn),
  [models],
);
```

### 2. Effect 只做“与外部系统同步”

- 适用：订阅、定时器、非 React 组件操作、日志上报
- 事件驱动的逻辑放事件处理器，不放 Effect（“响应提交”是事件不是 Effect）
- prop 变化需要重置内部状态时，优先用 `key` 重挂载而非 Effect 监听

```tsx
// ❌ 事件逻辑塞进 Effect（提交后弹窗）
useEffect(() => {
  if (submitted) {
    toast('提交成功');
  }
}, [submitted]);

// ✅ 事件处理器中直接处理
function handleSubmit() {
  save(model).then(() => toast('提交成功'));
}
```

### 3. 依赖与清理

- 依赖数组完整（`exhaustive-deps`），不撒谎不省略；确实无需随依赖重跑的逻辑移入事件处理器
- 订阅 / 定时器 / 请求必须返回清理函数；数据请求用 `AbortController` 取消过期请求

```tsx
useEffect(() => {
  const controller = new AbortController();
  fetch(`/api/models/${id}`, { signal: controller.signal })
    .then((r) => r.json())
    .then(setModel)
    .catch((e) => {
      if (e.name !== 'AbortError') reportError(e);
    });
  return () => controller.abort();
}, [id]);
```

### 4. 状态选型

- 局部 `useState` → 迁移逻辑复杂 `useReducer` → 跨组件提升或状态库（Zustand/Redux Toolkit 等按项目选型，本 skill 不强绑）
- `useContext` 用于低频全局上下文（主题、当前用户），不作为高频更新的数据总线

## 三、组件设计与状态

- **组合优于继承**：`children`、组合组件、Render Props/自定义 Hook 复用逻辑
- **状态就近**：放实际使用的最低层级；共享时提升到最近公共父级
- **单一数据源**：同一数据只存一处，其余派生；避免 `useEffect` 把 props 同步进 state（反模式；确需“随 prop 重置”用 `key`）

```tsx
// ❌ props 同步进 state 的反模式
const [list, setList] = useState(props.items);
useEffect(() => setList(props.items), [props.items]);

// ✅ 用 key 重置，或直接派生
<ModelList key={props.version} items={props.items} />
```

- **受控组件为默认**；`<input type="file">` 等用非受控（`defaultValue` + ref）
- **命名约定**：组件 PascalCase（`ModelCard.tsx`）；回调 props `onXxx`（`onSelect`），组件内处理器 `handleXxx`（`handleSelect`）；自定义 Hook `useXxx`；布尔 props `is/has/can` 前缀（`isCompact`）

```tsx
// ✅ Props 类型化 + 默认值（不用已废弃的 defaultProps）
interface ModelCardProps {
  model: ModelInfo;
  isCompact?: boolean;
  onSelect?: (id: number) => void;
}
function ModelCard({ model, isCompact = false, onSelect }: ModelCardProps) {
}
```

## 四、渲染性能

- **key 用稳定业务 id**：index 作 key 在重排/过滤时引发状态错乱（同 Vue A 级规则）

```tsx
// ❌
{models.map((m, i) => <ModelRow key={i} model={m} />)}
// ✅
{models.map((m) => <ModelRow key={m.id} model={m} />)}
```

- **条件渲染 && 数字陷阱**：

```tsx
// ❌ count 为 0 时渲染出 "0"
{count && <ModelList models={models} />}
// ✅
{count > 0 && <ModelList models={models} />}
```

- `useMemo` / `useCallback` / `memo` 不预防性滥用：先确认瓶颈或引用相等性有语义；启用 React Compiler 的项目交由编译器
- 长列表虚拟滚动（react-window 等按项目选型）；重组件 `React.lazy` + `Suspense` 分包
- `dangerouslySetInnerHTML` 仅限完全可信内容；用户输入先消毒（DOMPurify 类库）
- 防止 `javascript:` URL 注入：动态 href 校验协议白名单

## 五、数据获取原则

- Effect 中获取数据必须可取消（AbortController）并处理竞态（仅接受最后一次结果）
- 请求去重、缓存、重试属于横切能力：优先项目选型的请求库（React Query/SWR 等，本 skill 不强绑）或框架内置机制（Next.js/RSC），避免每个组件手写 Effect 获取
- 服务端数据不复制进 state 长存；以请求层缓存为单一来源
