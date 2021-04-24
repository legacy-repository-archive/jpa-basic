# JPA 시작
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

  
## 엔티티 매니저 설정   
### 엔티티 매니저 팩토리 
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

### 엔티티 매니저
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

## 트랜잭션 관리  


## 비즈니스 로직 
   


  

 

