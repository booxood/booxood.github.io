title: Data Model 设计
date: 2015-08-12 14:29:50
tags: Mongodb
categories: Mongodb
---

传统的关系型数据库，设计 Data Model 时可以完全参照[范式](https://zh.wikipedia.org/wiki/%E6%95%B0%E6%8D%AE%E5%BA%93%E8%A7%84%E8%8C%83%E5%8C%96)来设计，虽然，你也可以把非关系型数据库的格式也按照范式来设计，但那样就跟用关系型数据库没什么差别了。

关系型数据库里两个 Model 之间的关系都是 `use references`，而非关系型数据库除了可以是 `use references` 还可以是 `embed`。所以在为非关系型数据库设计 Data Model 时，关键要考虑的问题就是什么情况用 `use references` 什么情况用 `embed`。



*优先考虑使用 `embed`*

- 一对一时，两个 Model 之间是`包含`或`属于`关系，例如：用户和地址
- 一对多时，其他的 Model 都完全`属于`另外一个 Model，例如：用户拥有多个邮箱

这种情况下，查询效率高，因为只需要查询一次就可以拿到所有的信息，写的效率可能相对来说就要低一点了，另外还需要数据还会在冗余。


*优先考虑使用 `use references`*

- 多对多关系时

这种情况下，Model 间的关联更灵活，结构清晰，数据冗余少。


### 参考

https://docs.mongodb.org/manual/core/data-model-design/

