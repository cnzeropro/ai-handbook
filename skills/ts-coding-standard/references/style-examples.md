# TypeScript 编码风格完整示例

> 收录各主题完整示例与市面高频正反例（来源：TypeScript 官方 Handbook 与 Do's & Don'ts、Google TypeScript Style Guide、社区审查常见问题）。规则正文见 `SKILL.md` 与 `general-quality.md`；本文件供代码审查对照与深入学习。

## 目录

- 1. any / unknown 与收窄
- 2. 判别联合建模状态（含穷举检查）
- 3. 枚举替代（as const 完整示例）
- 4. 断言与非空断言
- 5. interface vs type
- 6. 泛型设计
- 7. 工具类型组合
- 8. type-only 导入与模块组织
- 9. 常见正反例速查

## 1. any / unknown 与收窄

```ts
// ❌ any 贯穿全链路，错误推迟到运行时
function parseUser(input: string): any {
  return JSON.parse(input);
}
const user = parseUser(raw);
user.nmae;              // 拼写错误不报错

// ✅ unknown 强制收窄
function parseUser(input: string): unknown {
  return JSON.parse(input);
}

const user = parseUser(raw);
if (isUser(user)) {
  console.log(user.name);   // 收窄后可安全访问
}
```

## 2. 判别联合建模状态（含穷举检查）

```ts
type RequestState =
  | { status: 'idle' }
  | { status: 'loading' }
  | { status: 'success'; data: Model[] }
  | { status: 'error'; message: string };

function render(state: RequestState): string {
  switch (state.status) {
    case 'idle':    return '待发起';
    case 'loading': return '加载中';
    case 'success': return `已加载 ${state.data.length} 条`;
    case 'error':   return state.message;
    default: {
      // ✅ 穷举检查：新增状态而漏处理时编译期报错
      const _exhaustive: never = state;
      return _exhaustive;
    }
  }
}
```

```ts
// ❌ 可选字段堆叠：非法状态可表示（success 与 error 同时存在）
type BadState = {
  status: 'success' | 'error';
  data?: Model[];
  message?: string;
};
```

## 3. 枚举替代（as const 完整示例）

```ts
// ❌ enum：数值枚举可隐式转 number、首个成员为 falsy；const enum 有隔离编译问题
enum ModelStatus {
  Draft = 'draft',
  Deployed = 'deployed',
}
if (model.status === ModelStatus.Draft) { … }   // Draft == 0 的历史陷阱

// ✅ as const 对象 + 字面量联合
const MODEL_STATUS = {
  Draft: 'draft',
  Deployed: 'deployed',
} as const;
type ModelStatus = (typeof MODEL_STATUS)[keyof typeof MODEL_STATUS];

if (model.status === MODEL_STATUS.Draft) { … }
```

需要“由值反查名”时，as const 对象天然支持：

```ts
function statusName(value: ModelStatus): string {
  return Object.entries(MODEL_STATUS).find(([, v]) => v === value)![0];
}
```

## 4. 断言与非空断言

```ts
// ❌ 同体系断言：掩盖类型设计问题
const user = rawData as User;

// ❌ 双重断言：数据流设计失败
const btn = document.querySelector('.btn') as unknown as HTMLButtonElement;

// ✅ 边界断言（编译器无法得知的 DOM 事实）
const canvas = document.getElementById('cv') as HTMLCanvasElement | null;
```

非空断言 `!` 的唯一豁免形态（不变量注释 + 替代方案对比）：

```ts
// ❌ 裸非空断言
const ctx = canvas.getContext('2d')!;

// ✅ 显式判空（优先）
const ctx = canvas?.getContext('2d');
if (!ctx) throw new Error('Canvas 2D context unavailable');

// ✅ 不变量确实成立时，注释说明
const ctx = canvas.getContext('2d')!;   // 不变量：canvas 宽高均 > 0，context 必非空
```

## 5. interface vs type

```ts
// ✅ 对象形状：interface
interface ModelInfo {
  id: number;
  name: string;
  tags: string[];
}
interface ModelDetail extends ModelInfo {
  description: string;
}

// ✅ 联合/映射/条件/工具类型：type
type ModelFilter = Pick<ModelInfo, 'name'> | 'all';
type Nullable<T> = T | null;
type ModelMap = Record<number, ModelInfo>;
```

❌ 同一声明混用与业务代码声明合并：

```ts
interface ModelInfo { id: number }
interface ModelInfo { name: string }   // 声明合并：来源难追踪，业务代码禁用
type ModelInfo = { … }                 // 与同名 interface 冲突
```

## 6. 泛型设计

约束 + keyof 的取值函数：

```ts
// ❌ 无约束，函数体不可检查；返回 unknown 失去信息
function pluck(items: unknown[], key: string) { … }

// ✅ 约束让函数体可检查
function pluck<T, K extends keyof T>(items: T[], key: K): T[K][] {
  return items.map((item) => item[key]);
}
```

默认类型参数：

```ts
interface FetchOptions<TBody = unknown> {
  url: string;
  body?: TBody;
  retries?: number;
}
```

重载从具体到宽泛：

```ts
// ✅
function parse(input: string): ModelInfo;
function parse(input: ModelInfo): ModelInfo;
function parse(input: unknown): ModelInfo { … }
```

## 7. 工具类型组合

```ts
// ❌ 手写重复形状（字段变更需同步三处）
interface CreateModelDTO { name: string; status: ModelStatus; tags: string[] }
interface UpdateModelDTO { name?: string; status?: ModelStatus; tags?: string[] }

// ✅ 组合工具类型，单一来源
interface ModelDraft {
  name: string;
  status: ModelStatus;
  tags: string[];
}
type CreateModelDTO = ModelDraft;
type UpdateModelDTO = Partial<ModelDraft>;
```

常用组合：

```ts
type PublicModel = Omit<ModelInfo, 'internalNote'>;
type ModelLookup = Record<number, ModelInfo>;
type Handler<T> = (payload: T) => void;
type AwaitedData<P> = Awaited<ReturnType<P>>;
```

## 8. type-only 导入与模块组织

```ts
// ❌ 值导入方式引入纯类型：打包器无法安全裁剪
import { ModelInfo, ModelService } from './model';

// ✅ 类型与值分离
import type { ModelInfo } from './model';
import { ModelService } from './model';

// ✅ 导出类型同理
export type { ModelInfo, RequestState };
export { ModelService };
```

模块组织：

```ts
// ❌ namespace 组织代码
namespace Models { … }

// ✅ ES Modules + 文件划分
// models/model-info.ts / models/model-service.ts
```

## 9. 常见正反例速查

| 问题 | ❌ 常见写法 | ✅ 正确写法 |
| --- | --- | --- |
| 关闭类型检查 | `any` | `unknown` + 收窄 |
| 断言滥用 | `as any` / `as unknown as T` | 收窄或从源头修类型 |
| 非空赌注 | `map.get(k)!` | 判空 / 注明不变量 |
| 遮蔽报错 | `@ts-ignore` | `@ts-expect-error` + 原因 |
| 枚举 | `enum Status` | `as const` 对象 + 联合 |
| 命名空间 | `namespace Foo` | ES Modules 拆文件 |
| 可选字段堆叠 | 全可选字段建模状态 | 判别联合 |
| 恒真谓词 | `v is T` 恒 true | 真实校验 |
| I 前缀 | `IModelInfo` | `ModelInfo` |
| 缺穷举 | switch 漏分支 | default 赋 `never` |
| 手写形状 | 重复 interface | `Partial` / `Pick` / `Omit` |
| 类型值混导 | `import { Type }` | `import type` |
