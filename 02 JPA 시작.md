> 해당 파트는 실제 실습 환경을 구성하면서 진행하기에   
> 중요하다고 생각하는 부분만 정리하겠습니다.  
    
# persistence.xml 설정    
JPA는 `persistence.xml`을 사용해서 필요한 설정 정보를 관리한다.     
이 설정 파일이 `META-INF/persistence.xml`클래스 패스 경로에 있으면    
별도의 추가 설정 없이 JPA가 인식할 수 있다.        

**실습에서 제공된 persistence.xml**
```xml   
<?xml version="1.0" encoding="UTF-8"?>
<persistence version="2.2"
             xmlns="http://xmlns.jcp.org/xml/ns/persistence" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
             xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/persistence http://xmlns.jcp.org/xml/ns/persistence/persistence_2_2.xsd">
    <persistence-unit name="hello">
        <properties>
            <!-- 필수 속성 -->
            <property name="javax.persistence.jdbc.driver" value="org.h2.Driver"/>
            <property name="javax.persistence.jdbc.user" value="sa"/>
            <property name="javax.persistence.jdbc.password" value=""/>
            <property name="javax.persistence.jdbc.url" value="jdbc:h2:tcp://localhost/~/test"/>
            <property name="hibernate.dialect" value="org.hibernate.dialect.H2Dialect"/>
            <!-- 옵션 -->
            <property name="hibernate.show_sql" value="true"/>
            <property name="hibernate.format_sql" value="true"/>
            <property name="hibernate.use_sql_comments" value="true"/>
            <!--<property name="hibernate.hbm2ddl.auto" value="create" />-->
        </properties>
    </persistence-unit>
</persistence>
```
위 `xml` 에 대해서 분석을 진행해보겠다.   
   
**필수 속성**   
* `javax.persistence.jdbc.driver` : JDBC 드라이버 
* `javax.persistence.jdbc.user` : 데이터베이스 접속 ID 
* `javax.persistence.jdbc.password` : 데이터베이스 접속 PASSWORD
* `javax.persistence.jdbc.url` : 데이터베이스 접속 URL  
* `hibernate.dialect` : 데이터베이스 방언 설정 
      
**선택 속성**     
* `hibernate.show_sql` : hibernate가 실행한 SQL을 출력한다.    
* `hibernate.format_sql` : hibernate가 실행한 SQL을 보기 좋게 정렬한다.      
* `hibernate.use_sql_comments` : hibernate가 실행한 SQL을 출력할 때 주석도 같이 출력한다.      
* `hibernate.id.new_generator_mappings` :JPA 표준에 맞춘 새로운 키 생성 전략을 사용한다. (이후 정리)    
   
**참고**   
JPA 구현체들은 보통 엔티티 클래스를 자동으로 인식한다.      
하지만, 환경에 따라 인식하지 못하는 경우도 있다.   
그런 경우 아래와 같은 태그를 사용해서 인식하도록 하자.   
  
```xml  
<persistence-unit name="jpabook">   
    <class>jpabook.start.Member</class>
    <properties>
        ...
...        
```
  
# 데이터베이스 방언 
> 해당 부분은 개념적으로만 이해하기 위해서 정리합니다.  
  
JPA는 데이터베이스에 종속적이지 않다.     
하지만, 각 데이터베이스마다 제공되는 SQL 문법이 다르다는 문제가 있다.      
물론, 표준 SQL도 존재하지만 데이터베이스마다 다른 고유한 SQL도 있다.  
 
**대표적인 예**  
* **데이터 타입 :**    
    * MYSQL : VARCHAR  
    * ORACLE : VARCHAR2   
* **다른 함수명 :**    
    * MYSQL : SUBSTRING   
    * ORACLE : SUBSTR     
* **페이징 처리 :**   
    * MYSQL : LIMIT
    * ORACLE : ROWNUM 
   
JPA 또한 해당 데이터베이스의 방언에 맞게끔 SQL 변환 작업을 거쳐야한다.        
따라서, 특정 데이터베이스에 대한 방언 정보를 아래와 같이 넘겨주자     
  
```xml
<property name="hibernate.dialect" value="org.hibernate.dialect.H2Dialect"/>
```  
     
**hibernate.dialect 속성**
* **MySQL :** `org.hibernate.dialect.MySQL5InnoDBDialect`
* **Oracle :** `org.hibernate.dialect.Oracle10gDialect`
* **H2 :** `org.hibernate.dialect.H2Dialect`    
* 이 외에도 40가지 이상의 데이터베이스 방언 지원한다.    

