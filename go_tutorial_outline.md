# Go 官方教程学习大纲

> 来源: https://go.dev/doc/tutorial/
> 整理日期: 2026-03-26

---

## 1. Getting Started — 入门

**目标**: 安装 Go，编写第一个 Hello World 程序

### 核心知识点
- 安装 Go (`go.dev/dl`)
- `go mod init` 创建模块，生成 `go.mod` 文件进行依赖追踪
- 模块路径 (module path) 的命名规范（如 `github.com/mymodule`）
- `go run .` 编译并运行程序
- `package main` 和 `func main()` 的含义（独立程序入口）

### 调用外部包
- 在 `pkg.go.dev` 上查找已发布的模块
- `import` 引入外部包（如 `rsc.io/quote`）
- `go mod tidy` 自动下载并整理依赖
- `go.sum` 文件用于模块认证

---

## 2. Create a Module — 创建模块

**目标**: 创建可供他人导入的库模块，并从另一个模块中调用

### 核心知识点
- 模块 (module) 与包 (package) 的关系：包被组织在模块中
- `go mod init example.com/greetings` 创建模块
- **导出名 (Exported name)**: 首字母大写的函数可被其他包调用
- `:=` 短变量声明（声明 + 初始化）
- `fmt.Sprintf` 格式化字符串，`%v` 占位符

### 多部分教程结构（7 个子话题）
1. 创建可复用模块
2. 从另一个模块调用代码
3. 返回并处理错误
4. 返回随机问候语
5. 为多人返回问候语 (map)
6. 添加测试
7. 编译并安装应用程序

---

## 3. Multi-module Workspaces — 多模块工作区

**目标**: 在同一工作区中同时开发多个模块

### 核心知识点
- `go work init ./hello` 创建 `go.work` 文件
- `go.work` 语法类似 `go.mod`，用 `use` 指令声明模块
- `go work use` 添加模块到工作区
- 工作区允许跨模块引用本地代码（替代 `replace` 指令）
- 工作区目录下可直接 `go run ./hello` 运行子模块

### 工作流
1. 创建工作区目录 → `go work init`
2. 创建/克隆多个模块 → `go work use` 添加
3. 在模块间共享本地修改
4. 完成后发布模块版本，更新 `go.mod` 中的依赖版本

### 相关命令
| 命令 | 用途 |
|------|------|
| `go work init` | 初始化工作区 |
| `go work use [-r] [dir]` | 添加模块到 `go.work` |
| `go work edit` | 编辑 `go.work`（类似 `go mod edit`）|
| `go work sync` | 同步工作区依赖 |

---

## 4. Accessing a Relational Database — 数据库访问

**目标**: 使用 `database/sql` 标准库访问关系型数据库 (MySQL)

### 核心知识点

#### 连接数据库
- 导入数据库驱动（如 `github.com/go-sql-driver/mysql`）
- `sql.Open("mysql", dsn)` 获取 `*sql.DB` 数据库句柄
- `db.Ping()` 验证连接可用性
- 使用 `mysql.Config` 结构体构建 DSN（比拼接字符串更清晰）
- 通过环境变量传递敏感信息 (`os.Getenv`)

#### 查询多行 — `db.Query`
- 返回 `*sql.Rows`，需循环遍历
- `rows.Scan(&field1, &field2, ...)` 将列值映射到结构体字段
- **必须** `defer rows.Close()` 释放资源
- 循环后检查 `rows.Err()` 捕获整体查询错误
- **参数化查询防 SQL 注入**: `db.Query("SELECT * FROM album WHERE artist = ?", name)`

#### 查询单行 — `db.QueryRow`
- 不直接返回错误，错误延迟到 `Scan` 时
- `sql.ErrNoRows` 表示无匹配结果

#### 插入数据 — `db.Exec`
- 用于不返回数据的 SQL 语句 (INSERT/UPDATE/DELETE)
- `result.LastInsertId()` 获取新插入行的 ID

### 完整代码结构
```
Album struct { ID, Title, Artist, Price }
main()            → 连接数据库
albumsByArtist()  → 按艺术家查询 (多行)
albumByID()       → 按ID查询 (单行)
addAlbum()        → 插入新专辑
```

---

## 5. Developing a RESTful API with Go and Gin — 用 Gin 构建 RESTful API

**目标**: 使用 Gin Web 框架构建 RESTful API

### API 端点设计
| 端点 | 方法 | 说明 |
|------|------|------|
| `/albums` | GET | 获取所有专辑 |
| `/albums` | POST | 添加新专辑 |
| `/albums/:id` | GET | 按 ID 获取专辑 |

### 核心知识点

#### 数据结构
- 结构体标签 `json:"artist"` 控制 JSON 序列化时的字段名
- 内存存储 (slice) 作为简单数据源

#### Gin 路由与处理器
- `gin.Default()` 初始化路由器（含中间件）
- `router.GET("/path", handler)` 注册路由
- `router.Run("localhost:8080")` 启动服务器

