## 模型定义

复杂的模型定义不是必须的，此功能用作数据库数据转换和自动建表

## Struct Tag 设置参数
```go
orm:"null;rel(fk)"
```
	
通常每个 Field 的 StructTag 里包含两种类型的设置，类似 null 的 bool 型设置，还有 类似 rel(fk) 的指定值设置，bool 型默认为 false，指定以后即表示为 true

多个设置间使用 `;` 分隔，设置的值如果是多个，使用 `,` 分隔。

#### 忽略字段

设置 `-` 即可忽略 struct 中的字段

```go
type User struct {
...
	AnyField string `orm:"-"`
...
```

#### auto

设置为 Autoincrement Primary Key

#### pk

设置为 Primary Key

#### null

数据库表默认为 `NOT NULL`，设置 null 代表 `ALLOW NULL`

#### blank

设置 string 类型的字段允许为空，否则 clean 会返回错误

#### index

为字段增加索引

#### unique

为字段增加 unique 键

#### column

为字段设置 db 字段的名称
```go
Name `orm:"column(user_name)"`
```
#### default

为字段设置默认值，类型必须符合
```go
type User struct {
	...
	Status int `orm:"default(1)"`
```
#### size (string)

string 类型字段设置 size 以后，db type 将使用 varchar
```go
Title string `orm:"size(60)"`
```
#### digits / decimals

设置 float32, float64 类型的浮点精度
```go
Money float64 `orm:"digits(12);decimals(4)"`
```
总长度 12 小数点后 4 位 eg: `99999999.9999`

#### auto_now / auto_now_add
```go
Created     time.Time `auto_now_add`
Updated     time.Time `auto_now`
```
* auto_now 每次 model 保存时都会对时间自动更新
* auto_now_add 第一次保存时才设置时间

对于批量的 update 此设置是不生效的

#### type

设置为 date, time.Time 字段的对应 db 类型使用 date
```go
Created time.Time `orm:"auto_now_add;type(date)"`
```
## 表关系设置

#### rel / reverse

**RelOneToOne**:
```go
type User struct {
	...
	Profile *Profile `orm:"null;rel(one);on_delete(set_null)"`
```
对应的反向关系 **RelReverseOne**:
```go
type Profile struct {
	...
	User *User `orm:"reverse(one)" json:"-"`
```
**RelForeignKey**:
```go
type Post struct {
	...
	User*User `orm:"rel(fk)"` // RelForeignKey relation
```
对应的反向关系 **RelReverseMany**:
```go
type User struct {
	...
	Posts []*Post `orm:"reverse(many)" json:"-"` // fk 的反向关系
```
**RelManyToMany**:
```go
type Post struct {
	...
	Tags []*Tag `orm:"rel(m2m)"` // ManyToMany relation
```
对应的反向关系 **RelReverseMany**:
```go
type Tag struct {
	...
	Posts []*Post `orm:"reverse(many)" json:"-"`
```
#### rel_table / rel_through

此设置针对 `orm:"rel(m2m)"` 的关系字段

	rel_table       设置自动生成的 m2m 关系表的名称
	rel_through     如果要在 m2m 关系中使用自定义的 m2m 关系表
	                通过这个设置其名称，格式为 pkg.path.ModelName
	                eg: app.models.PostTagRel
	                PostTagRel 表需要有到 Post 和 Tag 的关系

当设置 rel_table 时会忽略 rel_through

#### on_delete

设置对应的 rel 关系删除时，如何处理关系字段。

	cascade        级联删除(默认值)
	set_null       设置为 NULL，需要设置 null = true
	set_default    设置为默认值，需要设置 default 值
	do_nothing     什么也不做，忽略

```go
type User struct {
	...
	Profile *Profile `orm:"null;rel(one);on_delete(set_null)"`
...
type Profile struct {
	...
	User *User `orm:"reverse(one)" json:"-"`

// 删除 Profile 时将设置 User.Profile 的数据库字段为 NULL
```


## 模型字段与数据库类型的对应

在此列出 orm 推荐的对应数据库类型，自动建表功能也会以此为标准。

默认所有的字段都是 **NOT NULL**

#### MySQL

| go		   |mysql
| :---   	   | :---
| bool | bool
| string - 设置 size 时 | varchar(size)
| string | longtext
| time.Time - 设置 type 为 date 时 | date
| time.TIme | datetime
| byte | tinyint unsigned
| rune | integer
| int | integer
| int8 | tinyint
| int16 | smallint
| int32 | integer
| int64 | bigint
| uint | integer unsigned
| uint8 | tinyint unsigned
| uint16 | smallint unsigned
| uint32 | integer unsigned
| uint64 | bigint unsigned
| float32 | double precision
| float64 | double precision
| float64 - 设置 digits, decimals 时  | numeric(digits, decimals)

#### Sqlite3

| go		   | sqlite3
| :---   	   | :---
| bool | bool
| string - 设置 size 时 | varchar(size)
| string | text
| time.Time - 设置 type 为 date 时 | date
| time.TIme | datetime
| byte | tinyint unsigned
| rune | integer
| int | integer
| int8 | tinyint
| int16 | smallint
| int32 | integer
| int64 | bigint
| uint | integer unsigned
| uint8 | tinyint unsigned
| uint16 | smallint unsigned
| uint32 | integer unsigned
| uint64 | bigint unsigned
| float32 | real
| float64 | real
| float64 - 设置 digits, decimals 时  | decimal

#### PostgreSQL

| go		   | postgres
| :---   	   | :---
| bool | bool
| string - 设置 size 时 | varchar(size)
| string | text
| time.Time - 设置 type 为 date 时 | date
| time.TIme | timestamp with time zone
| byte | smallint CHECK("column" >= 0 AND "column" <= 255)
| rune | integer
| int | integer
| int8 | smallint CHECK("column" >= -127 AND "column" <= 128)
| int16 | smallint
| int32 | integer
| int64 | bigint
| uint | bigint CHECK("column" >= 0)
| uint8 | smallint CHECK("column" >= 0 AND "column" <= 255)
| uint16 | integer CHECK("column" >= 0)
| uint32 | bigint CHECK("column" >= 0)
| uint64 | bigint CHECK("column" >= 0)
| float32 | double precision
| float64 | double precision
| float64 - 设置 digits, decimals 时  | numeric(digits, decimals)


## 关系型字段

其字段类型取决于对应的主键。

* RelForeignKey
* RelOneToOne
* RelManyToMany
* RelReverseOne
* RelReverseMany