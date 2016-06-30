# Comparison Query Operators for mongo DB

## find()に使える演算子
### 準備
テスト用のデータ

    $ mongo
    > use test;
    > db.createCollection("test");

    > db.test.insert({"name" : "tom", "id" : 1});
    > db.test.insert({"name" : "jack", "id" : 2});
    > db.test.insert({"name" : "john", "id" : 3});
    > db.test.insert({"name" : "mike", "id" : 4});

### ORDER BY
mongoDB内ではsortを使う。
```
db.[コレクション名].find().sort({x : n});  
```

nには1か-1を指定できる。
以下の例ではidについて昇順、降順でソートを行っている。

```
> db.test.find().sort({"id" : 1});
{ "_id" : ObjectId("56bc77ba75ea20ec73750742"), "name" : "tom", "id" : 1 }
{ "_id" : ObjectId("56bc77bf75ea20ec73750743"), "name" : "jack", "id" : 2 }
{ "_id" : ObjectId("56bc77c375ea20ec73750744"), "name" : "john", "id" : 3 }
{ "_id" : ObjectId("56bc77c875ea20ec73750745"), "name" : "mike", "id" : 4 }
```

```
> db.test.find().sort({"id" : -1});
{ "_id" : ObjectId("56bc77c875ea20ec73750745"), "name" : "mike", "id" : 4 }
{ "_id" : ObjectId("56bc77c375ea20ec73750744"), "name" : "john", "id" : 3 }
{ "_id" : ObjectId("56bc77bf75ea20ec73750743"), "name" : "jack", "id" : 2 }
{ "_id" : ObjectId("56bc77ba75ea20ec73750742"), "name" : "tom", "id" : 1 }
```

クエリ発行後、レコードがidについて昇順/降順で出力されているのがわかる。各レコードの先頭に表示されている"\_id"要素は、MongoDBが自動で付加している物であり、ユニークな値を持つ。  
sort内の第二引数で指定する値は-1でdesc(降順)で、1でasc(昇順)。  
その他の値を指定するとエラーになる。

### LIMIT
mongoDB内でもlimitを使う。
```
db.[コレクション名].find().limit(n);
```

nには上限に設定したい値を指定できる。例えば、limit(2)を指定すると、

```
> db.test.find().limit(2);
{ "_id" : ObjectId("56bc77ba75ea20ec73750742"), "name" : "tom", "id" : 1 }
{ "_id" : ObjectId("56bc77bf75ea20ec73750743"), "name" : "jack", "id" : 2 }
```

以上のように出力されるレコードは挿入した順番に先頭から二つだけになる。

### BETWEEN
mongoDBではfindの引数で条件式を指定する。
```
db.[コレクション名].find({[対象としたいカラム] : {"$gt": n, "$lt": m}});
```
find()の引数に指定しているのは言わば条件式で、nには最小値、mには最大値が入る。  
条件式内で使うことのできる範囲指定記号は以下になる:  
- "$gt" → 指定した値を超える値
- "$gte" -> 指定した値以上の値
- "$lt" → 指定した値未満の値
- "$lte" → 指定した値以下の値

例えば、

```
db.test.find({"id" : {"$gt": 2, "$lte" : 4}});
{ "_id" : ObjectId("56bc77c375ea20ec73750744"), "name" : "john", "id" : 3 }
{ "_id" : ObjectId("56bc77c875ea20ec73750745"), "name" : "mike", "id" : 4 }
```
この例ではidが2を超える値かつ4以下の値を持つレコードが出力される。

### IN
mongoDBではfindの引数内で$in演算子を用いる。
```
db.[コレクション名].find([対象としたいカラム] : {$in : [n1, n2, n3, ...]});
```
find()内の引数で$inを用いて指定している。n1, n2, n3に値を指定すればレコードと一致する値が存在した場合、一致したレコードを返却する。

例えば、

```
db.test.find({"id" : {$in : [1, 3, 4]}});
{ "_id" : ObjectId("56bc77ba75ea20ec73750742"), "name" : "tom", "id" : 1 }
{ "_id" : ObjectId("56bc77c375ea20ec73750744"), "name" : "john", "id" : 3 }
{ "_id" : ObjectId("56bc77c875ea20ec73750745"), "name" : "mike", "id" : 4 }
```
以上の例では、idが1と3と4であるレコードを出力が出力される。

### OFFSET
mongoDB内ではskipを使う。
```
db.[コレクション名].find().skip(n);
```
nに指定した値分結果をスキップする。

例えば、skip(1)としたクエリを発行すると、

```
db.test.find().skip(1);
{ "_id" : ObjectId("56bc77bf75ea20ec73750743"), "name" : "jack", "id" : 2 }
{ "_id" : ObjectId("56bc77c375ea20ec73750744"), "name" : "john", "id" : 3 }
{ "_id" : ObjectId("56bc77c875ea20ec73750745"), "name" : "mike", "id" : 4 }
```

のように挿入した順番で言う最初のレコードは飛ばされて3つのレコードしか出力されない。
limitと使い合わせることでレコードがソートされた状態で中間に位置するレコードのデータを取得することができる。

```
db.test.find().sort({"id":1}).skip(1).limit(2);
{ "_id" : ObjectId("56bc77bf75ea20ec73750743"), "name" : "jack", "id" : 2 }
{ "_id" : ObjectId("56bc77c375ea20ec73750744"), "name" : "john", "id" : 3 }
```

以上の例ではid順にソートされたレコードのうち、中間に位置する二つのみが返って来ていることがわかる。

### 参考URL
http://qiita.com/ANTON072/items/e0534daa4b2fb0f553eb  
http://taka512.hatenablog.com/entry/20110220/1298195574  
http://www.cuspy.org/diary/2012-04-17/the-little-mongodb-book-ja.pdf  
https://docs.mongodb.org/v3.0/reference/operator/query-comparison/  
