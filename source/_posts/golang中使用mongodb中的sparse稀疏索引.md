title: golang中使用mongodb中的sparse稀疏索引
date: 2015-10-23 14:15:10
tags: [golang,mongodb,sparse]
categories: mongodb
---
### 稀疏索引
“sparse”的作用就是当索引字段在文档中不存在，则不进入索引，从而避免唯一索引问题。

##### 举个栗子，当我们有一个表user1，有mobile字段，在刚开始注册的时候,允许mobile为空，但对于mobile来说需要是unique的，这是就不能够允许多个用户存在了。这时如果加上sparse:true,就没有问题了。
<!--more-->
##### 直接看例子
<div bgcolor=#dddddd>
            > db.user1.insert({"name":"hx1","mobile":13912345678})
            WriteResult({ "nInserted" : 1 })
            > db.user1.createIndex({mobile:1},{unique:true})
            {
            	"createdCollectionAutomatically" : false,
            	"numIndexesBefore" : 1,
            	"numIndexesAfter" : 2,
            	"ok" : 1
            }
            > db.user1.find()
            { "_id" : ObjectId("5629d27de589416ab6babc4c"), "name" : "hx1", "mobile" : 13912345678 }
</div>
这时候如果在插入就会出现duplicate
            > db.user1.insert({"name":"hx2","mobile":13912345678})
            WriteResult({
            	"nInserted" : 0,
            	"writeError" : {
            		"code" : 11000,
            		"errmsg" : "E11000 duplicate key error index: asot.user1.$mobile_1 dup key: { : 13912345678.0 }"
            	}
            })
如果插入的mobile为空值,可以成功
            > db.user1.insert({"name":"hx2"})
            WriteResult({ "nInserted" : 1 })
            > db.user1.find()
            { "_id" : ObjectId("5629d50be589416ab6babc50"), "name" : "hx1", "mobile" : 13912345678 }
            { "_id" : ObjectId("5629d52fe589416ab6babc52"), "name" : "hx2" }
当第二次再插入的时候就失败了
            > db.user1.insert({"name":"hx2"})
            WriteResult({
            	"nInserted" : 0,
            	"writeError" : {
            		"code" : 11000,
            		"errmsg" : "E11000 duplicate key error index: asot.user1.$mobile_1 dup key: { : null }"
            	}
            })
好，现在精彩的来了，使用sparse配合unique一起使用。
先把mobile的unique唯一索引删除，然后添加sparse配合unique共同索引
            > db.user1.dropIndex({mobile:1})
            { "nIndexesWas" : 2, "ok" : 1 }
            > db.user1.getIndexes()
            [
            	{
            		"v" : 1,
            		"key" : {
            			"_id" : 1
            		},
            		"name" : "_id_",
            		"ns" : "asot.user1"
            	}
            ]
            > db.user1.createIndex({mobile:1},{unique:true,sparse:true})
            {
            	"createdCollectionAutomatically" : false,
            	"numIndexesBefore" : 1,
            	"numIndexesAfter" : 2,
            	"ok" : 1
            }
            > db.user1.getIndexes()
            [
            	{
            		"v" : 1,
            		"key" : {
            			"_id" : 1
            		},
            		"name" : "_id_",
            		"ns" : "asot.user1"
            	},
            	{
            		"v" : 1,
            		"unique" : true,
            		"key" : {
            			"mobile" : 1
            		},
            		"name" : "mobile_1",
            		"ns" : "asot.user1",
            		"sparse" : true
            	}
            ]
索引已经建好，这次在尝试插入空值
            > db.user1.find()
            { "_id" : ObjectId("5629d50be589416ab6babc50"), "name" : "hx1", "mobile" : 13912345678 }
            { "_id" : ObjectId("5629d52fe589416ab6babc52"), "name" : "hx2" }
            > db.user1.insert({"name":"hx2"})
            WriteResult({ "nInserted" : 1 })
            > db.user1.find()
            { "_id" : ObjectId("5629d50be589416ab6babc50"), "name" : "hx1", "mobile" : 13912345678 }
            { "_id" : ObjectId("5629d52fe589416ab6babc52"), "name" : "hx2" }
            { "_id" : ObjectId("5629d708e589416ab6babc54"), "name" : "hx2" }
打工告成！

谨记：对于golang来说,struct在映射mongodb表的时候，使用时需要添加 bson:",omitempty"。 允许为空！