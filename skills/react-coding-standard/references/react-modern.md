# React 现行规范（函数组件 + Hooks）

> 本文件为 React **现行体系**（函数组件 + Hooks，React 17 ~ 19 同写法）的特有约定，须与版本通用的 `general-quality.md` 配合使用。class 组件遗留环境见 `react-legacy.md`。

## 目录

- 一、Rules of React —— 纯粹性、由 React 调用、Rules of Hooks
- 二、Hooks 使用约定 —— 派生不建 state、Effect 正确场景与清理、依赖数组、状态选型
- 三、渲染性能 —— memo 三件套与 React Compiler、虚拟滚动、lazy
- 四、数据获取原则 —— AbortController 与竞态、请求库原则层

## 一、Rules of React

官方定性为“规则而非指南”——违反即 bug。配合 StrictMode 与 `eslint-plugin-react-hooks` 使用。

### 1. 组件与 Hooks 必须纯粹

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

### 2. 组件与 Hooks 由 React 调用

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

### 3. Rules of Hooks

```tsx
// ❌ 条件/循环/嵌套中调用 Hook；早 return 之后调用
function ModelList({ expanded }: Props) {
  if (!expanded) return null;
  const [page, setPage] = useState(1);      // 违反顶层规则
}

// ✅ Hook 全部在顶层
function ModelList({ expanded }: Props) {
  const [page, setPage] = useState(1);
  if (!expanded) return null;
}
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

## 三、渲染性能

- `useMemo` / `useCallback` / `memo` 不预防性滥用：先确认瓶颈或引用相等性有语义；启用 React Compiler 的项目交由编译器自动记忆化
- 长列表虚拟滚动（react-window 等按项目选型）；重组件 `React.lazy` + `Suspense` 分包

## 四、数据获取原则

- Effect 中获取数据必须可取消（AbortController）并处理竞态（仅接受最后一次结果）
- 请求去重、缓存、重试属于横切能力：优先项目选型的请求库（React Query/SWR 等）或框架内置机制（Next.js/RSC），避免每个组件手写 Effect 获取
- 服务端数据不复制进 state 长存；以请求层缓存为单一来源
