# 엔티티 매니저 설정   
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

# CRUD
## Create
```java
em.persist(object);
```
`영속성 컨텍스트`에 저장    
      
## Read
```java
Something something = em.find(Something.class, id); // 반환 타입, ID와 
```
`영속성 컨텍스트`에 조회를 하고 없으면 `DB`에서 조회      
      
## Update   
```java
Something something = em.find(Something.class, id);
something.setName("ssomething");
```     
`더티 체크`를 이용하여 `DB`의 값을 수정하도록 한다.      
하지만, 이 조차 `불변`을 지키지 않게 만드므로 `도메인`과 `엔티티`를 분리하여             
`불변한 상태를 만들자`는 이야기도 나오고 있다.         
     
## Delete   
```java   
em.remove(object);  
```
영속성에서 제거하는 것 뿐만 아니라 `DELTE SQL`을 만들어서 `DB`에 전달한다.    



