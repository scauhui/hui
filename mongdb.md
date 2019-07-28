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
