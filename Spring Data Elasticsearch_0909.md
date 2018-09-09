## Spring Data Elasticsearch
BioMed Central Development Team Version 3.0.9.RELEASE, 2018-07-26

---
© 2013-2015 The original author(s).

```text
Copies of this document may be made for your own use and for distribution to others, 
provided that you do not charge any fee for such copies and further provided that 
each copy contains this Copyright Notice, whether distributed in print or electronically.

本文件的副本可供您自行使用并分发给他人，但您无需为此收取任何费用，
并可在每份副本中包含本版权声明，无论以印刷形式或电子形式分发。
```

**Table of Contents**

```
Preface
Project Metadata
Requirements
1. Working with Spring Data Repositories
  1.1. Core concepts
  1.2. Query methods
  1.3. Defining Repository Interfaces
    1.3.1. Fine-tuning Repository Definition
    1.3.2. Null Handling of Repository Methods
    1.3.3. Using Repositories with Multiple Spring Data Modules
  1.4. Defining Query Methods
    1.4.1. Query Lookup Strategies
    1.4.2. Query Creation
    1.4.3. Property Expressions
    1.4.4. Special parameter handling
    1.4.5. Limiting Query Results
    1.4.6. Streaming query results
    1.4.7. Async query results
  1.5. Creating Repository Instances
    1.5.1. XML configuration
    1.5.2. JavaConfig
    1.5.3. Standalone usage
  1.6. Custom Implementations for Spring Data Repositories
    1.6.1. Customizing Individual Repositories
    1.6.2. Customize the Base Repository
  1.7. Publishing Events from Aggregate Roots
  1.8. Spring Data Extensions
    1.8.1. Querydsl Extension
    1.8.2. Web support
    1.8.3. Repository Populators
Reference Documentation
2. Elasticsearch Repositories
  2.1. Introduction
    2.1.1. Spring Namespace
    2.1.2. Annotation based configuration
    2.1.3. Elasticsearch Repositores using CDI
  2.2. Query methods
    2.2.1. Query lookup strategies
    2.2.2. Query creation
    2.2.3. Using @Query Annotation
3. Miscellaneous Elasticsearch Operation Support
  3.1. Filter Builder
  3.2. Using Scan And Scroll For Big Result Set
Appendix
Appendix A: Namespace reference
The <repositories /> Element
Appendix B: Populators namespace reference
The <populator /> element
Appendix C: Repository query keywords
Supported query keywords
Appendix D: Repository query return types
Supported Query Return Types
```

## Preface

The Spring Data Elasticsearch project applies core Spring concepts to the development of solutions using the Elasticsearch Search Engine. We have povided a "template" as a high-level abstraction for storing,querying,sorting and faceting documents. You will notice similarities to the Spring data solr and mongodb support in the Spring Framework.

Spring Data Elasticsearch项目将Spring核心概念应用于使用Elasticsearch搜索引擎开发解决方案。我们提出了一个“模板”，作为存储、查询、排序和处理文档的高级抽象。您将注意到与Spring框架中的Spring Data solr和mongodb支持的相似之处。

## Project Metadata

* Version Control - https://github.com/spring-projects/spring-data-elasticsearch
* Bugtracker - https://jira.spring.io/browse/DATAES
* Release repository - https://repo.spring.io/libs-release
* Milestone repository - https://repo.spring.io/libs-milestone
* Snapshot repository - https://repo.spring.io/libs-snapshot

## Requirements
Requires Elasticsearch 0.20.2 and above or optional dependency or not even that if you are using Embedded Node Client

要求Elasticsearch 0.20.2或以上，或可选依赖项，甚至如果您使用的是嵌入式节点客户端

## 1. Working with Spring Data Repositories
The goal of the Spring Data Repository abstraction is to significantly reduce the amount of boilerplate code required to implement data access layers for various persistence stores.

Spring Data Repository 抽象的目标是显著减少实现各种持久性存储的数据访问层所需的样板代码数量。

```
Spring Data Repository documentation and your module

Spring Data存储库文档和模块

This chapter explains the core concepts and interfaces of Spring Data repositories. 
The information in this chapter is pulled from the Spring Data Commons module. 
It uses the configuration and code samples for the Java Persistence API (JPA) module. 
You should adapt the XML namespace declaration and the types to be extended to 
the equivalents of the particular module that you use. 
“Namespace reference” covers XML configuration, which is supported across all 
Spring Data modules supporting the repository API. 
“Repository query keywords” covers the query method keywords supported by 
the repository abstraction in general. For detailed information on 
the specific features of your module, see the chapter on that module of this document.

本章将解释Spring Data Repository 的核心概念和接口。本章中的信息来自Spring Data Commons模块。它使用Java Persistence API (JPA)模块的配置和代码示例。您应该调整XML名称空间声明和类型，以便扩展到您使用的特定模块的等价物。“名称空间引用”涵盖了XML配置，支持存储库API的所有Spring Data module都支持XML配置。“存储库查询关键字”通常包括存储库抽象支持的查询方法关键字。有关模块的具体特性的详细信息，请参阅本文档中关于该模块的章节。
```

### 1.1. Core concepts
The central interface in the Spring Data Repository abstraction is Repository. It takes the domain class to manage as well as the ID type of the domain class as type arguments. This interface acts primarily as a marker interface to capture the types to work with and to help you to discover interfaces that extend this one. The CrudRepository provides sophisticated CRUD functionality for the entity class that is being managed.

Spring Data Repository 抽象中的中心接口是存储库。它将域类和域类的ID类型作为类型参数进行管理。此接口主要充当标记接口，用于捕获要使用的类型，并帮助您发现扩展此接口的接口。CrudRepository为被管理的实体类提供了复杂的CRUD功能。

Example 1. CrudRepository interface

```java
public interface CrudRepository<T, ID extends Serializable>
  extends Repository<T, ID> {

  <S extends T> S save(S entity);      

  Optional<T> findById(ID primaryKey); 

  Iterable<T> findAll();               

  long count();                        

  void delete(T entity);               

  boolean existsById(ID primaryKey);   

  // … more functionality omitted.
}
1. Saves the given entity. // 保存给定的实体。
2. Returns the entity identified by the given ID. //返回由给定ID标识的实体。
3. Returns all entities. // 返回所有实体。
4. Returns the number of entities. // 返回实体的数量。
5. Deletes the given entity. // 删除给定的实体。
6. Indicates whether an entity with the given ID exists. // 指示具有给定ID的实体是否存在。
```
```
We also provide persistence technology-specific abstractions, such as JpaRepository or MongoRepository. Those interfaces extend CrudRepository and expose the capabilities of the underlying persistence technology in addition to the rather generic persistence technology-agnostic interfaces such as CrudRepository.
```
On top of the CrudRepository, there is a PagingAndSortingRepository abstraction that adds additional methods to ease paginated access to entities:

在CrudRepository之上，有一个分页和排序存储库抽象，它添加了额外的方法来简化对实体的分页访问:

Example 2. PagingAndSortingRepository interface
```java
public interface PagingAndSortingRepository<T, ID extends Serializable>
  extends CrudRepository<T, ID> {

  Iterable<T> findAll(Sort sort);

  Page<T> findAll(Pageable pageable);
}
```
To access the second page of User by a page size of 20, you could do something like the following:
要以20页大小访问用户的第二页，您可以执行以下操作:
```java
PagingAndSortingRepository<User, Long> repository = // … get access to a bean
Page<User> users = repository.findAll(new PageRequest(1, 20));
```
In addition to query methods, query derivation for both count and delete queries is available. The following list shows the interface definition for a derived count query:
除了查询方法之外，计数和删除查询的查询派生也是可用的。下面的列表显示了派生计数查询的接口定义:

Example 3. Derived Count Query
```java
interface UserRepository extends CrudRepository<User, Long> {

  long countByLastname(String lastname);
}
```
The following list shows the interface definition for a derived delete query:
下面的列表显示了派生删除查询的接口定义:

Example 4. Derived Delete Query
```java
interface UserRepository extends CrudRepository<User, Long> {

  long deleteByLastname(String lastname);

  List<User> removeByLastname(String lastname);
}
```

### 1.2. Query methods
Standard CRUD functionality repositories usually have queries on the underlying datastore. With Spring Data, declaring those queries becomes a four-step process:
标准CRUD功能存储库通常对底层数据存储进行查询。使用Spring Data，声明这些查询将成为一个四步过程:

1. Declare an interface extending Repository or one of its subinterfaces and type it to the domain class and ID type that it should handle, as shown in the following example:
声明一个接口扩展库或其子接口之一，并将其输入到它应该处理的域类和ID类型，如下例所示:
```java
interface PersonRepository extends Repository<Person, Long> { … }
```
2. Declare query methods on the interface.
在接口上声明查询方法。
```java
interface PersonRepository extends Repository<Person, Long> {
  List<Person> findByLastname(String lastname);
}
```  
3. Set up Spring to create proxy instances for those interfaces, either with JavaConfig or with XML configuration.
通过使用JavaConfig或XML配置，设置Spring来为这些接口创建代理实例。

* To use Java configuration, create a class similar to the following:
要使用Java配置，创建一个类似于以下的类:
```java
import org.springframework.data.jpa.repository.config.EnableJpaRepositories;

@EnableJpaRepositories
class Config {}
```
* To use XML configuration, define a bean similar to the following:
要使用XML配置，请定义一个类似于以下的bean:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
   xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
   xmlns:jpa="http://www.springframework.org/schema/data/jpa"
   xsi:schemaLocation="http://www.springframework.org/schema/beans
     http://www.springframework.org/schema/beans/spring-beans.xsd
     http://www.springframework.org/schema/data/jpa
     http://www.springframework.org/schema/data/jpa/spring-jpa.xsd">

   <jpa:repositories base-package="com.acme.repositories"/>

</beans>
```

The JPA namespace is used in this example. If you use the repository abstraction for any other store, you need to change this to the appropriate namespace declaration of your store module. In other words, you should exchange jpa in favor of, for example, mongodb.
这个示例中使用了JPA名称空间。如果将存储库抽象用于任何其他存储库，则需要将其更改为存储模块的适当名称空间声明。换句话说，您应该交换jpa以支持，例如mongodb。

+ Also, note that the JavaConfig variant does not configure a package explicitly, because the package of the annotated class is used by default. To customize the package to scan, use one of the basePackage… attributes of the data-store-specific repository’s @Enable${store}Repositories-annotation.
另外，注意JavaConfig变体没有显式地配置包，因为默认情况下使用带注释的类的包。要定制要扫描的包，使用特定于 Data Repository 的@Enable${store}Repositories-annotation的一个basePackage…属性。

4. Inject the repository instance and use it, as shown in the following example:
注入存储库实例并使用它，如下例所示:
```java
class SomeClient {

  private final PersonRepository repository;

  SomeClient(PersonRepository repository) {
    this.repository = repository;
  }

  void doSomething() {
    List<Person> persons = repository.findByLastname("Matthews");
  }
}
```
The sections that follow explain each step in detail:
* Defining Repository Interfaces
* Defining Query Methods
* Creating Repository Instances
* Custom Implementations for Spring Data Repositories
接下来的章节详细解释了每个步骤:
* 定义存储库接口
* 定义查询方法
* 创建存储库实例
* Spring Data Repository 的自定义实现

## 1.3. Defining Repository Interfaces
First, define a domain class-specific repository interface. The interface must extend Repository and be typed to the domain class and an ID type. If you want to expose CRUD methods for that domain type, extend CrudRepository instead of Repository.
首先，定义一个特定于域类的存储库接口。接口必须扩展存储库，并将其类型化到域类和ID类型。如果您想为该域类型公开CRUD方法，请扩展CrudRepository而不是存储库。

### 1.3.1. Fine-tuning Repository Definition
Typically, your repository interface extends Repository, CrudRepository, or PagingAndSortingRepository. Alternatively, if you do not want to extend Spring Data interfaces, you can also annotate your repository interface with @RepositoryDefinition. Extending CrudRepository exposes a complete set of methods to manipulate your entities. If you prefer to be selective about the methods being exposed, copy the methods you want to expose from CrudRepository into your domain repository.
通常，存储库接口扩展存储库、CrudRepository或分页和排序存储库。另外，如果您不想扩展Spring Data接口，也可以使用@RepositoryDefinition注释存储库接口。扩展CrudRepository公开了一组完整的方法来操作您的实体。如果您倾向于选择公开的方法，那么将您想要公开的方法从CrudRepository复制到您的域存储库中。


Doing so lets you define your own abstractions on top of the provided Spring Data Repositories functionality.
The following example shows how to selectively expose CRUD methods (findById and save, in this case):
这样做可以让您在提供的Spring Data Repository 功能之上定义自己的抽象。
下面的示例展示了如何有选择地公开CRUD方法(在本例中是findById和save):

Example 5. Selectively exposing CRUD methods

```java
@NoRepositoryBean
interface MyBaseRepository<T, ID extends Serializable> extends Repository<T, ID> {

