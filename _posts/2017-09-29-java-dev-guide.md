---
layout: post
title:  Java Dev Guide
date:  2017-9-29 08:46:05 +0800
categories: [Java]
tags: [ali, java]
published: true
---


# Java Dev Guide

> [阿里巴巴 Java 开发手册下载](http://techforum-img.cn-hangzhou.oss-pub.aliyun-inc.com/%E9%98%BF%E9%87%8C%E5%B7%B4%E5%B7%B4Java%E5%BC%80%E5%8F%91%E6%89%8B%E5%86%8C%EF%BC%88%E7%BB%88%E6%9E%81%E7%89%88%EF%BC%89.pdf)

愿景：码出高效，码出质量。

此篇规范可以作为自己的编写规范。也可留作日后 team 的开发规范。

以后所有项目，直接引入这一篇规范。


# 编程风格
  
## 命名风格

- 【强制】命名不可以 `_`/`$` 为开始或者结束。

- 【强制】禁止拼音与英文混淆。建议纯英文。

- 【强制】类名遵循 UpperCamelCase。除却 DO/BO/DTO/VO/AO

- 【强制】方法名、参数名、成员变量、局部变量统一使用 lowerCamelCase

- 【强制】常量命名(枚举)全部大写，`_` 分隔。力求完整清晰。

- 【强制】抽象类名以 Abstract/Base 开头。异常类以 Exception 结尾。测试类以类名开始，Test 结尾。

这是一种约定，可以快速分类出类属于哪一部分。比如约定工具类都以 `Util` 结尾。

- 【强制】POJO 类布尔变量，不要加 is。否则部分框架解析会导致序列化错误。

- 【强制】包名统一小写。`.` 分隔符之间仅有一个自然语义的英文单词。包名统一**单数**形式。类名可用复数。

- 【强制】杜绝不规范的缩写。

英文名过长，可进行对应缩写。约定对应缩写规范。

- 【推荐】为了代码自解释。任意元素命名，尽量为完整语义。

- 【推荐】若模块、接口、类、方法使用了设计模式，可以直接命名时体现。

- 【推荐】接口方法/属性不要加任何修饰符。(public 也不应该，因为默认就是。)

- 接口与实现

【强制】Service/DAO 类，基于 [SOA](https://www.ibm.com/developerworks/cn/webservices/ws-soa-design/) 理念。
暴露接口，实现为 Impl 后缀与之区分。

【推荐】形容能力的接口，(-able) 的形式。


- 【推荐】枚举使用 Enum 后缀，常量使用 Constant 后缀。

- Service/DAO 命名规约

觉得原来的文档设计的不够好。可以从区分 DAO/Service 的角度来命名。其他可参考原文档。

DAO: insert/delete/update/select 与 SQL 保持一致。

Service: save/remove/edit/query 等与服务语义保持一致。(可自行约定)

-  领域模型命名规范

1. 数据对象。xxxDO, xxx 为表名

2. 数据传输对象。xxxDTO, xxx 为业务领域相关名称

3. 展示对象。xxxVo， xxx 一般为网页名称。

4. POJO 是以上统称，禁止命名 xxxPOJO。

## 常量定义

- 【强制】代码中禁止直接出现魔法值。(未经定义的常量)

- 【强制】Long 变量初始化时，后缀应使用大写 L。

- 【推荐】常量类应该按照功能归类，分开维护。

- 【推荐】常量的复用广度不同，可分为以下 5 层：

1、跨应用共享变量。放在二方库中。如 client.jar/constant

2、应用内共享变量。放在一方库中。如 modules/constant

3、子工程内共享变量。当前子工程 constant 下。

4、包内共享常量。当前包下单独的 constant 下。

5、类内共享常量。类内部：private static final。

- 【推荐】枚举优于常量就不多言了。

可以多看看《Effective Java》之类的书。

## 代码风格

- 【强制】大括号，默认编辑器格式。

- 【强制】左右括号之间与字符之间不应该有空格。

正：if (a == b)

反： if ( a == b )

- 【强制】if/for/while/switch/do 等保留字与括号之间**必须**加空格。

- 【强制】任何二目、三目运算符的左右两边都需要加一个空格。

- 【强制】采用 4 空格缩进，禁止使用 tab。

IDE 设置 tab 为 4 个空格时，请不要勾选 **Use tab character**;


- 【强制】注释的 `//` 与注释内容之间，**有且仅有**一个空格。

- 【强制】单行字符数不超过 120。(IDE 有个分割线)

1. 换行后。第二行缩进 4 个空格。第三行则不用继续缩进。


- 【强制】多个参数定义传入时，多个参数 `,` 后必须加空格。

- 【强制】IDE 编码统一为 `UTF-8`，换行符统一为 `Unix`，而不是 `Windows`。

- 【推荐】没有必要增加若干空格使得字符对齐。

- 【推荐】不同的语义之间，插入一行空格分隔即可。相同逻辑和语义不需要。

## OOP

- 【强制】避免通过类的对象引用调用访问其静态方法/变量(会增加编译器解析成本)，直接使用 ClassName 访问。

- 【强制】所有方法覆写，添加 `@Override` 注解。

- 【强制】尽量避免使用可变参数编程

有些场景就挺合适的，比如 Arrays.asList(...)

- 【强制】外部正在调用或者二方库依赖的接口，禁止修改方法签名。应使用 `@Depretched` 废弃掉，并注明最新的使用替代方案。

Java 的这个注解没有 .Net 的设计的好，无法注解中指定新的方法是什么。


- 【强制】请勿使用过时的方法/类

- 【强制】比较时，尽量使用确定的值和变量对比。推荐使用 Objects.equals();

正：  

```java
"test".equals(obejct)
```

- 【强制】相同类型的包装类值比较，必须使用 equals();

Integer val = ? (-128~127) 会复用已有对象，大坑。

- 【强制】所有 POJO 都必须为包装数据类型，且不要设置默认值。

保证有些值入库可以为 null，所有校验由使用者保证。

- 【推荐】所有局部变量使用基本数据类型。

- 【强制】序列化类，不要修改 `serialVersionUID`，避免反序列化失败

看过一本书，应该是 《Thinking in Java》。类中其实不需要这个变量，个人建议不要有此属性。
 
- 【强制】构造方法中禁止有业务代码。如果有初始化逻辑，放在 init() 中

- 【强制】POJO 必须添加 toString() 方法。为了日志更容易追踪。

个人建议使用 Commons-lang3 反射实现 ToString(); 自己写项目，为了简单可以使用 lombok。

有一天，系统需要对有些字段进行脱敏。如用户名，手机号，密码。统一使用 Commons-lang3 性能消耗不高，可以统一生成。
对某些字段添加字段，就可脱敏，方便统一管理。当然日志脱敏也是可行的。

- 【推荐】类中的方法应该按照名称顺序等，便于阅读的放在一起。

建议顺序：public > protect > private > getter/setter > toString() > equals()/hashcode()

- 【推荐】getter/setter 方法要保证纯粹。

这一点很好，比如最近我想在 get 的时候，如果没有就默认给个初始值。是在 get 方法中加的业务逻辑，检讨。

个人想法：在获取到之后，在对属性进行默认值初始化等处理。

- 【推荐】 String 循环体字符串的拼接，使用 StringBuilder.append();

- 【推荐】类成员/方法的访问控制从严：

1) 仅仅本类使用，必须为 private;

2) 考虑与子类共享，则为 protect;

