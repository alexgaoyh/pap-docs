# ElasticSearch _bulk 使用与实战：批量操作、查询、冲突（模拟电商下单/查询）

## 0、背景

&ensp;本文所描述的示例，仅是为了对概念进行解释，方便理解，部分示例可能并不完美，特此说明。

## 1、概念

&ensp;_bulk 操作 可以在单个请求中一次执行多个新增、修改、删除操作，使用这种方式可以极大的提升索引性能。

&ensp;_bulk 的请求体内部分为多行，其中连续的两行数据构成了一次操作，第一行是操作类型可以index，create，update，或者delete，第二行就是我们的可选的数据体

&ensp;针对不同的操作类型，第二行里面的可选的数据体是不一样的，如下：
1. index 和 create  第二行是source数据体
2. delete 没有第二行
3. update 第二行可以是partial doc，upsert或者是script

## 2、实战

&ensp;在电商场景中，经常会出现下单操作，比如初始化SKU的信息之后，如果遇到下单操作，除了生成订单信息之外，还需要同步更改SKU的商品数量，如下命令所示：
```
POST localhost:9200/_bulk
```
``` json
{ "index" :  { "_index" : "sku", "_type" : "_doc", "_id" : "978020137962" } }
{ "skuId" : "978020137962", "skuName" : "零售商品一", "initNum": 100, "skuNum" : 100 }
{ "index" :  { "_index" : "sku", "_type" : "_doc", "_id" : "978020137963" } }
{ "skuId" : "978020137963", "skuName" : "零售商品二", "initNum": 200, "skuNum" : 200 }
{ "index" :  { "_index" : "order", "_type" : "_doc", "_id" : "20230101" } }
{ "orderNo" : "20230101", "skus": [{"skuId" : "978020137962", "skuNum" : 1}, {"skuId" : "978020137963", "skuNum" : 3}] }
{ "update" : { "_index" : "sku", "_type" : "_doc", "_id" : "978020137962" } }
{ "doc" : {"skuNum" : 99} }
{ "update" : { "_index" : "sku", "_type" : "_doc", "_id" : "978020137963" } }
{ "doc" : {"skuNum" : 197} }

```
&ensp;对命令进行解释，按照 _bulk 的语法结构，连续的两行构成一次操作：
1. 首先是生成 978020137962 对应的 零售商品一（初始化数量initNum = 100，当前数量skuNum = 100）
2. 接下来生成 978020137963 对应的 零售商品二（初始化数量initNum = 200, 当前数量skuNum = 200）
3. 创建订单（20230101），订单下包含如上两种商品，订单商品数量分别为1和3
4. 更新 978020137962（零售商品一）的商品数量 skuNum = 99
4. 更新 978020137963（零售商品二）的商品数量 skuNum = 197

&ensp;同理其他类似的业务场景也可以按照如上的方式进行处理。

### 2.1、查找

&ensp; 在 ES 中，如果想要查找订单 20230101 下的 SKU 信息，一般情况下会分成多条查询语句，首先查询 订单信息，之后再查询 SKU 信息，略显麻烦。
这种查询方式，放到 MYSQL 中，相当于  select * from sku where sku_id in (select sku_id from order_sku_rel where order_no = '20230101')。
在 ES 中，有 terms lookup 语法可以使用，可以等价理解为 MYSQL 的联表查询，具体查询语句如下所示：
```
http://localhost:9200/sku/_search
```
``` json
{
    "query": {
        "terms": {
            "skuId": {
                "index": "order",
                "type": "_doc",
                "id": "20230101",
                "path": "skus.skuId"
            }
        }
    }
}
```
&ensp;对命令进行解释，按照 terms lookup 的语法结构，实现 根据订单号，查询订单下的 SKU明细 信息。
1. index：从中获取索引，本例从 order 这个索引进行查询。
2. type：从中获取索引类型，本例使用 _doc 。
3. id：用于获取文档的ID，是源字段_id,而不是我们自定义的字段id，本例查询的是 20230101 这个订单。
4. path：指定为获取terms过滤器实际值的路径的字段，本例从 skus.skuId 中查询出来 SKU编号。

### 2.2 retry_on_conflict

&ensp; 请注意 ES 中的 _bulk 请求不是原子的，所以不能用它来实现事务控制，同时由于每个请求是单独处理的，所以一个请求的成功或失败不会影响其他的请求。
所以如果将 _bulk 进行实际应用，建议优先对将要执行的命令进行校验，针对 action = create、index、delete 这三个命令的校验较为简单，做一个 GET 请求查询一下即可，
但是 action = update 的操作，如果遇到请求量稍微大的情况，将会遇到文档冲突的情况。
```
POST localhost:9200/_bulk
```
``` json
{ "update" : { "_index" : "sku", "_type" : "_doc", "_id" : "1234567890" } }
{ "script" : { "source": "ctx._source.counter += params.baseNum", "lang" : "painless", "params" : {"baseNum" : 1}}, "upsert" : {"counter" : 1}}

```
&ensp; 针对如上命令，本例简单的使用 POSTMAN 的 RUNNER 命令，同时开了4个RUNNER，每个 RUNNER 设置循环 500次，按计划最终得到的 counter 应该为 2000， 但是实际的执行结果 counter = 1989，
为了解决这个问题，可以增加 retry_on_conflict 参数， 设置在发生版本冲突时应该重试更新的次数，使用同样的测试，得到的结果变为正常（测试并不完备，仅供说明）。
``` json
{ "update" : { "_index" : "sku", "_type" : "_doc", "_id" : "1234567890" , "retry_on_conflict": 3} }
{ "script" : { "source": "ctx._source.counter += params.baseNum", "lang" : "painless", "params" : {"baseNum" : 1}}, "upsert" : {"counter" : 1}}

```


## 参考

1. http://pap-docs.pap.net.cn/
2. https://gitee.com/alexgaoyh/
