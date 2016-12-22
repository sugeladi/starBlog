title: golang mysql 1461 and Invalid Connection
tags:
  - golang mysql
categories: golang mysql
date: 2016-12-22 13:24:52
---

### 1.Invalid Connection

##### go-mysql的连接池只有两个参数MaxIdleConns 与 MaxOpenConns
##### golang使用mysql官方驱动时，用了mysql连接池(如果MaxIdleConns 与 MaxOpenConns不相等)，有时会看到大量的  statement.go:27: Invalid Connection 一番查询之后，发现底层驱动有这个警告，最好设置MaxIdleConns和MaxOpenConns相等。就不会有这个warning信息了，不过这个warning信息时无害的。如果有强迫症的同学，还是设置相等吧。[GitHub地址](https://github.com/go-sql-driver/mysql/issues/185)



### 2.Error 1461: Can't create more than max_prepared_stmt_count statements (current value: 16382)
##### 这个需要注意，再使用了 Prepare 之后记得，返回的Stmt记得close()  另外在查询多列Query()返回的Rows 也要关闭