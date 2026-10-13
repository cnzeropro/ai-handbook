# TypeScript 通用质量规范

> 本文件为 ts-coding-standard 的**版本通用**质量规范，适用于 TS 4.x 至 5.x（示例以 TS 5.x 语法呈现）。写法规约见分册：TS 5.x 见 `ts5.md`，TS 4.x 及以下遗留见 `ts-legacy.md`。规则来源：TypeScript 官方 Handbook（含 Do's & Don'ts）、Google TypeScript Style Guide、TSConfig Reference。完整正反例见 `style-examples.md`（同目录）。JS 运行时语法规则遵循 `js-coding-standard`。

## 目录

- 一、tsconfig 基线原则
- 二、类型设计原则
- 三、any 与 unknown
- 四、类型收窄与守卫
- 五、断言与非空断言
- 六、枚举替代方案
- 七、interface 与 type
- 八、泛型
- 九、模块与命名
- 十、工具类型优先

## 一、tsconfig 基线原则

> 本 skill 不逐条规定 tsconfig 取值（跟随项目与构建链），但新建/审查 tsconfig 时遵循以下类别原则：

- `strict: true` 为默认起点（含 `noImplicitAny` / `strictNullChecks` / `strictFunctionTypes` 等全家桶）；禁为图省事关闭 strict
- 推荐补充：`noUncheckedIndexedAccess`（索引访问追加 undefined）、`noImplicitOverride`、`noFallthroughCasesInSwitch`、`isolatedModules`
- `skipLibCheck: true` 推荐开启（跳过第三方 .d.ts 检查，提速且不受三方类型质量拖累）
- 新项目建议 `erasableSyntaxOnly: true`：禁 enum / namespace / 参数属性等非可擦除语法，与 TS 7 原生编译器方向对齐（见 `ts5.md`）
- `allowJs` 按迁移策略；`module` / `moduleResolution` 跟随构建链（bundler/tsc/nodenext），不自造组合

## 二、类型设计原则

### 1. 让编译器推导，避免冗余标注

变量初始化、函数返回值等可推导处不手写类型；仅在参数、公开 API 边界、推导不出处标注：

```ts
// ❌ 推导得出，标注冗余
const name: string = model.name;
function getModels(): Promise<Model[]> { … }

// ✅ 标注边界（参数），返回值交给推导
function getModels(dept: string): Promise<Model[]> { … }
```

### 2. 用联合与判别联合建模状态，不用可选字段堆叠

互斥状态用判别联合（discriminated union），让“非法状态不可表示”：

```ts
// ❌ 可选字段堆叠：loaded 与 error 可能同时存在，语义不明
type RequestState = {
  loading: boolean;
  data?: Model[];
  error?: string;
};

// ✅ 判别联合：非法状态无法构造
type RequestState =
  | { status: 'loading' }
  | { status: 'success'; data: Model[] }
  | { status: 'error'; message: string };
```

### 3. 最小类型原则

函数参数与返回值只声明实际用到的能力（依赖倒置在类型上的体现）；消费方接受更宽、生产方返回更窄：

```ts
// ❌ 参数要求整个对象，实际只用一个字段
function getName(model: Model): string { return model.name; }

// ✅ 最小结构，调用方与测试都轻松
function getName(model: { name: string }): string { return model.name; }
```

## 三、any 与 unknown

### 1. 禁用 any

`any` 关闭整条链路上的类型检查，是类型系统的逃生舱而非工具：

```ts
// ❌
function parse(input: string): any { … }
const data: any = response;

// ✅ 外部输入用 unknown
function parse(input: string): unknown { … }
const data: unknown = response;
```

### 2. unknown + 类型收窄

`unknown` 是类型安全的 any——使用前必须收窄：

```ts
function isModel(value: unknown): value is Model {
  return typeof value === 'object' && value !== null && 'id' in value;
}
```

### 3. 确需 any 的例外

确属无法收窄的历史边界时：局部化 `any` 作用域（仅单参数）、注释说明原因，并严禁扩散到返回值。

## 四、类型收窄与守卫

- 优先使用编译器可识别的收窄写法：`typeof` / `instanceof` / `in` / 判别联合字段比较
- 跨边界复用校验逻辑时写自定义类型谓词（`value is T`）；谓词内部必须真实校验，不能恒真

```ts
// ❌ 恒真谓词，自欺欺人
function isModel(v: unknown): v is Model {
  return true as unknown as boolean;
}

// ✅
function isModel(v: unknown): v is Model {
  return (
    typeof v === 'object' && v !== null &&
    'id' in v && typeof (v as { id: unknown }).id === 'number'
  );
}
```

- switch + 判别联合时开启穷举检查（default 分支赋值 never）：

