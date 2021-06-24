# 엔티티 매니저 설정   
**기본 설정**  
```java
        EntityManagerFactory emf = Persistence.createEntityManagerFactory("hello");
        EntityManager em = emf.createEntityManager();
        EntityTransaction tx = em.getTransaction();
        
        try {
            tx.begin();
              
            // 로직  
              
            tx.commit();
        } catch (Exception e) {
            tx.rollback();
        } finally {
            em.close();
        }
        emf.close();
    }
```
**EntityTransaction tx**       
* **begin() :** 트랜잭션을 시작한다.   
* **commit() :** 데이터베이스에 반영한다.    
* **rollback() :** 이전 상태로 돌린다.     
  
**EntityManager em**     
* **flush() :** 영속성 컨텍스트의 변경 내용을 DB 에 반영한다.     
         
**flush vs commit**   
* **flush** : session state 와 DB를 sync하는것  
  `flush`는 DB에 반영, 단 트랜잭션이 commit 되지 않으면 롤백된다.  
* **commit** : flush + end of the unit of work    
  `commit`은 DB를 영구적으로 반영시킨다는 명령어이다.    
  영구적으로 반영하기 때문에 이전에 `flush`를 호출한다.        

## CRUD
### Create
```java
em.persist(object);
```
`영속성 컨텍스트`에 저장    
      
### Read
```java
Something something = em.find(Something.class, id); // 반환 타입, ID와 
```
`영속성 컨텍스트`에 조회를 하고 없으면 `DB`에서 조회      
      
### Update   
```java
Something something = em.find(Something.class, id);
something.setName("ssomething");
```     
`더티 체크`를 이용하여 `DB`의 값을 수정하도록 한다.      
하지만, 이 조차 `불변`을 지키지 않게 만드므로 `도메인`과 `엔티티`를 분리하여             
`불변한 상태를 만들자`는 이야기도 나오고 있다.         
     
### Delete   
```java   
em.remove(object);  
```
`영속성 컨텍스트`에서 제거하는 것 뿐만 아니라 `DELTE SQL`을 만들어서 `DB`에 전달한다.      

# 영속성 컨텍스트    
* **엔티티를 영구 저장하는 환경**을 의미한다.(필자는 엔티티 컨테이너라 생각한다.)           
* 일종의 `캐시`와 같은 역할을 수행하며, 내부적으로 엔티티를 보관하고 있다.              
* 논리적인 개념으로 `EntityManager`를 생성할 때 하나 만들어진다.        
* `EntityManger`를 이용하면 `영속성 컨텍스트`에 엔티티를 저장하거나 조회할 수 있다.       
* **커밋할 때 모아둔 쿼리를 한번에 데이터베이스에 보내는 `쓰기 지연 방식`을 채택했다.**     

```java
em.persist(member);    
```    
사실, `EntityManger`는 **엔티티를 영속성 컨텍스트에 보관하고 관리**만하는 역할이다.            
이후에, `commit`이 일어나면 `영속성 컨텍스트`에서 쿼리를 만들어 실제 데이터베이스에 반영을 한다.       
             
```   
하나의 엔티티 매니저가 하나의 영속성 컨텍스트에 접근하던가      
여러 엔티티 매니저가 하나의 영속성 컨텍스트에 접근하던가 합니다.    
``` 
  
## 엔티티의 생명주기
**엔티티 4가지 상태**

* 비영속 : 영속성 컨텍스트와 전혀 관계가 없는 상태
* 영속 : 영속성 컨텍스트에 저장된 상태
* 준영속 : 영속성 컨텍스트에 저장되었다가 분리된 상태
* 삭제 : 삭제된 상태

### 비영속
```java
Entity entity = new Entitiy();
```
엔티티 객체를 생성했지만 아직 저장하지 않은 상태

### 영속
```java
em.persist(entity);
```
엔티티가 영속성 컨텍스트에 저장된 상태  
**영속 상태 :** 영속성 컨텍스트에 의해 관리되는 엔티티 상태를 의미한다.    

### 준영속
```java
em.detach(entity)
```
영속 상태의 엔티티를 영속성 컨텍스트가 관리하지 않으면서 바뀌는 상태    
  
### 삭제
```java
em.remove(entity)
```
엔티티가 영속성 컨텍스트에서 삭제된 상태      
   
## 장점
* 1차 캐시
* 동일성 보장
* 트랜잭션을 지원하는 쓰기 지연
* 변경 감지
* 지연 로딩
