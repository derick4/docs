# 如何添加 API 接口文档

本文档详细说明如何在 Mintlify 文档系统中添加新的 API 接口文档。

## 目录

- [添加 API 接口文档的步骤](#添加-api-接口文档的步骤)
- [完整示例](#完整示例)
- [常见配置选项](#常见配置选项)
- [高级功能](#高级功能)
- [测试文档](#测试文档)

## 添加 API 接口文档的步骤

### 步骤 1: 在 OpenAPI 规范中定义 API 端点

编辑 `api-reference/openapi.json` 文件，在 `paths` 对象中添加新的端点。

**示例：添加一个产品管理接口 /products**

```json
{
  "paths": {
    "/products": {
      "get": {
        "description": "获取产品列表",
        "parameters": [
          {
            "name": "category",
            "in": "query",
            "description": "按类别筛选",
            "schema": {
              "type": "string"
            }
          }
        ],
        "responses": {
          "200": {
            "description": "成功返回产品列表",
            "content": {
              "application/json": {
                "schema": {
                  "type": "array",
                  "items": {
                    "$ref": "#/components/schemas/Product"
                  }
                }
              }
            }
          }
        }
      },
      "post": {
        "description": "创建新产品",
        "requestBody": {
          "required": true,
          "content": {
            "application/json": {
              "schema": {
                "$ref": "#/components/schemas/NewProduct"
              }
            }
          }
        },
        "responses": {
          "201": {
            "description": "产品创建成功",
            "content": {
              "application/json": {
                "schema": {
                  "$ref": "#/components/schemas/Product"
                }
              }
            }
          }
        }
      }
    }
  }
}
```

### 步骤 2: 定义数据模型（Schemas）

在 `openapi.json` 的 `components.schemas` 中定义数据结构。

```json
{
  "components": {
    "schemas": {
      "Product": {
        "required": ["id", "name", "price"],
        "type": "object",
        "properties": {
          "id": {
            "description": "产品ID",
            "type": "integer",
            "format": "int64"
          },
          "name": {
            "description": "产品名称",
            "type": "string"
          },
          "price": {
            "description": "价格",
            "type": "number",
            "format": "double"
          },
          "category": {
            "description": "产品类别",
            "type": "string"
          }
        }
      },
      "NewProduct": {
        "required": ["name", "price"],
        "type": "object",
        "properties": {
          "name": {
            "type": "string"
          },
          "price": {
            "type": "number"
          },
          "category": {
            "type": "string"
          }
        }
      }
    }
  }
}
```

### 步骤 3: 创建文档页面（MDX 文件）

在 `api-reference/endpoint/` 目录下创建对应的 `.mdx` 文件。

**文件：`api-reference/endpoint/products/list.mdx`**
```mdx
---
title: 'Get Products'
openapi: 'GET /products'
---

<API />
```

**文件：`api-reference/endpoint/products/create.mdx`**
```mdx
---
title: 'Create Product'
openapi: 'POST /products'
---

<API />
```

**关键点：**
- `title`: 页面标题，显示在导航和页面顶部
- `openapi`: 对应 openapi.json 中的方法和路径
- `<API />`: Mintlify 组件，自动生成交互式 API 界面

### 步骤 4: 更新导航配置

编辑 `docs.json`，在 navigation 中添加新的页面。

```json
{
  "navigation": {
    "tabs": [
      {
        "tab": "API reference",
        "groups": [
          {
            "group": "Product endpoints",
            "pages": [
              "api-reference/endpoint/products/list",
              "api-reference/endpoint/products/create"
            ]
          }
        ]
      }
    ]
  }
}
```

## 完整示例：添加一个订单接口

以下是一个完整的订单管理接口实现示例。

### OpenAPI 定义

```json
{
  "paths": {
    "/orders": {
      "get": {
        "description": "获取订单列表",
        "parameters": [
          {
            "name": "status",
            "in": "query",
            "description": "按订单状态筛选",
            "schema": {
              "type": "string",
              "enum": ["pending", "completed", "cancelled"]
            }
          },
          {
            "name": "limit",
            "in": "query",
            "description": "限制返回数量",
            "schema": {
              "type": "integer",
              "default": 20
            }
          }
        ],
        "responses": {
          "200": {
            "description": "订单列表",
            "content": {
              "application/json": {
                "schema": {
                  "type": "array",
                  "items": {"$ref": "#/components/schemas/Order"}
                }
              }
            }
          }
        }
      },
      "post": {
        "description": "创建新订单",
        "requestBody": {
          "required": true,
          "content": {
            "application/json": {
              "schema": {"$ref": "#/components/schemas/NewOrder"}
            }
          }
        },
        "responses": {
          "201": {
            "description": "订单创建成功",
            "content": {
              "application/json": {
                "schema": {"$ref": "#/components/schemas/Order"}
              }
            }
          }
        }
      }
    },
    "/orders/{id}": {
      "get": {
        "description": "获取单个订单详情",
        "parameters": [
          {
            "name": "id",
            "in": "path",
            "required": true,
            "description": "订单ID",
            "schema": {"type": "integer"}
          }
        ],
        "responses": {
          "200": {
            "description": "订单详情",
            "content": {
              "application/json": {
                "schema": {"$ref": "#/components/schemas/Order"}
              }
            }
          },
          "404": {
            "description": "订单未找到",
            "content": {
              "application/json": {
                "schema": {"$ref": "#/components/schemas/Error"}
              }
            }
          }
        }
      },
      "put": {
        "description": "更新订单",
        "parameters": [
          {
            "name": "id",
            "in": "path",
            "required": true,
            "schema": {"type": "integer"}
          }
        ],
        "requestBody": {
          "required": true,
          "content": {
            "application/json": {
              "schema": {"$ref": "#/components/schemas/UpdateOrder"}
            }
          }
        },
        "responses": {
          "200": {
            "description": "订单更新成功",
            "content": {
              "application/json": {
                "schema": {"$ref": "#/components/schemas/Order"}
              }
            }
          }
        }
      },
      "delete": {
        "description": "删除订单",
        "parameters": [
          {
            "name": "id",
            "in": "path",
            "required": true,
            "schema": {"type": "integer"}
          }
        ],
        "responses": {
          "204": {
            "description": "订单删除成功"
          },
          "404": {
            "description": "订单未找到"
          }
        }
      }
    }
  },
  "components": {
    "schemas": {
      "Order": {
        "type": "object",
        "required": ["id", "userId", "total", "status"],
        "properties": {
          "id": {
            "type": "integer",
            "description": "订单ID"
          },
          "userId": {
            "type": "integer",
            "description": "用户ID"
          },
          "total": {
            "type": "number",
            "format": "double",
            "description": "订单总金额"
          },
          "status": {
            "type": "string",
            "enum": ["pending", "completed", "cancelled"],
            "description": "订单状态"
          },
          "items": {
            "type": "array",
            "items": {"$ref": "#/components/schemas/OrderItem"},
            "description": "订单商品列表"
          },
          "createdAt": {
            "type": "string",
            "format": "date-time",
            "description": "创建时间"
          }
        }
      },
      "NewOrder": {
        "type": "object",
        "required": ["userId", "items"],
        "properties": {
          "userId": {
            "type": "integer"
          },
          "items": {
            "type": "array",
            "items": {"$ref": "#/components/schemas/OrderItem"}
          }
        }
      },
      "UpdateOrder": {
        "type": "object",
        "properties": {
          "status": {
            "type": "string",
            "enum": ["pending", "completed", "cancelled"]
          }
        }
      },
      "OrderItem": {
        "type": "object",
        "required": ["productId", "quantity", "price"],
        "properties": {
          "productId": {
            "type": "integer",
            "description": "商品ID"
          },
          "quantity": {
            "type": "integer",
            "description": "数量"
          },
          "price": {
            "type": "number",
            "format": "double",
            "description": "单价"
          }
        }
      }
    }
  }
}
```

### 创建文档页面

创建以下文件：

**`api-reference/endpoint/orders/list.mdx`**
```mdx
---
title: 'Get Orders'
openapi: 'GET /orders'
---

<API />
```

**`api-reference/endpoint/orders/get.mdx`**
```mdx
---
title: 'Get Order by ID'
openapi: 'GET /orders/{id}'
---

<API />
```

**`api-reference/endpoint/orders/create.mdx`**
```mdx
---
title: 'Create Order'
openapi: 'POST /orders'
---

<API />
```

**`api-reference/endpoint/orders/update.mdx`**
```mdx
---
title: 'Update Order'
openapi: 'PUT /orders/{id}'
---

<API />
```

**`api-reference/endpoint/orders/delete.mdx`**
```mdx
---
title: 'Delete Order'
openapi: 'DELETE /orders/{id}'
---

<API />
```

### 更新导航

在 `docs.json` 中添加：

```json
{
  "group": "Order endpoints",
  "pages": [
    "api-reference/endpoint/orders/list",
    "api-reference/endpoint/orders/get",
    "api-reference/endpoint/orders/create",
    "api-reference/endpoint/orders/update",
    "api-reference/endpoint/orders/delete"
  ]
}
```

## 常见配置选项

### 参数类型

#### 路径参数（Path Parameters）
```json
{
  "name": "id",
  "in": "path",
  "required": true,
  "description": "资源的唯一标识符",
  "schema": {
    "type": "integer",
    "format": "int64"
  }
}
```

#### 查询参数（Query Parameters）
```json
{
  "name": "page",
  "in": "query",
  "description": "页码",
  "schema": {
    "type": "integer",
    "default": 1,
    "minimum": 1
  }
}
```

#### Header 参数
```json
{
  "name": "X-API-Key",
  "in": "header",
  "required": true,
  "description": "API密钥",
  "schema": {
    "type": "string"
  }
}
```

### 请求体（Request Body）

```json
{
  "requestBody": {
    "required": true,
    "description": "创建用户的数据",
    "content": {
      "application/json": {
        "schema": {
          "$ref": "#/components/schemas/NewUser"
        }
      }
    }
  }
}
```

### 响应定义（Responses）

```json
{
  "responses": {
    "200": {
      "description": "请求成功",
      "content": {
        "application/json": {
          "schema": {
            "$ref": "#/components/schemas/User"
          }
        }
      }
    },
    "400": {
      "description": "请求参数错误",
      "content": {
        "application/json": {
          "schema": {
            "$ref": "#/components/schemas/Error"
          }
        }
      }
    },
    "401": {
      "description": "未授权"
    },
    "404": {
      "description": "资源未找到"
    },
    "500": {
      "description": "服务器内部错误"
    }
  }
}
```

### 数据类型

#### 基本类型
```json
{
  "type": "string"           // 字符串
  "type": "integer"          // 整数
  "type": "number"           // 数字（包括小数）
  "type": "boolean"          // 布尔值
  "type": "array"            // 数组
  "type": "object"           // 对象
}
```

#### 格式化类型
```json
{
  "type": "string",
  "format": "email"          // 邮箱
}
{
  "type": "string",
  "format": "date-time"      // 日期时间
}
{
  "type": "string",
  "format": "password"       // 密码
}
{
  "type": "integer",
  "format": "int64"          // 64位整数
}
{
  "type": "number",
  "format": "double"         // 双精度浮点数
}
```

#### 枚举类型
```json
{
  "type": "string",
  "enum": ["admin", "user", "guest"],
  "default": "user"
}
```

#### 数组类型
```json
{
  "type": "array",
  "items": {
    "$ref": "#/components/schemas/Item"
  }
}
```

### 验证规则

```json
{
  "type": "string",
  "minLength": 3,
  "maxLength": 50,
  "pattern": "^[a-zA-Z0-9]+$"
}

{
  "type": "integer",
  "minimum": 1,
  "maximum": 100
}

{
  "type": "array",
  "minItems": 1,
  "maxItems": 10
}
```

## 高级功能

### 1. 添加示例数据

```json
{
  "components": {
    "schemas": {
      "User": {
        "type": "object",
        "properties": {
          "name": {
            "type": "string",
            "example": "张三"
          },
          "email": {
            "type": "string",
            "format": "email",
            "example": "zhangsan@example.com"
          },
          "age": {
            "type": "integer",
            "example": 25
          }
        }
      }
    }
  }
}
```

### 2. 使用 allOf 组合 Schema

```json
{
  "NewUser": {
    "allOf": [
      {
        "$ref": "#/components/schemas/BaseUser"
      },
      {
        "type": "object",
        "required": ["id"],
        "properties": {
          "id": {
            "type": "integer"
          }
        }
      }
    ]
  }
}
```

### 3. 添加详细文档说明

在 MDX 文件中可以添加更多内容：

```mdx
---
title: 'Create User'
openapi: 'POST /users'
---

## 接口说明

此接口用于创建新用户账户。

### 功能特性

- 支持自动生成用户ID
- 邮箱自动验证
- 密码自动加密存储

## 注意事项

<Warning>
邮箱必须唯一，重复邮箱会返回 400 错误
</Warning>

<Info>
密码最小长度为 8 位，必须包含字母和数字
</Info>

- 默认角色为 'user'
- 创建后自动发送欢迎邮件

<API />

## 响应示例

成功创建用户后，将返回：

```json
{
  "id": 12345,
  "name": "John Doe",
  "email": "john@example.com",
  "role": "user",
  "createdAt": "2024-01-15T10:30:00Z"
}
```

## 代码示例

### JavaScript/Node.js

```javascript
const response = await fetch('http://api.example.com/users', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'Authorization': 'Bearer your-token-here'
  },
  body: JSON.stringify({
    name: 'John Doe',
    email: 'john@example.com',
    password: 'securepass123'
  })
})

const data = await response.json()
console.log(data)
```

### Python

```python
import requests

url = "http://api.example.com/users"
headers = {
    "Content-Type": "application/json",
    "Authorization": "Bearer your-token-here"
}
data = {
    "name": "John Doe",
    "email": "john@example.com",
    "password": "securepass123"
}

response = requests.post(url, json=data, headers=headers)
print(response.json())
```

### cURL

```bash
curl -X POST http://api.example.com/users \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer your-token-here" \
  -d '{
    "name": "John Doe",
    "email": "john@example.com",
    "password": "securepass123"
  }'
```

## 常见错误

| 状态码 | 说明 | 解决方案 |
|--------|------|----------|
| 400 | 邮箱已存在 | 使用不同的邮箱地址 |
| 400 | 密码格式不正确 | 确保密码至少8位且包含字母数字 |
| 401 | 未授权 | 检查 Authorization header |
| 500 | 服务器错误 | 联系技术支持 |
```

### 4. 添加认证配置

在 openapi.json 顶层添加安全配置：

```json
{
  "security": [
    {
      "bearerAuth": []
    }
  ],
  "components": {
    "securitySchemes": {
      "bearerAuth": {
        "type": "http",
        "scheme": "bearer",
        "bearerFormat": "JWT",
        "description": "输入 JWT Bearer token"
      },
      "apiKey": {
        "type": "apiKey",
        "in": "header",
        "name": "X-API-Key",
        "description": "API密钥认证"
      }
    }
  }
}
```

为特定端点设置不同的认证：

```json
{
  "paths": {
    "/public/info": {
      "get": {
        "security": [],  // 无需认证
        "description": "公开接口"
      }
    },
    "/admin/users": {
      "get": {
        "security": [
          {"bearerAuth": []},
          {"apiKey": []}
        ],
        "description": "需要管理员权限"
      }
    }
  }
}
```

## 测试文档

### 本地预览

1. 安装 Mintlify CLI（如果还没安装）：
```bash
npm install -g mintlify
```

2. 启动本地开发服务器：
```bash
mintlify dev
```

3. 访问文档：
打开浏览器访问 `http://localhost:3000`

### 测试 API 接口

在文档页面中，你会看到：

1. **参数输入区域**
   - 路径参数
   - 查询参数
   - 请求体

2. **认证区域**
   - Bearer Token 输入框
   - API Key 输入框

3. **操作按钮**
   - "Send Request" 按钮
   - "Copy as cURL" 按钮

4. **响应区域**
   - 状态码
   - 响应头
   - 响应体

### 验证清单

在发布文档前，请检查：

- [ ] OpenAPI 定义语法正确
- [ ] 所有 Schema 引用都存在
- [ ] MDX 文件的 openapi 字段与实际端点匹配
- [ ] 导航配置正确，页面路径准确
- [ ] 所有必需参数都标记为 required
- [ ] 响应状态码定义完整
- [ ] 示例数据准确且有意义
- [ ] 认证配置正确
- [ ] 文档描述清晰易懂

## 目录结构示例

```
docs/
├── api-reference/
│   ├── openapi.json                    # OpenAPI 规范文件
│   ├── introduction.mdx                # API 文档介绍
│   └── endpoint/
│       ├── users/
│       │   ├── list.mdx               # GET /users
│       │   ├── get.mdx                # GET /users/{id}
│       │   ├── create.mdx             # POST /users
│       │   ├── update.mdx             # PUT /users/{id}
│       │   └── delete.mdx             # DELETE /users/{id}
│       ├── products/
│       │   ├── list.mdx
│       │   ├── create.mdx
│       │   └── ...
│       └── orders/
│           ├── list.mdx
│           ├── create.mdx
│           └── ...
└── docs.json                           # 导航配置文件
```

## 常见问题

### Q: OpenAPI 定义更新后文档没有变化？

A: 尝试重启 Mintlify 开发服务器：
```bash
# 停止服务器（Ctrl+C）
# 重新启动
mintlify dev
```

### Q: 如何添加多个服务器地址？

A: 在 openapi.json 中配置：
```json
{
  "servers": [
    {
      "url": "https://api.example.com/v1",
      "description": "生产环境"
    },
    {
      "url": "https://staging-api.example.com/v1",
      "description": "测试环境"
    },
    {
      "url": "http://localhost:3000/v1",
      "description": "本地开发"
    }
  ]
}
```

### Q: 如何处理文件上传接口？

A: 使用 multipart/form-data：
```json
{
  "requestBody": {
    "content": {
      "multipart/form-data": {
        "schema": {
          "type": "object",
          "properties": {
            "file": {
              "type": "string",
              "format": "binary"
            },
            "description": {
              "type": "string"
            }
          }
        }
      }
    }
  }
}
```

### Q: 如何添加版本控制？

A: 可以在路径或服务器 URL 中包含版本：
```json
{
  "servers": [
    {"url": "https://api.example.com/v1"}
  ],
  "paths": {
    "/v2/users": {
      "get": {...}
    }
  }
}
```

## 参考资源

- [OpenAPI 3.1 规范](https://swagger.io/specification/)
- [Mintlify 官方文档](https://mintlify.com/docs)
- [JSON Schema 规范](https://json-schema.org/)

## 总结

添加 API 接口文档的核心步骤：

1. 在 `openapi.json` 中定义端点和数据模型
2. 创建对应的 MDX 文档页面
3. 在 `docs.json` 中配置导航
4. 本地测试验证
5. 发布文档

遵循这些步骤，你可以快速创建专业、交互式的 API 文档。