3) 不允许 new 来创建，则构造器必须 private;

一句话：尽量将访问级别控制在**最低**。


## 集合处理


- 【强制】关于 hashCode 和 equals 的处理，遵循如下规则:

1) 只要重写equals，就必须重写hashCode。

2) 因为Set存储的是不重复的对象，依据hashCode和equals进行判断，所以Set存储的 对象必须重写这两个方法。

3) 如果自定义对象做为Map的键，那么必须重写hashCode和equals。

- 【强制】ArrayList的subList结果不可强转成ArrayList，否则会抛出ClassCastException 异常，即java.util.Random


- 【强制】在 subList 场景中，高度注意对原集合元素个数的修改，会导致子列表的遍历、增加、 删除均会产生ConcurrentModificationException 异常。

子列表是个大坑，建议对原来的列表进行复制成一个新列表。

- 【强制】使用集合转数组的方法，必须使用集合的toArray(T[] array)，传入的是类型完全 一样的数组，大小就是 list.size()。

- 【强制】使用工具类 Arrays.asList() 把数组转换成集合时，不能使用其修改集合相关的方 法，它的 add/remove/clear 方法会抛出 UnsupportedOperationException 异常。 

说明: asList 的返回对象是一个 Arrays 内部类，并没有实现集合的修改方法。Arrays.asList 体现的是适配器模式，只是转换接口，后台的数据**仍是数组**。


