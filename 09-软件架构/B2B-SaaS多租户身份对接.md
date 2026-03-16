# B2B SaaS 多租户身份对接

> 更新：2026-03-16
> 场景：你做了一个 SaaS 服务，要部署给多个企业客户。每个客户有自己的用户管理系统（Okta、Azure AD、Google Workspace...），你的服务怎么对接？

---

## 一、问题本质

你的 SaaS 产品有自己的用户系统。客户 A 用 Okta，客户 B 用 Azure AD，客户 C 没有 SSO 只想用邮箱注册。你需要：

1. **客户员工用自己公司账号登录**（不用再注册一套新账号）
2. **你的系统知道这个人属于哪个客户**（数据隔离）
3. **不同客户可以有不同的权限配置**（角色、配额）
4. **加新客户时不改代码**

这是 B2B SaaS 的经典问题。2026 年有成熟的解法。

---

## 二、核心概念

### BYOI — Bring Your Own Identity

"自带身份"。客户不在你的系统里创建账号，而是用自己公司的 IdP（Identity Provider）登录。

### Identity Broker（身份中间人）

你的应用不直接对接每个客户的 IdP，而是通过一个中间层统一翻译：

```
客户 A 的 Okta       ──┐
                        ├──→  Identity Broker  ──→  你的应用
客户 B 的 Azure AD   ──┘      (Logto/Auth0/       只信任 Broker
                                Keycloak)          签发的 JWT
散客（邮箱注册）     ──────────────┘
```

**核心思想**：你的应用只对接一个 IdP（Broker），Broker 负责对接 N 个客户的 IdP。

### Organization（组织）

在 Broker 中，每个客户是一个 Organization。用户归属到 Organization 下。JWT 中携带 `org_id`，你的应用据此做数据隔离。

### Enterprise SSO Connector

Broker 侧的配置，每个 Organization 可以绑定一个 SSO Connector（SAML 或 OIDC），指向客户自己的 IdP。

### JIT Provisioning（即时配置）

客户员工第一次登录时，Broker 自动创建用户账号并加入对应 Organization。不需要手动导入用户。

---

## 三、为什么不直接对接多个 IdP？

| 方案 | 工作量 | 维护成本 | 安全风险 |
|------|--------|---------|---------|
| 应用直接对接 N 个 IdP | 每个客户改代码/配置 | N 套 JWKS 缓存、证书轮换 | N 个攻击面 |
| **Identity Broker** | 应用只对接 1 个 | Broker 统一管理 | 1 个攻击面 |

直接对接的问题：
- **协议差异**：有的客户用 SAML，有的用 OIDC，有的用 LDAP。你要全部支持。
- **证书管理**：每个 SAML IdP 有自己的签名证书，会过期需要轮换。
- **代码耦合**：每加一个客户的 IdP，要改应用配置甚至代码。

Broker 模式下，这些复杂度全部封装在 Broker 里。你的应用只看标准 JWT。

---

## 四、架构方案

### 整体流程

```
1. 客户员工打开你的应用
2. 应用重定向到 Identity Broker 的登录页
3. Broker 根据邮箱域名识别客户（@acme.com → Organization Acme）
4. Broker 重定向到客户的 IdP（Okta/Azure AD）
5. 员工在自己公司的 IdP 认证
6. IdP 回传认证结果给 Broker
7. Broker 签发自己的 JWT（包含 org_id、roles）
8. 应用收到 JWT，提取 org_id 做业务隔离
```

### JWT Token 结构

```jsonc
{
  "sub": "user-abc123",              // Broker 内部用户 ID（稳定）
  "iss": "https://auth.example.com", // Broker 的 issuer
  "aud": "your-app",                 // 你的应用标识
  "org_id": "org-acme",              // 所属 Organization
  "org_roles": ["member"],           // 组织内角色
  "email": "zhang@acme.com",
  "exp": 1710003600
}
```

**你的应用只需要**：
1. 用 JWKS 验证 JWT 签名
2. 提取 `sub`（用户）、`org_id`（租户）、`org_roles`（角色）
3. 据此做数据隔离和权限控制

### 数据隔离层

```
请求进入 → JWT 验证 → 提取 org_id
  ↓
所有 DB 查询自动加 WHERE org_id = :orgId
  - SELECT * FROM sessions WHERE org_id = 'org-acme'
  - SELECT SUM(tokens) FROM usage WHERE org_id = 'org-acme'
  - ...

不同 Organization 的数据物理上在同一个 DB，逻辑上完全隔离。
```

### 配额与计量

```
两层配额检查：

Organization 级: 客户 A 的月度 token 总额度
  └── User 级: 客户 A 的张三的个人配额

用量统计按 org_id 聚合，账单按 Organization 出。
```

---

