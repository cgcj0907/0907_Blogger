---
title: "Gorm CRUD"
date: 2025-01-19
draft: false
tags: ["Languages", "Golang", "Gorm"]
ShowToc: true
---

## Create

### 创建记录的基础操作
GORM 提供了两种风格的 API 来创建记录：**泛型 API**（较新）和**传统 API**

*   **传统API（常用）**
    ```go
    user := User{Name: "Jinzhu", Age: 18, Birthday: time.Now()}
    // 插入单条记录，传入数据的指针
    result := db.Create(&user)

    user.ID             // 获取插入数据的主键
    result.Error        // 获取错误信息
    result.RowsAffected // 获取插入的记录数
    ```
    **关键提醒**：必须传递数据指针（如 `&user`）给 `Create` 方法，不能直接传递结构体

*   **插入多条记录**
    ```go
    users := []*User{
      {Name: “Jinzhu1”},
      {Name: “Jinzhu2”},
    }
    result := db.Create(users) // 传递切片以插入多行
    ```

### 选择/忽略字段创建
创建记录时，可以指定只插入或排除某些字段
*   **`Select`**：仅创建指定字段
    ```go
    db.Select(“Name”, “Age”, “CreatedAt”).Create(&user)
    ```
*   **`Omit`**：创建时忽略指定字段
    ```go
    db.Omit(“Name”, “Age”, “CreatedAt”).Create(&user)
    ```

### 批量插入
向 `Create` 方法传递一个切片可以高效插入大量数据。GORM 会生成一条 SQL 语句插入所有数据，并回填主键值，同时也会调用关联的 Hook 方法
```go
var users = []User{{Name: “jinzhu1”}, {Name: “jinzhu2”}}
db.Create(&users)
```
*   **指定批次大小**：使用 `CreateInBatches` 可以控制每批插入的数量，这在处理极大量数据时有助于优化性能
    ```go
    var users = []User{{Name: “jinzhu_1”}, …, {Name: “jinzhu_10000”}}
    // 每批插入100条
    db.CreateInBatches(users, 100)
    ```
*   **全局批次大小**：可以在初始化 GORM 或创建会话时设置 `CreateBatchSize` 选项，之后所有的 `INSERT` 操作（包括关联创建）都会遵循这个批次大小

### 使用 Hook
GORM 允许为 `BeforeSave`、`BeforeCreate`、`AfterSave`、`AfterCreate` 生命周期定义自定义的 Hook 方法
```go
func (u *User) BeforeCreate(tx *gorm.DB) (err error) {
  u.UUID = uuid.New()
  if u.Role == “admin” {
    return errors.New(“invalid role”)
  }
  return
}
```
如果需要**跳过 Hook** 的执行，可以使用 `SkipHooks` 会话模式：
```go
DB.Session(&gorm.Session{SkipHooks: true}).Create(&user)
```

### 从 Map 创建
GORM 支持从 `map[string]interface{}` 或 `[]map[string]interface{}{}` 创建记录
```go
db.Model(&User{}).Create(map[string]interface{}{
  “Name”: “jinzhu”, “Age”: 18,
})
```
**重要提示**：从 Map 创建时，**Hook 方法不会被调用，关联不会被保存，主键值也不会被回填**

### 使用 SQL 表达式/自定义类型创建
有时需要直接使用数据库函数或表达式来插入数据，有两种方式：
1.  **从 Map 创建，值使用 `clause.Expr`**
    ```go
    db.Model(User{}).Create(map[string]interface{}{
      “Location”: clause.Expr{SQL: “ST_PointFromText(?)”, Vars: []interface{}{“POINT(100 100)”}},
    })
    ```
2.  **使用实现了 `GormValuer` 接口的自定义数据类型**
    ```go
    type Location struct { X, Y int }
    func (loc Location) GormValue(ctx context.Context, db *gorm.DB) clause.Expr {
      return clause.Expr{
        SQL: “ST_PointFromText(?)”,
        Vars: []interface{}{fmt.Sprintf(“POINT(%d %d)”, loc.X, loc.Y)},
      }
    }
    db.Create(&User{Name: “jinzhu”, Location: Location{X: 100, Y: 100}})
    ```

### 高级功能

#### 1. 创建关联记录
创建数据时，如果其关联字段不是零值，这些关联记录会被**一并创建（Upsert）**，其 Hook 方法也会被调用
```go
db.Create(&User{
  Name: “jinzhu”,
  CreditCard: CreditCard{Number: “411111111111”} // CreditCard会被同时创建
})
```
*   **跳过关联保存**：使用 `Select` 或 `Omit`
    ```go
    db.Omit(“CreditCard”).Create(&user) // 跳过特定关联
    db.Omit(clause.Associations).Create(&user) // 跳过所有关联
    ```