- 【强制】PECS(Producer Extends Consumer Super)原则:

第一、频繁往外读取内 容的，适合用<? extends T>。

第二、经常往里插入的，适合用<? super T>

- 【强制】不要在 foreach 循环里进行元素的 remove/add 操作。remove 元素请使用 Iterator 方式，如果并发操作，需要对 Iterator 对象加锁。


- 【推荐】集合初始化时，指定集合初始值大小。

这个涉及到 HashMap 的原理。

HashMap使用HashMap(int initialCapacity) 初始化，

正例: `initialCapacity = (需要存储的元素个数 / 负载因子) + 1`。注意负载因子(即 loader factor)默认为 0.75，如果暂时无法确定初始值大小，请设置为 16(即默认值)。

也就是说，当数量较多时，应该设置初始值。为：initialCapacity = (num / 0.75)+1;


- 【推荐】使用 entrySet 遍历 Map 类集合 KV，而不是 keySet 方式进行遍历。
 
> DESC

values() 返回的是 V 值集合，是一个 list 集合对象; keySet() 返回的是 K 值集合，是 一个 Set 集合对象; entrySet() 返回的是 K-V 值组合集合。


## 并发处理


- 【强制】获取单例对象需要保证线程安全，其中的方法也要保证线程安全。

- 【强制】创建线程或线程池时请指定有意义的线程名称，方便出错时回溯。

- 【强制】线程资源必须通过线程池提供，不允许在应用中自行显式创建线程。

- 【强制】线程池不允许使用 Executors 去创建，而是通过 ThreadPoolExecutor 的方式，这样 的处理方式让写的同学更加明确线程池的运行规则，规避资源耗尽的风险。 

说明:Executors 返回的线程池对象的弊端如下:

1)FixedThreadPool 和 SingleThreadPool:

允许的请求队列长度为 Integer.MAX_VALUE，可能会堆积大量的请求，从而导致 OOM。 

2)CachedThreadPool 和 ScheduledThreadPool:

允许的创建线程数量为 Integer.MAX_VALUE，可能会创建大量的线程，从而导致 OOM。


- SimpleDateFormat 是线程不安全的类，一般不要定义为static变量，如果定义为static，必须加锁，或者使用 DateUtils 工具类。

> DESC

如果是 JDK8 的应用，可以使用 Instant 代替 Date，LocalDateTime 代替 Calendar， DateTimeFormatter 代替 SimpleDateFormat，
官方给出的解释:simple beautiful strong immutable thread-safe。

- 【强制】尽可能使加锁的代码块工作量尽可能的小，避免在锁代码块中调用 RPC 方法。

类似的。在事务中尽量不要有 bean 的构建，外部的调用。这些尽量放在事务外面。

- 【强制】并发修改同一记录时，避免更新丢失，需要加锁。如果每次访问冲突概率小于 20%，推荐使用乐观锁，否则使用悲观锁。乐观锁的重试次数不得小于 3 次。


- 【强制】多线程并行处理定时任务时，Timer 运行多个 TimeTask 时，只要其中之一没有捕获 抛出的异常，其它任务便会自动终止运行，使用 `ScheduledExecutorService` 则没有这个问题。


- 【推荐】避免 Random 实例被多线程使用，虽然共享该实例是线程安全的，但会因竞争同一 seed 导致的性能下降。