  Optional<T> findById(ID id);

  <S extends T> S save(S entity);
}

interface UserRepository extends MyBaseRepository<User, Long> {
  User findByEmailAddress(EmailAddress emailAddress);
}
```

In the prior example, you defined a common base interface for all your domain repositories and exposed findById(…) as well as save(…).These methods are routed into the base repository implementation of the store of your choice provided by Spring Data (for example, if you use JPA, the implementation is SimpleJpaRepository), because they match the method signatures in CrudRepository. So the UserRepository can now save users, find individual users by ID, and trigger a query to find Users by email address.
在前面的示例中，您为所有域存储库定义了一个公共基接口，并公开了findById(…)和save(…)。这些方法被路由到Spring Data提供的存储库的基本实现中(例如，如果使用JPA，实现是SimpleJpaRepository)，因为它们与CrudRepository中的方法签名匹配。因此，UserRepository现在可以保存用户，通过ID找到单个用户，并触发查询，通过电子邮件地址找到用户。

The intermediate repository interface is annotated with @NoRepositoryBean. Make sure you add that annotation to all repository interfaces for which Spring Data should not create instances at runtime.
1.3.2. Null Handling of Repository Methods
As of Spring Data 2.0, repository CRUD methods that return an individual aggregate instance use Java 8’s Optional to indicate the potential absence of a value. Besides that, Spring Data supports returning the following wrapper types on query methods:
中间存储库接口用@NoRepositoryBean注释。确保将该注释添加到所有存储库接口，Spring Data不应该为这些接口在运行时创建实例。
1.3.2。存储库方法的空处理
从Spring Data 2.0开始，返回单个聚合实例的repository CRUD方法使用Java 8的可选方法来表示可能缺少值。除此之外，Spring Data还支持在查询方法上返回以下包装器类型:

* com.google.common.base.Optional

* scala.Option

* io.vavr.control.Option

* javaslang.control.Option (deprecated as Javaslang is deprecated)

Alternatively, query methods can choose not to use a wrapper type at all. The absence of a query result is then indicated by returning null. Repository methods returning collections, collection alternatives, wrappers, and streams are guaranteed never to return null but rather the corresponding empty representation. See “Repository query return types” for details.
或者，查询方法可以选择完全不使用包装器类型。然后返回null表示没有查询结果。存储库方法返回集合、集合替代方案、包装器和流，保证永远不会返回null，而是相应的空表示。有关详细信息，请参阅“存储库查询返回类型”。

#### Nullability Annotations
You can express nullability constraints for repository methods by using Spring Framework’s nullability annotations. They provide a tooling-friendly approach and opt-in null checks during runtime, as follows:
您可以使用Spring Framework的nullability注释来表示存储库方法的nullability约束。它们提供了一种工具友好的方法，并在运行时进行空检查，如下所示:

@NonNullApi: Used on the package level to declare that the default behavior for parameters and return values is to not accept or produce null values.
在包级别上用于声明参数和返回值的默认行为是不接受或生成空值。

@NonNull: Used on a parameter or return value that must not be null (not needed on a parameter and return value where @NonNullApi applies).
用于不为空的参数或返回值(在@NonNullApi应用的参数和返回值中不需要)。

@Nullable: Used on a parameter or return value that can be null.
用于可为空的参数或返回值。

Spring annotations are meta-annotated with JSR 305 annotations (a dormant but widely spread JSR). JSR 305 meta-annotations let tooling vendors such as IDEA, Eclipse, and Kotlin provide null-safety support in a generic way, without having to hard-code support for Spring annotations. To enable runtime checking of nullability constraints for query methods, you need to activate non-nullability on the package level by using Spring’s @NonNullApi in package-info.java, as shown in the following example:
Spring注释是用JSR 305注释(一种休眠但广泛传播的JSR)进行元注释的。JSR 305元注释允许工具供应商(如IDEA、Eclipse和Kotlin)以通用的方式提供空安全支持，而不必硬编码对Spring注释的支持。要启用查询方法的空性约束的运行时检查，您需要在包信息中使用Spring的@NonNullApi在包级别上激活非空性。java，如下例所示:

Example 6. Declaring Non-nullability in package-info.java
```
@org.springframework.lang.NonNullApi
package com.acme;
```

Once non-null defaulting is in place, repository query method invocations get validated at runtime for nullability constraints. If a query execution result violates the defined constraint, an exception is thrown. This happens when the method would return null but is declared as non-nullable (the default with the annotation defined on the package the repository resides in). If you want to opt-in to nullable results again, selectively use @Nullable on individual methods. Using the result wrapper types mentioned at the start of this section continues to work as expected: An empty result is translated into the value that represents absence.
一旦非null默认值就位，在运行时将验证存储库查询方法调用是否为空。如果查询执行结果违反定义的约束，则抛出异常。当方法返回null但声明为不可空时(默认情况下存储库驻留的包上定义了注释)，就会发生这种情况。如果您想再次选择可空的结果，请在各个方法上选择性地使用@Nullable。使用本节开头所提到的结果包装器类型继续正常工作:将空结果转换为表示缺席的值。

The following example shows a number of the techniques just described:
下面的例子展示了刚才描述的一些技术:

Example 7. Using different nullability constraints
```java
package com.acme;                                                       

import org.springframework.lang.Nullable;

interface UserRepository extends Repository<User, Long> {

  User getByEmailAddress(EmailAddress emailAddress);                    

  @Nullable
  User findByEmailAddress(@Nullable EmailAddress emailAdress);          

  Optional<User> findOptionalByEmailAddress(EmailAddress emailAddress); 
}
1. The repository resides in a package (or sub-package) for which we have defined non-null behavior.
存储库驻留在一个包(或子包)中，我们为它定义了非空行为。
2. Throws an EmptyResultDataAccessException when the query executed does not produce a result. Throws an IllegalArgumentException when the emailAddress handed to the method is null.
当执行的查询没有产生结果时，抛出EmptyResultDataAccessException。当传递给方法的电子邮件地址为空时，抛出一个IllegalArgumentException。
当执行的查询没有产生结果时，返回null。还接受null作为电子邮件地址的值。
3. Returns null when the query executed does not produce a result. Also accepts null as the value for emailAddress.
当执行的查询没有产生结果时，返回null。还接受null作为电子邮件地址的值。
4. Returns Optional.empty() when the query executed does not produce a result. Throws an IllegalArgumentException when the emailAddress handed to the method is null.
当执行查询时，返回option .empty()，不会产生结果。当传递给方法的电子邮件地址为空时，抛出一个IllegalArgumentException。
```

#### Nullability in Kotlin-based Repositories
Kotlin has the definition of nullability constraints baked into the language. Kotlin code compiles to bytecode, which does not express nullability constraints through method signatures but rather through compiled-in metadata. Make sure to include the kotlin-reflect JAR in your project to enable introspection of Kotlin’s nullability constraints. Spring Data repositories use the language mechanism to define those constraints to apply the same runtime checks, as follows:
Kotlin语言中包含了可空性约束的定义。Kotlin代码编译为字节码，字节码不通过方法签名来表示空性约束，而是通过编译后的元数据。确保在项目中包含Kotlin - reflection JAR，以启用对Kotlin的可空性约束的自省。Spring Data Repository 使用语言机制来定义这些约束，以应用相同的运行时检查，如下所示:

Example 8. Using nullability constraints on Kotlin repositories
```java
interface UserRepository : Repository<User, String> {

  fun findByUsername(username: String): User     

  fun findByFirstname(firstname: String?): User? 
}
```
The method defines both the parameter and the result as non-nullable (the Kotlin default). The Kotlin compiler rejects method invocations that pass null to the method. If the query execution yields an empty result, an EmptyResultDataAccessException is thrown.
该方法将参数和结果都定义为不可空的(Kotlin默认值)。Kotlin编译器拒绝向方法传递null的方法调用。如果查询执行产生空的结果，就会抛出EmptyResultDataAccessException。

This method accepts null for the firstname parameter and returns null if the query execution does not produce a result.
此方法接受firstname参数为null，如果查询执行没有产生结果，则返回null。

### 1.3.3. Using Repositories with Multiple Spring Data Modules
Using a unique Spring Data module in your application makes things simple, because all repository interfaces in the defined scope are bound to the Spring Data module. Sometimes, applications require using more than one Spring Data module. In such cases, a repository definition must distinguish between persistence technologies. When it detects multiple repository factories on the class path, Spring Data enters strict repository configuration mode. Strict configuration uses details on the repository or the domain class to decide about Spring Data module binding for a repository definition:
在应用程序中使用唯一的Spring Data module使事情变得简单，因为定义范围内的所有存储库接口都绑定到Spring Data module。有时，应用程序需要使用多个Spring Data module。在这种情况下，存储库定义必须区分持久性技术。当它检测到类路径上的多个存储库工厂时，Spring Data将进入严格的存储库配置模式。严格的配置使用存储库或域类的详细信息来决定存储库定义的Spring Data module绑定:

If the repository definition extends the module-specific repository, then it is a valid candidate for the particular Spring Data module.
如果存储库定义扩展了特定于模块的存储库，那么它就是特定Spring Data module的有效候选对象。

If the domain class is annotated with the module-specific type annotation, then it is a valid candidate for the particular Spring Data module. Spring Data modules accept either third-party annotations (such as JPA’s @Entity) or provide their own annotations (such as @Document for Spring Data MongoDB and Spring Data Elasticsearch).
如果用特定于模块的类型注释注释了域类，那么它就是特定Spring Data module的有效候选对象。Spring Data module可以接受第三方注释(如JPA的@Entity)，也可以提供自己的注释(如Spring Data MongoDB和Spring Data Elasticsearch的@Document)。

The following example shows a repository that uses module-specific interfaces (JPA in this case):
下面的示例显示了一个使用特定于模块的接口(在本例中是JPA)的存储库:

Example 9. Repository definitions using module-specific interfaces
```java
interface MyRepository extends JpaRepository<User, Long> { }

@NoRepositoryBean
interface MyBaseRepository<T, ID extends Serializable> extends JpaRepository<T, ID> {
  …
}

interface UserRepository extends MyBaseRepository<User, Long> {
  …
}
```
MyRepository and UserRepository extend JpaRepository in their type hierarchy. They are valid candidates for the Spring Data JPA module.
MyRepository和UserRepository在其类型层次结构中扩展了JpaRepository。它们是Spring Data JPA模块的有效候选对象。

The following example shows a repository that uses generic interfaces:
下面的示例显示了使用通用接口的存储库:

Example 10. Repository definitions using generic interfaces
```java
interface AmbiguousRepository extends Repository<User, Long> {
 …
}

@NoRepositoryBean
interface MyBaseRepository<T, ID extends Serializable> extends CrudRepository<T, ID> {
  …
}

interface AmbiguousUserRepository extends MyBaseRepository<User, Long> {
  …
}
```
AmbiguousRepository and AmbiguousUserRepository extend only Repository and CrudRepository in their type hierarchy. While this is perfectly fine when using a unique Spring Data module, multiple modules cannot distinguish to which particular Spring Data these repositories should be bound.
歧义存储库和歧义用户存储库仅在其类型层次结构中扩展存储库和CrudRepository。虽然这在使用唯一的Spring Data module时非常好，但是多个模块无法区分这些存储库应该绑定到哪个特定的Spring Data。


The following example shows a repository that uses domain classes with annotations:
下面的示例显示了使用带注释的域类的存储库:

Example 11. Repository definitions using domain classes with annotations
```java
interface PersonRepository extends Repository<Person, Long> {
 …
}

@Entity
class Person {
  …
}

interface UserRepository extends Repository<User, Long> {
 …
}

@Document
class User {
  …
}
PersonRepository references Person, which is annotated with the JPA @Entity annotation, so this repository clearly belongs to Spring Data JPA. UserRepository references User, which is annotated with Spring Data MongoDB’s @Document annotation.