#### 2. 字段默认值
可以使用 `default` 标签为字段定义默认值，在插入时，对于零值字段将使用该默认值
```go
type User struct {
  Name string `gorm:“default:galeone”` // 零值“”会触发默认值
  Age  int64  `gorm:“default:18”`      // 零值0会触发默认值
}
```
*   **零值问题**：像 `0`、`“”`、`false` 这样的零值，对于定义了 `default` 的字段，**不会**被存入数据库，而是会使用默认值。如果需要区分“零值”和“未设置”，可以考虑使用**指针类型**或 `sql.NullBool` 等类型
*   **数据库默认值/生成值**：如果数据库字段本身有默认值（如 `uuid_generate_v3()`）或是生成列，你需要在 GORM 模型中使用 `default` 标签指明。如果想在迁移时跳过 GORM 的默认值定义（完全交由数据库处理），可以使用 `default:(-)`
*   **SQLite 限制**：SQLite 在批量插入时，不支持部分记录使用默认值（如 `INSERT … VALUES (“dog”), (DEFAULT)`）。一个可行的替代方案是在 `BeforeCreate` Hook 中为字段分配默认值

#### 3. Upsert / 冲突处理
GORM 通过 `clause.OnConflict` 为不同数据库提供了兼容的 Upsert（插入或更新）支持
*   **冲突时无操作**：
    ```go
    db.Clauses(clause.OnConflict{DoNothing: true}).Create(&user)
    ```
*   **冲突时更新指定列为默认值**：
    ```go
    db.Clauses(clause.OnConflict{
      Columns:   []clause.Column{{Name: “id”}},
      DoUpdates: clause.Assignments(map[string]interface{}{“role”: “user”}),
    }).Create(&users)
    ```
*   **冲突时更新指定列为新值（排除主键和有默认值的列）**：
    ```go
    db.Clauses(clause.OnConflict{
      Columns:   []clause.Column{{Name: “id”}},
      DoUpdates: clause.AssignmentColumns([]string{“name”, “age”}), // 更新这些列
    }).Create(&users)
    ```
*   **冲突时更新所有列**：
    ```go
    db.Clauses(clause.OnConflict{
      UpdateAll: true,
    }).Create(&users)
    ```

## Query

### 检索单个对象
GORM 提供了 `First`、`Take`、`Last` 方法来检索单个对象，它们会在查询时自动添加 `LIMIT 1` 子句。如果未找到记录，会返回 `gorm.ErrRecordNotFound` 错误

| 方法 | 传统 API 示例 | 生成 SQL (示例) | 说明与注意 |
| :--- | :--- | :--- | :--- |
| **`First`** | `db.First(&user)` | `SELECT * FROM users ORDER BY id LIMIT 1;` | 按**主键升序**返回第一条记录。需传入结构体指针或使用 `db.Model()` |
| **`Take`** | `db.Take(&user)` | `SELECT * FROM users LIMIT 1;` | **随机**返回一条记录（无特定排序） |
| **`Last`** | `db.Last(&user)` | `SELECT * FROM users ORDER BY id DESC LIMIT 1;` | 按**主键降序**返回第一条记录（即最后一条） |

**关键行为与提示**：
*   **错误处理**：建议使用 `errors.Is(err, gorm.ErrRecordNotFound)` 来判断是否“未找到记录”
*   **避免 `ErrRecordNotFound`**：如果不希望触发此错误，可使用 `db.Limit(1).Find(&user)`
*   **无主键模型**：若模型未定义主键，`First` 和 `Last` 会按**第一个字段**排序
*   **性能警告**：切勿使用 `db.Find(&user)`（无Limit）查询单条记录，它会查询全表且行为不确定

#### 根据主键检索
可以直接将主键值作为参数传入查询方法。
```go
// 使用数值型主键
db.First(&user, 10) // SELECT * FROM users WHERE id = 10;
// 使用主键列表（检索多个对象）
db.Find(&users, []int{1,2,3}) // SELECT * FROM users WHERE id IN (1,2,3);
// 字符串主键（如UUID）需使用内联条件
db.First(&user, "id = ?", "uuid-string-here")
```
**特殊场景**：当传入的结构体对象本身已包含主键值时（如 `user := User{ID: 10}`），GORM 会默认使用该主键作为查询条件

