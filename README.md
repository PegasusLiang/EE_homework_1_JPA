## SpringBoot JPA Demo
### 完成的点
实现了基础的注册登录功能，同时展示t_gym跟t_user两表的数据。

实现了两表关联，UserEntity跟Gym是多对一的关系，在Gym类中建立关联。

实现Spring Cache，多次查询在第一次查询后不再从数据库中查询。

实现了分页，但未在前端页面显示。

实现了Auditing，创建时间，更新时间自动生成。

前端的展示我用了thymeleaf来映射数据。

### 注册登录

![1556506128732](http://github.com/PegasusLiang/EE_homework_1_JPA/raw/master/作业截图/1556506128732.png)

![1556506151019](http://github.com/PegasusLiang/EE_homework_1_JPA/raw/master/作业截图/1556506151019.png)

登录信息的验证通过重现JPA的方法，findByUsernameAndPassword

![1556506976017](http://github.com/PegasusLiang/EE_homework_1_JPA/raw/master/作业截图/1556506976017.png)



### 数据展示

![1556506189869](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\1556506189869.png)

![1556506198204](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\1556506198204.png)

Gym跟User的数据展示，其中用户跟场馆是多对一的关系。

一端(UserEntity)使用@OneToMany,多端(Gym)使用@ManyToOne。关系由Gym场馆维护

![1556506371299](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\1556506371299.png)

使用@ManyToOne和@JoinColumn来注释属性,ManyToOne即表示此UserEntity为多端，@JoinColumn设置在gym表中的关联外键。

![1556506509999](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\1556506509999.png)

之前也尝试以多对多的关系来对应，也成功的对应上了。但是比较麻烦的是在前端上的展示，我对于thymeleaf是在不熟，网上例子也没改对。多对多关系需要建立中间表，通过jpa可以自动帮我们创建两表间的对应关系。

![1556506671279](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\1556506671279.png)

### 数据查询

数据查询我只做了路径映射，在地址栏输入进行跳转查询。

如通过地址查询运动馆信息。

![1556506772722](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\1556506772722.png)

![1556506830869](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\1556506830869.png)



### SpringCache

```java
    @Autowired
    GymJPA gymJPA;

    @Caching(
            cacheable = {@Cacheable(key="#name")},
            put={ @CachePut(key = "#result.id"),
                    @CachePut(key = "#result.address")
            }
    )
    public Gym queryGymByName(String name){
        System.out.println("查询的名字："+name);
        return gymJPA.queryByName(name);
    }

    @Transactional
    public List<Gym> getAllgyms() {
        return (List<Gym>) gymJPA.findAll();
    }

    @Transactional
    @Cacheable(key = "#address")
    public List<Gym> findByAddress(String address) {
        return gymJPA.nativeQuery(address);
    }

    @Transactional
    @Cacheable(key = "#id")
    public Gym getById(Long id) {
        return gymJPA.findById(id).get();
    }

    @Transactional
    public void deletePerson(Long personId) {
        gymJPA.deleteById(personId);
    }

    @Transactional
    @CacheEvict(cacheNames = "gym",allEntries = true)
    public boolean addGym(Gym gym) {
        return gymJPA.save(gym) != null;
    }
```

发现在通过地址查询测试时，第二次后在后台不会再调用sql语句

![1556507458931](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\1556507458931.png)