The following bad example shows a repository that uses domain classes with mixed annotations:

Example 12. Repository definitions using domain classes with mixed annotations

```java
interface JpaPersonRepository extends Repository<Person, Long> {
 …
}

interface MongoDBPersonRepository extends Repository<Person, Long> {
 …
}

@Entity
@Document
class Person {
  …
}
```

This example shows a domain class using both JPA and Spring Data MongoDB annotations. It defines two repositories, JpaPersonRepository and MongoDBPersonRepository. One is intended for JPA and the other for MongoDB usage. Spring Data is no longer able to tell the repositories apart, which leads to undefined behavior.
这个例子展示了一个同时使用JPA和Spring Data MongoDB注释的域类。它定义了两个存储库:JpaPersonRepository和MongoDBPersonRepository。一个用于JPA，另一个用于MongoDB。Spring Data不再能够区分存储库，这导致了未定义的行为。

Repository type details and distinguishing domain class annotations are used for strict repository configuration to identify repository candidates for a particular Spring Data module. Using multiple persistence technology-specific annotations on the same domain type is possible and enables reuse of domain types across multiple persistence technologies. However, Spring Data can then no longer determine a unique module with which to bind the repository.
存储库类型详细信息和区分领域类注释用于严格的存储库配置，以识别特定Spring Data module的存储库候选对象。在同一个域类型上使用多个特定于持久性技术的注释是可能的，并且允许跨多个持久性技术重用域类型。然而，Spring Data不再能够确定绑定存储库的唯一模块。

The last way to distinguish repositories is by scoping repository base packages. Base packages define the starting points for scanning for repository interface definitions, which implies having repository definitions located in the appropriate packages. By default, annotation-driven configuration uses the package of the configuration class. The base package in XML-based configuration is mandatory.
最后一种区分存储库的方法是确定存储库基础包的范围。基本包定义扫描存储库接口定义的起点，这意味着在适当的包中有存储库定义。默认情况下，注解驱动的配置使用配置类的包。基于xml的配置中的基本包是强制性的。

The following example shows annotation-driven configuration of base packages:
下面的示例显示了基本包的注释驱动配置:

Example 13. Annotation-driven configuration of base packages
```java
@EnableJpaRepositories(basePackages = "com.acme.repositories.jpa")
@EnableMongoRepositories(basePackages = "com.acme.repositories.mongo")
interface Configuration { }
```
## 1.4. Defining Query Methods
## 1.4. 定义查询方法
The repository proxy has two ways to derive a store-specific query from the method name:

By deriving the query from the method name directly.

By using a manually defined query.

Available options depend on the actual store. However, there must be a strategy that decides what actual query is created. The next section describes the available options.

存储库代理有两种方法从方法名派生特定于存储的查询:

通过直接从方法名派生查询。

通过使用手动定义的查询。

可用的选项取决于实际的存储。但是，必须有一个策略来决定创建什么实际查询。下一节将介绍可用的选项。

### 1.4.1. Query Lookup Strategies 
### 1.4.1. 查询查找策略
The following strategies are available for the repository infrastructure to resolve the query. With XML configuration, you can configure the strategy at the namespace through the query-lookup-strategy attribute. For Java configuration, you can use the queryLookupStrategy attribute of the Enable${store}Repositories annotation. Some strategies may not be supported for particular datastores.
存储库基础设施可以使用以下策略来解决查询。通过XML配置，您可以通过查询查找策略属性在名称空间配置策略。对于Java配置，您可以使用Enable${store} repository注释的queryLookupStrategy属性。对于特定的数据存储，可能不支持某些策略。

CREATE attempts to construct a store-specific query from the query method name. The general approach is to remove a given set of well known prefixes from the method name and parse the rest of the method. You can read more about query construction in “Query Creation”.
创建尝试从查询方法名构造特定于存储的查询。通常的方法是从方法名中删除一组已知前缀，并解析方法的其余部分。您可以在“查询创建”中阅读关于查询构造的更多信息。

USE_DECLARED_QUERY tries to find a declared query and throws an exception if cannot find one. The query can be defined by an annotation somewhere or declared by other means. Consult the documentation of the specific store to find available options for that store. If the repository infrastructure does not find a declared query for the method at bootstrap time, it fails.
USE_DECLARED_QUERY试图寻找已声明的查询，如果找不到就抛出异常。查询可以通过某个注释定义，也可以通过其他方式声明。查阅特定存储的文档以找到该存储的可用选项。如果储存库基础设施在引导时没有找到方法的声明查询，那么它将失败。

CREATE_IF_NOT_FOUND (default) combines CREATE and USE_DECLARED_QUERY. It looks up a declared query first, and, if no declared query is found, it creates a custom method name-based query. This is the default lookup strategy and, thus, is used if you do not configure anything explicitly. It allows quick query definition by method names but also custom-tuning of these queries by introducing declared queries as needed.
CREATE_IF_NOT_FOUND(默认)合并了CREATE和USE_DECLARED_QUERY。它首先查找声明的查询，如果没有找到声明的查询，它将创建一个基于名称的定制方法查询。这是默认的查找策略，因此，如果没有显式地配置任何内容，就会使用这种策略。它允许通过方法名快速定义查询，还可以根据需要引入声明的查询，从而对这些查询进行定制调优。

### 1.4.2. Query Creation
### 1.4.2. 创建查询
The query builder mechanism built into Spring Data Repository infrastructure is useful for building constraining queries over entities of the repository. The mechanism strips the prefixes find…By, read…By, query…By, count…By, and get…By from the method and starts parsing the rest of it. The introducing clause can contain further expressions, such as a Distinct to set a distinct flag on the query to be created. However, the first By acts as delimiter to indicate the start of the actual criteria. At a very basic level, you can define conditions on entity properties and concatenate them with And and Or. The following example shows how to create a number of queries:
构建在Spring Data Repository 基础结构中的查询构建器机制对于构建针对存储库实体的约束查询非常有用。该机制从方法中剥离前缀find. By、read. By、query. By、count. By和get. By，并开始解析其余部分。引入子句可以包含进一步的表达式，例如，一个Distinct可以在要创建的查询上设置一个惟一的标志。但是，第一个By充当分隔符来指示实际标准的开始。在非常基本的级别上，您可以定义实体属性的条件，并将它们与and和Or连接起来。下面的例子展示了如何创建一些查询:

Example 14. Query creation from method names
```java
interface PersonRepository extends Repository<User, Long> {

  List<Person> findByEmailAddressAndLastname(EmailAddress emailAddress, String lastname);

  // Enables the distinct flag for the query
  List<Person> findDistinctPeopleByLastnameOrFirstname(String lastname, String firstname);
  List<Person> findPeopleDistinctByLastnameOrFirstname(String lastname, String firstname);

  // Enabling ignoring case for an individual property
  List<Person> findByLastnameIgnoreCase(String lastname);
  // Enabling ignoring case for all suitable properties
  List<Person> findByLastnameAndFirstnameAllIgnoreCase(String lastname, String firstname);

  // Enabling static ORDER BY for a query
  List<Person> findByLastnameOrderByFirstnameAsc(String lastname);
  List<Person> findByLastnameOrderByFirstnameDesc(String lastname);
}
```
The actual result of parsing the method depends on the persistence store for which you create the query. However, there are some general things to notice:
解析方法的实际结果取决于创建查询的持久性存储。然而，有一些事情需要注意:

* The expressions are usually property traversals combined with operators that can be concatenated. You can combine property expressions with AND and OR. You also get support for operators such as Between, LessThan, GreaterThan, and Like for the property expressions. The supported operators can vary by datastore, so consult the appropriate part of your reference documentation.
表达式通常是与可以连接的运算符组合的属性遍历。可以使用AND和OR组合属性表达式。您还可以获得对诸如Between、less、GreaterThan和Like等运算符的支持。受支持的操作符可能因数据存储而异，因此请参阅参考文档的适当部分。

* The method parser supports setting an IgnoreCase flag for individual properties (for example, findByLastnameIgnoreCase(…)) or for all properties of a type that supports ignoring case (usually String instances — for example, findByLastnameAndFirstnameAllIgnoreCase(…)). Whether ignoring cases is supported may vary by store, so consult the relevant sections in the reference documentation for the store-specific query method.
方法解析器支持为单个属性(例如，findByLastnameIgnoreCase(…))或支持忽略大小写类型的所有属性设置一个IgnoreCase标记(通常是字符串实例，例如，findbylastnameandfirstnameobamnorecase(…))。是否支持忽略用例可能因存储而异，因此请参阅参考文档中有关存储特定查询方法的部分。

* You can apply static ordering by appending an OrderBy clause to the query method that references a property and by providing a sorting direction (Asc or Desc). To create a query method that supports dynamic sorting, see “Special parameter handling”.
可以通过在引用属性的查询方法中添加OrderBy子句并提供排序方向(Asc或Desc)来应用静态排序。要创建支持动态排序的查询方法，请参阅“特殊参数处理”。

### 1.4.3. Property Expressions
### 1.4.3. 属性表达式
Property expressions can refer only to a direct property of the managed entity, as shown in the preceding example. At query creation time, you already make sure that the parsed property is a property of the managed domain class. However, you can also define constraints by traversing nested properties. Consider the following method signature:
属性表达式只能引用托管实体的直接属性，如上例所示。在查询创建时，您已经确保已解析的属性是托管域类的属性。但是，您也可以通过遍历嵌套属性来定义约束。考虑以下方法签名:
```java
List<Person> findByAddressZipCode(ZipCode zipCode);
```

Assume a Person has an Address with a ZipCode. In that case, the method creates the property traversal x.address.zipCode. The resolution algorithm starts by interpreting the entire part (AddressZipCode) as the property and checks the domain class for a property with that name (uncapitalized). If the algorithm succeeds, it uses that property. If not, the algorithm splits up the source at the camel case parts from the right side into a head and a tail and tries to find the corresponding property — in our example, AddressZip and Code. If the algorithm finds a property with that head, it takes the tail and continues building the tree down from there, splitting the tail up in the way just described. If the first split does not match, the algorithm moves the split point to the left (Address, ZipCode) and continues.
假设某人有一个有邮编的地址。在这种情况下，该方法创建遍历x.address.zipCode的属性。解析算法首先将整个部分(AddressZipCode)解释为属性，然后检查域类是否具有该名称(未大写)的属性。如果算法成功，它将使用该属性。如果不是，该算法将驼峰箱部分的源代码从右侧分割为头部和尾部，并试图找到相应的属性——在我们的示例中是AddressZip和Code。如果这个算法找到了那个头的属性，它就会取下尾巴，继续从那里建立树，像刚才描述的那样把尾巴分开。如果第一次分割不匹配，算法将分割点向左移动(地址、邮编)并继续。

Although this should work for most cases, it is possible for the algorithm to select the wrong property. Suppose the Person class has an addressZip property as well. The algorithm would match in the first split round already, choose the wrong property, and fail (as the type of addressZip probably has no code property).
虽然这在大多数情况下都是可行的，但是算法可能会选择错误的属性。假设Person类也有addressZip属性。该算法将在第一轮分割中匹配，选择错误的属性，然后失败(因为addressZip类型可能没有代码属性)。

To resolve this ambiguity you can use \_ inside your method name to manually define traversal points. So our method name would be as follows:
要解决这种模糊性，可以在方法名中使用\_来手动定义遍历点。因此，我们的方法名如下:

```
List<Person> findByAddress_ZipCode(ZipCode zipCode);
```
Because we treat the underscore character as a reserved character, we strongly advise following standard Java naming conventions (that is, not using underscores in property names but using camel case instead).
因为我们将下划线字符视为保留字符，所以我们强烈建议遵循标准的Java命名约定(即，在属性名称中不使用下划线，而是使用驼峰大小写)。

### 1.4.4. Special parameter handling
### 1.4.4. 特殊参数处理
To handle parameters in your query, define method parameters as already seen in the preceding examples. Besides that, the infrastructure recognizes certain specific types like Pageable and Sort, to apply pagination and sorting to your queries dynamically. The following example demonstrates these features:
要处理查询中的参数，请像前面示例中那样定义方法参数。除此之外，基础结构还识别某些特定类型，如可分页和排序，以便动态地对查询应用分页和排序。下面的例子演示了这些特性:

