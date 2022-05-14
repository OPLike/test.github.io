# SpringBoot集成mongo MongoDBTemplate 增删改查





## 添加依赖

```xml
<!-- springboot 整合 mongodb   -->
<dependency> 
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-mongodb</artifactId>
    <version>2.0.0.RELEASE</version>
</dependency>
```



## 引用

```java
@Autowired
private MongoTemplate mongoTemplate;
```



## 插入数据

插入一个集合

insert第二个参数为集合名称

```java
List<Student> stuList = new ArrayList<>();
stuList.add(new Student(1,"小明","18888888888"));
mongoTemplate.insert(stuList, "student");
```



插入一个实体类

```java
Student student = new Student(1,"小明","18888888888");
mongoTemplate.insert(student, "student");
```





## 查询数据

### 精确匹配

```java
// 查询单个数据
JSONObject doc = mongoTemplate.findById(id, JSONObject.class, "DEV_ALARM_LIST");

public Student findMongo() {
    Query query = new Query(Criteria.where("phone").is("18888888888"));
    Student findOne = mongoTemplate.findOne(query, Student.class, "student");
    return findOne;
}
```



### 模糊匹配

```java
public Student findMongo() {
    Pattern pattern = Pattern.compile("^.*8$",Pattern.CASE_INSENSITIVE);
    Query query = new Query(Criteria.where("phone").regex(pattern));
    Student findOne = mongoTemplate.findOne(query,Student.class,"student");
    return findOne;
}
```



### 多条件查询

```java
// 查询列表
List<JSONObject> docList = mongoTemplate.find(query, JSONObject.class, "DEV_ALARM_LIST");

public List<Student> findList(){
    Criteria c = new Criteria();
    c.and(key).is(value);
    c.and(key).is(value);
    Query query = Query.query(c);
    List<Student> findList = mongoTemplate.find(query, Student.class,"student");
    return findList;
}
```



### 分页

```java
// 分页查询
int startPage = pageIndex != null ? pageIndex : 1;
int pageDtoSize = pageSize != null ? pageSize : 10;
int startCount = (startPage - 1) * pageDtoSize;
Query query = new Query();
query.addCriteria(new Criteria().and("字段名").is(1));
query.skip(startCount).limit(pageDtoSize).with(Sort.by(Sort.Order.desc("alarmTime")));
List<JSONObject> docList = mongoTemplate.find(query, JSONObject.class, "DEV_ALARM_LIST");

public List<Student> findList(){
    Criteria c = new Criteria();
    c.and(key).is(value);
    c.and(key).is(value);
    Query query = Query.query(c);
    // 设置分页
    query.skip().limit();
    List<Student> findList = mongoTemplate.find(query, Student.class,"student");
    return findList;
}
```

### 排序

```java
//查询单个数据
public Student findMongo() {
    Query query = new Query(Criteria.where("phone").is("18888888888"));
    query.with(new Sort(Sort.Direction.DESC,"phone"));
    Student findOne = mongoTemplate.findOne(query,Student.class,"student");
    return findOne;
}
```

### 统计

```java
// 查询统计query条件下的数量
public long  findMongo() {
    Query query = new Query();
	query.addCriteria(new Criteria().and("字段名").is(1));
    return Long.valueOf(mongoTemplate.count(query, "mongo表名")).intValue();
}
```



## 更新

### 更新文档

```java
// 更新
Update update = new Update();
// 已有key则更新，没有会添加
update.set(ConstantPool.AUDIT_LEVEL, 0);
mongoTemplate.updateMulti(query, update, String.class, "DEV_ALARM_LIST");

public int update() {
	Query query = new Query(); 
	query.addCriteria(Criteria.where("_id").is(1));  //_id区分引号 "1"和1
	Update update = Update.update("name", "zzzzz");
//	WriteResult upsert = mongoTemplate.updateMulti(query, update, "userList"); //查询到的全部更新
//	WriteResult upsert = mongoTemplate.updateFirst(query, update, "userList"); //查询更新第一条
	WriteResult upsert = mongoTemplate.upsert(query, update, "userList");      //有则更新，没有则新增
	return upsert.getN();       //返回执行的条数
}
```



### 内嵌文档

#### 添加内嵌文档

```java
//添加内嵌文档数据（有则直接加入，没有则进行新增）
public int update1() {
    Query query = Query.query(Criteria.where("_id").is("11"));
    Student student = new Student(1,"小明","18888888888");
    Update update = new Update();
    update.addToSet("student", student);
    WriteResult upsert = mongoTemplate.upsert(query, update, "student");
    return upsert.getN();
}
```



#### 修改内嵌文档

```java
//修改内嵌文档中数据
public int update2() {
	//查询_id为11并且其中userList文档的_id为1的
	Query query = Query.query(Criteria.where("_id").is("11").and("users._id").is(1));
	Update update = Update.update("users.$.name", "zhangsan");
	WriteResult upsert = mongoTemplate.upsert(query, update, "student");
	return upsert.getN();
}
```

#### 删除内嵌文档数据

```java
//删除内嵌文档中数据
public int delete() {
	//查询_id为11并且其中userList文档的_id为1的
	Query query = Query.query(Criteria.where("_id").is("11").and("users._id").is(1));
	Update update = new Update();
	update.unset("users.$");
	WriteResult upsert = mongoTemplate.updateFirst(query, update, "student");
	return upsert.getN();
}
```



## 删除文档

```java
//查询单个数据
public Student findMongo() {
    Query query = new Query(Criteria.where("phone").is("18888888888"));
    mongoTemplate.remove(query,Student.class,"student");
}
```