보자 자세한 내용은 [공식 레퍼런스](https://docs.jboss.org/hibernate/orm/3.5/javadocs/org/hibernate/dialect/package-summary.html)를 통해서 확인하자   
   
# 코드 분석하기       
> 해당 부분은 일종의 에피타이저입니다.      
> 즉, 간단하게 내용에 대해 맛 보기 위해서 정리합니다.       
    
```java
package hellojpa;

import javax.persistence.EntityManager;
import javax.persistence.EntityManagerFactory;
import javax.persistence.EntityTransaction;
import javax.persistence.Persistence;
import java.util.List;

public class JpaMain {
    public static void main(String[] args) {
        EntityManagerFactory emf = Persistence.createEntityManagerFactory("hello");
        EntityManager em = emf.createEntityManager();

        EntityTransaction tx = em.getTransaction();
        
        try {
            tx.begin();
            logic(em);
            tx.commit();
        } catch (Exception e) {
            tx.rollback();
        } finally {
            em.close();
        }

        emf.close();
    }
    
    private static void logic(EntityManager em) {
        List<Member> result = em.createQuery("select m from Member as m", Member.class)
            .setFirstResult(5)
            .setMaxResults(0)
            .getResultList();
    }
}
```
코드는 크게 3부분으로 나뉘어 있다.   

* 엔티티 매니저 설정 
* 트랜잭션 관리 
* 비즈니스 로직 관리   

  
# 엔티티 매니저 설정   
## 엔티티 매니저 팩토리 
```xml
<?xml version="1.0" encoding="UTF-8"?>
<persistence version="2.2"
             xmlns="http://xmlns.jcp.org/xml/ns/persistence" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
             xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/persistence http://xmlns.jcp.org/xml/ns/persistence/persistence_2_2.xsd">
    <persistence-unit name="hello"> <!--여기 이름과-->
```
```java
        EntityManagerFactory emf = Persistence.createEntityManagerFactory("hello"); // 여기 이름이 동일해야한다.  
```
 
JPA를 시작하려면 우선,     
`persistence.xml`의 설정 정보를 사용해서 `EntityManagerFactory`를 생성해야 한다.           
이때, `persistence.xml`의 영속성 유닛의 이름과 동일한 인자값을 넘겨줘야한다.       
 
`EntityManagerFactory`는 `persistence.xml`를 통해서     
JPA를 동작시키기 위한 기반 객체 생성 및, `DB Connection Pool` 도 생성한다.      
즉, 비용이 아주 큰 작업을 하는 셈이다.   
**따라서, `EntityManagerFactory`는 애플리케이션 전체에서 딱 한번만 생성하고 공유하면서 사용하자**     
참고로 공유하면서 사용할 수 있다는 것은 `thread-safe`하게 설계 되었다는 뜻이기도 하다.   

## 엔티티 매니저
```java
        EntityManagerFactory emf = Persistence.createEntityManagerFactory("hello");
        EntityManager em = emf.createEntityManager();
```
`EntityManagerFactory` 인스턴스를 통해 `EntityManager`인스턴스를 생성한다.   
JPA의 대부분의 기능들은 이 `EntityManager`인스턴스가 수행한다. (CRUD)   
    
`EntityManagerFactory` 인스턴스는 내부에     
`데이터 소스(커넥션)`을 유지하면서 데이터베이스와 통신한다.     
따라서 `EntityManagerFactory`를 가상의 DB로 생각할 수 있다.    
참고로, **`EntityManagerFactory`는 스레드간에 공유하거나 재사용하면 안된다.** 
(데이터베이스 커넥션과 밀접한 관계가 있기 때문이다.)    
          
추가적으로, **영속성 컨텍스트가 이 `EntityManagerFactory` 인스턴스에 존재한다.**          
(인스턴스에 존재한다는 뜻은 각 객체마다 존재하고 같은 라이프사이클을 가진다는 뜻이겠죠? ㅎㅎ)       
      
**종료**   
```java
            em.close();
``` 
사용이 끝난 엔티티 매니저는 종료해야한다.     
앞서 말했듯이, 엔티티 매니저는 **데이터베이스 커넥션**을 소유하고 있기에     
종료하지 않을 경우 해당 커넥션을 낭비하고, 성능 저하의 큰 원인이 될 것이다.   

```java
        emf.close();
```

엔티티 매니저 팩토리는 애플리케이션 종료 직전에 종료시키자.  
  
# 트랜잭션 관리   
JPA를 사용하면 항상 트랜잭션 안에서 데이터를 변경해야 한다.       
트랜잭션 없이 데이터를 변경하면 예외를 발생시키기도 한다.   

```java
        EntityTransaction tx = em.getTransaction();
        try {
            tx.begin();
            logic(em);
            tx.commit();
        } catch (Exception e) {
            tx.rollback();
        } finally {
            em.close();
        }

        emf.close();
    }
```
트랜잭션을 시작하려면 `엔티티 매니저`에서 `트랜잭션 API`를 받아와야한다.   
`트랜잭션 API`를 받아온후 `begin()`을 통해 실행 및 `commit()`을 통해 반영시킨다.   
단, 에러가 발생할 경우 `rollback()`을 통해 이전 상태로 돌린다.   

* begin() : 트랜잭션을 시작한다.  
* commit() : 데이터베이스에 반영한다.  
* rollback() : 이전 상태로 돌린다.  
* flush() : 영속성 컨텍스트의 변경 내용을 DB 에 반영한다.   
  
**flush vs commit**
* flush : session state 와 DB를 sync하는것
* commit : flush + end of the unit of work
* 필자 생각 :    
    * flush : DB 반영, 단 트랜잭션에서 보류중인 작업    
    * commit : 반영된 DB 내용을 지속시도록 한다.     
* 이 [사이트](https://classmethod.dev/ko/questions/4201455)도 참고하면 좋다 :)    
  
보다 자세한 내용은 다음 챕터에 정리할 예정이지만     
미리 맛보고 싶다면 [heejeong Kwon 님의 블로그를 통해 확인하자](https://gmlwjd9405.github.io/2019/08/07/what-is-flush.html)   
  
# 비즈니스 로직     
> 업무에 필요한 데이터 처리를 수행하는 응용프로그램의 일부     
> 데이터 입력, 수정, 조회 및 보고서 처리 등을 수행하는 루틴     
> 좀더 엄밀히 말하면 보이는 것의 그 뒤에서 일어나는 각종 처리를 의미한다.    

JPA에서의 비즈니스 로직은 `EntityManager`를 통해 데이터를 처리하는 과정을 말한다.   
이 과정에서 CRUD 작업을 할 수 있는데 아래에서 맛 보기로 설명해보겠다.   


## 등록 
```java
em.persist(object);
```
엔티티 매니저의 `persist()` 메서드에 저장할 엔티티(인스턴스)를 넘겨주면 된다.     
JPA에서는 엔티티 매핑 정보를 분석해 SQL을 만들어서 DB에 전달한다.   

## 조회
```java
Something something = em.find(Something.class, id);
```
엔티티 매니저의 `find()` 메서드에 아래와 같은 인자값을 넣어주면 된다.   
    
1. 클래스 정보 `.class`    
2. `@Id`로 매핑된 컬럼에서 조회할 수 있는 값
    * 예를 들어 1번 데이터일 경우, `1L`      

JPA에서는 인자 값을 매핑하여 SQL을 만든 후 DB에 전달하고     
이후 조회 된 데이터를 엔티티의 형식에 알맞게 반환한다.       
   
## 수정
```java
Something something = em.find(Something.class, id);
something.setName("ssomething");
```
엔티티 매니저를 사용하지 않고,       
엔티티 매니저로 찾은 엔티티의 값을 변경하면 된다.         
   
다음 챕터에서 기술하겠지만     
엔티티 매니저의 영속성 컨텍스트에 저장된 객체들은      
값의 변동이 일어나면 이에 알맞게 수접 SQL을 만들어 보내기 때문이다.  
    
## 삭제
```java
em.remove(object);   
```  
엔티티 매니저의 `remove()` 메서드에 삭제할 엔티티(인스턴스)를 넘겨주면 된다.        
JPA에서는 엔티티 매핑 정보를 분석해 SQL을 만들어서 DB에 전달한다.     
  
# JPQL  
JPA를 사용하면 **엔티티 객체를 중심으로 개발**하고 **데이터베이스에 대한 처리는 JPA 맡긴다.**       
하지만, 검색에 있어 이 같은 원칙을 지키기가 힘들다        
쉬운 예를 들면, 검색을 하기 위해 모든 데이터를 애플리케이션에 로드하고 검색하는 로직을 만들어야한다.     
이는 매우 비효율적인 작업이다.                 
그렇기에 JPA는 **JPQL**이라는 쿼리 언어로 `SQL`을 실행하도록 지원해준다.     
  
```java  
em.createQuery("JPQL쿼리", JPQL에 사용되는 클래스타입);
```
엔티티 매니저의 `createQuery()` 를 이용하여 JPQL을 사용할 수 있다.     
보다 자세한 내용은 후반 챕터에 기술하겠다.     
     
```java
        List<Member> result = em.createQuery("select m from Member as m", Member.class)
            .setFirstResult(5)
            .setMaxResults(0)
            .getResultList();
```

JPA는 SQL을 추상화한 `JPQL`이라는 객체지향 쿼리 언어를 제공한다.       
`JPQL`은 SQL과 거의 유사해서 `SELECT`, `FROM`, `WHERE`, `GROUP BY`, `HAVING`, `JOIN`등을 사용할 수 있다.     

**JPQL VS SQL**   
* **JPQL :**     
  **엔티티 객체를 대상**으로 쿼리한다.   
  즉, 쿼리와 필드를 대상으로 쿼리한다.   
* **SQL :**  
  데이터베이스 **테이블을 대상**으로 쿼리한다.   
    
이러한 차이점 때문인지, JPQL 쿼리를 보면    
`select m from Member as m` SQL과 다르게 실제 엔티티가 `from`의 대상이 된다.     