Example 15. Using Pageable, Slice, and Sort in query methods
Example 15. 在查询方法中使用可分页、切片和排序
```java
Page<User> findByLastname(String lastname, Pageable pageable);

Slice<User> findByLastname(String lastname, Pageable pageable);

List<User> findByLastname(String lastname, Sort sort);

List<User> findByLastname(String lastname, Pageable pageable);
```
The first method lets you pass an org.springframework.data.domain.Pageable instance to the query method to dynamically add paging to your statically defined query. A Page knows about the total number of elements and pages available. It does so by the infrastructure triggering a count query to calculate the overall number. As this might be expensive (depending on the store used), you can instead return a Slice. A Slice only knows about whether a next Slice is available, which might be sufficient when walking through a larger result set.
第一个方法允许您传递org.springframework.data.domain。可分页查询方法的实例，以动态地将分页添加到静态定义的查询中。一个页面知道可用元素和页面的总数。它通过基础结构触发count查询来计算总体数量。由于这可能很昂贵(取决于所使用的商店)，您可以返回一个片。一个片只知道下一个片是否可用，这在遍历较大的结果集时可能就足够了。

Sorting options are handled through the Pageable instance, too. If you only need sorting, add an org.springframework.data.domain.Sort parameter to your method. As you can see, returning a List is also possible. In this case, the additional metadata required to build the actual Page instance is not created (which, in turn, means that the additional count query that would have been necessary is not issued). Rather, it restricts the query to look up only the given range of entities.
排序选项也通过可分页实例来处理。如果只需要排序，添加一个org.springframework.data.domain。对方法的参数进行排序。如您所见，返回一个列表也是可能的。在这种情况下，不创建构建实际页面实例所需的额外元数据(这又意味着不发出必要的额外count查询)。相反，它限制查询只查找给定的实体范围。

To find out how many pages you get for an entire query, you have to trigger an additional count query. By default, this query is derived from the query you actually trigger.
要查找整个查询的页面数，必须触发一个额外的count查询。默认情况下，此查询派生自您实际触发的查询。

1.4.5. Limiting Query Results
1.4.5. 限制查询结果
The results of query methods can be limited by using the first or top keywords, which can be used interchangeably. An optional numeric value can be appended to top or first to specify the maximum result size to be returned. If the number is left out, a result size of 1 is assumed. The following example shows how to limit the query size:
查询方法的结果可以通过使用第一个或最上面的关键字来限制，这些关键字可以互换使用。可以将可选的数值追加到顶部或首先指定要返回的最大结果大小。如果省略了该数字，则假设结果大小为1。下面的例子展示了如何限制查询大小:

Example 16. Limiting the result size of a query with Top and 
Example 16. 使用Top和First限制查询的结果大小
```java
User findFirstByOrderByLastnameAsc();

User findTopByOrderByAgeDesc();

Page<User> queryFirst10ByLastname(String lastname, Pageable pageable);

Slice<User> findTop3ByLastname(String lastname, Pageable pageable);

List<User> findFirst10ByLastname(String lastname, Sort sort);

List<User> findTop10ByLastname(String lastname, Pageable pageable);
```
The limiting expressions also support the Distinct keyword. Also, for the queries limiting the result set to one instance, wrapping the result into with the Optional keyword is supported.
极限表达式还支持Distinct关键字。此外，对于将结果集限制为一个实例的查询，还支持使用可选关键字将结果包装起来。

If pagination or slicing is applied to a limiting query pagination (and the calculation of the number of pages available), it is applied within the limited result.
如果分页或切片应用于有限的查询分页(以及计算可用页面的数量)，那么它将在有限的结果中应用。

Limiting the results in combination with dynamic sorting by using a Sort parameter lets you express query methods for the 'K' smallest as well as for the 'K' biggest elements.
通过使用排序参数将结果与动态排序结合使用，可以为最小的K和最大的K元素表达查询方法。

1.4.6. Streaming query results
1.4.6. 流查询结果
The results of query methods can be processed incrementally by using a Java 8 Stream<T> as return type. Instead of wrapping the query results in a Stream data store-specific methods are used to perform the streaming, as shown in the following example:
查询方法的结果可以通过使用Java 8 Stream<T>作为返回类型递增地处理。使用特定于流数据存储的方法来执行流，而不是将查询结果包装在流数据存储中，如下例所示:

Example 17. Stream the result of a query with Java 8 Stream<T>
使用Java 8 Stream<T>来流查询结果
```java
@Query("select u from User u")
Stream<User> findAllByCustomQueryAndStream();

Stream<User> readAllByFirstnameNotNull();

@Query("select u from User u")
Stream<User> streamAllPaged(Pageable pageable);
```
A Stream potentially wraps underlying data store-specific resources and must, therefore, be closed after usage. You can either manually close the Stream by using the close() method or by using a Java 7 try-with-resources block, as shown in the following example:
流可能会包装特定于数据存储的底层资源，因此必须在使用后关闭。您可以使用close()方法手动关闭流，也可以使用Java 7 try-with-resources块，如下例所示:
Example 18. Working with a Stream<T> result in a try-with-resources block
```
try (Stream<User> stream = repository.findAllByCustomQueryAndStream()) {
  stream.forEach(…);
}
```
Not all Spring Data modules currently support Stream<T> as a return type.
不是所有的Spring Data模块都支持Stream<T>作为返回类型。
### 1.4.7. Async query results
### 1.4.7. 异步查询结果
Repository queries can be run asynchronously by using Spring’s asynchronous method execution capability. This means the method returns immediately upon invocation while the actual query execution occurs in a task that has been submitted to a Spring TaskExecutor. Asynchronous query execution is different from reactive query execution and should not be mixed. Refer to store-specific documentation for more details on reactive support. The following example shows a number of asynchronous queries:
通过使用Spring的异步方法执行功能，可以异步运行存储库查询。这意味着该方法在调用时立即返回，而实际的查询执行发生在提交给Spring TaskExecutor的任务中。异步查询执行与响应式查询执行不同，不应该混合执行。有关反应性支持的详细信息，请参阅特定于存储的文档。下面的示例显示了一些异步查询:
```
@Async
Future<User> findByFirstname(String firstname);               

@Async
CompletableFuture<User> findOneByFirstname(String firstname); 

@Async
ListenableFuture<User> findOneByLastname(String lastname);    
  
1. Use java.util.concurrent.Future as the return type.
使用java . util . concurrent, Future作为返回类型。
2. Use a Java 8 java.util.concurrent.CompletableFuture as the return type.
使用Java 8 Java .util.concurrent, CompletableFuture作为返回类型。
3. Use a org.springframework.util.concurrent.ListenableFuture as the return type.
使用org.springframework.util.concurrent, ListenableFuture作为返回类型。
```
## 1.5. Creating Repository Instances
## 1.5. 创建存储库实例
In this section, you create instances and bean definitions for the defined repository interfaces. One way to do so is by using the Spring namespace that is shipped with each Spring Data module that supports the repository mechanism, although we generally recommend using Java configuration.
在本节中，您将为已定义的存储库接口创建实例和bean定义。一种方法是使用支持存储库机制的每个Spring Data module附带的Spring名称空间，尽管我们通常建议使用Java配置。

1.5.1. XML configuration
Each Spring Data module includes a repositories element that lets you define a base package that Spring scans for you, as shown in the following example:
每个Spring Data module都包含一个存储库元素，它允许您定义Spring为您扫描的基本包，如下例所示:

Example 19. Enabling Spring Data repositories via XML
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans:beans xmlns:beans="http://www.springframework.org/schema/beans"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xmlns="http://www.springframework.org/schema/data/jpa"
  xsi:schemaLocation="http://www.springframework.org/schema/beans
    http://www.springframework.org/schema/beans/spring-beans.xsd
    http://www.springframework.org/schema/data/jpa
    http://www.springframework.org/schema/data/jpa/spring-jpa.xsd">

  <repositories base-package="com.acme.repositories" />

</beans:beans>
```
In the preceding example, Spring is instructed to scan com.acme.repositories and all its sub-packages for interfaces extending Repository or one of its sub-interfaces. For each interface found, the infrastructure registers the persistence technology-specific FactoryBean to create the appropriate proxies that handle invocations of the query methods. Each bean is registered under a bean name that is derived from the interface name, so an interface of UserRepository would be registered under userRepository. The base-package attribute allows wildcards so that you can define a pattern of scanned packages.
在前面的示例中，指示Spring扫描com.acme。存储库及其所有用于扩展存储库或其中一个子接口的接口的子包。对于找到的每个接口，基础设施注册持久化技术专用的FactoryBean，以创建处理查询方法调用的适当代理。每个bean都在从接口名派生的bean名称下注册，因此UserRepository的接口将在UserRepository下注册。基本包属性允许通配符，以便您可以定义扫描包的模式。

#### Using filters
By default, the infrastructure picks up every interface extending the persistence technology-specific Repository sub-interface located under the configured base package and creates a bean instance for it. However, you might want more fine-grained control over which interfaces have bean instances created for them. To do so, use <include-filter /> and <exclude-filter /> elements inside the <repositories /> element. The semantics are exactly equivalent to the elements in Spring’s context namespace. For details, see the Spring reference documentation for these elements.
默认情况下，基础结构会获取扩展位于配置的基本包下的持久性技术特定存储库子接口的每个接口，并为其创建一个bean实例。但是，您可能希望对为其创建bean实例的接口进行更细粒度的控制。为此，在< repository />元素中使用< includefilter />和<exclude-filter />元素。语义完全等同于Spring上下文命名空间中的元素。有关详细信息，请参阅这些元素的Spring参考文档。

For example, to exclude certain interfaces from instantiation as repository beans, you could use the following configuration:
例如，要从实例化中排除某些接口作为存储库bean，您可以使用以下配置:

Example 20. Using exclude-filter element
  
```xml
<repositories base-package="com.acme.repositories">
  <context:exclude-filter type="regex" expression=".*SomeRepository" />
</repositories>
```
The preceding example excludes all interfaces ending in SomeRepository from being instantiated.
前面的例子排除了以SomeRepository结尾的所有接口被实例化。

1.5.2. JavaConfig
The repository infrastructure can also be triggered by using a store-specific @Enable${store}Repositories annotation on a JavaConfig class. For an introduction into Java-based configuration of the Spring container, see JavaConfig in the Spring reference documentation.
通过在JavaConfig类上使用特定于存储的@Enable${store}存储库注释，还可以触发存储库基础设施。有关基于java的Spring容器配置的介绍，请参阅Spring参考文档中的JavaConfig。

A sample configuration to enable Spring Data repositories resembles the following:
启用Spring Data Repository 的示例配置如下:

Example 21. Sample annotation based repository configuration
```java
@Configuration
@EnableJpaRepositories("com.acme.repositories")
class ApplicationConfiguration {

  @Bean
  EntityManagerFactory entityManagerFactory() {
    // …
  }
}
``` 
The preceding example uses the JPA-specific annotation, which you would change according to the store module you actually use. The same applies to the definition of the EntityManagerFactory bean. See the sections covering the store-specific configuration.
前面的示例使用特定于jpa的注释，您将根据实际使用的存储模块更改该注释。这同样适用于EntityManagerFactory bean的定义。请参阅涉及存储特定配置的部分。

### 1.5.3. Standalone usage
### 1.5.3. 独立使用
You can also use the repository infrastructure outside of a Spring container — for example, in CDI environments. You still need some Spring libraries in your classpath, but, generally, you can set up repositories programmatically as well. The Spring Data modules that provide repository support ship a persistence technology-specific RepositoryFactory that you can use as follows:
您还可以在Spring容器之外使用存储库基础设施——例如，在CDI环境中。在类路径中仍然需要一些Spring库，但通常也可以通过编程的方式设置存储库。提供存储库支持的Spring Data module提供了一个持久性技术专用的存储库工厂，您可以按照如下方式使用它:

Example 22. Standalone usage of repository factory
```
RepositoryFactorySupport factory = … // Instantiate factory here
UserRepository repository = factory.getRepository(UserRepository.class);
```
## 1.6. Custom Implementations for Spring Data Repositories
## 1.6. 用于Spring Data Repositories的自定义实现
This section covers repository customization and how fragments form a composite repository.
本节讨论存储库定制以及片段如何形成复合存储库。
  
When a query method requires a different behavior or cannot be implemented by query derivation, then it is necessary to provide a custom implementation. Spring Data repositories let you provide custom repository code and integrate it with generic CRUD abstraction and query method functionality.
当查询方法需要不同的行为或不能通过查询派生实现时，则需要提供自定义实现。Spring Data Repository 允许您提供定制的存储库代码，并将其与通用CRUD抽象和查询方法功能集成在一起。
  
### 1.6.1. Customizing Individual Repositories
### 1.6.1. 定制个人存储库
To enrich a repository with custom functionality, you must first define a fragment interface and an implementation for the custom functionality, as shown in the following example:
要使用自定义功能丰富存储库，您必须首先为自定义功能定义一个片段接口和实现，如下例所示:
Example 23. Interface for custom repository functionality

```java
interface CustomizedUserRepository {
  void someCustomMethod(User user);
}
```

Then you can let your repository interface additionally extend from the fragment interface, as shown in the following example:
然后，您可以让您的存储库接口从片段接口进一步扩展，如下例所示:
Example 24. Implementation of custom repository functionality

```java
class CustomizedUserRepositoryImpl implements CustomizedUserRepository {

