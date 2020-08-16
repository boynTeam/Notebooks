我们知道,Gorm是无法支持数组类型的元素的,但是在PgSQL中,恰好支持数组类型.

所以这个时候要如何使Gorm支持这个类型呢?

# 直接使用数组

假设我们有这样的Model

```go
type Product struct {
	gorm.Model
	Code string `json:"code"`
	Price uint `json:"price"`
	Names [] `json:"names" gorm:"type:varchar(100)[]"`
}
```

并且以这样的方式启动pg数据库

```go
db, err := gorm.Open("postgres", "host=localhost port=5433 user=postgres dbname=gorm sslmode=disable password=**")
	if err != nil {
		panic(err)
	}
	db.AutoMigrate(&model.Product{})
```

那么,我们在启动的时候,gorm就会报错,称不支持这种方式

# 使用lib/pg

在gorm的一个[issue](https://github.com/jinzhu/gorm/issues/1588)中,我查找到了关于这个的解决方法,就是使用"github.com/lib/pq"

我们导入最新的"github.com/lib/pq"

并将上面的Product struct改成

```go
type Product struct {
	gorm.Model
	Code string `json:"code"`
	Price uint `json:"price"`
	Names  pq.StringArray `json:"names" gorm:"type:varchar(100)[]"`
}
```

就可以了