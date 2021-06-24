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
* **flush() :**- 영속성 컨텍스트의 변경 내용을 DB 에 반영한다.     
         
**flush vs commit**   
* **flush** : session state 와 DB를 sync하는것  
    * `flush`는 DB에 반영, 단 트랜잭션이 commit 되지 않으면 롤백된다.  
* **commit** : flush + end of the unit of work    
    * `commit`은 DB를 영구적으로 반영시킨다는 명령어이다.    
    * 영구적으로 반영하기 때문에 이전에 `flush`를 호출한다.        





