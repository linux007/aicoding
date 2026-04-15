# 用户登录功能实现方案

## 需求分析

实现用户登录功能，支持两种登录方式：
1. 手机号登录
2. 邮箱登录

## 技术方案

根据 DigiCore 项目现有架构模式，登录功能实现需要以下组件：

### 1. 请求/响应结构定义 (`integrity/`)

需要定义以下 Schema（通过 `integrity.py` 生成）：

```go
// 登录请求
type R_LoginRequest struct {
    LoginType  string `json:"loginType"`  // "phone" 或 "email"
    Account    string `json:"account"`    // 手机号或邮箱
    Password   string `json:"password"`   // 密码（可选，若支持验证码则不同）
    VerifyCode string `json:"verifyCode"` // 验证码（可选）
}

// 登录响应
type R_LoginResponse struct {
    UserId    string `json:"userId"`
    Token     string `json:"token"`
    ExpiresAt int64  `json:"expiresAt"`
}
```

### 2. HTTP Handler (`controllers/http/login.go`)

遵循项目 Handler 模式：

```go
func Login(ctx *gin.Context) {
    var request *integrity.R_LoginRequest
    if err := ctx.ShouldBindJSON(&request); err != nil {
        zlog.Errorf(ctx, "params invalid:%s", err.Error())
        base.RenderJsonFail(ctx, components.ErrorInvalidParam)
        return
    }

    // 参数校验
    if request.LoginType != "phone" && request.LoginType != "email" {
        base.RenderJsonFail(ctx, components.ErrorInvalidParam)
        return
    }

    // 调用业务逻辑
    response, err := service.Login(ctx, request)
    if err != nil {
        base.RenderJsonFail(ctx, err)
        return
    }

    base.RenderJsonSucc(ctx, response)
}
```

### 3. Service 层 (`service/process/login.go`)

根据项目架构，可选两种实现方式：

**方案A：同步处理（适合简单登录验证）**

```go
func Login(ctx *gin.Context, req *integrity.R_LoginRequest) (*integrity.R_LoginResponse, error) {
    // 调用 Activity 进行登录验证
    result, err := data.LoginActivity(ctx, req)
    if err != nil {
        return nil, err
    }
    return result, nil
}
```

**方案B：Temporal Workflow（适合需要异步处理的场景）**

```go
func LoginWorkflow(ctx workflow.Context, req *integrity.R_LoginRequest) (*integrity.R_LoginResponse, error) {
    ao := workflow.ActivityOptions{
        StartToCloseTimeout: 30 * time.Second,
    }
    ctx = workflow.WithActivityOptions(ctx, ao)

    var result *integrity.R_LoginResponse
    err := workflow.ExecuteActivity(ctx, data.LoginActivity, req).Get(ctx, &result)
    return result, err
}
```

### 4. Activity 层 (`data/login.go`)

```go
func LoginActivity(ctx context.Context, req *integrity.R_LoginRequest) (*integrity.R_LoginResponse, error) {
    ginCtx := rivercli.NewGinContext(ctx)

    var user *models.User
    var err error

    switch req.LoginType {
    case "phone":
        user, err = models.GetUserByPhone(ginCtx, req.Account)
    case "email":
        user, err = models.GetUserByEmail(ginCtx, req.Account)
    }

    if err != nil {
        return nil, components.ErrorDbSelectFailed.Wrap(err)
    }

    if user == nil {
        return nil, components.ErrorUserNotFound
    }

    // 验证密码
    if !validatePassword(req.Password, user.Password) {
        return nil, components.ErrorInvalidPassword
    }

    // 生成 Token
    token, expiresAt := generateToken(user.ID)

    return &integrity.R_LoginResponse{
        UserId:    user.ID,
        Token:     token,
        ExpiresAt: expiresAt,
    }, nil
}
```

### 5. 数据模型 (`models/user/user.go`)

```go
type User struct {
    ID        int64  `gorm:"column:id;primaryKey;autoIncrement"`
    Phone     string `gorm:"column:phone"`
    Email     string `gorm:"column:email"`
    Password  string `gorm:"column:password"`
    Deleted   int64  `gorm:"column:deleted;default:0"`
    CreateTime int64 `gorm:"column:create_time;autoCreateTime"`
    UpdateTime int64 `gorm:"column:update_time;autoUpdateTime"`
}

func GetUserByPhone(ctx *gin.Context, phone string) (*User, error) {
    var user User
    err := getDB(ctx).Where("phone = ?", phone).First(&user).Error
    if err == gorm.ErrRecordNotFound {
        return nil, nil
    }
    return &user, err
}

func GetUserByEmail(ctx *gin.Context, email string) (*User, error) {
    var user User
    err := getDB(ctx).Where("email = ?", email).First(&user).Error
    if err == gorm.ErrRecordNotFound {
        return nil, nil
    }
    return &user, err
}
```

### 6. 路由注册 (`router/http.go`)

```go
api := router.Group("/digicore/api")
{
    // ... 现有路由
    api.POST("/login", http.Login)
}
```

### 7. 错误码定义 (`components/error.go`)

```go
var (
    // 用户相关错误 (6000001-6000099)
    ErrorUserNotFound      = base.ErrorRange(6000001, "user not found")
    ErrorInvalidPassword   = base.ErrorRange(6000002, "invalid password")
    ErrorAccountLocked     = base.ErrorRange(6000003, "account locked")
    ErrorTooManyAttempts   = base.ErrorRange(6000004, "too many login attempts")
)
```

## 实现步骤

1. **定义 Schema** - 在 `integrity/` 目录添加登录请求/响应的 JSON Schema
2. **生成类型** - 运行 `python3 integrity.py integrity/` 生成 Go 类型
3. **定义错误码** - 在 `components/error.go` 添加用户相关错误码
4. **创建模型** - 在 `models/user/` 目录创建 User 模型
5. **实现 Activity** - 在 `data/login.go` 实现登录验证逻辑
6. **实现 Service** - 在 `service/process/login.go` 实现业务编排
7. **实现 Handler** - 在 `controllers/http/login.go` 实现 HTTP 接口
8. **注册路由** - 在 `router/http.go` 注册登录端点
9. **编写测试** - 遵循 TDD，编写单元测试和集成测试

## 安全考虑

1. **密码加密** - 使用 bcrypt 存储密码哈希
2. **Token 安全** - 使用 JWT 并设置合理过期时间
3. **登录限制** - Redis 记录失败次数，防止暴力破解
4. **HTTPS** - 生产环境必须使用 HTTPS

## 测试用例

```go
func TestLogin(t *testing.T) {
    // 手机号登录成功
    // 邮箱登录成功
    // 无效手机号
    // 无效邮箱
    // 密码错误
    // 账号不存在
    // 参数缺失
}
```

## 文件清单

| 文件路径 | 用途 |
|---------|------|
| `integrity/login.json` | 登录 Schema 定义 |
| `components/error.go` | 错误码定义（新增） |
| `models/user/user.go` | User 模型 |
| `data/login.go` | 登录 Activity |
| `service/process/login.go` | 登录 Service |
| `controllers/http/login.go` | HTTP Handler |
| `router/http.go` | 路由注册 |