  public void someCustomMethod(User user) {
    // Your custom implementation
  }
}
```

The most important part of the class name that corresponds to the fragment interface is the Impl postfix.
The implementation itself does not depend on Spring Data and can be a regular Spring bean. Consequently, you can use standard dependency injection behavior to inject references to other beans (such as a JdbcTemplate), take part in aspects, and so on.
与片段接口相对应的类名中最重要的部分是Impl后缀。
实现本身不依赖于Spring Data，可以是常规的Spring bean。因此，您可以使用标准的依赖注入行为来注入对其他bean(例如JdbcTemplate)的引用，参与方面，等等。

You can let your repository interface extend the fragment interface, as shown in the following example:
您可以让存储库接口扩展片段接口，如下例所示:

Example 25. Changes to your repository interface

```java
interface UserRepository extends CrudRepository<User, Long>, CustomizedUserRepository {

  // Declare query methods here
}
```

Extending the fragment interface with your repository interface combines the CRUD and custom functionality and makes it available to clients.
使用存储库接口扩展片段接口结合了CRUD和自定义功能，并使其对客户机可用。

Spring Data repositories are implemented by using fragments that form a repository composition. Fragments are the base repository, functional aspects (such as QueryDsl), and custom interfaces along with their implementation. Each time you add an interface to your repository interface, you enhance the composition by adding a fragment. The base repository and repository aspect implementations are provided by each Spring Data module.
Spring Data Repository 是通过使用组成存储库组合的片段实现的。片段是基本存储库、功能方面(如QueryDsl)以及自定义接口及其实现。每次将接口添加到存储库接口时，都通过添加一个片段来增强组合。基本存储库和存储库方面实现由每个Spring Data module提供。

The following example shows custom interfaces and their implementations:
下面的例子展示了自定义接口及其实现:

Example 26. Fragments with their implementations

```java
interface HumanRepository {
  void someHumanMethod(User user);
}

class HumanRepositoryImpl implements HumanRepository {

  public void someHumanMethod(User user) {
    // Your custom implementation
  }
}

interface ContactRepository {

  void someContactMethod(User user);

  User anotherContactMethod(User user);
}

class ContactRepositoryImpl implements ContactRepository {

  public void someContactMethod(User user) {
    // Your custom implementation
  }

  public User anotherContactMethod(User user) {
    // Your custom implementation
  }
}
```
The following example shows the interface for a custom repository that extends CrudRepository:
下面的示例显示了扩展CrudRepository的定制存储库的接口:

Example 27. Changes to your repository interface

```java
interface UserRepository extends CrudRepository<User, Long>, HumanRepository, ContactRepository {

  // Declare query methods here
}
```

Repositories may be composed of multiple custom implementations that are imported in the order of their declaration. Custom implementations have a higher priority than the base implementation and repository aspects. This ordering lets you override base repository and aspect methods and resolves ambiguity if two fragments contribute the same method signature. Repository fragments are not limited to use in a single repository interface. Multiple repositories may use a fragment interface, letting you reuse customizations across different repositories.
存储库可以由多个按照声明顺序导入的自定义实现组成。自定义实现的优先级高于基本实现和存储库方面。这种顺序允许您重写基本存储库和方面方法，并解决如果两个片段提供相同的方法签名时的歧义。存储库片段不限于在单个存储库接口中使用。多个存储库可能使用片段接口，允许您跨不同存储库重用定制。

The following example shows a repository fragment and its implementation:
下面的示例展示了一个存储库片段及其实现:
  
Example 28. Fragments overriding save(…)

```java
interface CustomizedSave<T> {
  <S extends T> S save(S entity);
}

class CustomizedSaveImpl<T> implements CustomizedSave<T> {

  public <S extends T> S save(S entity) {
    // Your custom implementation
  }
}
```

The following example shows a repository that uses the preceding repository fragment:
下面的示例显示了使用前一个存储库片段的存储库:
Example 29. Customized repository interfaces

```java
interface UserRepository extends CrudRepository<User, Long>, CustomizedSave<User> {
}

interface PersonRepository extends CrudRepository<Person, Long>, CustomizedSave<Person> {
}
```

#### Configuration
If you use namespace configuration, the repository infrastructure tries to autodetect custom implementation fragments by scanning for classes below the package in which it found a repository. These classes need to follow the naming convention of appending the namespace element’s repository-impl-postfix attribute to the fragment interface name. This postfix defaults to Impl. The following example shows a repository that uses the default postfix and a repository that sets a custom value for the postfix:
如果使用名称空间配置，存储库基础结构将通过扫描找到存储库的包下面的类来尝试自动检测自定义实现片段。这些类需要遵循将名称空间元素的repository- postfix属性附加到片段接口名称的命名约定。这个后缀默认为Impl。下面的示例显示了使用默认后缀的存储库和为后缀设置自定义值的存储库:

Example 30. Configuration example

```xml
<repositories base-package="com.acme.repository" />

<repositories base-package="com.acme.repository" repository-impl-postfix="MyPostfix" />
```

The first configuration in the preceding example tries to look up a class called com.acme.repository.CustomizedUserRepositoryImpl to act as a custom repository implementation. The second example tries to lookup com.acme.repository.CustomizedUserRepositoryMyPostfix.
前面示例中的第一个配置尝试查找一个名为com.acme.repository.CustomizedUserRepositoryImpl作为自定义存储库实现。第二个示例尝试查找com.acme.repository.CustomizedUserRepositoryMyPostfix.

#### Resolution of Ambiguity
#### 解决歧义
If multiple implementations with matching class names are found in different packages, Spring Data uses the bean names to identify which one to use.
如果在不同的包中发现了具有匹配类名的多个实现，Spring Data将使用bean名称来确定使用哪个。

Given the following two custom implementations for the CustomizedUserRepository shown earlier, the first implementation is used. Its bean name is customizedUserRepositoryImpl, which matches that of the fragment interface (CustomizedUserRepository) plus the postfix Impl.
鉴于前面显示的定制userrepository有以下两个自定义实现，因此将使用第一个实现。它的bean名称是customzeduserrepositoryimpl，与片段接口(customzeduserrepository)和postfix Impl相匹配。

Example 31. Resolution of amibiguous implementations

```java
package com.acme.impl.one;

class CustomizedUserRepositoryImpl implements CustomizedUserRepository {

  // Your custom implementation
}
package com.acme.impl.two;

@Component("specialCustomImpl")
class CustomizedUserRepositoryImpl implements CustomizedUserRepository {

  // Your custom implementation
}
```

If you annotate the UserRepository interface with @Component("specialCustom"), the bean name plus Impl then matches the one defined for the repository implementation in com.acme.impl.two, and it is used instead of the first one.
如果您使用@Component("specialCustom")注释UserRepository接口，那么bean名称加上Impl将与com.acme.impl.two中为存储库实现定义的名称匹配, 它被用来代替第一个。

#### Manual Wiring
#### 手动连接
If your custom implementation uses annotation-based configuration and autowiring only, the preceding approach shown works well, because it is treated as any other Spring bean. If your implementation fragment bean needs special wiring, you can declare the bean and name it according to the conventions described in the preceding section. The infrastructure then refers to the manually defined bean definition by name instead of creating one itself. The following example shows how to manually wire a custom implementation:
如果您的自定义实现只使用基于注释的配置和自动连接，那么前面所示的方法工作得很好，因为它被视为任何其他Spring bean。如果您的实现片段bean需要特殊的连接，您可以根据前一节描述的约定声明bean并命名它。然后，基础结构根据名称引用手动定义的bean定义，而不是自己创建bean定义。下面的例子展示了如何手动连接自定义实现:

Example 32. Manual wiring of custom implementations

```xml
<repositories base-package="com.acme.repository" />

<beans:bean id="userRepositoryImpl" class="…">
  <!-- further configuration -->
</beans:bean>
```

### 1.6.2. Customize the Base Repository
### 1.6.2. 自定义基本存储库
The approach described in the preceding section requires customization of each repository interfaces when you want to customize the base repository behavior so that all repositories are affected. To instead change behavior for all repositories, you can create an implementation that extends the persistence technology-specific repository base class. This class then acts as a custom base class for the repository proxies, as shown in the following example:
当您想要定制基本存储库行为以便所有存储库都受到影响时，上一节中描述的方法需要定制每个存储库接口。要更改所有存储库的行为，您可以创建一个扩展特定于持久性技术的存储库基类的实现。然后这个类作为存储库代理的自定义基类，如下例所示:

Example 33. Custom repository base class

```java
class MyRepositoryImpl<T, ID extends Serializable>
  extends SimpleJpaRepository<T, ID> {

  private final EntityManager entityManager;

  MyRepositoryImpl(JpaEntityInformation entityInformation,
                          EntityManager entityManager) {
    super(entityInformation, entityManager);

    // Keep the EntityManager around to used from the newly introduced methods.
    this.entityManager = entityManager;
  }

  @Transactional
  public <S extends T> S save(S entity) {
    // implementation goes here
  }
}
``` 

The class needs to have a constructor of the super class which the store-specific repository factory implementation uses. If the repository base class has multiple constructors, override the one taking an EntityInformation plus a store specific infrastructure object (such as an EntityManager or a template class).
The final step is to make the Spring Data infrastructure aware of the customized repository base class. In Java configuration, you can do so by using the repositoryBaseClass attribute of the @Enable${store}Repositories annotation, as shown in the following example:
类需要有一个超类的构造函数，这个超类是特定于存储库的存储库工厂实现所使用的。如果存储库基类有多个构造函数，则重写获取EntityInformation和存储特定基础结构对象(例如EntityManager或模板类)的构造函数。
最后一步是让Spring Data基础设施知道定制的存储库基类。在Java配置中，可以通过使用@Enable${store} repository注释的repositoryBaseClass属性来完成此操作，如下例所示:

Example 34. Configuring a custom repository base class using JavaConfig
```
@Configuration
@EnableJpaRepositories(repositoryBaseClass = MyRepositoryImpl.class)
class ApplicationConfiguration { … }
```
A corresponding attribute is available in the XML namespace, as shown in the following example:
在XML名称空间中可以使用相应的属性，如下例所示:
Example 35. Configuring a custom repository base class using XML
```
<repositories base-package="com.acme.repository"
     base-class="….MyRepositoryImpl" />
```
## 1.7. Publishing Events from Aggregate Roots
## 1.7. 从聚合根发布事件
Entities managed by repositories are aggregate roots. In a Domain-Driven Design application, these aggregate roots usually publish domain events. Spring Data provides an annotation called @DomainEvents that you can use on a method of your aggregate root to make that publication as easy as possible, as shown in the following example:
存储库管理的实体是聚合根。在域驱动的设计应用程序中，这些聚合根通常发布域事件。Spring Data提供了一个名为@DomainEvents的注释，您可以在聚合根的方法上使用该注释，以使发布尽可能简单，如下例所示:

Example 36. Exposing domain events from an aggregate root
```
class AnAggregateRoot {

    @DomainEvents 
    Collection<Object> domainEvents() {
        // … return events you want to get published here
    }

