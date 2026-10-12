# React 编码风格完整示例

> 收录各主题完整示例与高频正反例（来源：react.dev Rules of React 与官方文档高频问题）。规则正文见 `SKILL.md` 与 `general-quality.md`；本文件供代码审查对照与深入学习。

## 目录

- 1. 组件骨架正反例
- 2. Rules of React 正反例（纯粹性/不可变/直接调用/Hooks 规则）
- 3. Effect 误用高频案例（“你可能不需要 Effect”）
- 4. 列表 key 与条件渲染陷阱
- 5. 数据获取与竞态
- 6. 命名约定完整示例
- 7. 常见正反例速查

## 1. 组件骨架正反例

```tsx
// ✅ 函数组件 + 类型化 props + 参数默认值 + onXxx/handleXxx 命名
interface ModelCardProps {
  model: ModelInfo;
  isCompact?: boolean;
  onSelect?: (id: number) => void;
}

function ModelCard({ model, isCompact = false, onSelect }: ModelCardProps) {
  function handleSelect() {
    onSelect?.(model.id);
  }

  return (
    <article className={isCompact ? 'card card--compact' : 'card'}>
      <h3 className="card__title">{model.name}</h3>
      <button type="button" onClick={handleSelect}>查看</button>
    </article>
  );
}
```

```tsx
// ❌ class 组件（新增代码）；props 被变异；默认值用 defaultProps
class ModelCard extends React.Component {
  render() {
    this.props.model.tags.push('x');     // 变异 props
    return <div>{this.props.model.name}</div>;
  }
}
ModelCard.defaultProps = { isCompact: false };   // React 19 已废弃
```

## 2. Rules of React 正反例

纯粹性（渲染期禁副作用与不纯读取）：

```tsx
// ❌ 渲染期副作用、随机数、外部可变状态
function ModelList({ models }: Props) {
  if (models.length > 0) {
    api.log('rendered');                       // 副作用
  }
  const seed = Date.now();                     // 不纯
  return <ul>{models.map((m) => <ModelRow key={m.id} model={m} />)}</ul>;
}

// ✅ 副作用进事件处理器/Effect；派生值确定性
function ModelList({ models }: Props) {
  return <ul>{models.map((m) => <ModelRow key={m.id} model={m} />)}</ul>;
}
```

props 与 state 不可变：

```tsx
// ❌ 直接变异
function handleRename(model: ModelInfo, name: string) {
  model.name = name;                       // 变异 props
  setModels((prev) => {
    prev[0].name = name;                   // 变异 state
    return prev;
  });
}

// ✅ 不可变更新
function handleRename(model: ModelInfo, name: string) {
  setModels((prev) =>
    prev.map((m) => (m.id === model.id ? { ...m, name } : m)),
  );
}
```

直接调用组件函数（官方规则）：

```tsx
// ❌ 不经 React 渲染，无 hook/上下文
const card = ModelCard({ model });

// ✅
const card = <ModelCard model={model} />;
```

Rules of Hooks：

```tsx
// ❌ 条件/循环/嵌套中调用；早 return 之后调用
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

## 3. Effect 误用高频案例（“你可能不需要 Effect”）

派生数据不建 state：

```tsx
// ❌ Effect 同步派生值：多一次渲染，且可能失步
const [fullName, setFullName] = useState('');
useEffect(() => {
  setFullName(`${first} ${last}`);
}, [first, last]);

// ✅ 渲染期派生
const fullName = `${first} ${last}`;
```

事件逻辑放进 Effect：

```tsx
// ❌ "提交后弹提示"是事件，不是 Effect
const [submitted, setSubmitted] = useState(false);
useEffect(() => {
  if (submitted) toast('提交成功');
}, [submitted]);

// ✅
async function handleSubmit() {
  await save(model);
  toast('提交成功');
}
```

props 变化重置 state 用 key，不用 Effect：

```tsx
// ❌
const [draft, setDraft] = useState(props.initial);
useEffect(() => setDraft(props.initial), [props.initial]);

// ✅ 父级用 key 重挂载
<ModelEditor key={props.modelId} initial={props.initial} />
```

## 4. 列表 key 与条件渲染陷阱

```tsx
// ❌ index 作 key：删除/排序时状态错乱
{models.map((m, i) => <ModelRow key={i} model={m} />)}

// ✅ 稳定业务 id
{models.map((m) => <ModelRow key={m.id} model={m} />)}
```

```tsx
// ❌ count 为 0 时渲染出字面量 "0"
{count && <ModelList models={models} />}

// ✅
{count > 0 && <ModelList models={models} />}
```

## 5. 数据获取与竞态

Effect 获取必须可取消、防竞态：

```tsx
// ❌ 无取消、无竞态防护：快速切换 id 时旧响应可能覆盖新响应
useEffect(() => {
  fetch(`/api/models/${id}`)
    .then((r) => r.json())
    .then(setModel);
}, [id]);

// ✅ AbortController + 忽略已取消
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

> 请求去重/缓存/重试属横切能力，优先项目选型的请求库或框架内置机制，避免逐组件手写。

## 6. 命名约定完整示例

```tsx
// ✅ 组件 PascalCase；回调 props onXxx；内部处理器 handleXxx；自定义 Hook useXxx；布尔 props is/has 前缀
interface ModelListProps {
  models: ModelInfo[];
  isEditable?: boolean;
  onSelect?: (id: number) => void;
}

function useModelFilter(models: ModelInfo[]) {
  const [keyword, setKeyword] = useState('');
  const filtered = useMemo(
    () => models.filter((m) => m.name.includes(keyword)),
    [models, keyword],
  );
  return { keyword, setKeyword, filtered };
}

function ModelList({ models, isEditable = false, onSelect }: ModelListProps) {
  const { keyword, setKeyword, filtered } = useModelFilter(models);

  function handleSearchChange(value: string) {
    setKeyword(value);
  }

  return (/* ... */);
}
```

## 7. 常见正反例速查

| 问题 | ❌ 常见写法 | ✅ 正确写法 |
| --- | --- | --- |
| 变异 state | `list.push(x)` | `setList([...list, x])` |
| 变异 props | `model.tags.splice(...)` | emit/回调请求父级不可变更新 |
| index 作 key | `key={i}` | `key={item.id}` |
| && 数字陷阱 | `{count && <List />}` | `{count > 0 && <List />}` |
| 派生值建 state | Effect 同步 fullName | 渲染期派生（+useMemo） |
| 事件逻辑进 Effect | 监听 submitted 弹 toast | 事件处理器内 toast |
| Effect 无清理 | 订阅/定时器裸注册 | 返回清理函数 |
| 请求无竞态防护 | 裸 fetch.then | AbortController + 忽略取消 |
| Hook 在条件中 | `if (x) { useState() }` | 顶层调用 |
| 直接调用组件 | `ModelCard({ model })` | `<ModelCard model={model} />` |
| 危险 HTML | `dangerouslySetInnerHTML` 直用 | 仅可信内容 + 消毒 |
| 滥用 memo 三件套 | 全量包 useMemo/useCallback/memo | 确认瓶颈后使用 |
