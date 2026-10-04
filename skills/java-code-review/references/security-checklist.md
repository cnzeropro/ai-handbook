# 安全审查清单

> 本文件为 `../SKILL.md` 的补充检查清单：安全维度的深度审查点。按优先级逐类排查。

## 1. 访问控制

- **认证**：敏感接口是否要求认证（登录态、网关、Token 校验）
- **授权**：是否使用权限控制注解（`@PreAuthorize`、`@Secured`）或权限框架校验
- **越权（IDOR，最高频）**：
  - 水平越权：按 `id` 操作的接口（`getById` / `updateById` / `removeById` 等）是否校验该资源属于当前用户/租户——例如用户 A 传入他人订单号即可查看/修改
  - 垂直越权：低权限角色是否可调用高权限接口
- **敏感操作**：支付、审批、管理员操作是否有二次校验（密码/验证码/审批流）

## 2. 注入类

- **SQL 注入**：MyBatis 使用 `#{}`（安全）而非 `${}`（危险）；字符串拼接 SQL 一律禁止

  ```java
  // ❌ 危险：字符串拼接 SQL
  String sql = "SELECT * FROM users WHERE name = '" + name + "'";

  // ✅ 安全：参数化查询
  @Select("SELECT * FROM users WHERE name = #{name}")
  User findByName(@Param("name") String name);
  ```
- **反序列化**：禁止 `ObjectInputStream` 直接反序列化外部数据；fastjson/jackson 不得启用不安全的类型解析配置（fastjson 的 `enableDefaultTyping` / `@type` 自动识别，jackson 的 `activateDefaultTyping`）
- **日志注入（log forging）**：用户输入未经清洗直接进入日志，攻击者可用换行符伪造日志条目，注意日志占位符中的用户输入应过滤 `\n` / `\r` 等控制字符
- **正则 ReDoS**：校验外部输入的正则是否存在灾难性回溯（多重 `+` / `*` 嵌套分组）
- **命令注入**：`Runtime.exec` / `ProcessBuilder` 拼接外部输入

## 3. 输出与数据安全

- **XSS**：模板引擎（如 Thymeleaf）是否启用自动 HTML 转义；`@ResponseBody` 返回 JSON 本身不做 HTML 转义，前端渲染侧需安全处理
- **CSRF**：表单、AJAX 提交是否做 CSRF 安全验证（Token/Referer 校验）
- **脱敏展示**：手机号、身份证、银行卡等敏感数据展示时脱敏（如 `139****1219`）
- **敏感数据存储与传输**：密码使用强哈希存储（禁用明文/可逆加密与 MD5/SHA-1 弱哈希；加盐并使用 bcrypt/argon2 等）；token、密钥不明文传输与存储；配置文件中的密码必须加密
- **超大整数 String 返回**：超过 `2^53 - 1`（9007199254740991，JS 最大安全整数）的 Long 返回前端必须用 String（JS Number 精度损失，订单号等 16 位以上场景必现问题）
- **异常信息不泄露内部细节**：错误响应与日志不含 SQL、堆栈、内部路径、框架版本

## 4. 网络与资源类

- **SSRF**：服务端发起外部 URL 请求（下载、回调、抓取）时必须做目标白名单/内网地址拦截
- **文件上传**：文件大小、类型严格校验；保存路径防路径穿越（`../`）；禁止可执行类型
- **开放重定向**：`redirect` 目标地址必须白名单过滤，防钓鱼
- **防重放/防刷**：短信、邮件、支付、下单等平台资源调用必须限流、疲劳度控制、验证码校验，防资损
- **XXE**：XML 解析（`DocumentBuilderFactory` / `SAXParser` 等）必须禁用外部实体（`setFeature` 关闭 DTD/外部实体）
- **CORS 与安全配置**：`@CrossOrigin` 不得配置任意来源（`*`）；actuator/调试端点仅内部开放；无默认凭据
- **依赖组件漏洞**：新增/升级依赖版本是否含已知高危 CVE（如 log4j2 类），以依赖扫描结果为准

## 5. 参数校验

- 外部输入有效性：`pageSize` 上限、`orderBy` 字段白名单（防恶意 order by）、批量操作大小上限（防内存爆掉）
- 类型与范围校验：数值范围、枚举合法性、必填项（对齐 `java-feature-standard` 的参数校验规范）

## 6. 安全审计日志

- 登录失败、越权尝试、敏感操作（删除/导出/权限变更）是否有审计日志留痕
- 审计日志是否独立于业务日志、含操作人/时间/IP/对象标识