    @AfterDomainEventPublication 
    void callbackMethod() {
       // … potentially clean up domain events list
    }
}
```
The method using @DomainEvents can return either a single event instance or a collection of events. It must not take any arguments.
After all events have been published, we have a method annotated with @AfterDomainEventPublication. It can be used to potentially clean the list of events to be published (among other uses).
The methods are called every time one of a Spring Data Repository’s save(…) methods is called.
使用@DomainEvents的方法可以返回单个事件实例或事件集合。它不能有任何争论。
在所有事件发布之后，我们有一个用@AfterDomainEventPublication注释的方法。它可以用于清理将要发布的事件列表(以及其他用途)。
每次调用Spring Data Repository 的save(…)方法时都会调用这些方法。

## 1.8. Spring Data Extensions
## 1.8. Spring Data 扩展
This section documents a set of Spring Data extensions that enable Spring Data usage in a variety of contexts. Currently, most of the integration is targeted towards Spring MVC.
本节记录一组Spring Data扩展，这些扩展支持在各种上下文中使用Spring Data。目前，大多数集成都是针对Spring MVC。
### 1.8.1. Querydsl Extension
### 1.8.1. Querydsl 扩展
Querydsl is a framework that enables the construction of statically typed SQL-like queries through its fluent API.
Querydsl是一个框架，支持通过它的fluent API构建静态类型化sql查询。

Several Spring Data modules offer integration with Querydsl through QuerydslPredicateExecutor, as shown in the following example:
几个Spring Data module通过QuerydslPredicateExecutor提供与Querydsl的集成，如下例所示:

Example 37. QuerydslPredicateExecutor interface

```java
public interface QuerydslPredicateExecutor<T> {

  Optional<T> findById(Predicate predicate);  

  Iterable<T> findAll(Predicate predicate);   

  long count(Predicate predicate);            

  boolean exists(Predicate predicate);        

  // … more functionality omitted.
}
1. Finds and returns a single entity matching the Predicate.
查找并返回与断言匹配的单个实体。
2. Finds and returns all entities matching the Predicate.
查找并返回与断言匹配的所有实体。
3. Returns the number of entities matching the Predicate.
返回匹配断言的实体的数量。
4. Returns whether an entity that matches the Predicate exists.
返回匹配断言的实体是否存在。
```

To make use of Querydsl support, extend QuerydslPredicateExecutor on your repository interface, as shown in the following example
要使用Querydsl支持，在存储库接口上扩展QuerydslPredicateExecutor，如下例所示

Example 38. Querydsl integration on repositories

```java
interface UserRepository extends CrudRepository<User, Long>, QuerydslPredicateExecutor<User> {
}
```
The preceding example lets you write typesafe queries using Querydsl Predicate instances, as shown in the following example:
前面的示例允许您使用Querydsl断言实例编写类型安全查询，如下例所示:
```java
Predicate predicate = user.firstname.equalsIgnoreCase("dave")
	.and(user.lastname.startsWithIgnoreCase("mathews"));

userRepository.findAll(predicate);
```

### 1.8.2. Web support
### 1.8.2. Web支持
This section contains the documentation for the Spring Data web support as it is implemented in the current (and later) versions of Spring Data Commons. As the newly introduced support changes many things, we kept the documentation of the former behavior in [web.legacy].
Spring Data modules that support the repository programming model ship with a variety of web support. The web related components require Spring MVC JARs to be on the classpath. Some of them even provide integration with Spring HATEOAS. In general, the integration support is enabled by using the @EnableSpringDataWebSupport annotation in your JavaConfig configuration class, as shown in the following example:
本节包含Spring Data web support的文档，因为它是在Spring Data Commons的当前(和以后)版本中实现的。随着新引入的支持改变了许多东西，我们在[web.legacy]中保留了以前行为的文档。
支持存储库编程模型的Spring Data module附带了各种web支持。web相关组件要求Spring MVC jar位于类路径上。其中一些甚至提供了与Spring HATEOAS的集成。通常，通过在JavaConfig配置类中使用@EnableSpringDataWebSupport注释来启用集成支持，如下例所示:
  
Example 39. Enabling Spring Data web support

```java
@Configuration
@EnableWebMvc
@EnableSpringDataWebSupport
class WebConfiguration {}
```

The @EnableSpringDataWebSupport annotation registers a few components we will discuss in a bit. It will also detect Spring HATEOAS on the classpath and register integration components for it as well if present.
@EnableSpringDataWebSupport注释注册了一些组件，我们将稍后讨论。它还将检测classpath上的Spring HATEOAS，并为其注册集成组件(如果存在的话)。

Alternatively, if you use XML configuration, register either SpringDataWebConfiguration or HateoasAwareSpringDataWebConfiguration as Spring beans, as shown in the following example (for SpringDataWebConfiguration):
或者，如果您使用XML配置，注册SpringDataWebConfiguration或HateoasAwareSpringDataWebConfiguration作为Spring bean，如下例所示(对于SpringDataWebConfiguration):

Example 40. Enabling Spring Data web support in XML

```xml
<bean class="org.springframework.data.web.config.SpringDataWebConfiguration" />

<!-- If you use Spring HATEOAS, register this one *instead* of the former -->
<bean class="org.springframework.data.web.config.HateoasAwareSpringDataWebConfiguration" />
```

#### Basic Web Support
The configuration shown in the previous section registers a few basic components:
前一节所示的配置注册了一些基本组件:
A DomainClassConverter to let Spring MVC resolve instances of repository-managed domain classes from request parameters or path variables.
DomainClassConverter允许Spring MVC从请求参数或路径变量解析存储管理的域类的实例。

HandlerMethodArgumentResolver implementations to let Spring MVC resolve Pageable and Sort instances from request parameters.
HandlerMethodArgumentResolver实现可以让Spring MVC解析可分页的实例，并从请求参数中对实例进行排序。

#### DomainClassConverter
The DomainClassConverter lets you use domain types in your Spring MVC controller method signatures directly, so that you need not manually lookup the instances through the repository, as shown in the following example:
DomainClassConverter允许您在Spring MVC控制器方法签名中直接使用域类型，这样您就不需要通过存储库手动查找实例，如下面的示例所示:
    
Example 41. A Spring MVC controller using domain types in method signatures

```java
@Controller
@RequestMapping("/users")
class UserController {

  @RequestMapping("/{id}")
  String showUserForm(@PathVariable("id") User user, Model model) {

    model.addAttribute("user", user);
    return "userForm";
  }
}
```

As you can see, the method receives a User instance directly, and no further lookup is necessary. The instance can be resolved by letting Spring MVC convert the path variable into the id type of the domain class first and eventually access the instance through calling findById(…) on the repository instance registered for the domain type.
如您所见，该方法直接接收用户实例，不需要进一步查找。实例可以通过让Spring MVC首先将路径变量转换为域类的id类型，并最终通过调用为域类型注册的存储库实例上的findById(…)来访问实例来解决。

Currently, the repository has to implement CrudRepository to be eligible to be discovered for conversion.
HandlerMethodArgumentResolvers for Pageable and Sort
The configuration snippet shown in the previous section also registers a PageableHandlerMethodArgumentResolver as well as an instance of SortHandlerMethodArgumentResolver. The registration enables Pageable and Sort as valid controller method arguments, as shown in the following example:
目前，存储库必须实现CrudRepository，才有资格被发现进行转换。
可分页和排序的HandlerMethodArgumentResolvers
前面部分中显示的配置片段还注册了PageableHandlerMethodArgumentResolver以及SortHandlerMethodArgumentResolver实例。注册使可分页和排序成为有效的控制器方法参数，如下例所示:

Example 42. Using Pageable as controller method argument

```java
@Controller
@RequestMapping("/users")
class UserController {

  private final UserRepository repository;

  UserController(UserRepository repository) {
    this.repository = repository;
  }

  @RequestMapping
  String showUsers(Model model, Pageable pageable) {

    model.addAttribute("users", repository.findAll(pageable));
    return "users";
  }
}
```

The preceding method signature causes Spring MVC try to derive a Pageable instance from the request parameters by using the following default configuration:
前面的方法签名使Spring MVC试图通过使用以下默认配置从请求参数中派生一个可分页实例:
    
Table 1. Request parameters evaluated for Pageable instances
为可分页实例评估请求参数
    
| 参数 | 分析 |
|---|---|
|page|Page you want to retrieve. 0-indexed and defaults to 0.要检索的页面，0索引，默认为0。|
|size|Size of the page you want to retrieve. Defaults to 20.要检索的页面的大小。默认为20。|
|sort|Properties that should be sorted by in the format property,property(ASC\DESC). Default sort direction is ascending. Use multiple sort parameters if you want to switch directions — for example, ?sort=firstname&sort=lastname,asc.应该按格式属性、属性(ASC\DESC)排序的属性。默认排序方向是递增的。如果你想转换方向，可以使用多个排序参数——例如?sort=firstname&sort=lastname,asc。|

To customize this behavior, register a bean implementing the PageableHandlerMethodArgumentResolverCustomizer interface or the SortHandlerMethodArgumentResolverCustomizer interface, respectively. Its customize() method gets called, letting you change settings, as shown in the following example:
要定制此行为，分别注册一个实现pageablehandlermethodargumentresolvercustomzer接口或sorthandlermethodargumentresolvercustomzer接口的bean。它的customize()方法被调用，允许您更改设置，如下例所示:

```java
@Bean SortHandlerMethodArgumentResolverCustomizer sortCustomizer() {
    return s -> s.setPropertyDelimiter("<-->");
}
```

If setting the properties of an existing MethodArgumentResolver is not sufficient for your purpose, extend either SpringDataWebConfiguration or the HATEOAS-enabled equivalent, override the pageableResolver() or sortResolver() methods, and import your customized configuration file instead of using the @Enable annotation.
如果设置现有的MethodArgumentResolver的属性不足以满足您的目的，那么扩展SpringDataWebConfiguration或启用hateoas的等效项，覆盖pageableResolver()或sortResolver()方法，导入定制的配置文件，而不是使用@Enable注释。

If you need multiple Pageable or Sort instances to be resolved from the request (for multiple tables, for example), you can use Spring’s @Qualifier annotation to distinguish one from another. The request parameters then have to be prefixed with ${qualifier}_. The followig example shows the resulting method signature:
如果需要从请求中解析多个可分页实例或排序实例(例如，对于多个表)，可以使用Spring的@Qualifier注释来区分它们。然后，请求参数必须以${qualifier}_为前缀。followig示例显示得到的方法签名:

```java
String showUsers(Model model,
      @Qualifier("thing1") Pageable first,
      @Qualifier("thing2") Pageable second) { … }
```

you have to populate thing1_page and thing2_page and so on.
你需要填充thing1_page和thing2_page等等。
    
The default Pageable passed into the method is equivalent to a new PageRequest(0, 20) but can be customized by using the @PageableDefault annotation on the Pageable parameter.
传入方法的默认Pageable等同于一个新的PageRequest(0,20)，但是可以通过在Pageable参数上使用@PageableDefault注释进行定制。
#### Hypermedia Support for Pageables
#### 超媒体支持分页    
    
Spring HATEOAS ships with a representation model class (PagedResources) that allows enriching the content of a Page instance with the necessary Page metadata as well as links to let the clients easily navigate the pages. The conversion of a Page to a PagedResources is done by an implementation of the Spring HATEOAS ResourceAssembler interface, called the PagedResourcesAssembler. The following example shows how to use a PagedResourcesAssembler as a controller method argument:
Spring HATEOAS附带了一个表示模型类(PagedResources)，它允许使用必要的页面元数据以及链接来丰富页面实例的内容，以便客户端轻松地导航页面。页面到PagedResources的转换是通过Spring HATEOAS ResourceAssembler接口(称为PagedResourcesAssembler)的实现完成的。下面的例子展示了如何使用PagedResourcesAssembler作为控制器方法参数:

Example 43. Using a PagedResourcesAssembler as controller method argument

``` java
@Controller
class PersonController {

  @Autowired PersonRepository repository;