## 五、Identity Broker 选型

### 开源方案

| 产品 | 特点 | 适合 |
|------|------|------|
| **Logto** | 现代 UI，Organization 支持好，SAML/OIDC SSO，JIT Provisioning | 中小团队，快速上手 |
| **Keycloak** | 功能最全，Red Hat 支持，企业级 | 大团队，有运维能力 |
| **Authentik** | Python 实现，简洁，支持 SCIM | 中小团队 |

### 商用方案

| 产品 | 特点 | 定价 |
|------|------|------|
| **Auth0** | 功能最全，Organizations 支持好 | 按 MAU，企业贵 |
| **WorkOS** | 专注 B2B SSO，开发者体验好 | 按 SSO 连接数 |
| **Clerk** | 开发者体验极佳，内置 UI 组件 | 按 MAU |
| **Descope** | 无代码/低代码 | 按 MAU |

### 选型建议

```
预算有限 + 想控制数据    → Logto（开源，自托管）
企业客户多 + 不想自运维  → Auth0 或 WorkOS
只需要 SSO 不需要用户管理 → WorkOS（最专注）
```

---

## 六、实施路径

### 阶段 1: 基础多租户（3 天）

```
你的应用需要改的：
  1. JWT 解析加 org_id 提取           ← 几行代码
  2. DB schema 关键表加 org_id 字段    ← 一次 migration
  3. Repository 查询统一加 org_id 过滤 ← 一天工作量
  4. 配额按 org_id 统计               ← 半天

Broker 侧：
  5. 创建 Organization                ← Console 点几下
  6. 配置第一个 Enterprise SSO         ← Console + 客户提供 IdP 信息
```

**完成后**：加新客户只需在 Broker Console 操作，应用零改动。

### 阶段 2: 自助化（1-2 周）

```
  7. Admin API — Organization CRUD
  8. 客户自助配置 SSO（提供 IdP 信息，自动配置）
  9. 按 Organization 的 billing dashboard
```

### 阶段 3: 高级特性（按需）

```
  10. SCIM 同步（客户 IdP 的用户变更自动同步）
  11. 每个 Organization 独立的 Guardrails 配置
  12. 数据驻留（某些客户要求数据存特定区域）
  13. Organization 级别的 audit log 导出
```

---

## 七、常见陷阱

### 1. 混淆 User 和 Tenant

```
❌ 一个用户只能属于一个 Organization
✅ 一个用户可能属于多个 Organization（咨询公司的人服务多个客户）
```

需要在请求级别确定"当前上下文是哪个 Organization"，不是在用户级别。

### 2. 自建身份系统

Microsoft 架构指南的原话：

> "Building a modern identity platform is complex. It requires support for a range of protocols and standards, and an incorrect implementation can introduce security vulnerabilities."

不要自己实现 SAML 解析、密码存储、MFA。用成熟的 Broker。

### 3. Token 跨租户泄露

```
❌ 共享 OIDC client，不同 Organization 的 token 互通
✅ 每个 Organization 的 token 在验证时检查 org_id 匹配

安全检查清单：
  - API 接口是否验证了 org_id 和当前用户的归属关系？
  - 用户能否通过修改请求参数访问其他 Organization 的数据？
  - 管理员 API 是否限制了只能管理自己 Organization 的用户？
```

### 4. 先做了用户系统再加多租户

事后加 `org_id` 字段 = 全表 migration + 所有查询审查。如果一开始就知道要多租户，**第一天就加 org_id**，即使初始值是 `default`。

---

## 八、与前面几篇文章的关系

```
生产级 Agent 工程清单.md
  └── "安全与 Guardrails" 章节提到了认证
      └── 本文展开：多客户场景下的身份对接具体怎么做

云端 Agent 服务架构.md
  └── "多租户隔离" 章节提到了 org_id
      └── 本文展开：org_id 从哪来？怎么对接客户 IdP？
```

---

## 参考资料

- [Logto: Multi-tenant Architecture for B2B](https://docs.logto.io/introduction/plan-your-architecture/b2b)
- [Logto: Organizations for SaaS](https://logto.io/products/organizations)
- [Microsoft: Identity Approaches for Multitenant Solutions](https://learn.microsoft.com/en-us/azure/architecture/guide/multitenant/approaches/identity)
- [WorkOS: The Developer's Guide to B2B User Management](https://workos.com/blog/user-management-for-b2b-saas)
- [ScaleKit: SaaS Authentication Guide](https://www.scalekit.com/saas-authentication)
- [Auth0: B2B Multi-Tenant SaaS](https://auth0.com/b2b-saas)
- [Datawiza: How to Add B2B SSO (BYOI)](https://www.datawiza.com/blog/industry/how-to-add-b2b-sso-byoi-without-a-ciam-vendor/)