在 JDK7 之后，可以直接使用 API ThreadLocalRandom。

- 【推荐】在并发场景下，通过双重检查锁(double-checked locking)实现延迟初始化的优 化问题隐患(可参考 The "Double-Checked Locking is Broken" Declaration)，
推荐解决方案中较为简单一种(适用于 JDK5 及以上版本)，将目标属性声明为 volatile 型。

- 【参考】volatile 解决多线程内存不可见问题。对于**一写多读，是可以解决变量同步问题**，但是如果多写，同样无法解决线程安全问题。

如果是 count++ 操作，使用如下类实现: AtomicInteger count = new AtomicInteger(); count.addAndGet(1); 

如果是 JDK8，推 荐使用 `LongAdder` 对象，比 AtomicLong 性能更好(减少乐观锁的重试次数)。

- 【参考】ThreadLocal 无法解决共享对象的更新问题，ThreadLocal 对象建议使用 **static** 修饰。

这个变量是针对一个线程内所有操作共享的，所以设置为静态变量，所有此类实例共享此静态变量 ，也就是说在类第一次被使用时装载，只分配一块存储空间，所有此类的对象(只要是这个线程内定义的)都可以操控这个变量。


## 控制语句

- 【推荐】除常用方法(如 getXxx/isXxx)等外，不要在条件判断中执行其它复杂的语句，将复 杂逻辑判断的结果赋值给一个有意义的布尔变量名，以提高可读性。

建议声明一个方法，并起一个好名字。

- 【推荐】循环体中的语句要考量性能，以下操作尽量移至循环体外处理，如定义对象、变量、获取数据库连接，进行不必要的 try-catch 操作(这个 try-catch 是否可以移至循环体外)。


- 【参考】下列情形，需要进行参数校验:

1) 调用频次低的方法。

2) 执行时间开销很大的方法。此情形中，参数校验时间几乎可以忽略不计，但如果因为参数错误导致中间执行回退，或者错误，那得不偿失。

3) 需要极高稳定性和可用性的方法。

4) 对外提供的开放接口，不管是RPC/API/HTTP接口。 

5) 敏感权限入口。

一般，所有对外接口都认为别人是恶意调用，必须入参校验。快速失败。

## 注释规约

- 【强制】类、类属性、类方法的注释必须使用 Javadoc 规范，使用 `/**内容*/` 格式，不得使用 `// xxx` 方式。

- 【强制】所有的抽象方法(包括接口中的方法)必须要用 Javadoc 注释、除了返回值、参数、异常说明外，还必须指出该方法做什么事情，实现什么功能。

- 【强制】所有的类都必须添加创建者和创建日期

便于日后甩锅。。。

- 【强制】所有的枚举类型字段必须要有注释，说明每个数据项的用途。

所有的常量和 POJO 也应该如此。

- 【推荐】与其“半吊子”英文来注释，不如用中文注释把问题说清楚。专有名词与关键字保持英文原文即可。

- 【参考】谨慎注释掉代码。在上方详细说明，而不是简单地注释掉。如果无用，则删除。

有 VCS 的出现，代码中应该**杜绝被注释掉的代码**，直接删掉。

- 【参考】特殊注释标记，请注明标记人与标记时间。注意及时处理这些标记，通过标记扫描， 经常清理此类标记。线上故障有时候就是来源于这些标记处的代码。

// todo 这个是很有用的代码。不过 Eclipse 下有些开发者，经常清理这些代码。

## 其他

- 【强制】在使用正则表达式时，利用好其预编译功能，可以有效加快正则匹配速度。

- 【强制】注意 Math.random() 这个方法返回是 double 类型，注意取值的范围 0≤x<1(能够 取到零值，注意除零异常)。

如果想获取整数类型的随机数，不要将 x 放大 10 的若干倍然后 取整，直接使用 Random 对象的 nextInt 或者 nextLong 方法。

- 【强制】获取当前毫秒数 System.currentTimeMillis(); 而不是 new Date().getTime(); 说明:如果想获取更加精确的纳秒级时间值，使用 System.nanoTime()的方式。
在 JDK8 中， 针对统计时间等场景，推荐使用 `Instant` 类。


