# 🧪 TypeScript单元测试最佳实践指南 v1.0.0

> **CreateDate**: *2026-01-15*
>
> **UpdateDate**: *2026-01-15*

## 📋 目录
- [版本历史](#版本历史)
- [🚀 快速开始](#快速开始)
- [🎯 核心原则](#核心原则)
- [🏗️ 测试架构](#测试架构)
- [🧩 测试模式](#测试模式)
- [🔧 实用技巧](#实用技巧)
- [🚫 避免的陷阱](#避免的陷阱)
- [📈 测试度量](#测试度量)
- [🔍 代码示例](#代码示例)

---

## 📜 版本历史

| 版本  | 日期       | 变更说明                   | 负责人 |
| ----- | ---------- | -------------------------- | ------ |
| 1.0.0 | 2026-01-15 | 初始版本，包含核心最佳实践 |        |

---

## 🚀 快速开始

### 1. 📦 安装基础依赖
```bash
npm install --save-dev jest typescript ts-jest @types/jest
npm install --save-dev @testing-library/react @testing-library/dom  # React项目
npm install --save-dev sinon chai mocha  # 备选工具链
```

### 2. ⚙️ 基础配置 (jest.config.js)
```javascript
module.exports = {
  preset: 'ts-jest',
  testEnvironment: 'node',
  roots: ['<rootDir>/src'],
  testMatch: ['**/?(*.)+(spec|test).+(ts|tsx|js)'],
  collectCoverageFrom: [
    'src/**/*.{ts,tsx}',
    '!src/**/*.d.ts',
    '!src/**/*.stories.{ts,tsx}',
    '!src/**/index.ts'
  ],
  setupFilesAfterEnv: ['<rootDir>/src/setupTests.ts']
};
```

---

## 🎯 核心原则

### 🏆 FIRST 原则
| 原则                       | 描述               | 示例                     |
| -------------------------- | ------------------ | ------------------------ |
| **F**ast 快速              | 测试应该快速执行   | `< 100ms/测试用例`       |
| **I**solated 隔离          | 测试之间不相互依赖 | 使用`beforeEach`清理状态 |
| **R**epeatable 可重复      | 在任何环境都能运行 | 避免依赖外部API          |
| **S**elf-validating 自验证 | 自动判断结果       | 明确的`expect`断言       |
| **T**imely 及时            | 与代码同步编写     | TDD或紧接开发            |

### 📐 测试金字塔
```
        ⛰️  少量 E2E 测试 (5-10%)
      🏢🏢   适量集成测试 (10-20%)
   🧱🧱🧱🧱  大量单元测试 (70-80%)
```
- **🧱 单元测试**: 测试单个函数/类
- **🏢 集成测试**: 测试模块间交互  
- **⛰️ E2E测试**: 测试完整用户流程

---

## 🏗️ 测试架构

### 📁 文件结构
```
src/
├── components/
│   ├── Button/
│   │   ├── Button.tsx
│   │   ├── Button.test.tsx      # 测试文件与源码相邻 ✅
│   │   └── Button.stories.tsx
├── services/
│   ├── UserService.ts
│   └── UserService.test.ts
├── utils/
│   ├── formatters.ts
│   └── formatters.test.ts
└── __mocks__/                   # 全局mock目录
    ├── axios.ts
    └── fileMock.js
```

### 🏷️ 命名规范
```typescript
// ✅ 推荐命名
describe('UserService', () => {           // 测试套件：测试的类名
  describe('createUser()', () => {        // 分组：测试的方法名
    it('should create user with valid email', () => { ... })
    it('should throw error when email is invalid', () => { ... })
  })
})

// ❌ 避免的命名
describe('test user service', () => {     // 太模糊
  it('test 1', () => { ... })            // 无意义名称
})
```

---

## 🧩 测试模式

### 1. ✅ AAA模式 (Arrange-Act-Assert)
```typescript
describe('Calculator', () => {
  it('should add two numbers correctly', () => {
    // Arrange - 准备阶段 🛠️
    const calculator = new Calculator();
    const a = 5;
    const b = 3;
    
    // Act - 执行阶段 🎬
    const result = calculator.add(a, b);
    
    // Assert - 断言阶段 ✅
    expect(result).toBe(8);
    expect(calculator.lastResult).toBe(8); // 可多个断言
  });
});
```

### 2. 🎭 测试替身 (Test Doubles)
```typescript
// ✅ 正确使用mock
interface IEmailService {
  send(email: string): Promise<void>;
}

describe('UserNotifier', () => {
  let emailService: jest.Mocked<IEmailService>;
  let notifier: UserNotifier;
  
  beforeEach(() => {
    // 🛡️ 创建mock实例
    emailService = {
      send: jest.fn().mockResolvedValue(undefined)
    };
    notifier = new UserNotifier(emailService);
  });
  
  it('should send welcome email to new user', async () => {
    // Arrange
    const user = { email: 'test@example.com', name: 'John' };
    
    // Act
    await notifier.sendWelcomeEmail(user);
    
    // Assert
    expect(emailService.send).toHaveBeenCalledTimes(1);
    expect(emailService.send).toHaveBeenCalledWith(
      expect.objectContaining({
        to: user.email,
        subject: expect.stringContaining('Welcome')
      })
    );
  });
});
```

### 3. 🔄 异步测试模式
```typescript
describe('UserAPI', () => {
  // ✅ 使用async/await
  it('should fetch user by id', async () => {
    const user = await UserAPI.getUserById('123');
    expect(user.id).toBe('123');
  });
  
  // ✅ 测试Promise拒绝
  it('should reject when user not found', async () => {
    await expect(UserAPI.getUserById('invalid'))
      .rejects
      .toThrow('User not found');
  });
  
  // ✅ 测试回调函数
  it('should call callback with data', (done) => {
    UserAPI.getUser('123', (error, data) => {
      expect(error).toBeNull();
      expect(data).toBeDefined();
      done(); // ⚠️ 必须调用done()
    });
  });
});
```

---

## 🔧 实用技巧

### 🎪 测试数据管理
```typescript
// ✅ 使用工厂函数创建测试数据
const createTestUser = (overrides = {}) => ({
  id: 'user-123',
  name: 'Test User',
  email: 'test@example.com',
  createdAt: new Date('2023-01-01'),
  ...overrides  // 🎯 允许覆盖特定字段
});

describe('UserService', () => {
  it('should update user name', () => {
    // 🎨 清晰的测试数据
    const user = createTestUser({ name: 'Old Name' });
    const service = new UserService();
    
    const updated = service.updateUserName(user, 'New Name');
    
    expect(updated.name).toBe('New Name');
    expect(updated.id).toBe(user.id); // 🎯 确保其他属性不变
  });
});
```

### 🧹 清理与重置
```typescript
describe('DatabaseService', () => {
  let db: DatabaseService;
  
  beforeEach(() => {
    // 🧼 每个测试前清理
    db = new DatabaseService();
    db.connect();
  });
  
  afterEach(() => {
    // 🗑️ 每个测试后清理
    db.disconnect();
    jest.clearAllMocks(); // 🎭 清理所有mocks
  });
  
  afterAll(() => {
    // 🏁 所有测试后清理
    DatabaseService.cleanup();
  });
});
```

### 🎯 精准断言
```typescript
// ✅ 推荐断言方式
expect(result).toBe(expected);           // 严格相等
expect(result).toEqual(expected);        // 深度相等
expect(result).toBeDefined();            // 非undefined
expect(result).toBeTruthy();             // 真值
expect(result).toBeNull();               // null
expect(result).toBeInstanceOf(User);     // 类型检查
expect(result).toHaveLength(3);          // 数组长度
expect(result).toContain('item');        // 包含元素
expect(result).toMatch(/regex/);         // 正则匹配
expect(result).toMatchObject({           // 部分匹配
  id: expect.any(String),
  name: expect.stringContaining('John')
});

// ❌ 避免模糊断言
expect(!!result).toBe(true);            // 不必要转换
expect(typeof result).toBe('object');   // 类型检查不精确
```

---

## 🚫 避免的陷阱

### ❌ 反模式：过度mock
```typescript
// ❌ 过度mock，测试变成实现细节的验证
it('should call method A then B then C', () => {
  const mock = {
    a: jest.fn(),
    b: jest.fn(),
    c: jest.fn()
  };
  
  service.doSomething(mock);
  
  expect(mock.a).toHaveBeenCalledBefore(mock.b); // 太脆弱！
  expect(mock.b).toHaveBeenCalledBefore(mock.c);
});

// ✅ 测试行为而非实现
it('should process data correctly', () => {
  const result = service.process(input);
  
  expect(result).toEqual(expectedOutput); // 测试结果！
});
```

### ❌ 反模式：测试私有方法
```typescript
// ❌ 不要测试私有方法
class UserService {
  private validateEmail(email: string): boolean {
    // 私有方法
  }
}

// 错误：通过特殊手段测试私有方法
it('should validate email', () => {
  const service = new UserService();
  const result = (service as any).validateEmail('test@example.com'); // 🚫
  
  expect(result).toBe(true);
});

// ✅ 通过公共API测试
it('should accept valid email', () => {
  const service = new UserService();
  const user = service.createUser('test@example.com'); // 公共方法
  
  expect(user).toBeDefined();
});
```

---

## 📈 测试度量

### 📊 覆盖率目标
```yaml
语句覆盖率: > 80%   # 基本要求
分支覆盖率: > 75%   # 重要决策点
函数覆盖率: > 85%   # 方法覆盖
行覆盖率:    > 80%   # 代码行覆盖
```

### 📋 覆盖率配置 (jest.config.js)
```javascript
coverageThreshold: {
  global: {
    branches: 75,
    functions: 85,
    lines: 80,
    statements: 80
  },
  './src/services/': {  // 关键模块更高要求
    branches: 90,
    functions: 95,
    lines: 90,
    statements: 90
  }
}
```

### 🔍 覆盖率盲点检查
```bash
# 生成详细报告
npm test -- --coverage --collectCoverageFrom="src/**/*.ts"

# 查看未覆盖代码
open coverage/lcov-report/index.html
```

重点关注：
- 🚨 `if/else` 分支
- 🚨 `try/catch` 块
- 🚨 边界条件
- 🚨 错误处理路径

---

## 🔍 代码示例

### 🎨 React组件测试
```typescript
import React from 'react';
import { render, screen, fireEvent } from '@testing-library/react';
import { Button } from './Button';

describe('Button Component', () => {
  it('should render with correct text', () => {
    // 🎯 渲染组件
    render(<Button label="Click me" />);
    
    // 🎯 查询元素
    const button = screen.getByRole('button', { name: /click me/i });
    
    // 🎯 断言
    expect(button).toBeInTheDocument();
    expect(button).toBeEnabled();
  });
  
  it('should call onClick when clicked', () => {
    // 🎭 创建mock回调
    const handleClick = jest.fn();
    
    render(<Button label="Click" onClick={handleClick} />);
    
    // 🎬 模拟用户交互
    fireEvent.click(screen.getByText('Click'));
    
    // ✅ 验证行为
    expect(handleClick).toHaveBeenCalledTimes(1);
  });
  
  it('should show loading state', () => {
    render(<Button label="Submit" loading={true} />);
    
    // 🔍 多种查询方式
    expect(screen.getByText('Loading...')).toBeInTheDocument();
    expect(screen.getByRole('button')).toBeDisabled();
    expect(screen.queryByText('Submit')).toBeNull(); // 确保文本已替换
  });
});
```

### 🗄️ API服务测试
```typescript
import axios from 'axios';
import { UserAPI } from './UserAPI';

// 🎭 Mock整个模块
jest.mock('axios');
const mockedAxios = axios as jest.Mocked<typeof axios>;

describe('UserAPI', () => {
  beforeEach(() => {
    // 🧹 清理mock历史
    jest.clearAllMocks();
  });
  
  describe('getUser', () => {
    it('should return user data on success', async () => {
      // 🎨 准备mock响应
      const mockUser = { id: '123', name: 'John' };
      mockedAxios.get.mockResolvedValue({ data: mockUser });
      
      // 🎬 执行测试
      const user = await UserAPI.getUser('123');
      
      // ✅ 验证结果
      expect(user).toEqual(mockUser);
      expect(mockedAxios.get).toHaveBeenCalledWith(
        '/api/users/123',
        expect.any(Object)
      );
    });
    
    it('should throw error on 404', async () => {
      // 🎭 Mock错误响应
      mockedAxios.get.mockRejectedValue({
        response: { status: 404, data: { message: 'Not found' } }
      });
      
      // 🎯 验证错误被抛出
      await expect(UserAPI.getUser('invalid'))
        .rejects
        .toThrow('User not found');
    });
  });
  
  describe('createUser', () => {
    it('should send POST request with user data', async () => {
      const userData = { name: 'Jane', email: 'jane@example.com' };
      mockedAxios.post.mockResolvedValue({ data: { id: '456', ...userData } });
      
      const result = await UserAPI.createUser(userData);
      
      expect(mockedAxios.post).toHaveBeenCalledWith(
        '/api/users',
        userData,
        expect.objectContaining({
          headers: { 'Content-Type': 'application/json' }
        })
      );
      expect(result.id).toBe('456');
    });
  });
});
```

### 🧠 复杂业务逻辑测试
```typescript
describe('OrderProcessor', () => {
  let processor: OrderProcessor;
  let mockPayment: jest.Mocked<IPaymentService>;
  let mockInventory: jest.Mocked<IInventoryService>;
  
  beforeEach(() => {
    // 🏗️ 构建复杂的测试依赖
    mockPayment = {
      charge: jest.fn().mockResolvedValue({ success: true, transactionId: 'txn-123' }),
      refund: jest.fn()
    };
    
    mockInventory = {
      checkStock: jest.fn().mockResolvedValue(true),
      reserveItem: jest.fn().mockResolvedValue('reservation-456'),
      releaseItem: jest.fn()
    };
    
    processor = new OrderProcessor(mockPayment, mockInventory);
  });
  
  describe('processOrder', () => {
    it('should complete order successfully', async () => {
      // 🎨 复杂测试场景
      const order = {
        id: 'order-789',
        items: [
          { productId: 'prod-1', quantity: 2 },
          { productId: 'prod-2', quantity: 1 }
        ],
        total: 99.99,
        customer: { id: 'cust-123', email: 'test@example.com' }
      };
      
      const result = await processor.processOrder(order);
      
      // 🎯 验证完整流程
      expect(result.success).toBe(true);
      expect(result.orderId).toBe(order.id);
      expect(result.transactionId).toBe('txn-123');
      
      // 🔍 验证关键交互
      expect(mockInventory.checkStock).toHaveBeenCalledWith('prod-1', 2);
      expect(mockInventory.reserveItem).toHaveBeenCalledTimes(2);
      expect(mockPayment.charge).toHaveBeenCalledWith(
        order.total,
        order.customer.email
      );
    });
    
    it('should rollback on payment failure', async () => {
      // 🎭 模拟支付失败
      mockPayment.charge.mockRejectedValue(new Error('Insufficient funds'));
      
      const order = { id: 'order-999', items: [], total: 50, customer: { email: 'test@example.com' } };
      
      await expect(processor.processOrder(order)).rejects.toThrow('Payment failed');
      
      // 🎯 验证回滚逻辑
      expect(mockInventory.releaseItem).toHaveBeenCalled(); // 库存应释放
      expect(mockPayment.refund).not.toHaveBeenCalled(); // 未成功不应退款
    });
    
    // 📝 参数化测试
    const invalidOrders = [
      { order: { items: [], total: 50 }, reason: 'empty items' },
      { order: { items: [{ productId: 'prod-1', quantity: 0 }], total: 0 }, reason: 'zero quantity' },
      { order: { items: [{ productId: 'prod-1', quantity: -1 }], total: -10 }, reason: 'negative values' }
    ];
    
    test.each(invalidOrders)('should reject invalid order: $reason', async ({ order }) => {
      await expect(processor.processOrder(order as any))
        .rejects
        .toThrow('Invalid order');
    });
  });
});
```

---

## 🎉 总结 Checklist

### 每次提交前检查：
- [ ] ✅ 所有测试通过
- [ ] 🧪 新增代码有对应测试
- [ ] 📊 覆盖率达标
- [ ] 🚫 没有跳过的测试
- [ ] 🔄 异步测试正确处理
- [ ] 🎭 Mock使用恰当
- [ ] 📝 测试名称清晰描述行为
- [ ] 🧹 测试数据已清理

### 代码审查检查：
- [ ] 🎯 测试行为而非实现
- [ ] 🏗️ 使用AAA模式
- [ ] 🎨 测试数据可读
- [ ] 🚀 测试快速执行
- [ ] 🎭 Mock适度使用
- [ ] 📖 错误信息清晰

---

## 🔮 未来更新计划

### 🚧 待添加内容
1. **🔄 快照测试最佳实践**
2. **🌐 E2E测试集成指南**
3. **⚡ 性能测试模式**
4. **🔒 安全测试考量**
5. **🧪 测试重构策略**

---

> **💡 提示**: 最佳实践需要团队共识！定期回顾和更新此文档，适应项目演进和团队成长。

---
*📚 本指南将持续更新，欢迎贡献你的经验！*