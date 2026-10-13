# Angular 通用质量规范

> 本文件为 angular-coding-standard 的通用质量规范，以**官方 Style Guide**（angular.dev/style-guide）与 angular.dev 官方最佳实践为纲，基线为现代 Angular（v17+ 写法为推荐方向；v14~16 遗留写法保持一致即可）。完整正反例见 `style-examples.md`（同目录）。TS 类型层规则遵循 `ts-coding-standard`。

## 目录

- 一、单一职责与文件组织
- 二、命名约定
- 三、组件与模板
- 四、服务与依赖注入
- 五、状态：Signals 与 RxJS 边界
- 六、生命周期
- 七、路由与懒加载
- 八、性能
- 九、安全

## 一、单一职责与文件组织

- 一个文件一个组件 / 服务 / 管道 / 指令；单文件超过 400 行考虑拆分（官方 SRP 规则）
- 服务只做一件事：超出单一用途时拆分（如 `UserDataService` 与 `UserAuthService` 分立）
- 按特性域组织目录（`features/users/`），共享件入 `shared/`；应用级单例入 `core/`（跟随项目既有结构约定）
- 新项目/新代码使用**独立组件（standalone）**，不再新建 NgModule；遗留 NgModule 项目维护时保持一致

## 二、命名约定

- 文件名点分隔：`user-profile.component.ts`、`user-data.service.ts`、`highlight.directive.ts`、`ellipsis.pipe.ts`
- 类名 PascalCase + 类型后缀：`UserProfileComponent` / `UserDataService`；测试名 `user-profile.component.spec.ts`
- 选择器带应用前缀（如 `app-`），camelCase：`appUserCard`；指令名同前缀规则
- 组件/服务后缀按项目统一（新版 CLI 允许省略 `Component` 后缀，默认保留以便全文检索）
- `boolean` 成员用 `is` / `has` / `can` 前缀（`isVisible`）

## 三、组件与模板

### 1. 输入输出现代写法

```ts
@Component({ … })
export class UserProfileComponent {
  // ✅ 函数式 signal API；被 Angular 初始化的成员标 readonly
  readonly userId = input.required<number>();
  readonly userSaved = output<void>();
}
```

- 被模板访问、但非公开 API 的成员标 `protected`；被 Angular 初始化的成员（`input` / `model` / `output` / queries）标 `readonly`（官方 Style Guide）

### 2. 模板规则

- 新控制流优先：`@if` / `@else` / `@switch` / `@for` 替代 `*ngIf` / `*ngFor`；`@for` 必带 `track`（稳定业务 id）

```html
@for (model of models; track model.id) {
  <app-model-card [model]="model" />
}
```

- **事件处理器按动作命名**（官方规则）：`(click)="saveUserData()"` 优于 `handleClick()`——模板读即知用途；键盘组合事件同理（`(keydown.control.enter)="commitNotes()"`）
- **优先 `class` / `style` 绑定**，优于 `ngClass` / `ngStyle`（语法直观且无额外指令开销）：

```html
<div [class.admin]="isAdmin" [style.color]="textColor">…</div>
```

- 模板只放简单表达式与具名方法调用；复杂逻辑进 `computed` / 组件方法
- 生命周期方法保持简单：`ngOnInit` 只编排具名方法调用（钩子名表达"何时"，不表达"做什么"）；实现对应接口（`implements OnInit`）

## 四、服务与依赖注入

- 新代码用 `inject()` 函数注入（字段初始化、组合式场景）；构造器注入仅在遗留代码保持一致
- 通用服务 `providedIn: 'root'`；限定特定 injector 时显式声明；跨实例令牌用 `InjectionToken` 并附描述性泛型

```ts
@Injectable({ providedIn: 'root' })
export class UserDataService {
  private readonly http = inject(HttpClient);
  private readonly config = inject(APP_CONFIG);
}
```

- 服务保持单一职责；有状态服务明确说明生命周期语义（root 单例共享 vs 组件级）
- 配置注入用 `InjectionToken`，不用字符串令牌

## 五、状态：Signals 与 RxJS 边界

### 1. Signals（新代码首选）

- 本地状态 `signal`；派生数据 `computed`（缓存、依赖自动跟踪）；副作用 `effect`（默认清理、慎用于业务流）

```ts
export class ModelListComponent {
  private readonly modelService = inject(ModelDataService);

  readonly models = this.modelService.models;
  readonly keyword = signal('');
  readonly filtered = computed(() =>
    this.models().filter((m) => m.name.includes(this.keyword())),
  );
}
```

- 组件输入派生用 `computed(() => transform(this.input()))`，不复制进新 state

### 2. RxJS 使用边界

- RxJS 用于：事件流、多值异步编排、与既有 Observable 生态集成
- 模板消费用 `async` 管道（自动退订）；命令式订阅必须配对退订（`takeUntilDestroyed`，v16+）
- 简单单值异步优先 `async/await` 或 `toSignal`，不为单值请求拉 RxJS 管道

## 六、生命周期

- 实现对应接口（`implements OnInit` / `OnDestroy`）保证方法名正确
- `ngOnInit` 保持编排简洁：只调用 `this.loadModels()` 这类具名方法
- 订阅 / 定时器 / 事件监听在 `ngOnDestroy` 清理，或使用 `takeUntilDestroyed(this.destroyRef)` / `DestroyRef`

## 七、路由与懒加载

- 路由懒加载默认：`loadComponent: () => import('./features/users/user-list.component')`；特性域用 `loadChildren`
- 守卫用函数式（`canActivate: [authGuard]`，`inject()` 内取依赖）
- 路由参数用 `input()` 绑定（`withComponentInputBinding()` 启用后），替代手工订阅 ActivatedRoute

## 八、性能

- 组件默认 `changeDetection: ChangeDetectionStrategy.OnPush`（配合 Signals / 不可变更新）
- `@for` 必带 `track`（稳定业务 id）；大列表虚拟滚动（CDK Scrolling）
- 纯展示转换用纯管道；管道内保持无副作用
- 首屏：路由级懒加载 + `defer`（@defer 延迟加载非关键块）

## 九、安全

- `innerHTML` 必须经 `DomSanitizer` 处理并最小化信任范围；禁直接绕过（`bypassSecurityTrust*` 需注释审查）
- 模板中禁拼接执行类绑定；XSS 面与 `html-coding-standard` 九章一致