```ts
function render(state: RequestState): string {
  switch (state.status) {
    case 'loading': return '加载中';
    case 'success': return '成功';
    case 'error': return state.message;
    default: {
      const _exhaustive: never = state;
      return _exhaustive;
    }
  }
}
```

## 五、断言与非空断言

### 1. as 仅用于编译器无法得知的边界

DOM 查询、第三方无类型库等；同体系类型之间的断言应改写代码消除：

```ts
// ❌ 同体系断言：说明类型设计有问题
const user = data as User;

// ✅ 边界断言
const canvas = document.getElementById('canvas') as HTMLCanvasElement | null;
```

- 禁 `as any`；禁双重断言 `as unknown as T`（出现即说明数据流设计有问题，回到源头修）

### 2. 非空断言 `!` 默认禁用

`!` 是把检查推给运行时的赌注；仅当不变量确实成立且无法用类型表达时使用，必须注释：

```ts
// ❌ 无注释的 !，map 取值可能 undefined
const first = map.get('primary')!;

// ✅ 注明不变量，或改为显式判空
const first = map.get('primary');   // 不变量：初始化流程必写入 primary
```

### 3. 抑制报错用 @ts-expect-error

`@ts-expect-error` 在报错消失时会自身报错（不会沦为长期遮羞布），且必须注明原因：

```ts
// ✅
// @ts-expect-error 三方库缺类型声明，已提 issue #123
import { parseV1 } from 'legacy-lib';
```

## 六、枚举替代方案

**禁用 enum**（Google TS Guide 限制使用；社区主流放弃；TS 7 原生编译器的 erasable-only 方向亦不支持，见 `ts5.md`“TS 6/7 方向规约”）：数值枚举可隐式转换为 number（首个成员还是 falsy 值），const enum 在隔离编译下有问题，运行时对象也与类型系统割裂。

替代写法：`as const` 对象 + 字面量联合：

```ts
// ❌
enum ModelStatus {
  Draft = 'draft',
  Deployed = 'deployed',
}

// ✅
const MODEL_STATUS = {
  Draft: 'draft',
  Deployed: 'deployed',
} as const;
type ModelStatus = (typeof MODEL_STATUS)[keyof typeof MODEL_STATUS];
```

需要“由值反查名”时，`as const` 对象天然支持（`Object.values(MODEL_STATUS)`），无需 enum 的逆映射。

## 七、interface 与 type

- 对象形状：用 `interface`（可 `extends`、可被声明合并——为第三方声明补丁留口子）
- 联合、映射类型、条件类型、工具类型组合：用 `type`
- 同一声明不混用两者；团队统一后保持一致

```ts
interface ModelInfo {
  id: number;
  name: string;
}

type ModelFilter = Pick<ModelInfo, 'name'> | 'all';
type Nullable<T> = T | null;
```

- 声明合并（interface 自动合并）是双刃剑：业务代码禁用（难以追踪来源），仅限类型补丁场景

## 八、泛型

- 泛型参数命名：单字符 `T` / `K` / `V` 或描述名（`TModel`），同一文件内统一风格
- 有约束写约束（`T extends { id: number }`），让函数体可检查；需要默认给默认（`T = string`）
- 不过度抽象：只用一次且类型固定的函数不需要泛型

```ts
// ❌ 无约束，函数体内无法访问属性
function pluck<T>(items: T[], key: string): unknown[] { … }

// ✅ 约束 + keyof
function pluck<T, K extends keyof T>(items: T[], key: K): T[K][] {
  return items.map((item) => item[key]);
}
```

- 函数重载：声明从具体到宽泛排列；实现签名对外不可见，保持与重载兼容

## 九、模块与命名

- **禁 namespace**（Google TS Guide 明令）：代码组织用 ES Modules 与文件划分；`namespace Foo { … }` 构造禁用
- 类型-only 导入导出：`import type { Model } from './model'`、`export type { … }`——帮助编译器与打包器确认纯类型、可安全裁剪
- 命名：类型 PascalCase；**接口不加 `I` 前缀**（`ModelInfo` 而非 `IModelInfo`）；泛型参数描述名或单大写；类型文件导出保持与文件名对应
- 模块导入：优先具名导入；大 API（如 `* as THREE`）可用命名空间导入（与 JS skill 的通配符规则区分：TS 中 namespace import 对大三方库是 Google 认可形态，业务模块仍用具名导入）

## 十、工具类型优先

内置工具类型不手写：`Partial` / `Required` / `Readonly` / `Pick` / `Omit` / `Record` / `ReturnType` / `Parameters` / `Awaited` / `NonNullable`；组合使用表达复杂形状：

```ts
// ❌ 手写重复形状
interface ModelPatch {
  name?: string;
  status?: ModelStatus;
}

// ✅
type ModelPatch = Partial<Pick<ModelInfo, 'name' | 'status'>>;
```

自定义工具类型放项目 `types` 工具文件并导出复用，禁止同名重复定义。