  @RequestMapping(value = "/persons", method = RequestMethod.GET)
  HttpEntity<PagedResources<Person>> persons(Pageable pageable,
    PagedResourcesAssembler assembler) {

    Page<Person> persons = repository.findAll(pageable);
    return new ResponseEntity<>(assembler.toResources(persons), HttpStatus.OK);
  }
}
```

Enabling the configuration as shown in the preceding example lets the PagedResourcesAssembler be used as a controller method argument. Calling toResources(…) on it has the following effects:
启用如上例所示的配置，可以将PagedResourcesAssembler用作控制器方法参数。调用它的toResources(…)有以下效果:
    
* The content of the Page becomes the content of the PagedResources instance.
页面的内容成为PagedResources实例的内容。
    
* The PagedResources object gets a PageMetadata instance attached, and it is populated with information from the Page and the underlying PageRequest.
PagedResources对象获得一个附加的PageMetadata实例，并用来自页面和底层PageRequest的信息填充它。
    
* The PagedResources may get prev and next links attached, depending on the page’s state. The links point to the URI to which the method maps. The pagination parameters added to the method match the setup of the PageableHandlerMethodArgumentResolver to make sure the links can be resolved later.
PagedResources可能会得到prev和next链接，这取决于页面的状态。这些链接指向方法映射到的URI。添加到方法中的分页参数与PageableHandlerMethodArgumentResolver的设置相匹配，以确保链接可以在以后得到解决。

Assume we have 30 Person instances in the database. You can now trigger a request (GET http://localhost:8080/persons) and see output similar to the following:
假设数据库中有30个Person实例。现在您可以触发一个请求(获取http://localhost:8080/persons)，并看到与以下类似的输出:

```json
{ "links" : [ { "rel" : "next",
                "href" : "http://localhost:8080/persons?page=1&size=20 }
  ],
  "content" : [
     … // 20 Person instances rendered here
  ],
  "pageMetadata" : {
    "size" : 20,
    "totalElements" : 30,
    "totalPages" : 2,
    "number" : 0
  }
}
```

You see that the assembler produced the correct URI and also picked up the default configuration to resolve the parameters into a Pageable for an upcoming request. This means that, if you change that configuration, the links automatically adhere to the change. By default, the assembler points to the controller method it was invoked in, but that can be customized by handing in a custom Link to be used as base to build the pagination links, which overloads the PagedResourcesAssembler.toResource(…) method.
您可以看到，汇编程序生成了正确的URI，并获得了默认配置，以便将参数解析为即将到来的请求的可分页的。这意味着，如果您更改了配置，链接将自动与更改保持一致。默认情况下，汇编程序指向调用它的控制器方法，但是可以通过提交一个定制的链接来定制，该链接用作构建分页链接的基础，这会重载pagedresourcesassembly . toresource(…)方法。

#### Web Databinding Support
#### Web数据绑定支持
Spring Data projections (described in [projections]) can be used to bind incoming request payloads by either using JSONPath expressions (requires Jayway JsonPath or XPath expressions (requires XmlBeam), as shown in the following example:
Spring Data投影(在[projection]中描述)可以通过使用JSONPath表达式(需要Jayway JSONPath或XPath表达式(需要XmlBeam)来绑定传入的请求有效负载，如下例所示:

Example 44. HTTP payload binding using JSONPath or XPath expressions

```java
@ProjectedPayload
public interface UserPayload {

  @XBRead("//firstname")
  @JsonPath("$..firstname")
  String getFirstname();

  @XBRead("/lastname")
  @JsonPath({ "$.lastname", "$.user.lastname" })
  String getLastname();
}
```

The type shown in the preceding example can be used as a Spring MVC handler method argument or by using ParameterizedTypeReference on one of RestTemplate's methods. The preceding method declarations would try to find firstname anywhere in the given document. The lastname XML lookup is performed on the top-level of the incoming document. The JSON variant of that tries a top-level lastname first but also tries lastname nested in a user sub-document if the former does not return a value. That way, changes in the structure of the source document can be mitigated easily without having clients calling the exposed methods (usually a drawback of class-based payload binding).
上面示例中显示的类型可以用作Spring MVC处理程序方法参数，也可以使用RestTemplate方法之一的参数化pereference。前面的方法声明将尝试在给定文档中的任何位置查找firstname。在传入文档的顶层执行lastname XML查找。它的JSON变体首先尝试顶级的lastname，但如果lastname不返回值，则尝试嵌套在用户子文档中的lastname。这样，无需客户调用公开的方法，源文档结构的更改就可以很容易地得到缓解(这通常是基于类的有效负载绑定的缺点)。

Nested projections are supported as described in [projections]. If the method returns a complex, non-interface type, a Jackson ObjectMapper is used to map the final value.
嵌套投影在[投影]中描述。如果该方法返回复杂的非接口类型，则使用Jackson ObjectMapper映射最终值。

For Spring MVC, the necessary converters are registered automatically as soon as @EnableSpringDataWebSupport is active and the required dependencies are available on the classpath. For usage with RestTemplate, register a ProjectingJackson2HttpMessageConverter (JSON) or XmlBeamHttpMessageConverter manually.
对于Spring MVC，只要@EnableSpringDataWebSupport处于活动状态且所需依赖项在类路径上可用，就会自动注册必要的转换器。要使用RestTemplate，手动注册一个ProjectingJackson2HttpMessageConverter (JSON)或XmlBeamHttpMessageConverter。

For more information, see the web projection example in the canonical Spring Data Examples repository.
有关更多信息，请参阅规范化Spring Data示例存储库中的web投影示例。

#### Querydsl Web Support
#### Querydsl Web 支持
For those stores having QueryDSL integration, it is possible to derive queries from the attributes contained in a Request query string.
对于那些具有QueryDSL集成的存储，可以从请求查询字符串中包含的属性派生查询。

Consider the following query string:
思考以下查询字符串:

```
?firstname=Dave&lastname=Matthews
```
    
Given the User object from previous examples, a query string can be resolved to the following value by using the QuerydslPredicateArgumentResolver.
对于前面示例中的User对象，可以使用QuerydslPredicateArgumentResolver将查询字符串解析为以下值。

```
QUser.user.firstname.eq("Dave").and(QUser.user.lastname.eq("Matthews"))
```

The feature is automatically enabled, along with @EnableSpringDataWebSupport, when Querydsl is found on the classpath.
当在类路径中找到Querydsl时，该特性与@EnableSpringDataWebSupport一起自动启用。
    
Adding a @QuerydslPredicate to the method signature provides a ready-to-use Predicate, which can be run by using the QuerydslPredicateExecutor.
向方法签名中添加@QuerydslPredicate可以提供一个随时可用的断言，该断言可以通过使用QuerydslPredicateExecutor来运行。

Type information is typically resolved from the method’s return type. Since that information does not necessarily match the domain type, it might be a good idea to use the root attribute of QuerydslPredicate.
The following exampe shows how to use @QuerydslPredicate in a method signature:
类型信息通常从方法的返回类型解析。由于该信息不一定与域类型匹配，因此使用QuerydslPredicate的根属性可能是一个好主意。
下面的示例展示了如何在方法签名中使用@QuerydslPredicate:

```java
@Controller
class UserController {

  @Autowired UserRepository repository;

  @RequestMapping(value = "/", method = RequestMethod.GET)
  String index(Model model, @QuerydslPredicate(root = User.class) Predicate predicate,    
          Pageable pageable, @RequestParam MultiValueMap<String, String> parameters) {

    model.addAttribute("users", repository.findAll(predicate, pageable));

    return "index";
  }
}
```

Resolve query string arguments to matching Predicate for User.
为用户解决查询字符串参数匹配断言。

The default binding is as follows:
默认绑定如下:

* Object on simple properties as eq.
对象的简单属性，如eq。
* Object on collection like properties as contains.
对象的集合，如包含的属性。
* Collection on simple properties as in.
简单属性的集合。
    
Those bindings can be customized through the bindings attribute of @QuerydslPredicate or by making use of Java 8 default methods and adding the QuerydslBinderCustomizer method to the repository interface.
这些绑定可以通过@QuerydslPredicate的bindings属性进行定制，或者使用Java 8默认方法并将querydslbindercustomzer方法添加到存储库接口。
```java
interface UserRepository extends CrudRepository<User, String>,
                                 QuerydslPredicateExecutor<User>,                
                                 QuerydslBinderCustomizer<QUser> {               

  @Override
  default void customize(QuerydslBindings bindings, QUser user) {

    bindings.bind(user.username).first((path, value) -> path.contains(value))    
    bindings.bind(String.class)
      .first((StringPath path, String value) -> path.containsIgnoreCase(value)); 
    bindings.excluding(user.password);                                           
  }
}
1. QuerydslPredicateExecutor provides access to specific finder methods for Predicate.
QuerydslPredicateExecutor提供对断言的特定查找程序方法的访问。

2. QuerydslBinderCustomizer defined on the repository interface is automatically picked up and shortcuts @QuerydslPredicate(bindings=…​).
在存储库接口上定义的querydslbindercustomzer将被自动拾取并使用@QuerydslPredicate(bindings=…)的快捷方式。

3. Define the binding for the username property to be a simple contains binding.
将username属性的绑定定义为简单的包含绑定。

4. Define the default binding for String properties to be a case-insensitive contains match.
将字符串属性的默认绑定定义为不区分大小写的contains match。
    
5. Exclude the password property from Predicate resolution.
从断言解析中排除密码属性。
```

### 1.8.3. Repository Populators
### 1.8.3. Repository 思想
If you work with the Spring JDBC module, you are probably familiar with the support to populate a DataSource with SQL scripts. A similar abstraction is available on the repositories level, although it does not use SQL as the data definition language because it must be store-independent. Thus, the populators support XML (through Spring’s OXM abstraction) and JSON (through Jackson) to define data with which to populate the repositories.
如果您使用Spring JDBC模块，您可能熟悉用SQL脚本填充数据源的支持。在存储库级别上也可以使用类似的抽象，尽管它不使用SQL作为数据定义语言，因为它必须是与存储无关的。因此，填充器支持XML(通过Spring的OXM抽象)和JSON(通过Jackson)来定义用来填充存储库的数据。

Assume you have a file data.json with the following content:
假设您有一个文件数据。json，内容如下:
    
Example 45. Data defined in JSON

```json
[ { "_class" : "com.acme.Person",
 "firstname" : "Dave",
  "lastname" : "Matthews" },
  { "_class" : "com.acme.Person",
 "firstname" : "Carter",
  "lastname" : "Beauford" } ]
```

    
You can populate your repositories by using the populator elements of the repository namespace provided in Spring Data Commons. To populate the preceding data to your PersonRepository, declare a populator similar to the following:
您可以使用Spring Data Commons中提供的存储库名称空间的populator元素来填充存储库。要将前面的数据填充到您的个人存储库中，请声明一个与以下类似的填充器:

Example 46. Declaring a Jackson repository populator

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xmlns:repository="http://www.springframework.org/schema/data/repository"
  xsi:schemaLocation="http://www.springframework.org/schema/beans
    http://www.springframework.org/schema/beans/spring-beans.xsd
    http://www.springframework.org/schema/data/repository
    http://www.springframework.org/schema/data/repository/spring-repository.xsd">

  <repository:jackson2-populator locations="classpath:data.json" />

</beans>
```

The preceding declaration causes the data.json file to be read and deserialized by a Jackson ObjectMapper.
前面的声明导致数据。json文件由Jackson对象映射程序读取和反序列化。

The type to which the JSON object is unmarshalled is determined by inspecting the \_class attribute of the JSON document. The infrastructure eventually selects the appropriate repository to handle the object that was deserialized.
解析JSON对象的类型是通过检查JSON文档的\_class属性确定的。基础结构最终选择适当的存储库来处理反序列化的对象。

To instead use XML to define the data the repositories should be populated with, you can use the unmarshaller-populator element. You configure it to use one of the XML marshaller options available in Spring OXM. See the Spring reference documentation for details. The following example shows how to unmarshal a repository populator with JAXB:
要使用XML定义存储库应该填充的数据，可以使用unmarshaller-populator元素。您可以将其配置为使用Spring OXM中可用的XML编组器选项之一。有关详细信息，请参阅Spring参考文档。下面的示例展示了如何使用JAXB解压存储库填充器:

Example 47. Declaring an unmarshalling repository populator (using JAXB)

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xmlns:repository="http://www.springframework.org/schema/data/repository"
  xmlns:oxm="http://www.springframework.org/schema/oxm"
  xsi:schemaLocation="http://www.springframework.org/schema/beans
    http://www.springframework.org/schema/beans/spring-beans.xsd
    http://www.springframework.org/schema/data/repository
    http://www.springframework.org/schema/data/repository/spring-repository.xsd
    http://www.springframework.org/schema/oxm
    http://www.springframework.org/schema/oxm/spring-oxm.xsd">

  <repository:unmarshaller-populator locations="classpath:data.json"
    unmarshaller-ref="unmarshaller" />

  <oxm:jaxb2-marshaller contextPath="com.acme" />

</beans>
```

## Reference Documentation
## 2. Elasticsearch Repositories
This chapter includes details of the Elasticsearch repository implementation.
本章包括Elasticsearch存储库实现的详细信息。

### 2.1. Introduction
### 2.1.1. Spring Namespace
The Spring Data Elasticsearch module contains a custom namespace allowing definition of repository beans as well as elements for instantiating a ElasticsearchServer.
Spring Data Elasticsearch模块包含一个定制的名称空间，允许定义存储库bean以及实例化ElasticsearchServer的元素。

Using the repositories element looks up Spring Data repositories as described in Creating Repository Instances .
使用Repository元素查找Spring Data Repository ，如创建存储库实例中所述。
    
Example 48. Setting up Elasticsearch repositories using Namespace
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
xmlns:elasticsearch="http://www.springframework.org/schema/data/elasticsearch"
xsi:schemaLocation="http://www.springframework.org/schema/beans
http://www.springframework.org/schema/beans/spring-beans-3.1.xsd
http://www.springframework.org/schema/data/elasticsearch
http://www.springframework.org/schema/data/elasticsearch/spring-elasticsearch-1.0.xsd">

    <elasticsearch:repositories base-package="com.acme.repositories" />

</beans>
```
    
Using the Transport Client or Node Client element registers an instance of Elasticsearch Server in the context.
使用传输客户机或节点客户机元素在上下文中注册一个Elasticsearch服务器的实例。
    
Example 49. Transport Client using Namespace
使用命名空间的传输客户端

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
xmlns:elasticsearch="http://www.springframework.org/schema/data/elasticsearch"
xsi:schemaLocation="http://www.springframework.org/schema/beans
http://www.springframework.org/schema/beans/spring-beans-3.1.xsd
http://www.springframework.org/schema/data/elasticsearch
http://www.springframework.org/schema/data/elasticsearch/spring-elasticsearch-1.0.xsd">

    <elasticsearch:transport-client id="client" cluster-nodes="localhost:9300,someip:9300" />

</beans>
```
    
Example 50. Node Client using Namespace
使用命名空间的节点客户端

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
xmlns:elasticsearch="http://www.springframework.org/schema/data/elasticsearch"
xsi:schemaLocation="http://www.springframework.org/schema/beans
http://www.springframework.org/schema/beans/spring-beans-3.1.xsd
http://www.springframework.org/schema/data/elasticsearch
http://www.springframework.org/schema/data/elasticsearch/spring-elasticsearch-1.0.xsd">

    <elasticsearch:node-client id="client" local="true"" />

</beans>                                                        
```
                                                        
### 2.1.2. Annotation based configuration
The Spring Data Elasticsearch repositories support cannot only be activated through an XML namespace but also using an annotation through JavaConfig.
Spring Data Elasticsearch存储库支持不仅可以通过XML名称空间激活，还可以通过JavaConfig使用注释。

Example 51. Spring Data Elasticsearch repositories using JavaConfig

```java                                                     
@Configuration
@EnableElasticsearchRepositories(basePackages = "org/springframework/data/elasticsearch/repositories")
static class Config {

    @Bean
    public ElasticsearchOperations elasticsearchTemplate() {
        return new ElasticsearchTemplate(nodeBuilder().local(true).node().client());
    }
}
```

The configuration above sets up an Embedded Elasticsearch Server which is used by the ElasticsearchTemplate . Spring Data Elasticsearch Repositories are activated using the @EnableElasticsearchRepositories annotation, which essentially carries the same attributes as the XML namespace does. If no base package is configured, it will use the one the configuration class resides in.
上面的配置设置了一个内嵌的Elasticsearch服务器，这是Elasticsearch Template使用的。Spring Data Elasticsearch存储库使用@EnableElasticsearchRepositories注释激活，其本质上具有与XML名称空间相同的属性。如果没有配置基本包，它将使用配置类所在的包。

2.1.3. Elasticsearch Repositores using CDI
The Spring Data Elasticsearch repositories can also be set up using CDI functionality.
还可以使用CDI功能设置Spring Data Elasticsearch存储库。

Example 52. Spring Data Elasticsearch repositories using JavaConfig

```java
class ElasticsearchTemplateProducer {

    @Produces
    @ApplicationScoped
    public ElasticsearchOperations createElasticsearchTemplate() {
        return new ElasticsearchTemplate(nodeBuilder().local(true).node().client());
    }
}

class ProductService {

    private ProductRepository repository;

    public Page<Product> findAvailableBookByName(String name, Pageable pageable) {
        return repository.findByAvailableTrueAndNameStartingWith(name, pageable);
    }

    @Inject
    public void setRepository(ProductRepository repository) {
        this.repository = repository;
    }
}
```

### 2.2. Query methods
### 2.2.1. Query lookup strategies
### 2.2.1. 查询查找策略
The Elasticsearch module supports all basic query building feature as String,Abstract,Criteria or have it being derived from the method name.
Elasticsearch模块支持所有基本的查询构建功能，如字符串、抽象、标准或从方法名派生。

#### Declared queries
Deriving the query from the method name is not always sufficient and/or may result in unreadable method names. In this case one might make either use of @Query annotation (see Using @Query Annotation ).
从方法名派生查询并不总是足够的，并且/或可能导致不可读的方法名。在这种情况下，可以使用@Query注释(请参阅使用@Query注释)。

### 2.2.2. Query creation
Generally the query creation mechanism for Elasticsearch works as described in Query methods . Here’s a short example of what a Elasticsearch query method translates into:
通常情况下，Elasticsearch的查询创建机制在查询方法中描述。下面是一个关于Elasticsearch查询方法的简单示例:

Example 53. Query creation from method names

```java
public interface BookRepository extends Repository<Book, String>
{
    List<Book> findByNameAndPrice(String name, Integer price);
}
```

The method name above will be translated into the following Elasticsearch json query
上面的方法名称将转换为以下Elasticsearch json查询

```json
{ "bool" :
    { "must" :
        [
            { "field" : {"name" : "?"} },
            { "field" : {"price" : "?"} }
        ]
    }
}
```

A list of supported keywords for Elasticsearch is shown below.
下面显示了对Elasticsearch支持的关键字的列表。

Table 2. Supported keywords inside method names
Table 2. 方法名称中支持的关键字

| Keyword | Sample | Elasticsearch Query String |
|---|---|---|
| And | findByNameAndPrice |{"bool" : {"must" : [ {"field" : {"name" : "?"}}, {"field" : {"price" : "?"}} ]}}|
| And | findByNameAndPrice | {"bool" : {"should" : [ {"field" : {"name" : "?"}}, {"field" : {"price" : "?"}} ]} |
| Or | findByNameOrPrice | {"bool" : {"should" : [ {"field" : {"name" : "?"}}, {"field" : {"price" : "?"}} ]}}|
| Is | findByName | {"bool" : {"must" : {"field" : {"name" : "?"}}}} |
| Not | findByNameNot |  {"bool" : {"must_not" : {"field" : {"name" : "?"}}}} |
| Between | findByPriceBetween | {"bool" : {"must" : {"range" : {"price" : {"from" : ?,"to" : ?,"include_lower" : true,"include_upper" : true}}}}} |
| LessThanEqual | findByPriceLessThan | {"bool" : {"must" : {"range" : {"price" : {"from" : null,"to" : ?,"include_lower" : true,"include_upper" : true}}}}} |
| GreaterThanEqual | findByPriceGreaterThan | {"bool" : {"must" : {"range" : {"price" : {"from" : ?,"to" : null,"include_lower" : true,"include_upper" : true}}}}} |
| Before | findByPriceBefore | {"bool" : {"must" : {"range" : {"price" : {"from" : null,"to" : ?,"include_lower" : true,"include_upper" : true}}}}} |
| After | findByPriceAfter | {"bool" : {"must" : {"range" : {"price" : {"from" : ?,"to" : null,"include_lower" : true,"include_upper" : true}}}}} |
| Like | findByNameLike | {"bool" : {"must" : {"field" : {"name" : {"query" : "?*","analyze_wildcard" : true}}}}} |
| StartingWith | findByNameStartingWith | {"bool" : {"must" : {"field" : {"name" : {"query" : "?\*","analyze_wildcard" : true}}}}} |
| EndingWith | findByNameEndingWith | {"bool" : {"must" : {"field" : {"name" : {"query" : "\*?","analyze_wildcard" : true}}}}} |
| Contains/Containing | findByNameContaining | {"bool" : {"must" : {"field" : {"name" : {"query" : "?","analyze_wildcard" : true}}}}} |
| In | findByNameIn(Collection<String>names) | {"bool" : {"must" : {"bool" : {"should" : [ {"field" : {"name" : "?"}}, {"field" : {"name" : "?"}} ]}}}} |
| NotIn | findByNameNotIn(Collection<String>names) | {"bool" : {"must_not" : {"bool" : {"should" : {"field" : {"name" : "?"}}}}}} |
| Near | findByStoreNear | Not Supported Yet ! |
| True | findByAvailableTrue | {"bool" : {"must" : {"field" : {"available" : true}}}} |
| False | findByAvailableFalse | {"bool" : {"must" : {"field" : {"available" : false}}}} |
| OrderBy | findByAvailableTrueOrderByNameDesc | {"sort" : [{ "name" : {"order" : "desc"} }],"bool" : {"must" : {"field" : {"available" : true}}}}|

2.2.3. Using @Query Annotation
Example 54. Declare query at the method using the @Query annotation.
使用@Query注释在方法上声明查询。

```java
public interface BookRepository extends ElasticsearchRepository<Book, String> {
    @Query("{"bool" : {"must" : {"field" : {"name" : "?0"}}}}")
    Page<Book> findByName(String name,Pageable pageable);
}
```

## 3. Miscellaneous Elasticsearch Operation Support
This chapter covers additional support for Elasticsearch operations that cannot be directly accessed via the repository interface. It is recommended to add those operations as custom implementation as described in Custom Implementations for Spring Data Repositories .

本章将介绍不能通过存储库接口直接访问的Elasticsearch操作的额外支持。建议将这些操作作为定制实现添加到Spring Data Repository 的定制实现中。

### 3.1. Filter Builder
Filter Builder improves query speed.

Filter Builder提高了查询速度。

```java
private ElasticsearchTemplate elasticsearchTemplate;

SearchQuery searchQuery = new NativeSearchQueryBuilder()
    .withQuery(matchAllQuery())
    .withFilter(boolFilter().must(termFilter("id", documentId)))
    .build();

Page<SampleEntity> sampleEntities =
    elasticsearchTemplate.queryForPage(searchQuery,SampleEntity.class);
```

### 3.2. Using Scan And Scroll For Big Result Set
### 3.2. 使用扫描和滚动大的结果集

Elasticsearch has scan and scroll feature for getting big result set in chunks. 
Elasticsearch有扫描和滚动功能，可以获得大块的结果集。

ElasticsearchTemplate has scan and scroll methods that can be used as below.
ElasticsearchTemplate有扫描和滚动方法，可以如下所示使用。

Example 55. Using Scan and Scroll

```java
SearchQuery searchQuery = new NativeSearchQueryBuilder()
    .withQuery(matchAllQuery())
    .withIndices("test-index")
    .withTypes("test-type")
    .withPageable(new PageRequest(0,1))
    .build();
String scrollId = elasticsearchTemplate.scan(searchQuery,1000,false);
List<SampleEntity> sampleEntities = new ArrayList<SampleEntity>();
boolean hasRecords = true;
while (hasRecords){
    Page<SampleEntity> page = elasticsearchTemplate.scroll(scrollId, 5000L , new ResultsMapper<SampleEntity>()
    {
        @Override
        public Page<SampleEntity> mapResults(SearchResponse response) {
            List<SampleEntity> chunk = new ArrayList<SampleEntity>();
            for(SearchHit searchHit : response.getHits()){
                if(response.getHits().getHits().length <= 0) {
                    return null;
                }
                SampleEntity user = new SampleEntity();
                user.setId(searchHit.getId());
                user.setMessage((String)searchHit.getSource().get("message"));
                chunk.add(user);
            }
            return new PageImpl<SampleEntity>(chunk);
        }
    });
    if(page != null) {
        sampleEntities.addAll(page.getContent());
        hasRecords = page.hasNextPage();
    }
    else{
        hasRecords = false;
    }
    }
}
```