### 检索所有对象
使用 `Find` 方法获取所有匹配的记录
```go
result := db.Find(&users) // SELECT * FROM users;
// 可以通过 result 访问影响行数和错误
total := result.RowsAffected // 等于 len(users)
err := result.Error
```

### 查询条件（WHERE）
这是构造查询的核心，GORM 提供了多种方式

#### 1. 字符串条件
最直接的方式，使用占位符 `?` 防止 SQL 注入
```go
db.Where("name = ?", "jinzhu").First(&user)
db.Where("name IN ?", []string{"a", "b"}).Find(&users)
db.Where("name LIKE ? AND age > ?", "%jin%", 20).Find(&users)
db.Where("created_at BETWEEN ? AND ?", lastWeek, today).Find(&users)
```

#### 2. 结构体与 Map 条件
可以使用结构体或 Map 来构建条件，其中**非零值字段**会成为查询条件
```go
// Struct - 零值字段会被忽略
db.Where(&User{Name: "jinzhu", Age: 0}).Find(&users)
// SELECT * FROM users WHERE name = "jinzhu"; (Age为0，被忽略)
// Map - 所有键值都会作为条件
db.Where(map[string]interface{}{"Name": "jinzhu", "Age": 0}).Find(&users)
// SELECT * FROM users WHERE name = "jinzhu" AND age = 0;
```

#### 3. 指定结构体查询字段
可以通过 `Where` 方法的后续参数，显式指定使用结构体中的哪些字段作为条件
```go
db.Where(&User{Name: "jinzhu", Age: 0}, "Name", "Age").Find(&users)
// SELECT * FROM users WHERE name = "jinzhu" AND age = 0;
```

#### 4. 内联条件（Inline Conditions）
可以将条件直接内联到 `First`、`Find` 等方法中
```go
db.Find(&users, "name = ? AND age > ?", "jinzhu", 20)
db.Find(&users, User{Age: 20}) // 非零值字段作为条件
db.Find(&users, map[string]interface{}{"age": 20})
```

#### 5. NOT 与 OR 条件
```go
// NOT 条件
db.Not("name = ?", "jinzhu").First(&user)
db.Not([]int64{1,2,3}).First(&user) // ID NOT IN (1,2,3)
// OR 条件
db.Where("role = ?", "admin").Or("role = ?", "super_admin").Find(&users)
db.Where("name = 'jinzhu'").Or(User{Name: "jinzhu 2", Age: 18}).Find(&users)
```

### 选择特定字段
使用 `Select` 可以指定只返回哪些字段，默认情况下 GORM 会查询所有字段
```go
db.Select("name", "age").Find(&users) // SELECT name, age FROM users;
db.Select([]string{"name", "age"}).Find(&users)
```

### 排序、限制与分页
这些方法通常与 `Find` 链式调用

| 方法 | 示例 | 说明 |
| :--- | :--- | :--- |
| **`Order`** | `db.Order("age desc, name").Find(&users)` | 指定排序。可多次调用，顺序叠加 |
| **`Limit`** | `db.Limit(10).Find(&users)` | 限制返回的最大记录数。用 `-1` 可取消之前设置的 Limit |
| **`Offset`** | `db.Offset(5).Find(&users)` | 指定跳过的记录数，用于分页。用 `-1` 可取消 |

**分页典型用法**：
```go
db.Limit(pageSize).Offset((page - 1) * pageSize).Find(&users)
```

### 分组与统计（Group By & Having）
用于分组聚合查询
```go
type Result struct {
    Date  time.Time
    Total int
}
// Group By
db.Model(&User{}).Select("name, sum(age) as total").Group("name").Find(&results)
// Having
db.Model(&User{}).Select("name, sum(age) as total").Group("name").Having("sum(age) > ?", 100).Find(&results)
```
> **提示**：对于更复杂的 SQL 查询（如连接查询、子查询、锁等），GORM 在 **“Advanced Query”** 章节有详细介绍

### 去重 (Distinct)
用于去重查询
```go
db.Distinct("name", "age").Order("name, age desc").Find(&results)
```
对于 `Pluck` 和 `Count` 同样适用

### 扫描 (Scan)
把数据库里的每一行数据读出来，然后填入 Go 结构体字段
```go
type Result struct {
  Name string
  Age  int
}

var result Result
db.Table("users").Select("name", "age").Where("name = ?", "Antonio").Scan(&result)

// Raw SQL
db.Raw("SELECT name, age FROM users WHERE name = ?", "Antonio").Scan(&result)
```

## Update


