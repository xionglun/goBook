## SQL in Go
在Go语言中使用SQL的相关知识。

### 关于`sql.DB`
在Go语言中，访问数据库是通过`sql`包中的`DB`对象实现的。

首先，`sql.DB`并不是一个数据库连接！它也不对应于数据库理论中的任一概念，其只是一个接口的抽象以及数据库的表示。
数据库可以是一个本地文件、一个网络链接或者内存、进程数据程。

`sql.DB`默默地处理了一些重要的事情：
* 通过数据库驱动打开或者关闭与底层数据库的连接
* 若有需要，它会去管理维护一个数据库连接池

`sql.DB`这一层抽象让你不用操心与底层数据库之间的并发操作。当你通过一个连接进行数据库操作时，
它会标记连接为正在使用状态，然后在使用完毕时将其回收到连接池中。而如果你忘了释放连接回连接池，
将有可能打开大量未释放连接，然后耗尽资源无法正常使用。


### 关于数据库驱动
当操作数据库时，你除了用到Go标准库中的`sql`包外，还应该引用一个与真实数据库相关的驱动包。
不同的数据库需要不同的驱动。   
但是，一般来说，你不应该直接使用驱动包，而是应该使用标准包。这样会让你不限定于使用某一种数据库，
可以轻易切换底层数据库，同时也让你用Go标准模式而不是驱动提供的方式来操作数据库。

引用数据库驱动方式如下：
```go
import (
	"database/sql"
	_ "github.com/go-sql-driver/mysql"
)
```
这里代码使用Go标准库sql包，同时引入Go语言的**MySQL**驱动。


### 使用数据库
要使用数据库，就需要获得一个数据库对象`sql.DB`，如下所示：
```go
db, err := sql.Open("mysql", "user:password@tcp(127.0.0.1:3306)/hello")
if err != nil {
	log.Fatal(err)
}
defer db.Close()
```
这里通过`Open`方法来获得一个数据库引用。你应该总是检查是否有错误，来防止意外情况。

注意，这里的`Open`方法并没有建立与数据库的链接，也没有查检查驱动的语法是否正确。
如果你想要检查是否可能连接到数据库，可以使用`Ping`方法来实现：
```go
err = db.Ping()
if err != nil {
	// do something here
}
```

**注意** 虽然关闭一个已打开的数据库链接看起来是很自然的一件事，但是由于`sql.DB`对象的设计成长久存活的，
所以不要频繁地使用`Open`和`Close`方法。如果你频繁的开关数据库链接，将会引发很多网络问题，
比如大量TCP链接处于等待状态，资源耗尽。


### 从数据库中查询数据
从数据库中查询数据有几种方式：
* 直接使用查询语句进行查询
* 使用预声明进行多次查询
* 使用预声明进行单次查询
* 只获取单行数据的查询

#### 直接查询
示例如下：
```go
var (
	id int
	name string
)
rows, err := db.Query("select id, name from users where id = ?", 1)
if err != nil {
	log.Fatal(err)
}
defer rows.Close()
for rows.Next() {
	err := rows.Scan(&id, &name)
	if err != nil {
		log.Fatal(err)
	}
	log.Println(id, name)
}
err = rows.Err()
if err != nil {
	log.Fatal(err)
}
```
这里我们想要查询`id`为1的用户的ID与名字，使用了`db.Query`这个方法进行直接查询。

这里有很多值得注意的地方：
1. 注意检查错误，不要忽略错误
2. 使用完查询结果后，要注意关闭`rows.Close()`
3. 遍历`rows`使用的是`rows.Next`这个方法
4. 读取查询结果时使用的是`Scan`方法
5. 使用完`rows`之后要注意检查是否有错误`rows.Err()`

其中`rows.Close()`这个方法是一个无害的方法，可以重复调用。
但是，在使用这个方法前，必须先检查前面操作有没有错误产生。

#### 使用声明进行查询
示例如下：
```go
stmt, err := db.Prepare("select id, name from users where id = ?")
if err != nil {
	log.Fatal(err)
}
defer stmt.Close()
rows, err := stmt.Query(1)
if err != nil {
	log.Fatal(err)
}
defer rows.Close()
for rows.Next() {
	// ...
}
if err = rows.Err(); err != nil {
	log.Fatal(err)
}
```
这里使用`db.Prepare`方法创建一个声明(Statement)stmt来进行后续使用。
使用声明进行数据库操作要比字符串拼接直接查询好得多。


#### 单行数据查询
单行数据查询表示每次查询最多只会取得一行数据，示例如下：
```go
var name string
err = db.QueryRow("select name from users where id = ?", 1).Scan(&name)
if err != nil {
	log.Fatal(err)
}
fmt.Println(name)
```
在单行查询时，查询不会返回错误，只有在读取结果时候可能返回错误。

你同样也可以在预声明中使用单行查询
```go
stmt, err := db.Prepare("select name from users where id = ?")
if err != nil {
	log.Fatal(err)
}
var name string
err = stmt.QueryRow(1).Scan(&name)
if err != nil {
	log.Fatal(err)
}
fmt.Println(name)
```