# 异常日志

## 异常处理

-  【强制】Java 类库中定义的一类 RuntimeException 可以通过预先检查进行规避，而不应该 通过catch 来处理，比如:IndexOutOfBoundsException，NullPointerException等等。

上述写法可以让代码性能更好，但是代码变得很恶心。到处都是充满了判断，个人不太喜欢。

- 【强制】对于非稳定代码的 catch 尽可能进行区分 异常类型，再做对应的异常处理。

- 【强制】捕获异常是为了处理它，不要捕获了却什么都不处理而抛弃之，如果不想处理它，请 将该异常抛给它的调用者。最外层的业务使用者，必须处理异常，将其转化为用户可以理解的 内容。

或者落日志。

- 【强制】有 try 块放到了事务代码中，catch 异常后，如果需要回滚事务，一定要注意手动回 滚事务。

这个是 spring-tx 的小常识。应该自动回滚依赖于异常捕捉。

- 【强制】finally 块必须对资源对象、流对象进行关闭，有异常也要做 try-catch。

建议 JDK1.7+ 使用 TWR

- 【强制】不能在 finally 块中使用 return，finally 块中的 return 返回后方法结束执行，不会再执行 try 块中的 return 语句。

- 【推荐】定义时区分unchecked/checked 异常，避免直接抛出newRuntimeException()， 更不允许抛出 Exception 或者 Throwable，应使用有业务含义的自定义异常。
推荐业界已定义 过的自定义异常，如:DAOException / ServiceException等。

- 【参考】避免出现重复的代码(Don’t Repeat Yourself)，即DRY原则。

同样的代码写2次，意味着以后修改的时候要改2次。

往广处说，类似的事情尽量使用 AOP 也是这个道理。


## 日志规约

- 【强制】应用中不可直接使用日志系统(Log4j、Logback)中的 API，而应依赖使用日志框架
SLF4J 中的 API，使用门面模式的日志框架，有利于维护和各个类的日志处理方式统一。

考虑日志的性能，比如 Log4j2 也可放弃 SLF4j。就像是 DAO 层一样，难道说今天用 mysql，以后可能用 oracle。还要去用 hibernate? 平衡问题。

- 【强制】日志文件推荐至少保存 15 天，因为有些异常具备以“周”为频次发生的特点。

建议一直保留。

- 【强制】对 trace/debug/info 级别的日志输出，必须使用条件输出形式或者使用占位符的方 式。

有的日志，如 Log4j 是不能用 `{}` 的，所以应该放弃使用 Log4j，使用 Log4j2。以后有 Log4jN，就用最新的。
 
- 【推荐】谨慎地记录日志。

这些地方应该记录日志：

1. 调用的入参/出参。

2. 远程调用/数据库访问。

3. 异常日志/警告日志

尽量使得日志便于追踪，有些金融相关的系统很麻烦，日志尽量详细(门槛低)，有一定经验后，就是两个字——恰当。

# 单元测试

- 【强制】好的单元测试必须遵守 AIR 原则。

 A:Automatic(自动化): Unit test; 
 
 R:Repeatable(可重复): 保证数据库等多次跑没有数据污染。
 
 I:Independent(独立性)：单元测试用例之间 决不能互相调用，也不能依赖执行的先后次序。
 
 
建议和 CI 结合使用。


- 【强制】单元测试代码必须写在如下工程目录:src/test/java，不允许写在业务代码目录下。

不要在 java 中写个 main() 测试一下，建议将这些测试代码写成UT；

- 【推荐】编写单元测试代码遵守 BCDE 原则，以保证被测试模块的交付质量。

1.  B:Border，边界值测试，包括循环边界、特殊取值、特殊时间点、数据顺序等。 

2.  C:Correct，正确的输入，并得到预期的结果。

3.  D:Design，与设计文档相结合，来编写单元测试。

4.  E:Error，强制错误信息输入(如:非法数据、异常流程、非业务允许输入等)，并得 到预期的结果。


- 【推荐】对于数据库相关的查询，更新，删除等操作