#### 处理器函数
- 参数为 `*gin.Context`（携带请求详情、验证、JSON 序列化等）
- `c.IndentedJSON(statusCode, data)` 返回格式化 JSON
- `c.BindJSON(&obj)` 解析请求体 JSON
- `c.Param("id")` 获取路径参数
- HTTP 状态码: `http.StatusOK` (200), `http.StatusCreated` (201), `http.StatusNotFound` (404)

#### 请求处理流程
1. **GET 全部**: 直接序列化 slice → JSON 返回
2. **POST 新增**: `BindJSON` 解析 → 追加到 slice → 返回 201
3. **GET 单个**: `Param` 取 ID → 遍历查找 → 返回结果或 404

---

## 6. Getting Started with Generics — 泛型入门

**目标**: 使用泛型编写适用于多种类型的函数

### 核心知识点

#### 从具体到泛型的演进路径
1. **非泛型**: 为每种类型写单独函数 (`SumInts`, `SumFloats`)
2. **泛型函数**: 用类型参数统一 (`SumIntsOrFloats[K comparable, V int64 | float64]`)
3. **类型推断**: 调用时省略类型参数 (`SumIntsOrFloats(ints)`)
4. **类型约束接口**: 提取约束为可复用接口 (`type Number interface { int64 | float64 }`)

#### 关键概念
| 概念 | 说明 |
|------|------|
| 类型参数 (Type Parameter) | 函数声明中的泛型占位符，如 `[K, V]` |
| 类型约束 (Type Constraint) | 限定类型参数可接受的类型集合 |
| `comparable` | 内建约束，支持 `==` 和 `!=` 操作的类型 |
| `\|` (联合类型) | 约束中用 `\|` 分隔多个允许的类型 |
| 类型推断 | 编译器从函数参数自动推导类型参数 |

#### 语法
```go
// 泛型函数声明
func SumIntsOrFloats[K comparable, V int64 | float64](m map[K]V) V { ... }

// 类型约束接口
type Number interface {
    int64 | float64
}

// 使用约束接口的泛型函数
func SumNumbers[K comparable, V Number](m map[K]V) V { ... }
```

---

## 7. Getting Started with Fuzzing — 模糊测试入门

**目标**: 使用 Go 内建的 fuzzing 功能发现边界情况和安全漏洞

### 核心知识点

#### Fuzz 测试基础
- Fuzz 测试函数签名: `func FuzzXxx(f *testing.F)`
- `f.Add(...)` 提供种子语料库 (seed corpus)
- `f.Fuzz(func(t *testing.T, input string) { ... })` 定义模糊测试逻辑
- 运行: `go test -fuzz=Fuzz`
- 限时运行: `go test -fuzz=Fuzz -fuzztime 30s`

#### 测试策略（无法预测输出时）
- 验证**属性**而非具体值:
  - 双重反转等于原值: `Reverse(Reverse(s)) == s`
  - 输出保持 UTF-8 有效性: `utf8.ValidString(rev)`

#### Bug 发现与修复流程 (实战案例: 字符串反转)

**Bug 1: 多字节字符破坏**
- 问题: 按 byte 反转破坏 UTF-8 多字节字符（如中文 `泃`）
- 修复: 改为按 `rune` 反转 → `r := []rune(s)`

**Bug 2: 无效 UTF-8 输入**
- 问题: 无效 UTF-8 byte (`\x91`) 转 `[]rune` 时被替换为 Unicode 替换字符
- 修复: 增加输入验证 → `if !utf8.ValidString(s) { return s, errors.New(...) }`

#### 关键文件结构
- 失败输入自动保存到 `testdata/fuzz/FuzzXxx/` 目录
- 后续 `go test` 会自动运行已保存的失败用例

---

## 8. Getting Started with govulncheck — 漏洞检查

**目标**: 使用 govulncheck 扫描并修复依赖中的已知漏洞

### 核心知识点
- 安装: `go install golang.org/x/vuln/cmd/govulncheck@latest`
- 运行: `govulncheck ./...`
- 输出分两级:
  - **Affecting**: 代码直接或间接调用了漏洞函数（需修复）
  - **Informational**: 依赖含漏洞但代码未调用相关函数（可评估后决定）

### 修复流程
1. 阅读漏洞描述，判断是否影响自身场景
2. 查看 "Fixed in" 版本
3. `go get module@version` 升级依赖
4. 再次运行 `govulncheck` 确认修复

---

## 9. Govulncheck in IDE (VS Code) — IDE 中的漏洞检查

**目标**: 在 VS Code 中直接使用 govulncheck

### 操作步骤
1. Cmd+Shift+P → "Go: Toggle Vulncheck" 扫描 go.mod 中的依赖
2. 悬停依赖查看漏洞详情
3. 灯泡图标 → "Quick Fix" → "run govulncheck to verify" 精确验证
4. Code Action → "Upgrade" 一键升级到修复版本

---

## 10. Go 并发编程 (Concurrency)

> 来源: A Tour of Go (go.dev/tour/concurrency) + Effective Go

Go 的并发设计哲学:
> **不要通过共享内存来通信；而是通过通信来共享内存。**
> (Do not communicate by sharing memory; instead, share memory by communicating.)

