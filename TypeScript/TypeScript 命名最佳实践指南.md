# 📘 TypeScript 命名最佳实践指南

> **CreateDate**: *2026-01-15*
>
> **UpdateDate**: *2026-01-15*

> 💡 **让代码自我表达的艺术**  
> 一份为团队打造的 TypeScript 命名风格指南，让命名成为提升代码质量的利器

## 🎯 核心理念

```typescript
// ❌ 糟糕的命名让代码难以理解
const d = [];          // 这是什么数据？
function p() {}        // 这是什么功能？
const f = true;        // 这是什么标志？

// ✅ 优秀的命名让代码自我解释
const userList: User[] = [];
function processPayment() {}
const isAuthenticated: boolean = true;
```

## 📚 目录

- [1️⃣ 基础原则](#1️⃣-基础原则)
- [2️⃣ 变量命名](#2️⃣-变量命名)
- [3️⃣ 函数命名](#3️⃣-函数命名)
- [4️⃣ 类与接口](#4️⃣-类与接口)
- [5️⃣ 类型别名与泛型](#5️⃣-类型别名与泛型)
- [6️⃣ 文件与目录](#6️⃣-文件与目录)
- [7️⃣ 特殊情况](#7️⃣-特殊情况)
- [8️⃣ 检查清单](#8️⃣-检查清单)

## 1️⃣ 基础原则

### 🧠 命名即沟通

```typescript
// ❌ 计算机思维
const arr = users.filter(u => u.active);

// ✅ 领域思维
const activeUsers = users.filter(user => user.isActive);
```

### 📝 四大黄金法则

1. **清晰度 > 简洁度** - 宁可长而清，不要短而迷
2. **一致性** - 整个项目保持统一的命名风格
3. **上下文感知** - 命名要反映所在领域
4. **避免歧义** - 一个名字只能有一个含义

## 2️⃣ 变量命名

### 🔤 基本变量

```typescript
// ❌ 模糊不清
let data: any;          // 什么数据？
let flag: boolean;      // 什么标志？
let obj: Record<string, any>; // 什么对象？

// ✅ 清晰具体
let userProfile: User;
let isModalOpen: boolean;
let apiConfig: ApiConfig;
```

### 📊 集合类型命名

```typescript
// 数组/列表 - 使用复数或List后缀
const users: User[] = [];           // ✅ 推荐
const userList: User[] = [];        // ✅ 明确列表
const userArray: User[] = [];       // ⚠️ 较少使用

// Map/字典 - 使用Map后缀或业务语义
const userByIdMap: Map<string, User> = new Map();   // ✅ 明确映射
const userLookup: Record<string, User> = {};        // ✅ 强调查找
const userCache: Map<string, User> = new Map();     // ✅ 业务语义

// Set集合 - 使用Set后缀
const activeUserIds: Set<string> = new Set();       // ✅
const uniqueEmails: Set<string> = new Set();        // ✅
```

### 🎨 布尔变量

```typescript
// 使用 is/has/can/should 等前缀
const isLoaded: boolean = true;      // ✅ 状态
const hasPermission: boolean = true; // ✅ 拥有关系
const canEdit: boolean = true;       // ✅ 能力
const shouldUpdate: boolean = true;  // ✅ 条件
const needsRefresh: boolean = true;  // ✅ 需求

// ❌ 避免混淆
const open: boolean = true;          // ❓ 是动作还是状态？
const valid: boolean = true;         // ⚠️ 缺少前缀，可读性稍差
```

### 🔢 数字变量

```typescript
// 包含单位或含义
const timeoutMs: number = 5000;      // ✅ 包含单位
const retryCount: number = 3;        // ✅ 包含计数
const maxItems: number = 100;        // ✅ 包含限制
const totalUsers: number = 1500;     // ✅ 包含总数

// ❌ 意义不明
const num: number = 5;               // ❌ 什么数字？
const val: number = 100;             // ❌ 什么值？
```

## 3️⃣ 函数命名

### 🎪 函数命名模式

```typescript
// 动作 + 对象
getUserById(id: string): User        // ✅ 获取用户
createOrder(order: Order): void      // ✅ 创建订单
validateEmail(email: string): boolean // ✅ 验证邮箱
calculateTotal(price: number): number // ✅ 计算总计

// 查询函数使用 is/has/can
isEmailValid(email: string): boolean // ✅ 查询状态
hasPermission(user: User): boolean   // ✅ 查询拥有
canUserEdit(user: User): boolean     // ✅ 查询能力
```

### 📦 不同职责的函数

```typescript
// 命令函数 - 执行操作
function sendEmail(to: string, subject: string): void { ... }

// 查询函数 - 返回信息
function getUserEmail(userId: string): string | null { ... }

// 验证函数 - 检查条件
function isValidPassword(password: string): boolean { ... }

// 转换函数 - 数据转换
function formatDate(timestamp: number): string { ... }

// 工厂函数 - 创建实例
function createUser(props: UserProps): User { ... }
```

### 🔄 异步函数

```typescript
// 明确表示异步操作
async function fetchUserData(userId: string): Promise<User> { ... }
async function loadUserPreferences(): Promise<Preferences> { ... }
async function saveDocument(content: string): Promise<void> { ... }

// 可选：使用 Async 后缀（当有同步版本时）
function getUserSync(id: string): User | null { ... }
async function getUserAsync(id: string): Promise<User> { ... }
```

## 4️⃣ 类与接口

### 🏗️ 类命名

```typescript
// 使用 PascalCase，名词开头
class UserService { ... }            // ✅ 服务类
class PaymentProcessor { ... }       // ✅ 处理器类
class EmailValidator { ... }         // ✅ 验证器类
class ConfigurationManager { ... }   // ✅ 管理器类

// 避免模糊的类名
class Helper { ... }                  // ❌ 太泛
class Util { ... }                    // ❌ 太泛
class MyClass { ... }                 // ❌ 无意义
```

### 🤝 接口命名

```typescript
// 描述形状的接口 - 使用名词或形容词
interface User { ... }                // ✅ 实体
interface Configurable { ... }        // ✅ 能力/特征
interface Serializable { ... }        // ✅ 能力/特征

// 服务契约接口 - 使用able后缀或Service
interface Renderable { ... }          // ✅ 可渲染
interface PaymentService { ... }      // ✅ 支付服务

// 避免 I 前缀（TypeScript风格指南不推荐）
interface IUser { ... }               // ⚠️ 不推荐（C#风格）
```

## 5️⃣ 类型别名与泛型

### 📐 类型别名

```typescript
// 描述性名称，使用 PascalCase
type UserId = string;                 // ✅ ID类型
type EmailAddress = string;           // ✅ 语义化字符串
type HttpMethod = 'GET' | 'POST' | 'PUT' | 'DELETE'; // ✅ 联合类型

// 对象类型别名
type UserProfile = {
  id: UserId;
  name: string;
  email: EmailAddress;
};

// 函数类型别名
type ClickHandler = (event: MouseEvent) => void;
type Comparator<T> = (a: T, b: T) => number;
```

### 🌀 泛型参数

```typescript
// 单个泛型 - 使用 T
function identity<T>(value: T): T { return value; }

// 多个泛型 - 使用 T, U, V 或描述性名称
function map<T, U>(arr: T[], fn: (item: T) => U): U[] { ... }

// 有意义的泛型参数
function getValue<Key extends string, Value>(
  map: Record<Key, Value>,
  key: Key
): Value { ... }

// 约束泛型 - 明确约束
interface HasId {
  id: string;
}

function findBy<T extends HasId>(items: T[], id: string): T | undefined {
  return items.find(item => item.id === id);
}
```

## 6️⃣ 文件与目录

### 📁 文件命名

```typescript
// kebab-case 用于普通文件
user-service.ts              // ✅ 服务文件
payment-processor.ts         // ✅ 处理器文件
types.ts                     // ✅ 类型定义文件
utils.ts                     // ✅ 工具文件

// PascalCase 用于类/组件文件
UserModel.ts                 // ✅ 模型类
PaymentForm.tsx              // ✅ React组件
Header.component.ts          // ✅ Angular组件

// 特定文件命名
user.service.ts              // ✅ 服务（Angular风格）
user.controller.ts           // ✅ 控制器
user.module.ts               // ✅ 模块
```

### 🗂️ 目录结构

```bash
src/
├── components/              # React/Vue组件
├── services/               # 业务服务
├── models/                 # 数据模型
├── types/                  # TypeScript类型定义
├── utils/                  # 工具函数
├── hooks/                  # React Hooks
├── contexts/               # React Contexts
└── styles/                 # 样式文件
```

## 7️⃣ 特殊情况

### ⚠️ 临时变量

```typescript
// 循环变量 - 使用单数形式
for (const user of users) { ... }           // ✅
for (let i = 0; i < users.length; i++) {    // ✅ 传统循环

// 临时变量 - 尽量描述意义
const tempUser = findUser(id);              // ⚠️ 可接受
const interimResult = calculate(...);       // ✅ 更好

// 回调参数 - 保持简洁但有意义
users.map(user => user.name);               // ✅
users.forEach((user, index) => { ... });    // ✅
```

### 🔍 测试文件命名

```typescript
// 测试文件与被测文件同名
user.service.ts              // 源文件
user.service.spec.ts         // ✅ 测试文件
user.service.test.ts         // ✅ 测试文件

// 测试描述清晰
describe('UserService', () => {
  it('should return user by id', () => { ... });      // ✅
  it('returns user when valid id provided', () => { ... }); // ✅ 更详细
});
```

### 🌍 国际化与常量

```typescript
// 常量使用 UPPER_SNAKE_CASE
const MAX_RETRY_COUNT = 3;
const DEFAULT_TIMEOUT_MS = 5000;
const API_BASE_URL = 'https://api.example.com';

// 枚举使用 PascalCase
enum UserRole {
  ADMIN = 'admin',
  EDITOR = 'editor',
  VIEWER = 'viewer'
}

// 配置对象
const APP_CONFIG = {
  apiEndpoint: 'https://api.example.com',
  enableDebug: process.env.NODE_ENV !== 'production',
  maxUploadSize: 10 * 1024 * 1024 // 10MB
} as const;  // 使用 as const 确保只读
```

## 8️⃣ 检查清单

### ✅ 命名自查清单

在提交代码前，问自己这些问题：

1. **清晰度检查** 📖
   - 这个名字是否清晰表达了它的目的？
   - 新人能否理解这个名字的含义？
   - 6个月后的我能否理解这个名字？

2. **简洁度检查** ✂️
   - 这个名字是否过于冗长？
   - 能否在不损失清晰度的前提下缩短？
   - 是否有不必要的重复信息？

3. **一致性检查** 🔄
   - 是否与项目中其他类似元素命名一致？
   - 是否遵循了团队约定的命名模式？
   - 是否使用了正确的命名规范（camelCase/PascalCase/kebab-case）？

4. **准确性检查** 🎯
   - 这个名字是否准确反映了它代表的概念？
   - 是否有更好的领域术语可以使用？
   - 是否避免了技术术语的误用？

5. **搜索友好检查** 🔍
   - 这个名字是否易于在代码库中搜索？
   - 是否有太常见或太独特的词？
   - IDE的自动补全能否很好地处理它？

### 🚨 常见反模式

```typescript
// ❌ 魔法数字/字符串
setTimeout(() => {}, 30000);           // ❌ 30000是什么？
if (status === 'A') { ... }           // ❌ 'A'是什么？

// ✅ 使用有意义的常量
const TIMEOUT_30_SECONDS = 30000;
const STATUS_ACTIVE = 'A';

// ❌ 模糊的缩写
const usrCnt = 10;                     // ❌ userCount?
const calcTot = 0;                     // ❌ calculateTotal?

// ✅ 完整或清晰缩写
const userCount = 10;                  // ✅
const total = 0;                       // ✅
const config = {};                     // ✅ config是公认缩写
```

## 🎉 总结

记住这些核心原则：

1. **命名是代码的UI** - 好的命名让代码易于理解和使用
2. **为读者命名** - 考虑下一个维护者的体验
3. **一致性是关键** - 团队约定比个人偏好更重要
4. **重构不羞耻** - 随时改进糟糕的命名

> 💭 **最后思考**  
> 优秀的命名不是一蹴而就的，而是通过不断审视和重构逐渐形成的。  
> 每当你写出一个名字时，想象你是6个月后的自己——你还能理解它吗？

---

**🌟 推荐工具：**
- ESLint 命名规则：`@typescript-eslint/naming-convention`
- 正则表达式检查命名模式
- 代码审查时特别关注命名质量

**📚 延伸阅读：**
- [TypeScript 官方编码指南](https://github.com/microsoft/TypeScript/wiki/Coding-guidelines)
- [Clean Code 命名章节](https://www.oreilly.com/library/view/clean-code/9780136083238/)
- [Google TypeScript 风格指南](https://google.github.io/styleguide/tsguide.html)

---

*让每个名字都成为代码质量的见证，而不是技术债的起点。* 🚀