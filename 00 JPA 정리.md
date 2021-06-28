# 엔티티 매니저 설정   
**기본 설정**  
```java
public class JpaBasicApplication {

	public static void main(String[] args) {
		EntityManagerFactory entityManagerFactory = Persistence.createEntityManagerFactory("hello");
		EntityManager entityManager = entityManagerFactory.createEntityManager();
		EntityTransaction entityTransaction = entityManager.getTransaction();

		try {
			entityTransaction.begin();
			businessLogic(entityManager);
			entityTransaction.commit();
		} catch (Exception e) {
			entityTransaction.rollback();
		} finally {
			entityManager.close();
		}
		entityManagerFactory.close();
	}

	private static void businessLogic(EntityManager entityManager) {
		Order order = new Order();
		entityManager.persist(order);
	}

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
* **Create :** `영속성 컨텍스트`에 저장    
  ```java
  em.persist(object);
  ```
* **Read :** `영속성 컨텍스트`에 조회를 하고 없으면 `DB`에서 조회            
  ```java
  Something something = em.find(Something.class, id);
  ```  
* **Update :** `더티 체크`를 이용하여 `DB`의 값을 수정하도록 한다.   
  ```java
  Something something = em.find(Something.class, id);
  something.setName("ssomething");
  ```     
  최근에는, `더티 체크`가 결국 `불변`을 지키지 못한다는 의견이 있어     
  `도메인`과 `엔티티`를 분리하여 `불변한 상태를 만들자`는 이야기도 나오고 있다.      
  즉, `엔티티`를 불변하도록 만들고 이에 따라 `도메인도 불변`하도록 만드는 것이다.          
* **Delete :** `영속성 컨텍스트`에서 제거 및 `DB`에서도 제거      
  ```java
  em.remove(object);
  ```
  
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
  
**장점**    
* 1차 캐시
* 동일성 보장
* 쓰기 지연(트랜잭션을 지원)
* 변경 감지
* 지연 로딩

## 엔티티의 생명주기
**엔티티 4가지 상태**

* **비영속 :** 엔티티가 영속성 컨텍스트와 전혀 관계가 없는 상태  
  ```java
  Entity entity = new Entitiy();
  ```
* **영속 :** 엔티티가 영속성 컨텍스트에 의해 관리되는 엔티티 상태  
  ```java
  em.persist(entity);
  ```
* **준영속 :** 엔티티가 영속성 컨텍스트에 저장되었다가 분리된 상태
  ```java
  em.detach(entity);
  ```
* **삭제 :** 엔티티가 영속성 컨텍스트에서 삭제된 상태
  ```java
  em.remove(entity);
  ```
* **병합 :** 준영속 상태의 엔티티를 다시 영속 상태로 변경한다.   
  ```java
  Member member = em.find(Member.class, 200L);
  em.detach(member);   
  Member mergeMember = em.merge(member);   
  ```

# 엔티티 매핑 
* 객체와 테이블 : `@Entity`, `@Table`   
* 기본 키 매핑 : `@Id`, `@GeneratedValue(strategy = GenerationType.AUTO)`
* 필드와 컬럼 매핑 : `@Column`
* 연관관계 주인 엔티티 : `@ManyToOne`, `@JoinColumn`
* 연관관계 엔티티 : `@OneToMany`, `mappedBy=`

## 연관 관계의 주인     
객체의 양방향 연관관계 매핑시, 두 연관관계 중 하나를 연관관계의 주인으로 정해야한다.         
왜냐하면, **연관관계의 주인만이 DB 연관관계와 매핑할 수 있고 외래키를 관리할 수 있다.(CRUD)**     
즉, `연관 관계의 주인만이 DB 값을 조작할 수 있는 것`이다.  
반대로, **주인이 아닌 쪽은 읽기만 할 수 있다.**        
그래도, 주인이랑 주인이 아닌 객체 모두 값을 갱신하는 로직을 넣어주자     
        
참고로 연관 관계 참조 변수에 `null`을 넣으면 연관관계가 제거된다.  
   
**주인 객체 Member**      
```java
public class Member {
    ... // 생략 

    @ManyToOne(fetch=FetchType.LAZY)    
    @JoinColumn(name="TEAM_ID")        
    private Team team;    
    
    ... // 생략  
}
```
* **FK_ID 이름 설정 기준 :** `필드명`+ `_` + `참조하는 테이블의 기본키 컬럼명`   
     
**관계 객체 - Team**
```java
public class Team {
    ... // 생략 
     
    @OneToMany(mappedBy = "team")    
    private List<Member> members;     
    
    ... // 생략   
}
```
* **mappedBy 설정 기준 :** 매핑된 필드의 `변수 이름`   



