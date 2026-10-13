# Angular 编码风格完整示例

> 收录各主题完整示例与高频正反例（来源：angular.dev 官方 Style Guide 与最佳实践文档）。规则正文见 `SKILL.md` 与 `general-quality.md`；本文件供代码审查对照与深入学习。

## 目录

- 1. 组件骨架（现代写法完整示例）
- 2. 输入输出现代写法
- 3. 事件命名与模板规则
- 4. 依赖注入（inject 与 InjectionToken）
- 5. Signals 与 RxJS 边界
- 6. 路由懒加载与函数式守卫
- 7. 性能（track / OnPush / @defer）
- 8. 安全（DomSanitizer）
- 9. 常见正反例速查

## 1. 组件骨架（现代写法完整示例）

```ts
import { ChangeDetectionStrategy, Component, computed, inject } from '@angular/core';

@Component({
  selector: 'app-user-profile',
  changeDetection: ChangeDetectionStrategy.OnPush,
  template: `
    <article class="profile">
      <h2>{{ fullName() }}</h2>
      @if (canEdit()) {
        <button type="button" (click)="saveProfile()">保存</button>
      }
    </article>
  `,
})
export class UserProfileComponent {
  private readonly userApi = inject(UserApiService);

  // 输入：被 Angular 初始化，标 readonly
  readonly userId = input.required<number>();
  readonly compact = input(false);

  // 仅模板访问的派生成员：protected
  protected readonly user = this.userApi.byId(this.userId);
  protected readonly canEdit = computed(() => this.user()?.role === 'editor');
  protected readonly fullName = computed(
    () => `${this.user()?.firstName ?? ''} ${this.user()?.lastName ?? ''}`,
  );

  // 公开 API（供父组件/服务使用）才用 public
  saveProfile(): void { … }
}
```

❌ 遗留混搭（维护期保持一致，新代码禁止）：NgModule 声明、`@Input() userId = 0`、构造器注入、模板内复杂表达式链。

## 2. 输入输出现代写法

```ts
export class ModelCardComponent {
  // ✅ 必填 / 可选 / 别名 / 双向
  readonly model = input.required<ModelInfo>();
  readonly compact = input(false, { alias: 'dense' });
  readonly selected = model(true);            // 双向绑定

  // ✅ 输出：output 函数
  readonly opened = output<number>();
}
```

被 Angular 初始化的成员标 `readonly`（官方规则）：

```ts
// ✅ input / output / model / queries 均标 readonly
readonly userSaved = output<void>();
readonly paymentMethods = viewChildren(PaymentMethod);
```

## 3. 事件命名与模板规则

```html
<!-- ❌ 按触发事件命名：读模板不知用途 -->
<button (click)="handleClick()">Save</button>

<!-- ✅ 按动作命名 -->
<button (click)="saveUserData()">Save</button>

<!-- ✅ 键盘组合事件 -->
<textarea (keydown.control.enter)="commitNotes()">…</textarea>
```

class / style 绑定优于指令：

```html
<!-- ❌ -->
<div [ngClass]="{admin: isAdmin}" [ngStyle]="{'color': textColor}">

<!-- ✅ -->
<div [class.admin]="isAdmin" [style.color]="textColor">
```

生命周期保持简单（官方规则）：

```ts
// ❌ 钩子内展开长逻辑
ngOnInit() {
  this.logger.setMode('info');
  this.logger.monitorErrors();
  this.loadModels();
}

// ✅ 钩子只编排具名方法
ngOnInit() {
  this.startLogging();
  this.loadModels();
}
```

## 4. 依赖注入（inject 与 InjectionToken）

```ts
export const APP_CONFIG = new InjectionToken<AppConfig>('APP_CONFIG');

@Injectable({ providedIn: 'root' })
export class UserDataService {
  // ✅ inject()：字段初始化即可用，免去构造器样板
  private readonly http = inject(HttpClient);
  private readonly config = inject(APP_CONFIG);
}
```

```ts
// ❌ 构造器注入（新代码）
constructor(private http: HttpClient, private config: AppConfig) {}
```

## 5. Signals 与 RxJS 边界

```ts
// ❌ watch 风格：手工订阅互相嵌套
this.route.params.subscribe((p) => {
  this.userService.byId(p.id).subscribe((user) => { … });
});

// ✅ Signals 组合 + toSignal
private readonly route = inject(ActivatedRoute);
readonly userId = toSignal(this.route.params.pipe(map((p) => p['id'])));
protected readonly user = toSignal(
  toObservable(this.userId).pipe(switchMap((id) => this.userService.byId(id))),
);
```

订阅清理（v16+）：

```ts
private readonly destroyRef = inject(DestroyRef);

ngOnInit() {
  interval(5000).pipe(takeUntilDestroyed(this.destroyRef)).subscribe(this.refresh);
}
```

单值异步优先 async/await；事件流与多值编排才用 RxJS。

## 6. 路由懒加载与函数式守卫

```ts
export const routes: Routes = [
  {
    path: 'users',
    canActivate: [authGuard],                      // 函数式守卫（inject 取依赖）
    loadComponent: () =>
      import('./features/users/user-list.component').then((m) => m.UserListComponent),
  },
  {
    path: 'admin',
    loadChildren: () => import('./features/admin/admin.routes').then((m) => m.ADMIN_ROUTES),
  },
];
```

路由参数用 input 绑定（启用 `withComponentInputBinding()` 后）：

```ts
// ✅ 替代手工订阅 ActivatedRoute
readonly id = input.required<string>();
```

## 7. 性能（track / OnPush / @defer）

```html
<!-- ❌ 无 track：全量 diff -->
@for (model of models) { … }

<!-- ✅ 稳定业务 id -->
@for (model of models; track model.id) { … }
```

```html
@defer (on viewport) {
  <app-heavy-chart />
} @placeholder {
  <div class="h-64">加载图表…</div>
}
```

- 组件默认 `OnPush`；配合 Signals / 不可变更新
- 纯展示转换用纯管道；管道内无副作用

## 8. 安全（DomSanitizer）

```ts
// ❌ 用户内容直插
<div [innerHTML]="comment.body"></div>

// ✅ 先经消毒，并最小化信任范围
readonly safeBody = computed(() =>
  this.sanitizer.sanitize(SecurityContext.HTML, DOMPurify.sanitize(this.comment.body)),
);
```

## 9. 常见正反例速查

| 问题 | ❌ 常见写法 | ✅ 正确写法 |
| --- | --- | --- |
| 模板成员公开 | `public fullName` | `protected fullName = computed(…)` |
| 忘记 readonly | `userId = input.required<number>()` | `readonly userId = input.required<number>()` |
| 旧控制流 | `*ngIf` / `*ngFor` | `@if` / `@for`（`track` 必带） |
| ngClass/ngStyle | `[ngClass]="{…}"` | `[class.x]="expr"` / `[style.color]` |
| 事件名无语义 | `(click)="onClick()"` | `(click)="saveUserData()"` |
| 构造器注入 | 构造器参数注入 | `inject()` 字段初始化 |
| 手工订阅嵌套 | subscribe 嵌套 | `toSignal` + `computed` / `switchMap` |
| 默认变更检测 | Default 策略 | `OnPush` + Signals/不可变更新 |
| 路由参数手工订阅 | ActivatedRoute params 订阅 | `input()` 绑定 + `withComponentInputBinding` |
| 钩子内长逻辑 | ngOnInit 展开 | 编排具名方法 |
| 模块包裹新组件 | NgModule declarations | standalone 组件 |
| innerHTML 直插 | 用户内容直接绑定 | DomSanitizer + 消毒 |