### 关于预声明的缺点
使用预声明(Prepared Statement)有很多好处（安全、有效、方便等）。
但是，由于Go在`database/sql`这一层的抽象，所以预声明运作可能与你想要的有差异。

在某些实现中，预声明是与底层真实链接绑定的，然后操作都是在同一条链路上。而Go中则不是：
1. 当你定义了一个预声明，它会准备一个在连接池中的链接。
2. `stmt`对象会记住所使用的那个链接。
3. 当你使用`stmt`对象的时候，它会尝试使用原来的链接。但当原来的链接关闭或者被占用的时候，
它就会再从连接池里取出一条链接重新绑定，这样可能会造成大量的预声明泄露。


### 操作数据
在一般语言或者库中，读取数据与更新修改数据可能使用的是同样的接口。但是在Go语言中，这两者是有很大差别的。

在Go语言中，读取操作一般使用`db.Query`这个接口，而操作数据一般使用`db.Exec`这个接口。
因为在Go语言中，`Exec`方法返回的是`sql.Result`对象，而`Query`返回的是`Rows`对象。
而如果`Rows`对象里还有数据未消息的话，其将保持底层链接不释放直到关闭，很可能造成泄露。


### 事务处理
一个事务会保证事务内所有操作在一条链接上进行。所以，事务内的预声明将不会出现上面所述的缺点情况。

使用事务时候，不要将事务方法`Begin()`或者`Commit()`与SQL语句里的`BEGIN`或者`COMMIT`关键词混合使用。

在一个事务内，不要再去使用`db`对象，用`tx`对象执行所有操作。如下所示：
```go
tx, err := db.Begin()
if err != nil {
	log.Fatal(err)
}
defer tx.Rollback()
stmt, err := tx.Prepare("INSERT INTO foo VALUES (?)")
if err != nil {
	log.Fatal(err)
}
defer stmt.Close() // danger!
for i := 0; i < 10; i++ {
	_, err = stmt.Exec(i)
	if err != nil {
		log.Fatal(err)
	}
}
err = tx.Commit()
if err != nil {
	log.Fatal(err)
}
// stmt.Close() runs here!
```


### 处理错误
> 在进行数据库操作时，可能出现错误的地方，都不应该忽略!

在进行数据库操作时，有几处地方的错误需要特别注意：

1）遍历查询结果后可能出现的错误
```go
for rows.Next(); {
}
if err = rows.Err(); err != nil {
	// handle error here
}
```

2）关闭查询结果时可能出现的错误
```go
for rows.Next() {
	// ...
	break; // whoops, rows is not closed! memory leak...
}
// do the usual "if err = rows.Err()" [omitted here]...
// it's always safe to [re?]close here:
if err = rows.Close(); err != nil {
	// but what should we do if there's an error?
	log.Println(err)
}
```

3）查询单行数据时候出错
```go
var name string
err = db.QueryRow("select name from users where id = ?", 1).Scan(&name)
if err != nil {
	if err == sql.ErrNoRows {
		// there were no rows, but otherwise no error occurred
	} else {
		log.Fatal(err)
	}
}
fmt.Println(name)
```
在查询单行数据时，有可能是没有查到数据，而不是出现其它错误，所以有可能需要单独处理。


### 处理Null返回
由于某些列可以默认为空，所以返回的时候可能会返回Null，你可以这样来处理：
```go
for rows.Next() {
	var s sql.NullString
	err := rows.Scan(&s)
	// check err
	if s.Valid {
	   // use s.String
	} else {
	   // NULL value
	}
}
```
除了示例中的`NullString`之外，还有`NullInt64`、`NullBool`、`NullFloat64`这几种已定义的。

如果觉得上面这几种不能满足需求，可以仿照Go标准实现来处理自定义的类型。


### 处理某知的列
使用`Scan`方法时，你需要预先知道所取出来的数据有多少行。假如取出来的行数不定时，就不能用上面的方法。
这时候，就可以使用`Columns`方法，如下所示：
```go
cols, err := rows.Columns()
if err != nil {
	// handle the error
} else {
	dest := []interface{}{ // Standard MySQL columns
		new(uint64), // id
		new(string), // host
		new(string), // user
		new(string), // db
		new(string), // command
		new(uint32), // time
		new(string), // state
		new(string), // info
	}
	if len(cols) == 11 {
		// Percona Server
	} else if len(cols) > 8 {
		// Handle this case
	}
	err = rows.Scan(dest...)
	// Work with the values in dest
}
```

如果你既不知道列数也不知道列类型，可以这么做：
```go
cols, err := rows.Columns() // Remember to check err afterwards
vals := make([]interface{}, len(cols))
for i, _ := range cols {
	vals[i] = new(sql.RawBytes)
}
for rows.Next() {
	err = rows.Scan(vals...)
	// Now you can check each element of vals for nil-ness,
	// and you can use type introspection and type assertions
	// to fetch the column into a typed variable.
}
```

-------
关于Go中使用数据库的某些特殊问题参见：http://go-database-sql.org/surprises.html
