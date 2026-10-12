# React class 组件（遗留环境）规范

> 适用：React ≤16.x class 组件遗留代码库。仅在维护既有代码时使用；新组件一律函数组件 + Hooks（见 `general-quality.md`）。同一项目保持一致，不混用两种写法。

## 一、class 组件写法基线

- 继承 `React.Component`；确认浅比较足够时才用 `PureComponent`
- `state` 与事件处理器用**类字段**定义；处理器用箭头函数类字段，避免渲染期 `bind`（每次渲染生成新函数）

```jsx
class ModelList extends React.Component {
  state = { keyword: '' };

  handleSearchChange = (e) => {
    this.setState({ keyword: e.target.value });
  };

  render() {
    const { keyword } = this.state;
    const filtered = this.props.models.filter((m) => m.name.includes(keyword));
    return (
      <input value={keyword} onChange={this.handleSearchChange} />
      /* ... */
    );
  }
}
```

- 依赖前值时 `setState` 用函数式更新：`this.setState((prev) => ({ count: prev.count + 1 }))`；`setState` 是**合并**语义（对比 Hooks 的整体替换）
- 生命周期映射：`componentDidMount` / `componentWillUnmount` 中注册与清理订阅、定时器；`getDerivedStateFromProps` 谨慎——多数场景应派生渲染而非复制 props 到 state
- 列表 `key` 用稳定业务 id；`shouldComponentUpdate` / `PureComponent` 仅在确认瓶颈后使用

## 二、迁移到函数组件（Hooks 对照）

| class 写法 | Hooks 等价 |
| --- | --- |
| `this.state` / `this.setState` | `useState` / `useReducer` |
| `componentDidMount` / `componentDidUpdate` | `useEffect(fn, [deps])` |
| `componentWillUnmount` | `useEffect` 返回清理函数 |
| `shouldComponentUpdate` / `PureComponent` | `React.memo` |
| 实例字段 / 方法 | `useRef` / 函数（或 `useCallback`） |
| 生命周期分散的同域逻辑 | 按“关注点”聚合的自定义 Hook（`useXxx`） |

## 三、遗留代码最小侵入

- 维护期不顺手把 class 组件迁移为 Hooks；升级作为独立任务执行
- 遗留库中的新组件与项目整体风格保持一致，不混用两种写法
- 迁移顺序建议：叶子展示组件 → 容器组件；迁移后以 StrictMode + `eslint-plugin-react-hooks` 验证