可以使用内存运行的临时数据库进行测试。保证每次操作执行完数据都会回滚。

- 【推荐】单元测试作为一种质量保障手段，不建议项目发布后补充单元测试用例，建议在项 目提测前完成单元测试。

# 安全规约

- 【强制】权限验证

- 【强制】用户请求传入的任何参数必须做有效性验证。

page size 过大导致内存溢出

 恶意 order by 导致数据库慢查询
 任意重定向
 SQL 注入
 反序列化注入
 正则输入源串拒绝服务 ReDoS


- 【强制】表单、AJAX 提交必须执行 CSRF 安全过滤。

前段的 JS 是可以被人以修改的，只能提升用户体验，无法保证数据的准确性。

# MySQL

## 建表规约


- 【强制】表达是与否概念的字段，必须使用 is_xxx 的方式命名，数据类型是 unsigned tinyint ( 1表示是，0表示否)。

- 【强制】表名不使用复数名词

- 【强制】禁用保留字，如 desc、range、match、delayed 等，请参考 MySQL 官方保留字。

- 【强制】主键索引名为 pk_字段名;唯一索引名为 uk_字段名;普通索引名则为 idx_字段名。

idx_ 个人更喜欢 nk_(normal_key)


- 【强制】小数类型为 decimal，禁止使用 float 和 double。

- 【强制】varchar 是可变长字符串，不预先分配存储空间，长度不要超过 5000，如果存储长 度大于此值，定义字段类型为 text，独立出来一张表，用主键来对应，避免影响其它字段索 引效率。

- 【推荐】字段允许适当冗余，以提高查询性能，但必须考虑数据一致。

冗余字段应遵循: 
 
 1)不是频繁修改的字段。
 
 2)不是 varchar 超长字段，更不能是 text 字段。
 
- 【推荐】字段长度范围尽可能小。



## 索引规约

- 【强制】业务上具有唯一特性的字段，即使是多个字段的组合，也必须建成唯一索引。

正式的线上库，必须有一个 非 ID，用来唯一标识当前记录的唯一序列号。

- 【强制】超过三个表禁止 join。需要 join 的字段，数据类型必须绝对一致;多表关联查询时， 保证被关联的字段需要有索引。

如果超过 3 张表怎么办？

- 【强制】在 varchar 字段上建立索引时，必须指定索引长度，没必要对全字段建立索引，根据 实际文本区分度决定索引长度即可。 

说明:索引的长度与区分度是一对矛盾体，一般对字符串类型数据，长度为 20 的索引，区分度会高达 90%以上，可以使用 count(distinct left(列名, 索引长度))/count(*)的区分度来确定。


- 【强制】页面搜索**严禁**左模糊或者全模糊，如果需要请走搜索引擎来解决。 

说明:索引文件具有 B-Tree 的最左前缀匹配特性，如果左边的值未确定，那么无法使用此索引。

- 【推荐】如果有 order by 的场景，请注意利用索引的有序性。order by 最后的字段是组合 索引的一部分，并且放在索引组合顺序的最后，

避免出现 file_sort 的情况，影响查询性能。 

正例:where a=? and b=? order by c; 索引:a_b_c


- 【推荐】SQL 性能优化的目标:至少要达到 range 级别，要求是 ref 级别，如果可以是 consts 最好。

说明:

1) consts 单表中最多只有一个匹配行(主键或者唯一索引)，在优化阶段即可读取到数据。 

2) ref 指的是使用普通的索引(normal index)。

3) range 对索引进行范围检索。


- 【推荐】建组合索引的时候，区分度最高的在最左边。

正例:如果 where a=? and b=? ，a 列的几乎接近于唯一值，那么只需要单建 idx_a 索引即 可。


## SQL

- 【强制】不要使用 count(列名)或 count(常量)来替代 count(*)，count(*)是 SQL92 定义的 标准统计行数的语法，跟数据库无关，跟 NULL 和非 NULL 无关。 

