## Java使用MongoDB

### 1. Java访问mongo

#### 1.1 pom

```xml
<dependency>
	<groupId>org.mongodb</groupId>
	<artifactId>mongo-java-driver</artifactId>
	<version>3.12.14</version>
</dependency>
```

#### 1.2 代码

```java
public class MongoDBTest {
    public static void main (String[] args) {
        addDoc();
        find();
        
    }
    private static final MongoCollection<Document> collection;
    
    static{
        MongoClient mongoClient = new MongoClient(new MongoClientURI("mongodb://root:123456@localhost:27017"));
        MongoDatabase mongoDatabase = mongoClient.getDatabase("hello");
        collection = mongoDatabase.getCollection("user");
    }
    
    //添加文档
    public static void addDoc(){
        Document d1 = Document.parse("{name:'xiaobei',age:11,birthday:new ISODate('2020-01-01')}");
        Document d2 = Document.parse("{name:'laobai',age:23,birthday:new ISODate('2021-11-21')}");
        Document d3 = Document.parse("{name:'laoxing',age:30,birthday:new ISODate('2022-07-19')}");
        collection.insertOne(d1);
        collection.insertOne(d2);
        collection.insertOne(d3);
    }
    
    //查询文档
    public static void find(){
        FindIterable<Document> findList = collection.find().sort(Document.parse("{age:-1}"));
        for ( Document document:findList ){
            System.out.println(document);
        }
    }
    
}
```

### 2. SpringBoot整合mongo

#### 1.1 引入依赖

```xml
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-data-mongodb</artifactId>
</dependency>
```

#### 1.2 配置

```yaml
spring:
  data:
    mongodb:
      host: localhost
      port: 27017
      username: root
      password: 123456
      authentication-database: admin
      database: hello
```

#### 1.2 实体类

```java 
public class User {
    private String name;
    private Integer age;
    private Date birthday;
}
```

#### 1.3 测试

```java
@SpringBootTest
@RunWith(SpringRunner.class)
public class MongoDBTest {
@Autowired
    private MongoTemplate mongoTemplate;
    
    @Test
    public void insert() throws Exception {
        try{
            User user1 = new User("aa",11,new Date());
            User user2 = new User("bb",21,new SimpleDateFormat("yyyy-MM-dd").parse("2010-01-01"));
            mongoTemplate.insert(user1);
            mongoTemplate.insert(user2);
        }catch ( Exception e ){
            e.printStackTrace();
        }
        
    }
    
    @Test
    public void find(){
        try{
            Query query = new Query(Criteria.where("name").is("aa"));
            List<User> users = mongoTemplate.find(query, User.class);
            System.out.println(users);
        }catch ( Exception e ){
            e.printStackTrace();
        }
    }
}
```