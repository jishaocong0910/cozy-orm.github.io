---
title: 快速开始
weight: 2
---

# 快速开始

## 安装

```shell
go get github.com/jishaocong0910/cozy-orm
```

## 基础API

```go
package main

import (
	"database/sql"
	"encoding/json"
	"fmt"
	"time"

	_ "github.com/go-sql-driver/mysql"
	orm "github.com/jishaocong0910/cozy-orm"
)

type User struct {
	Id       *int64 `orm:"pk,auto"`
	Name     *string
	Age      *int32
	Address  *string
	Phone    *string
	Email    *string
	Status   *int8
	Level    *int8
	CreateAt *time.Time
}

func main() {
	// open a sql.DB
	sqlDB, err := sql.Open("mysql", "root:12345678@tcp(127.0.0.1:3306)/test?charset=utf8mb4&parseTime=True&loc=Local")
	if err != nil {
		panic(err)
	}

	// create a orm.DB
	db := orm.DbConfig{
		SqlDB:  sqlDB,
		DbType: orm.DbType_.MySQL,
	}.Build()

	// insert
	u := &User{
		Name:     new("Alice"),
		Age:      new(int32(20)),
		Address:  new("anytown"),
		Phone:    new("123456789"),
		Email:    new("demo@email.com"),
		Status:   new(int8(1)),
		Level:    new(int8(0)),
		CreateAt: new(time.Now()),
	}
	affected, err := db.Mutation(nil).MapTarget[User](u).BuildSql(func(b *orm.SqlBuilder) {
		b.Write("INSERT INTO user(name, age, address, phone, email, status, level, create_at) VALUES(?, ?, ?, ?, ?, ?, ?, ?)",
			u.Name, u.Age, u.Address, u.Phone, u.Email, u.Status, u.Level, u.CreateAt)
	}).Do()
	if err != nil {
		panic(err)
	}
	fmt.Printf("affected: %d, id: %d\n", affected, *u.Id) // auto increment key

	// query
	u2, err := db.Query[User](nil).BuildSql(func(b *orm.SqlBuilder) {
		b.Write("SELECT * FROM user WHERE id = ?", u.Id)
	}).Do()
	if err != nil {
		panic(err)
	}
	j, _ := json.Marshal(u2)
	fmt.Println(string(j))

	// update
	affected, err = db.Mutation(nil).BuildSql(func(b *orm.SqlBuilder) {
		b.Write("UPDATE user SET phone = ?, status = ? WHERE id = ?", "987654321", 2, u.Id)
	}).Do()
	if err != nil {
		panic(err)
	}
	fmt.Println("affected:", affected)

	// delete
	affected, err = db.Mutation(nil).BuildSql(func(b *orm.SqlBuilder) {
		b.Write("DELETE FROM user WHERE id = ?", u.Id)
	}).Do()
	if err != nil {
		panic(err)
	}
	fmt.Println("affected:", affected)
}
```

## 高级API

```go
package main

import (
	"database/sql"
	"encoding/json"
	"fmt"
	"time"

	_ "github.com/go-sql-driver/mysql"
	orm "github.com/jishaocong0910/cozy-orm"
)

type User struct {
	Id       *int64 `orm:"pk,auto"`
	Name     *string
	Age      *int32
	Address  *string
	Phone    *string
	Email    *string
	Status   *int8
	Level    *int8
	CreateAt *time.Time
}

func main() {
	// open a sql.DB
	sqlDB, err := sql.Open("mysql", "root:12345678@tcp(127.0.0.1:3306)/test?charset=utf8mb4&parseTime=True&loc=Local")
	if err != nil {
		panic(err)
	}

	// create a orm.DB
	db := orm.DbConfig{
		SqlDB:  sqlDB,
		DbType: orm.DbType_.MySQL,
	}.Build()

	// insert
	u := &User{
		Name:     new("Alice"),
		Age:      new(int32(20)),
		Address:  new("anytown"),
		Phone:    new("123456789"),
		Email:    new("demo@email.com"),
		Status:   new(int8(1)),
		Level:    new(int8(0)),
		CreateAt: new(time.Now()),
	}
	affected, err := db.Insert[User](nil).Entities(u).Do()
	if err != nil {
		panic(err)
	}
	fmt.Printf("affected: %d, id: %d\n", affected, *u.Id) // auto increment key

	// query
	u2, err := db.FindOne[User](nil).Condition(orm.Cond().Eq("id", u.Id)).Do()
	if err != nil {
		panic(err)
	}
	j, _ := json.Marshal(u2)
	fmt.Println(string(j))

	// update
	u3 := &User{
		Id:     u.Id,
		Phone:  new("987654321"),
		Status: new(int8(2)),
	}
	affected, err = db.Update[User](nil).Entity(u3).Do()
	if err != nil {
		panic(err)
	}
	fmt.Println("affected:", affected)

	// delete
	affected, err = db.Delete[User](nil).Condition(orm.Cond().Eq("id", u.Id)).Do()
	if err != nil {
		panic(err)
	}
	fmt.Println("affected:", affected)
}
```