说明:count(*)会统计值为 NULL 的行，而 count(列名)不会统计此列为 NULL 值的行。
  
  
- 【强制】count(distinct col) 计算该列除 NULL 之外的不重复行数，注意 count(distinct col1, col2) 如果其中一列全为NULL，那么即使另一列有不同的值，也返回为0。


- 【强制】当某一列的值全是 NULL 时，count(col)的返回结果为 0，但 sum(col)的返回结果为 NULL，因此使用 sum()时需注意 NPE 问题。 

正例:可以使用如下方式来避免sum的NPE问题: SELECT IF(ISNULL(SUM(g)),0,SUM(g)) FROM table;

- 【强制】使用 ISNULL()来判断是否为 NULL 值。

- 【强制】 在代码中写分页查询逻辑时，若 count 为 0 应直接返回，避免执行后面的分页语句。

- 【强制】不得使用外键与级联，一切外键概念必须在应用层解决。

- 【强制】禁止使用存储过程，存储过程难以调试和扩展，更没有移植性。

- 【强制】数据订正时，删除和修改记录时，要先 select，避免出现误删除，确认无误才能执行更新语句。

update 有时候会出现**死锁**。不同的查法，按照不同的索引，可能查出相同的数据，锁的顺序不一致就会导致死锁。

- 【推荐】in 操作能避免则避免，若实在避免不了，需要仔细评估 in 后边的集合元素数量，控制在 1000 个之内。

- 【参考】如果有全球化需要，所有的字符存储与表示，均以 utf-8 编码，注意字符统计函数 的区别。

## ORM

- 【强制】在表查询中，一律不要使用 `*` 作为查询的字段列表，需要哪些字段必须明确写明。

说明:

1)增加查询分析器解析成本。

2)增减字段容易与 resultMap 配置不一致。


-【强制】POJO 类的布尔属性不能加 is，而数据库字段必须加 is_，要求在 resultMap 中进行 字段与属性之间的映射。

说明:参见定义 POJO 类以及数据库字段定义规定， 在<resultMap> 中增加映射，是必须的。 在MyBatis Generator生成的代码中，需要进行对应的修改。

- 【强制】不要用 resultClass 当返回参数，即使所有类属性名与数据库字段一一对应，也需 要定义;反过来，每一个表也必然有一个与之对应。

说明:配置映射关系，使字段与 DO 类解耦，方便维护。

可以为不同的查询制定一个 XXXResult, 但是维护量较大。

-【强制】sql.xml 配置参数使用:#{}，#param# 不要使用${} 此种方式容易出现 SQL 注入。

-【强制】iBATIS自带的queryForList(String statementName,int start,int size)不推荐使用。

说明:其实现方式是在数据库取到 statementName 对应的 SQL 语句的所有记录，再通过 subList 取 start,size 的子集合。

正例:

```java
Map<String, Object> map = new HashMap<String, Object>(); map.put("start", start);
map.put("size", size);
```



- 【强制】不允许直接拿 HashMap 与 Hashtable 作为查询结果集的输出。

说明:resultClass=”Hashtable”，会置入字段名和属性值，但是值的类型不可控。 ——禁止用于商业用途，违者必究——

- 【强制】更新数据表记录时，必须同时更新记录对应的 gmt_modified 字段值为当前时间。

这里可以再 mySQL 中设置，让这个字段当记录被修改时就会自动变化。

- 【推荐】不要写一个大而全的数据更新接口。传入为 POJO 类，不管是不是自己的目标更新字 段，
都进行 update table set c1=value1,c2=value2,c3=value3; 这是不对的。
执行 SQL 时，不要更新无改动的字段，一是易出错;二是效率低;三是增加 binlog 存储。


- 【参考】@Transactional 事务不要滥用。事务会影响数据库的 QPS，另外使用事务的地方需 要考虑各方面的回滚方案，包括缓存回滚、搜索引擎回滚、消息补偿、统计修正等。

- 【参考】<isEqual>中的 compareValue 是与属性值对比的常量，一般是数字，表示相等时带 上此条件;<isNotEmpty>表示不为空且不为 null 时执行;<isNotNull>表示不为 null 值时 执行。




































   
   































* any list
{:toc}












 