### 10.1 Goroutine

- `go f(x, y)` 启动一个新的 goroutine 执行 `f(x, y)`
- Goroutine 不是线程，而是轻量级的协程，由 Go 运行时调度
- 拥有独立的调用栈，按需动态伸缩
- 可以轻松创建成千上万个 goroutine
- 一个程序中可能只有一个 OS 线程，却有数千个 goroutine

### 10.2 Channel（通道）

通道是 goroutine 之间通信的管道，类型化且并发安全。

#### 基本操作
```go
ch := make(chan int)    // 创建无缓冲通道
ch <- v                // 发送 v 到通道
v := <-ch              // 从通道接收值
```

#### 关键特性
- **同步性**: 无缓冲通道的发送和接收会阻塞，直到对方就绪
- 天然实现 goroutine 间的同步，无需显式锁

#### 示例: 并行求和
```go
func sum(s []int, c chan int) {
    sum := 0
    for _, v := range s {
        sum += v
    }
    c <- sum
}

func main() {
    s := []int{7, 2, 8, -9, 4, 0}
    c := make(chan int)
    go sum(s[:len(s)/2], c)
    go sum(s[len(s)/2:], c)
    x, y := <-c, <-c
    fmt.Println(x, y, x+y)
}
```

### 10.3 Buffered Channel（缓冲通道）

```go
ch := make(chan int, 100)  // 缓冲大小为 100
```

- 仅当缓冲区满时发送阻塞，缓冲区空时接收阻塞
- 可用作计数信号量（限制并发数）

```go
var limit = make(chan int, 3)  // 最多 3 个并发
for _, w := range work {
    go func(w func()) {
        limit <- 1
        w()
        <-limit
    }(w)
}
```

### 10.4 Range 和 Close

```go
close(ch)                        // 发送方关闭通道
v, ok := <-ch                    // ok == false 表示通道已关闭
for v := range ch { ... }        // 循环接收直到通道关闭
```

- **只有发送方**应关闭通道，接收方不应关闭
- 关闭不是必须的，仅在需要通知接收方"没有更多数据"时使用（如终止 `range` 循环）

### 10.5 Select

`select` 让 goroutine 同时等待多个通道操作，哪个就绪就执行哪个。

```go
select {
case c <- x:
    // 发送成功时执行
case v := <-ch:
    // 接收成功时执行
default:
    // 所有 case 都未就绪时执行（非阻塞）
}
```

#### 示例: Fibonacci with select
```go
func fibonacci(c, quit chan int) {
    x, y := 0, 1
    for {
        select {
        case c <- x:
            x, y = y, x+y
        case <-quit:
            fmt.Println("quit")
            return
        }
    }
}
```

- 多个 case 同时就绪时**随机选择**一个执行
- `default` 分支实现非阻塞操作（如非阻塞发送/接收、轮询）

### 10.6 sync.Mutex（互斥锁）

当不需要通信、只需要互斥访问共享变量时，使用 `sync.Mutex`。

```go
type SafeCounter struct {
    mu sync.Mutex
    v  map[string]int
}

func (c *SafeCounter) Inc(key string) {
    c.mu.Lock()
    c.v[key]++
    c.mu.Unlock()
}

func (c *SafeCounter) Value(key string) int {
    c.mu.Lock()
    defer c.mu.Unlock()  // 用 defer 确保解锁
    return c.v[key]
}
```

### 10.7 并发模式（Effective Go 进阶）

#### 模式 1: 用通道等待后台任务完成
```go
c := make(chan int)
go func() {
    list.Sort()
    c <- 1       // 发送信号
}()
doSomethingElse()
<-c              // 等待排序完成
```

#### 模式 2: Channel of Channels
- 通道可以传递通道，实现更复杂的通信模式
- 常用于实现 request-response 模式

#### 模式 3: 固定数量 Worker Pool
```go
func handle(queue chan Request) {
    for r := range queue {
        process(r)
    }
}

func Serve(clientRequests chan Request, quit chan bool) {
    for i := 0; i < MaxOutstanding; i++ {
        go handle(clientRequests)  // 启动固定数量的 worker
    }
    <-quit
}
```

### 10.8 并发 vs 并行

| 概念 | 含义 |
|------|------|
| **并发 (Concurrency)** | 程序结构上同时处理多件事的能力 |
| **并行 (Parallelism)** | 物理上同时执行多件事 |

- Go 的并发原语（goroutine + channel）关注的是**程序结构**
- 是否真正并行取决于硬件和 `GOMAXPROCS` 设置

### 延伸阅读
- [Go Concurrency Patterns](https://go.dev/talks/2012/concurrency.slide) — Rob Pike 经典演讲
- [Advanced Go Concurrency Patterns](https://go.dev/talks/2013/advconc.slide) — 进阶并发模式
- [Share Memory by Communicating](https://go.dev/doc/codewalk/sharemem/) — 代码走读
- [The Go Memory Model](https://go.dev/ref/mem) — 内存模型规范
