1.*连接MongoDB*

连接 MongoDB 我们需要使用 PyMongo 库里面的 MongoClient，一般来说传入 MongoDB 的 IP 及端口即可，第一个参数为地址 host，第二个参数为端口 port，端口如果不传默认是 27017。
```
import pymongo
client = pymongo.MongoClient(host='localhost', port=27017)
```
MongoClient 的第一个参数 host 还可以直接传MongoDB 的连接字符串，以 mongodb 开头，例如：
```
client = MongoClient('mongodb://localhost:27017/')
```

2.*指定数据库*   

MongoDB 中还分为一个个数据库,以 test 数据库为例进行说明，所以下一步需要在程序中指定要使用的数据库.
```
db = client.test
```
调用 client 的 test 属性也可返回 test 数据库。
```
db = client['test']
```

3.*指定集合*   

MongoDB 的每个数据库又包含了许多集合 Collection，也就类似与关系型数据库中的表，指定一个集合名称为 students，学生集合，还是和指定数据库类似，指定集合也有两种方式：
```
collection = db.students
```
```
collection = db['students']
```

4*插入数据*   

```
student = {
    'id': '20170101',
    'name': 'Jordan',
    'age': 20,
    'gender': 'male'
}


result = collection.insert(student)
print(result)
```
MongoDB 中，每条数据其实都有一个 _id 属性来唯一标识，如果没有显式指明 _id，MongoDB 会自动产生一个 ObjectId 类型的 _id 属性。insert() 方法会在执行后返回的 _id 值。

同时插入多条数据，只需要以列表形式传递即可
```
student1 = {
    'id': '20170101',
    'name': 'Jordan',
    'age': 20,
    'gender': 'male'
}

student2 = {
    'id': '20170202',
    'name': 'Mike',
    'age': 21,
    'gender': 'male'
}

result = collection.insert([student1, student2])
print(result)
```
结果:
```
[ObjectId('5932a80115c2606a59e8a048'), ObjectId('5932a80115c2606a59e8a049')]
```

推荐使用 insert_one() 和 insert_many() 方法将插入单条和多条记录分开

5.*查询*   

利用 find_one() 或 find() 方法进行查询，find_one() 查询得到是单个结果，find() 则返回一个生成器对象。
```
result = collection.find_one({'name': 'Mike'})
print(type(result))
print(result)
```
运行结果：
```
<class 'dict'>
{'_id': ObjectId('5932a80115c2606a59e8a049'), 'id': '20170202', 'name': 'Mike', 'age': 21, 'gender': 'male'}
```
可以发现它多了一个 _id 属性，这就是 MongoDB 在插入的过程中自动添加的。
如果查询结果不存在则会返回 None。


对于多条数据的查询，我们可以使用 find() 方法
```
results = collection.find({'age': 20})
print(results)
for result in results:
    print(result)
```
```
<pymongo.cursor.Cursor object at 0x1032d5128>
{'_id': ObjectId('593278c115c2602667ec6bae'), 'id': '20170101', 'name': 'Jordan', 'age': 20, 'gender': 'male'}
{'_id': ObjectId('593278c815c2602678bb2b8d'), 'id': '20170102', 'name': 'Kevin', 'age': 20, 'gender': 'male'}
{'_id': ObjectId('593278d815c260269d7645a8'), 'id': '20170103', 'name': 'Harden', 'age': 20, 'gender': 'male'}
```
返回结果是 Cursor 类型，相当于一个生成器,遍历,每一个结果都是字典类型

更详细用法在可以在 MongoDB 官方文档找到：
[https://docs.mongodb.com/manual/reference/operator/query/](https://docs.mongodb.com/manual/reference/operator/query/)


6*排序*   
调用 sort() 方法，传入排序的字段及升降序标志
```
results = collection.find().sort('name', pymongo.ASCENDING)
print([result['name'] for result in results])
```
运行结果：
```
['Harden', 'Jordan', 'Kevin', 'Mark', 'Mike']
```
调用了 pymongo.ASCENDING 指定升序，如果要降序排列可以传入 pymongo.DESCENDING

7.*偏移*   
可以利用skip() 方法偏移几个位置，比如偏移 2，就忽略前 2 个元素，得到第三个及以后的元素。
```
results = collection.find().sort('name', pymongo.ASCENDING).skip(2)
print([result['name'] for result in results])
```
```
['Kevin', 'Mark', 'Mike']
```
还可以用 limit() 方法指定要取的结果个数
```
results = collection.find().sort('name', pymongo.ASCENDING).skip(2).limit(2)
print([result['name'] for result in results])
```
```
['Kevin', 'Mark']
```
8.*更新*   
分了 update_one() 方法和 update_many() 方法,第二个参数需要使用 $ 类型操作符作为字典的键名(关于$字符，建议google)

```
condition = {'name': 'Kevin'}
student = collection.find_one(condition)
student['age'] = 26
result = collection.update_one(condition, {'$set': student})
print(result)
print(result.matched_count, result.modified_count)
```
调用了 update_one() 方法，第二个参数不能再直接传入修改后的字典，而是需要使用 {'$set': student} 这样的形式，其返回结果是 UpdateResult 类型，然后调用 matched_count 和 modified_count 属性分别可以获得匹配的数据条数和影响的数据条数。
```
<pymongo.results.UpdateResult object at 0x10d17b678>
1 0
```
调用 update_many() 方法，则会将所有符合条件的数据都更新


9.*删除*   
直接调用 remove() 方法指定删除的条件,符合条件的所有数据均会被删除
```
result = collection.remove({'name': 'Kevin'})
print(result)
```
```
{'ok': 1, 'n': 1}
```
delete_one() 和 delete_many() 方法
```
result = collection.delete_one({'name': 'Kevin'})
print(result)
print(result.deleted_count)
result = collection.delete_many({'age': {'$lt': 25}})
print(result.deleted_count)
```

```
<pymongo.results.DeleteResult object at 0x10e6ba4c8>
1
4
```
delete_one() 即删除第一条符合条件的数据，delete_many() 即删除所有符合条件的数据，返回结果是 DeleteResult 类型，可以调用 deleted_count 属性获取删除的数据条数。
