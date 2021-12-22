# JPA Basic

**:Contents**
* 01.JPA 소개
* 02.JPA 시작
* 03.영속성 관리
* 04.엔티티 매핑
* 05.연관관계 매핑 기초
* 06.다양한 연관관계 매핑
* 07.고급 매핑
* 08.프록시와 연관관계 관리
* 09.값 타입
* 10.객체지향 쿼리 언어[JPQL]
---

## 01. JPA 소개
jpa는 JAVA애플리케이션과 JDBC 사이에서 동작        
<br>

###JPA
- select sql 생성
- jdbc api 사용
- resultset 결과를 객체로 매핑
- 패러다임 불일치 해결 *** 

###JPA를 왜 사용해야 하는가?
- SQL 중심적인 개발에서 객체 중심으로 개발
- 생산성
- 유지보수
- 패러다임의 불일치 해결
- 성능
- 데이터 접근 추상화와 벤더 독립성
- 표준

###생산성 - JPA와 CRUD
> 저장: jpa.persist(member)   
> 조회: Member member = jpa.find(memberId)    
> 수정: member.setName(“변경할 이름”)  
> 삭제: jpa.remove(member)    

###JPA와 패러다임의 불일치 해결  
1.JPA와 상속   
2.JPA와 연관관계     
3.JPA와 객체 그래프 탐색    
4.JPA와 비교하기

###동일한 트랜잭션에서 조회한 엔티티는 같음을 보장  

###JPA의 성능 최적화 기능
1. 1차 캐시와 동일성(identity) 보장
2. 트랜잭션을 지원하는 쓰기 지연(transactional write-behind)
3. 지연 로딩(Lazy Loading)

###1차 캐시와 동일성 보장
1. 같은 트랜잭션 안에서는 같은 엔티티를 반환 - 약간의 조회 성능 향상
2. DB Isolation Level이 Read Commit이어도 애플리케이션에서 Repeatable Read 보장

###트랜잭션을 지원하는 쓰기 지연 - INSERT,update
1. 트랜잭션을 커밋할 때까지 INSERT,update SQL을 모음    
   //트랜잭션을 커밋하는 순간 데이터베이스에 SQL을 모아서 보낸다.
2. JDBC BATCH SQL 기능을 사용해서 한번에 SQL 전송

###지연 로딩과 즉시 로딩 (중요! 아래 둘중 하나를 옵션으로 선택할수있음) 
> ** 중요!! ** 애플리케이션 개발할때는 지연로딩쓰고, 실제 최적화할때만 즉시 로딩 쓰는 편  
• 지연 로딩: 객체가 실제 사용될 때 로딩    
• 즉시 로딩: JOIN SQL로 한번에 연관된 객체까지 미리 조회

-------------------------
## 02. JPA 시작

###JPA 설정하기 - persistence.xml
• JPA 설정 파일     
• /META-INF/persistence.xml 위치  
• persistence-unit name으로 이름 지정     
• javax.persistence로 시작: JPA 표준 속성      
• hibernate로 시작: 하이버네이트 전용 속성

persistence.xml   
![A](imgs/persistencexml.PNG)
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
### 데이터베이스 방언 (dialect)
• JPA는 특정 데이터베이스에 종속 X  
• 각각의 데이터베이스가 제공하는 SQL 문법과 함수는 조금씩 다름   
• 가변 문자: MySQL은 VARCHAR, Oracle은 VARCHAR2   
• 문자열을 자르는 함수: SQL 표준은 SUBSTRING(), Oracle은 SUBSTR()    
• 페이징: MySQL은 LIMIT , Oracle은 ROWNUM    
• 방언: SQL 표준을 지키지 않는 특정 데이터베이스만의 고유한 기능 
=> *** 중요!!!! *** hibernate.dialect 속성에 지정
• H2 : org.hibernate.dialect.H2Dialect  
• Oracle 10g : org.hibernate.dialect.Oracle10gDialect   
• MySQL : org.hibernate.dialect.MySQL5InnoDBDialect     
• 하이버네이트는 40가지 이상의 데이터베이스 방언 지원     

---------------------
## 03.영속성관리

###영속성 컨텍스트(Persistence Context)
- “엔티티를 영구 저장하는 환경”이라는 뜻
- EntityManager.persist(entity)
- 엔티티 매니저를 통해서 영속성 컨텍스트에 접근
- 저장 시 1차 캐시에 저장

###엔티티의 생명주기
• 비영속 (new/transient) : 영속성 컨텍스트와 전혀 관계가 없는 새로운 상태    
• 영속 (managed) : 영속성 컨텍스트에 관리되는 상태  (엔티티매니저를 가져와서 persist로 저장한 상태 = 영속)    
• 준영속 (detached) : 영속성 컨텍스트에 저장되었다가 분리된 상태      
• 삭제 (removed) : 삭제된 상태 [실제 db에서 삭제하고 싶을 때]
```java
   //객체를 생성한 상태(비영속)
   Member member = new Member();
   member.setId("member1");
   member.setUsername(“회원1”);
   EntityManager em = emf.createEntityManager();
   em.getTransaction().begin();
   
   //객체를 저장한 상태(영속)
   em.persist(member)
   
   //회원 엔티티를 영속성 컨텍스트에서 분리, 준영속 상태  => 영속성 컨텍스트에서 삭제함
   em.detach(member); 
   
   //객체를 삭제한 상태(삭제)  => 실제 db에서 삭제함
   em.remove(member)
```

###영속성 컨텍스트의 이점
• 1차 캐시  
> em.persist를 할 경우 1차 캐시에 저장됨   
> 1차 캐시에서 먼저 조회하는데 없을 경우 db 조회하여 1차 캐시에 저장 후 반환   

• 영속 엔티티의 동일성(identity) 보장 (1차 캐시가 있기 때문에 가능)   
> 1차 캐시로 반복 가능한 읽기(REPEATABLE READ) 등급의 트랜잭션 격리 수준을 데이터베이스가 아닌 애플리케이션 차원에서 제공   
```java
    Member a = em.find(Member.class, "member1");  
    Member b = em.find(Member.class, "member1");       
    System.out.println(a == b); //동일성 비교 true    
```
 > REPEATABLE READ    
   >> MySQL에서는 트랜잭션마다 트랜잭션 ID를 부여하여 트랜잭션 ID보다 작은 트랜잭션 번호에서 변경한 것만 읽게 된다.    
   Undo 공간에 백업해두고 실제 레코드 값을 변경한다.   
   백업된 데이터는 불필요하다고 판단하는 시점에 주기적으로 삭제한다.   
   Undo에 백업된 레코드가 많아지면 MySQL 서버의 처리 성능이 떨어질 수 있다.     
   이러한 변경방식은 MVCC(Multi Version Concurrency Control)라고 부른다.	      

• 트랜잭션을 지원하는 쓰기 지연(transactional write-behind)  - 버퍼링(write)    
> 트랜잭션 커밋 순간에 쓰기 지연 sql 저장소에서 flush가 되면서 db에 저장된다.   
> jdbc batch : 버퍼링(write)을 모아서 한번에 디비에 저장할 수 있는 옵션이 있음.  
> 트랜잭션 commit > 쓰기 지연 sql 저장소 > flush > db에 커밋 

• 변경 감지(Dirty Checking)  
> 커밋하면 flush 호출 > 엔티티와 스냅샷(최초로 값을 읽어온 것들을 스냅샷에 저장해둠)을 비교 > 쓰기지연 SQL 저장소에 업데이트 sql 생성 > flush > commit

• 지연 로딩(Lazy Loading)

#### ** jpa는 기본으로 내부적으로 리플렉션을 사용하는데 동적으로 객체를 생성해야하기 때문에 @Entity로 매핑한 클래스에 기본생성자가 있어야함.

###플러시(flush)
- 영속성 컨텍스트의 변경내용을 데이터베이스에 반영   
- db에 커밋될 때 플러시가 일어남 [플러시가 발생한다고 db에 커밋된 것은 아님]   
- 영속성 컨텍스트를 비우지 않음  
   [flush 해도 영속성 컨텍스트의 1차 캐시는 지워지지 않음. 쓰기 지연 sql 저장소에 업데이트 되는것들이 반영되는 것이라고 생각하면 됨]    
- 영속성 컨텍스트의 변경내용을 데이터베이스에 동기화  
- 트랜잭션이라는 작업 단위가 중요 -> 커밋 직전에만 동기화 하면 됨    
- flush는 트랜잭션 커밋되거나 qury가 날라갈때 flush됨      

###플러시 발생
1. 변경 감지 (Dirty Checking)가 일어남
2. 수정된 엔티티를 쓰기 지연 SQL 저장소에 등록
3. 쓰기 지연 SQL 저장소의 쿼리를 데이터베이스에 전송(등록, 수정, 삭제 쿼리)

###영속성 컨텍스트를 플러시하는 방법
• em.flush() - 직접 호출 			=> 거의 안씀     
• 트랜잭션 커밋 - 플러시 자동 호출      
• JPQL 쿼리 실행 - 플러시 자동 호출      

###플러시 모드 옵션
> **중요!!** 가급적 실무에서 AUTO 사용

• FlushModeType.AUTO : 커밋이나 쿼리를 실행할 때 플러시 (기본값)  
• FlushModeType.COMMIT : 커밋할 때만 플러시

>> em.flush() > em.clear() 후 persist()할 경우, 영속성 컨텍스트를 모두 비우고 1차 캐시도 삭제 후 persist()함 (실무에서 사용할 일은 거의 없으나, 로컬에서 쿼리 디버깅 등 사용에 용이)

###준영속 상태   
• 영속(em.persist 했을 때 or em.find했을 때 1차 캐시가 없을 경우 1차 캐시에 저장된 상태) -> 준영속  
• 영속 상태의 엔티티가 영속성 컨텍스트에서 분리(detached)된 것 [영속 상태였다가 빠지는 경우를 준영속 상태라 함]    
• 영속성 컨텍스트가 제공하는 기능을 사용 못함    

###준영속 상태로 만드는 방법
• em.detach(entity) : 특정 엔티티만 준영속 상태로 전환  
• em.clear() : 영속성 컨텍스트를 완전히 초기화    
• em.close() : 영속성 컨텍스트를 종료      

--------------

##04.엔티티매핑

###엔티티 매핑 소개   
• 객체와 테이블 매핑: @Entity, @Table    
• 필드와 컬럼 매핑: @Column    
• 기본 키 매핑: @Id    
• 연관관계 매핑: @ManyToOne,@JoinColumn      

###@Entity     
• @Entity가 붙은 클래스는 JPA가 관리, 엔티티라 한다.      
• JPA를 사용해서 테이블과 매핑할 클래스는 @Entity 필수      
> 주의!!!  
=> 기본 생성자 필수(파라미터가 없는 public 또는 protected 생성자) !!!    
=> final 클래스, enum, interface, inner 클래스 사용X    
=> 저장할 필드에 final 사용 X

###@Entity 속성 정리  
• 속성: name  
• JPA에서 사용할 엔티티 이름을 지정한다.  
• 기본값: 클래스 이름을 그대로 사용(예: Member)    
• ** 같은 클래스 이름이 없으면 가급적 기본값을 사용한다!!  

###@Table
• @Table은 엔티티와 매핑할 테이블 지정 : @Table(name="member")

###데이터베이스 스키마 자동 생성
=> 속성 : hibernate.hbm2ddl.auto   
=> 생성된 DDL은 개발 장비에서만 사용.[운영 장비에는 절대 옵션인 create, create-drop, update 사용하면 안된다]    
=> 옵션 중 validate : 엔티티와 테이블이 정상 매핑되었는지만 확인      
=> • 개발 초기 단계는 create 또는 update     
   • 같이 쓰는 테스트 서버 / 스테이징과 운영 서버는 validate만..    
=> DDL 생성 기능은 DDL을 자동 생성할 때만 사용되고 JPA의 실행 로직에는 영향을 주지 않는다      

###@Column : 컬럼 매핑   
-name 필드와 매핑할 테이블의 컬럼 이름   
-@Column의 unique 속성은 잘 안씀. 이름이 이상하게 생성되서 실무에 별로... 대신 @Table의 uniqueConstraints 속성으로 이름까지 직접 지정하도록 사용한다. 

###@Temporal : 날짜 타입 매핑 
> 최신버전일 경우 이 어노테이션 없이 localdateTime, localdate로 사용

###@Enumerated : 자바 enum 타입 매핑
> *** 중요! *** 주의! : ORDINAL 사용X, 무조건 STRING으로 사용   
> • EnumType.ORDINAL: enum 순서를 데이터베이스에 저장 => 중간에 추가되거나 했을때 운영상 큰 문제  
  • EnumType.STRING: enum 이름을 데이터베이스에 저장

###@Lob : BLOB, CLOB 매핑 (지정 속성 없음)
###@Transient 특정 필드를 컬럼에 매핑하지 않음(매핑 무시)
- 필드 매핑X / 데이터베이스에 저장X, 조회X /주로 메모리상에서만 임시로 어떤 값을 보관하고 싶을 때 사용

### * id..seq 값은 무조건 Long 권장

###기본 키 매핑 방법
• 직접 할당: @Id만 사용  
• 자동 생성(@GeneratedValue )

=> DB에 맞게 아래 전략 중 선택해서 사용하면 됨

###IDENTITY 전략 - 특징			=>모았다가(버퍼링) 커밋하는게 불가
• 기본 키 생성을 데이터베이스에 위임   
• 주로 MySQL, PostgreSQL, SQL Server, DB2에서 사용(예: MySQL의 AUTO_ INCREMENT)    
• JPA는 보통 트랜잭션 커밋 시점에 INSERT SQL 실행    
• AUTO_ INCREMENT는 데이터베이스에 INSERT SQL을 실행한 이후에 ID 값을 알 수 있음    
• IDENTITY 전략은 em.persist() 시점에 즉시 INSERT SQL 실행하고 DB에서 식별자를 조회      

###SEQUENCE 전략 : @SequenceGenerator =>모았다가(버퍼링) 커밋하는게 가능    
• 데이터베이스 시퀀스는 유일한 값을 순서대로 생성하는 특별한 데이터베이스 오브젝트(예: 오라클 시퀀스)     
• 오라클, PostgreSQL, DB2, H2 데이터베이스에서 사용    
 >> allocationSize 속성 : 시퀀스 한 번 호출에 증가하는 수   
   (성능 최적화에 사용됨. 데이터베이스 시퀀스 값이 하나씩 증가하도록 설정되어 있으면 이 값을 반드시 1로 설정해야 한다).    
   기본값 50 / 50~100정도가 적당한 것 같다      

###TABLE 전략 => 그닥 권장하지 않음  
• 키 생성 전용 테이블을 하나 만들어서 데이터베이스 시퀀스를 흉내내는 전략      
• 장점: 모든 데이터베이스에 적용 가능  
      (어떤 db는 auto_increment가 있고 어떤 db는 seq가 있기 때문에...그걸 커버하기 위해 만들어진 전략)    
• 단점: 성능    
 >> allocationSize 속성 : 시퀀스 한 번 호출에 증가하는 수(성능 최적화에 사용됨), 기본값 50    

###권장하는 식별자 전략
• 기본 키 제약 조건: null 아님, 유일, 변하면 안된다.    
• 미래까지 이 조건을 만족하는 자연키는 찾기 어렵다. 대리키(대체키)를 사용하자.     
• 예를 들어 주민등록번호도 기본 키로 적절하기 않다.      
• 권장: Long형 + 대체키 + 키 생성전략 사용    

###데이터 중심 설계의 문제점    
• 데이터 중심 설계 : 객체 설계를 테이블 설계에 맞춘 방식     
• 테이블의 외래키를 객체에 그대로 가져옴    
• 객체 그래프 탐색이 불가능     
• 참조가 없으므로 UML도 잘못됨     

#### *인덱스나 길이와 같은 정보도 엔티티에 같이 주면, 디비에서 확인할 필요없으니 편리함     
#### *부트는 기본적으로 자바의 카멜케이스(orderDate)를 > 소문자 언더스코어(order_date)로 매핑해줌     

---------------
##05.연관관계 매핑

###객체를 테이블에 맞추어 데이터 중심으로 모델링(참조 대신에 외래 키를 그대로 사용)하면, 협력 관계를 만들 수 없다.
>• 테이블은 외래 키로 조인을 사용해서 연관된 테이블을 찾는다.   
>• 객체는 참조를 사용해서 연관된 객체를 찾는다.  
>• 테이블과 객체 사이에는 이런 큰 간격이 있다.  

###단방향 연관관계 - 객체 지향 모델링 : (객체의 참조와 테이블의 외래 키를 매핑)

###양방향 연관관계와 연관관계의 주인 : 양방향 매핑
=> 양방향보다 단방향 연관관계 설계가 더 좋음..

###연관관계의 주인과 mappedBy   
• mappedBy = JPA의 멘탈붕괴 난이도    
• mappedBy는 처음에는 이해하기 어렵다.    
• 객체와 테이블간에 연관관계를 맺는 차이를 이해해야 한다    

###객체와 테이블이 관계를 맺는 차이   
- 객체 연관관계 = 2개    
  • 회원 -> 팀 연관관계 1개(단방향)   
  • 팀 -> 회원 연관관계 1개(단방향)   
- 테이블 연관관계 = 1개   
  • 회원 <-> 팀의 연관관계 1개(양방향) 

###객체의 양방향 관계
• 객체의 양방향 관계는 사실 양방향 관계가 아니라 서로 다른 단뱡향 관계 2개다.  
• 객체를 양방향으로 참조하려면 단방향 연관관계를 2개 만들어야 한다.      

###테이블의 양방향 연관관계
• 테이블은 외래 키 하나로 두 테이블의 연관관계를 관리  
• MEMBER.TEAM_ID 외래 키 하나로 양방향 연관관계 가짐(양쪽으로 조인할 수 있다.)    
=> 객체의 양방향 관계는 결국...단방향2개 이고, 객체 둘 중 하나로 외래 키를 관리해야 한다      

###연관관계의 주인(Owner) (= 양방향 매핑 규칙)    **** 중요!!! ****  
• 객체의 두 관계중 하나를 연관관계의 주인으로 지정    
• 연관관계의 주인만이 외래 키를 관리(등록, 수정)    
• 주인이 아닌쪽(mappedBy가 있는 쪽)은 읽기만 가능      
• *** 주인(진짜매핑)은 mappedBy 속성 사용X     
• *** 주인이 아니면(가짜매핑) mappedBy 속성으로 주인 지정      

###누구를 주인으로?
> ****중요!! ***    
> 무조건 외래 키(FK)가 있는 곳[N 쪽]을 주인으로 정해라!!!! [연관관계 편의 메소드를 사용할 경우 한쪽에만 사용]

###양방향 연관관계 주의 
• 순수 객체 상태를 고려해서 항상 양쪽에 값을 설정하자 /양방향 매핑시 연관관계의 주인에 값을 양쪽 다 입력해야 한다.  
• *** 중요!! *** "연관관계 편의 메소드를 생성하자" [연관관계 편의 메소드를 사용할 경우 한쪽에만 사용]    
• 양방향 매핑시에 무한 루프를 조심하자  
 > 예: toString() / lombok [롬복에서 tostring 만드는 부분 사용x]  
 > JSON 생성 라이브러리(엔티티를 직접 컨트롤러의 리스펀스로 반환할 경우 문제가 많이 생김[컨트롤러에서 절대 엔티티 반환x]. 즉 요청응답dto와 엔티티 분리하여 사용)   

###양방향 매핑 정리
• *** 중요!! *** 단방향 매핑만으로도 이미 연관관계 매핑은 완료
   > 설계할때 양방향 매핑하지 말고 단방향으로 매핑해야됨   

• ** 단방향 매핑을 잘 하고 양방향은 필요할 때 추가해도 됨(테이블에 영향을 주지 않음)  
   > 단방향으로 설계 하고 실제 애플리케이션 개발할 때 양방향이 필요할 경우 고민

• 양방향 매핑은 반대 방향으로 조회(객체 그래프 탐색) 기능이 추가된 것 뿐
• JPQL에서 역방향으로 탐색할 일이 많음

###연관관계의 주인을 정하는 기준     
• 비즈니스 로직을 기준으로 연관관계의 주인을 선택하면 안됨!  
• **** 연관관계의 주인은 외래 키의 위치를 기준으로 정해야함      

####* 테이블 설계 > 객체 설계 [참조를 사용하도록 설계]

####* 관례상 new로 초기화(널포인터 방지 등)
>@OneToMany(mappedBy = "member")  
List<Order> orders = new ArrayList<>()


------------------------------------

##06.다양한 연관관계 매핑 (***중요!!***)

###연관관계 매핑시 고려사항 3가지
• 다중성    
• 단방향, 양방향     
• 연관관계의 주인     

###다중성   
• 다대일: @ManyToOne    
• 일대다: @OneToMany    
• 일대일: @OneToOne     
• 다대다: @ManyToMany   => 사용 x     
>  @ManyToMany 는 편리한 것 같지만, 중간 테이블( CATEGORY_ITEM )에 컬럼을 추가할 수 없고, 세밀하게 쿼리를 실행하기 어렵기 때문에 실무에서 사용하기에는 한계가 있다.   
중간 엔티티( CategoryItem ) 를 만들고 @ManyToOne , @OneToMany 로 매핑해서 사용하자. 정리하면 대다대 매핑을 일대다, 다대일 매핑으로 풀어내서 사용하자.

###단방향, 양방향
[테이블]    
• 외래 키 하나로 양쪽 조인 가능  
• 사실 방향이라는 개념이 없음    
[객체]  
• 참조용 필드가 있는 쪽으로만 참조 가능    
• 한쪽만 참조하면 단방향    
• 양쪽이 서로 참조하면 양방향 [=단방향이 2개]     

###연관관계의 주인    
• 테이블은 외래 키 하나로 두 테이블이 연관관계를 맺음     
• 객체 양방향 관계는 A->B, B->A 처럼 참조가 2군데     
• 객체 양방향 관계는 참조가 2군데 있음. 둘중 테이블의 외래 키를 관리할 곳을 지정해야함      
• **** 연관관계의 주인: 외래 키를 관리하는 참조      
• **** 주인의 반대편: 외래 키에 영향을 주지 않음, 단순 조회만 가능 ,수정에 영향을 주지 않음, mappedBy 사용     

###다대일 단방향 정리     
• 가장 많이 사용하는 연관관계    

###일대다 단방향 정리   => *** 다대일 양방향 사용 권장      
• 일대다 단방향은 일대다(1:N)에서 일(1)이 연관관계의 주인... 운영이 힘들기 때문에 사용 X    
• 테이블 일대다 관계는 항상 다(N) 쪽에 외래 키가 있음   
• 객체와 테이블의 차이 때문에 반대편 테이블의 외래 키를 관리하는 특이한 구조    
• @JoinColumn을 꼭 사용해야 함. 그렇지 않으면 조인 테이블 방식을 사용함(중간에 테이블이 하나 추가됨)     
• 단점 : 엔티티가 관리하는 외래 키가 다른 테이블에 있음 / 연관관계 관리를 위해 추가로 UPDATE SQL 실행       

###일대다 양방향 정리  
• 이런 매핑은 공식적으로 존재X      
• @ManyToOne      
@JoinColumn(insertable=false, updatable=false) 사용     
• 읽기 전용 필드를 사용해서 양방향 처럼 사용하는 방법  
• *** 다대일 양방향 사용 권장     

###일대일 관계   
• 일대일 관계는 그 반대도 일대일     
• db입장에서는, 외래 키에 데이터베이스 유니크(UNI) 제약조건 추가된 경우가 일대일 관계     
• 주 테이블이나 대상 테이블 중에 외래 키 선택 가능      
1. 일대일: 주 테이블에 외래 키 양방향  => ***권장.  
   >@OneToOne , 다대일처럼 동일하게 외래키가 있는곳이 연관관계 주인, 주인이 아닌곳에는 mappedBy 적용    
2. 일대일: 대상 테이블에 외래 키 단방향 
   > jpa에서 지원 x [db 설계를 변경하던가 양방향 관계는 지원하기때문에 1번 방법을 사용]

###일대일 정리
<주 테이블에 외래 키>     
• 주 객체가 대상 객체의 참조를 가지는 것 처럼 주 테이블에 외래 키를 두고 대상 테이블을 찾음   
• 객체지향 개발자 선호  
• JPA 매핑 편리    
• 장점: 주 테이블만 조회해도 대상 테이블에 데이터가 있는지 확인 가능     
• 단점: 값이 없으면 외래 키에 null 허용    

<대상 테이블에 외래 키>    
• 대상 테이블에 외래 키가 존재      
• 전통적인 데이터베이스 개발자 선호       
• 장점: 주 테이블과 대상 테이블을 일대일에서 일대다 관계로 변경할 때 테이블 구조 유지    
• 단점: 프록시 기능의 한계로 지연 로딩으로 설정해도 항상 즉시 로딩됨(프록시는 뒤에서 설명)    

###다대다 (=@ManyToMany)
•실무에서는 @ManyToMany 사용X [why? 제약: 필드 추가X, 엔티티 테이블 불일치]

###다대다 한계 극복
•연결 테이블용 엔티티 추가(연결 테이블을 엔티티로 승격)    
•@ManyToMany -> @OneToMany, @ManyToOne로 변경하여 사용하고 중간 연결 엔티티는 아무 의미없는 id 값을 사용하기를 권고.(두개의 pk를 묶어서 id를 만드는 것보다 추후 유연성에 좋음)      
•테이블의 N:M 관계는 중간 테이블을 이용해서 1:N, N:1로 구현 : 실전에서는 중간 테이블이 단순하지 않다.     

###@JoinColumn
•외래 키를 매핑할 때 사용   
> 속성   
 >> name : 매핑할 외래 키 이름 (기본값 : 필드명 + _ + 참조하는 테 이블의 기본 키 컬럼명)    
 >> referencedColumnName : 외래 키가 참조하는 대상 테이블의 컬럼명 (기본값:참조하는 테이블의 기본 키 컬럼명)     

###@ManyToOne
•다대일 관계 매핑 (속성 중에 mappedBy가 없다. 즉 ManyToOne은 연관관계의 주인이 되어야한다는 뜻)
>속성
>> optional : false로 설정하면 연관된 엔티티가 항상 있어야 한다.	(기본값 : TRUE)
>> fetch : 글로벌 페치 전략을 설정한다. (기본값 : @ManyToOne=FetchType.EAGER - @OneToMany=FetchType.LAZY)
>> cascade : 영속성 전이 기능을 사용한다.

###@OneToMany
•다대일 관계 매핑
>속성
>> mappedBy : 연관관계의 주인 필드를 선택한다.
>> fetch : 글로벌 페치 전략을 설정한다. (기본값 : @ManyToOne=FetchType.EAGER - @OneToMany=FetchType.LAZY)
>> cascade : 영속성 전이 기능을 사용한다.

-----------------------------

##07.고급 매핑(상속관계 매핑)

###상속관계 매핑  
•관계형 데이터베이스는 상속 관계X  
•슈퍼타입 서브타입 관계라는 모델링 기법이 객체 상속과 유사   
•상속관계 매핑: 객체의 상속과 구조와 DB의 슈퍼타입 서브타입 관계를 매핑   
•슈퍼타입 서브타입 논리 모델을 실제 물리 모델로 구현하는 방법 (pdf 그림 참고)    
- 각각테이블로변환->조인전략     
- 통합 테이블로 변환 -> 단일 테이블 전략. [jpa 기본 전략]    
- 서브타입 테이블로 변환 -> 구현 클래스마다 테이블 전략      
  •주요 어노테이션    
- @Inheritance(strategy=InheritanceType.XXX)    
  => JOINED: 조인 전략      
  => SINGLE_TABLE: 단일 테이블 전략  
  => TABLE_PER_CLASS: 구현 클래스마다 테이블 전략     
- @DiscriminatorColumn(name="DTYPE") : 부모클래스에 선언. 기본이 DTYPE. name 변경가능. 컬럼명에 해당하는 테이블 컬럼에 입력되는 엔티티명이 들어감 (사용권장)      
- @DiscriminatorValue(“XXX”) : 자식클래스에서 선언. 위 @DiscriminatorColumn(name=“컬럼명”)의 테이블 컬럼에 입력되는 엔티티명 대신 선언해둔 @DiscriminatorValue(“XXX”)가 들어감        
  => * 싱글 테이블 전략에서는 @Discriminator 어노테이션이 반드시 들어가야함   
  •jpa가 상속관계일때 알아서 select 시 조인해서 가져오고 insert도 상속관계 함께 insert.  

### 조인 전략 [**기본적으로는 조인 전략이 가장 잘 맞다. *권장]
• 장점  
> 테이블 정규화   
> 외래 키 참조 무결성 제약조건 활용가능 (ex : 상위 클래스의 pk만으로 하위 클래스를 조회 할 수 있음.)      
> 저장공간 효율화  

• 단점
> 조회시 조인을 많이 사용,성능 저하 => 오히려 잘 사용하면 성능 저하 되지 않을 수 있음.    
> 조회 쿼리가 복잡함      
> 데이터 저장시 INSERT SQL 2번 호출    

### 단일 테이블 전략  
• 장점  
> 조인이 필요 없으므로 일반적으로 조회 성능이 빠름    
> 조회 쿼리가 단순함      

• 단점
> 자식 엔티티가 매핑한 컬럼은 모두 null 허용 (치명적)  
> 단일테이블에 모든것을 저장하므로 테이블이 커질 수 있다. 상황에 따라서 조회 성능이 오히려 느려질 수 있다.    

### 구현 클래스마다 테이블 전략 => 절대 사용 x
• 이 전략은 데이터베이스 설계자와 ORM 전문가 둘 다 추천X
• 장점
> 서브 타입을 명확하게 구분해서 처리할 때 효과적  
> not null 제약조건 사용 가능

• 단점
> 여러 자식 테이블을 함께 조회할 때 성능이 느림(UNION SQL 필요)   
> 자식 테이블을 통합해서 쿼리하기 어려움    

### *** 기본적으로 조인전략을 사용하고, 정말 단순하고 데이터가 얼마 안되고 확장가능성없을 경우 단일 테이블 전략을 사용.
### *** 비즈니스적으로 복잡하고 중요하면 대부분 조인 전략 사용. 나중에 억단위 데이터가 늘어날 경우에는.. 조인전략도 정답이 아닐 수 있음. 고민이 필요함

###@MappedSuperclass
• 상속관계 매핑이랑 별로 관계가 없음. 속성을 같이 여러군데에서 사용하고 싶을때 씀.   
• 전체 엔티티에서 공통 매핑 정보가 필요할 때 클래스에 @MappedSuperclass 어노테이션 사용(ex - id, name, 등록자, 등록일, 수정자, 수정일)하고 extends 하여 사용  
• 엔티티X, 테이블과 매핑X (baseEntity라는 테이블이 생성되지 않음. @Entity 어노테이션을 사용하지 않는다)   
• 부모 클래스를 상속 받는 자식 클래스에 매핑 정보만 제공 조회, 검색 불가(em.find(BaseEntity) 불가)  
• 직접 생성해서 사용할 일이 없으므로 추상 클래스 권장  
cf) 등록자, 등록일, 수정자, 수정일 이런 정보는 jpa가 제공하는 이벤트로 공통으로 관리할 수 있음. 

### * JPA에서 extends를 사용할 수 있는 경우 
>상위 클래스가 @Entity가 적용되어 있는 클래스(상속관계)이거나,   
>@MappedSuperclass(공통속성매핑)s로 지정한 클래스만 상속 가능.

---------------------------------------------------------------

## 08.프록시와 연관관계 관리

###em.find() vs em.getReference()
•em.find() : 데이터베이스를 통해서 실제 엔티티 객체 조회  
•em.getReference() : 데이터베이스 조회를 미루는 가짜(프록시) 엔티티 객체 조회    
[= db에 쿼리가 나가지 않는데 데이터를 조회하는 것]  
=> getReference() 시에는 db조회 하지 않고, getReference 후 값이 실제 사용되는 시점에 db에 쿼리를 날림 

###프록시 객체의 초기화
Member member = em.getReference(Member.class, “id1”);    
member.getName();    
> getName() 시 Member Target 가짜 객체에 값이 없으면,      
> 영속성 컨텍스트에 초기화 요청 > DB 조회 > 실제 엔티티 생성, 가짜객체인 Target과 매핑 시킴

###프록시 특징      
• 실제 클래스를 상속 받아서 만들어짐      
• 실제 클래스와 겉 모양이 같다.     
• 사용하는 입장에서는 진짜 객체인지 프록시 객체인지 구분하지 않고 사용하면 됨(이론상)     
• 프록시 객체는 실제 객체의 참조(target)를 보관     
• 프록시 객체를 호출하면 프록시 객체는 실제 객체의 메소드 호출      

• 프록시 객체는 처음 사용할때 한번만 초기화     
• 프록시 객체를 초기화 할 때, 프록시 객체가 실제 엔티티로 바뀌는 것은 아님, 초기화되면 프록시 객체를 통해서 실제 엔티티에 접근 가능    
• 영속성 컨텍스트에 찾는 엔티티가 이미 있으면 em.getReference()를 호출해도 실제 엔티티 반환
• 영속성 컨텍스트의 도움을 받을 수 없는 준영속 상태일 때, 프록시를 초기화하면 문제 발생      
(하이버네이트는 org.hibernate.LazyInitializationException 예외를 터트림) => ** could not initialize proxy : 영속성 컨텍스트에서 관리가 안되는구나라고 생각하면 됨 [트랜잭션 끝날 때 영속성 컨텍스트도 끝나도록 관리하지 않을 경우 발생한다!!!]    
> *** 중요!! *** 프록시 객체는 원본 엔티티를 상속받음, 따라서 타입 체크시 주의해야함 (== 비교 실패 / instance of 사용) - 프록시가 넘어올지, 실제 엔티티가 넘어올지 알 수 없기 때문

cf) jpa에서는 m1이던, 프록시던 상관 없이 동일한 트랜잭션 안에서 조회하는 pk가 같을 경우 동일한 타입 보장을 해준다. 즉 아래 true 반환이 보장된다   
```java
   Member m1 = em.find(Member.class, member1.getID());   	    			//getClass Member
   Member refrence = em.getReference(Member.class, member1.getID());		//getClass Member
   System.out.println("a == a : " + (m1 == refrence));   					// a == a : true

   Member m1 = em.getReference(Member.class, member1.getID());   			//getClass hibernateProxy
   Member refrence = em.getReference(Member.class, member1.getID());		//getClass hibernateProxy
   System.out.println("a == a : " + (m1 == refrence));  		   		    // a == a : true
```

###프록시 강제 초기화
• hibernate : org.hibernate.Hibernate.initialize(entity);      
• 참고: JPA 표준은 강제 초기화 없음 > 강제 호출로 사용: member.getName()

###지연로딩(LAZY)과 즉시로딩(EAGER) 
•즉시로딩(EAGER)   => 실무 사용 x
> 엔티티 조회 시 연관관계에 있는 데이터까지 한번에 조회해오는 기능    
> 즉시 로딩으로 조회된 엔티티의 연관관계 필드에는 실제 엔티티 객체가 반환된다.      
> ex) Member 클래스 안의 team에 즉시로딩을 하면, em.find() 시 바로 테이블을 읽어들이는데 연관된 테이블을 조인해서 같이 가져옴 => 실무에서 사용 X      

•지연로딩(LAZY) 
> 엔티티 조회 시점이 아닌, 엔티티 내 연관관계를 참조할 때 해당 연관관계에 대한 SQL이 질의      
> 엔티티 조회 시, 연관관계 필드는 프록시 객체로 제공된다.     
> ex) team에 지연로딩을 하면, 실제 team를 사용하는 시!점!(team.getName())에 db를 조회하는데 team 테이블만 읽음
```java 
  Member findMember = em.find(Member.class, 1L);
  member.getTeam(); 								//프록시 객체 초기화 X
  member.getTeam().getClass();			//프록시 객체
  member.getTeam().getName();  		//프록시 객체 초기화 및 SQL 질의

  //=> 위와 같이 지연로딩 되는 연관관계를 참조하기 전까지는 프록시 객체가 초기화 되지 않고,
  //   프록시 객체를 참조할 때, 프록시 객체가 초기화되고 SQL이 질의 된다
```

###지연로딩(LAZY)
• 실제로 fetch=FetchType.LAZY를 설정한 어떤 필드를 사용하는 시!점!에[getTeam()] 연관된 것을 프록시 객체로 가져옴     
• 지연로딩 LAZY를 사용해서 프록시로 조회     
>  Member member = em.find(Member.class, 1L);      
   Team team = member.getTeam();    
   team.getName(); 			// team을 읽을 경우 이 시점에!! 실제 team을 사용하는 시점에 초기화(DB 조회)

###프록시와 즉시로딩 주의
• *** 중요!! *** 무조건 실무에서 지연 로딩만 사용   
• 즉시 로딩을 적용하면 예상하지 못한 SQL이 발생    
• 즉시 로딩은 JPQL에서 N+1 문제를 일으킨다. (최초 쿼리 1, 그 이후 eager의 개수 만큼 N개의 쿼리가 각각 나감)      
   > ** eager 대신, 대부분 fetch join을 사용하는 방법도 있음, entity graph 어노테이션 사용하는 방법도 있고, 배치 사이즈를 사용하는 방법도 있음    

• @XToOne(@ManyToOne, @OneToOne)은 기본이 즉시 로딩 -> ** 중요!! ** 꼭!! LAZY로 설정    
• @OneToMany, @ManyToMany는 기본이 지연 로딩

### 모든 연관관계는 지연로딩으로 설정! *** 중요!! ***   
  -즉시로딩( EAGER )은 예측이 어렵고, 어떤 SQL이 실행될지 추적하기 어렵다. 특히 JPQL을 실행할 때 N+1 문제가 자주 발생한다.   
  (ex-member 테이블을 읽어들일때 연관된 모든 테이블을 다 읽어들임)     
  =>한건 조회할때는 eager 사용해도 되긴하는데, JPQL [select o From order o;] 실행 시 > SQL [select * from order 100+1(order)]과 같게 되어 100번 읽어 들이는 꼴..      
  그래서 사용안하는게 낫다.eager 대신 lazy와 fetch join을 함께 사용      
  -실무에서 모든 연관관계는 지연로딩( LAZY )으로 설정해야 한다. [-> order에 지연로딩을 하면 order테이블 읽을때 order만 읽음]      
  -연관된 엔티티를 함께 DB에서 조회해야 하면, fetch join 또는 엔티티 그래프 기능을 사용한다.
  > **(결론)     
  >  - @XToMany는 기본이 지연로딩(LAZY)이라 그대로 두면 되는데, @XToOne인 경우에는 무!조!건! 즉시로딩(EAGER)이 기본이라 지연로딩(LAZY)으로 설정을 해줘야함      
  >  - @XToOne(OneToOne, ManyToOne) 관계는 기본이 즉시로딩이므로 직접 지연로딩으로 설정해야 한다.  
  >  - @ManyToOne, @OneToMany(fetch=LAZY) 

###영속성 전이: CASCADE.
• 특정 엔티티를 영속 상태로 만들 때 연관된 엔티티도 함께 영속 상태로 만들도 싶을 때 (ex - parent 저장 시 cascade 적용된 child도 함께 바로 저장)      
• 예: 부모 엔티티를 저장할 때 자식 엔티티도 함께 저장.   
@OneToMany(mappedBy="parent", cascade=CascadeType.PERSIST)
> 사용할 수 있는 경우  
>- 라이프사이클이 거의 똑같을 경우 사용.   
>- 소유자가 하나일 경우 사용 
>- 하나의 부모가 자식들을 관리할 때[소유자가 하나일 경우]는 의미가 있음(ex: 게시판과 첨부파일)   
>  => 단, 쓰면 안되는 케이스 :   
   > *** 중요!! ***  Parent, Child가 CASCADE 관계인데, Child가 Parent 외에 다른 엔티티와 연관관계가 있을 경우 절대 사용하면 안됨  
>  => 단, 쓰면 안되는 케이스2 : 파일을 여러군데에서 관리하거나 부모 외에 다른 엔티티에서 관리 할 경우    
>>  • 주의  
>> - 영속성 전이는 연관관계를 매핑하는 것과 아무 관련이 없음     
>> - 엔티티를 영속화할 때 연관된 엔티티도 함께 영속화하는 편리함을 제공할 뿐     

###CASCADE의 종류
• ALL: 모두 적용   
• PERSIST: 영속 (=저장)  
• REMOVE: 삭제   
• MERGE: 병합    
• REFRESH: REFRESH      
• DETACH: DETACH     

###고아 객체
• 고아 객체 제거: 부모 엔티티와 연관관계가 끊어진 자식 엔티티를 자동으로 삭제      
• orphanRemoval = true     
   > @OneToMany(mappedBy="parent", orphanRemoval=true)

• 참조가 제거된 엔티티는 다른 곳에서 참조하지 않는 고아 객체로 보고 삭제하는 기능    
> 주의
>- ** 중요!! ** 참조하는 곳이 하나일 때 사용해야함! (ex- 첨부파일)
>- ** 중요!! ** 특정엔티티가 개인 소유할때 사용(소유자가 하나)
>- @OneToOne, @OneToMany만 가능
>- 참고: 개념적으로 부모를 제거하면 자식은 고아가 된다. 따라서 고아 객체 제거 기능을 활성화 하면, ** 부모를 제거할 때 자식도 함께 제거된다.      
>  이것은 CascadeType.REMOVE처럼 동작한다

###"영속성 전이 + 고아 객체" 함께 사용할 경우 생명주기     
>CascadeType.ALL + orphanRemovel=true 사용  
>>•두 옵션을 모두 활성화 하면 부모 엔티티를 통해서 자식의 생명 주기를 관리할 수 있음    
=> ex) parent의 생명주기는 영속성 컨텍스트가 관리한다    
> [•스스로 생명주기를 관리하는 엔티티는 em.persist()로 영속화, em.remove()로 제거]       
child는 영속성 컨텍스트가 아닌 parent가 생명주기를 관리한다.      
•도메인 주도 설계(DDD)의 Aggregate Root개념을 구현할 때 유용     
=> ex) parent가 agrregate root고 child가 agreegate root가 관리하는 애다     

###글로벌 페치 전략 설정   ** 중요!! **
  • 모든 연관관계를 지연 로딩으로    
  • @ManyToOne, @OneToOne은 기본이 즉시 로딩이므로 지연 로딩으로 변경


-------------------------------------------------------------
##09.값 타입 			
> ** 중요!! ** : 임베디드 타입(복합 값 타입), 값 타입 컬렉션

###JPA의 데이터 타입 분류 
1.엔티티 타입    
>• @Entity로 정의하는 객체      
>• 데이터가 변해도 식별자로 지속해서 추적 가능    
>• 예) 회원 엔티티의 키나 나이 값을 변경해도 식별자로 인식 가능    
 
2.값 타입
>• int, Integer, String처럼 단순히 값으로 사용하는 자바 기본 타입이나 객체      
>• 식별자가 없고 값만 있으므로 변경시 추적 불가      
>• 예) 숫자 100을 200으로 변경하면 완전히 다른 값으로 대체     

###값 타입 분류
1. 기본값 타입   
   • 자바 기본 타입(int, double)    
   • 래퍼 클래스(Integer, Long)    
   • String    
2. 임베디드 타입(embedded type, 복합 값 타입)        
   => ex) 좌표를 묶어서 쓰고 싶을 때 position과 같은 클래스 만들어서 쓸 때      
3. 컬렉션 값 타입(collection value type)     
   => ex) 자바 컬렉션에 기본값이나 임베디드 타입을 넣어서 쓰는 것을 컬렉션 값 타입      

###기본값 타입 => 사이드 이펙트 없이 쓸 수 있음
> **중요!!** 생명주기를 엔티티의 의존      
=> 예) 회원을 삭제하면 이름, 나이 필드도 함께 삭제

• 예): String name, int age    
• 값 타입은 공유하면 X (=사이드이펙트[원래의 목적과 다르게 다른 효과 또는 부작용]나 부수효과가 일어나면 안됨)    
• 예) 회원 이름 변경시 다른 회원의 이름도 함께 변경되면 안됨      
>• 참고: 자바의 기본 타입은 절대 공유가 되지 않음    
>- int, double 같은 기본 타입(primitive type)은 절대 공유X     
>- 기본 타입은 항상 값을 복사함      
>- Integer같은 래퍼 클래스나 String 같은 특수한 클래스는 공유 가능한 객체이지만 변경X     
>  => 래퍼클래스는 참조값(레퍼런스)을 끌고가기때문에 공유가 됨. 그러나 하나만 변경할 수가 없기 때문에 공유할 수가 없다.     

###임베디드 타입(복합 값 타입)
• 새로운 값 타입을 직접 정의할 수 있음    
• JPA는 임베디드 타입(embedded type)이라 함      
• 주로 기본 값 타입을 모아서 만들어서 복합 값 타입이라고도 함      
• int, String과 같은 값 타입 [변경하면 끝.]    
• 임베디드 타입의 값이 null이면 매핑한 컬럼 값은 모두 null    

###임베디드 타입 사용법    
• @Embeddable: 값 타입을 정의하는 곳에 표시        
• @Embedded: 값 타입을 사용하는 곳에 표시    
• 기본 생성자 필수    
• ex)
```java
 @Entity
 public class Member{

     @Id @GeneratedValue
     privte Long id;

     @Embedded
     private Address homeAddress;
     
     //한 엔티티에서 같은 값 타입 중복 사용하기 위해 필수로 속성 재정의 어노테이션[@AttributeOverrides] 필요 
     @Embedded
     @AttributeOverrides({
         @AttributeOverride(name="city",column=@Column(name="WORK_CITY")),
         @AttributeOverride(name="street",column=@Column(name="WORK_STREET"))
     })		
     private Address workAddress;
 }

 @Embeddable
 public class Address{

     private String city;
     private String street;	

     //기본생성자 구현
     public Address(){
     }
 }
```

###임베디드 타입[=복합 값 타입]의 장점
• 재사용, 공통화 가능  
• 높은 응집도    
• Period.isWork()처럼 해당 "값 타입"만 사용하는 의미 있는 메소드를 만들 수 있음      
• 임베디드 타입을 포함한 모든 값 타입은, 값 타입을 소유한 엔티티에 생명주기를 의존함     
=> 값 타입을 소유한 엔티티에 생명주기란 엔티티 생성되면 값들어오고 소멸되면 사라지는..    

###임베디드 타입과 테이블 매핑
> **중요!!** 임베디드 타입을 사용하기 전과 후에 매핑하는 테이블은 같다.          

• 임베디드 타입은 엔티티의 값일 뿐이다.    
• 객체와 테이블을 아주 세밀하게(find-grained) 매핑하는 것이 가능        
• 잘 설계한 ORM 애플리케이션은 매핑한 테이블의 수보다 클래스의 수가 더 많음      

###@AttributeOverride: 속성 재정의
• 한 엔티티에서 같은 값 타입을 사용하면?   
=> 컬럼 명이 중복됨 => 에러 발생 => 그때 사용하기 위해 필수로 속성 재정의 어노테이션 필요      
• @AttributeOverrides, @AttributeOverride를 사용해서 컬럼명 속성을 재정의 (소스코드 위 참고)

----
## 값 타입과 불변 객체
> * 값 타입은 복잡한 객체 세상을 조금이라도 단순화하려고 만든 개념이다. 따라서 값 타입은 단순하고 안전하게 다룰 수 있어야 한다
> * 임베디드 타입은 엔티티 조회 시 한번에 불러오고 한번에 저장됨[한 테이블에 저장됨]

### 값 타입 공유 참조 [절대 사용 X]
• 임베디드 타입 같은 값 타입을 여러 엔티티에서 공유하면 위험함 > 부작용(side effect) 발생     
ex)
```java
@Transactional{
   Address address = new Address("city","street")			//임베디드 타입이라고 가정
   
   Member member = new Member();
   member.setUsername("member1");
   member.setHomeAddress(adress);
   em.persist(member);
   
   Member member2 = new Member();
   member2.setUsername("member2");
   member2.setHomeAddress(adress);
   em.persist(member2);
   
   member.getHomeAdress().setCity("newCity");			
   // => 임베디드 타입이 값 타입인데, 이렇게 값 타입을 공유해서 사용할 경우 member, member2 둘다 newCity로 변경되는 사이드이펙트(부작용)가 발생됨
}
```

###값 타입 복사 [*** 중요!! ** 값 타입 공유 참조대신 복사를 사용하자]
• 값 타입의 실제 인스턴스인 값을 공유하는 것은 위험 > 값(인스턴스)를 복사해서 사용  
ex)
```java
@Transactional{
   Address address = new Address("city","street")
   
   Member member = new Member();
   member.setUsername("member1");
   member.setHomeAddress(adress);
   em.persist(member);
   
   Address copyAddress = new Address(address.getCity(),address.getStreet());	//값 복사하여 객체 생성
   
   Member member2 = new Member();
   member2.setUsername("member2");
   member2.setHomeAddress(copyAddress);
   em.persist(member2);
   
   member.getHomeAdress().setCity("newCity");	//member만 newCity로 변경됨.
}
```

###객체 타입의 한계
• 항상 값을 복사해서 사용하면 공유 참조로 인해 발생하는 부작용을 피할 수 있다.  
• 문제는 임베디드 타입처럼 직접 정의한 값 타입은 자바의 기본타입(primitive Type [=으로 값을 할당하면 값이 복사되서 넘어감])이 아니라 객체 타입(참조 값을 직접 대입하는 것)이다.    
• 자바 기본 타입에 값을 대입하면 값을 복사한다.     
• 객체 타입은 참조 값을 직접 대입하는 것을 막을 방법이 없다. > 객체의 공유 참조는 피할 수 없다. [공유가 되도록 코딩해도 컴파일 시 에러가 발생하지 않음]     
>• *** 중요!! *** 해결책 : 불변 객체 생성!!

###임베디드 타입을 불변 객체로 생성!! **중요!!**
• 객체 타입을 수정할 수 없게 만들면 부작용을 원천 차단    
• ***중요!! *** 값 타입은 불변 객체(immutable object)로 설계해야함       
• 불변 객체: 생성 시점 이후 절대 값을 변경할 수 없는 객체    
• 참고: Integer, String은 자바가 제공하는 대표적인 불변 객체      
• 생성자로만 값을 설정하고 수정자(Setter)를 만들지 않으면 됨 [최초만 값이 설정됨]         
=> 중간에 값을 변경하고 싶을 경우엔?     
ex)   
```java
@Transactional{
    Address address = new Address("city","street")

     Member member = new Member();
     member.setUsername("member1");
     member.setHomeAddress(adress);
     em.persist(member);
     
     Address newAddress = new Address("newCity",address.getStreet());	
     member.setHomeAddress(newAddress);		
     // => 객체를 새로 하나 생성해서 다시 셋팅해서 변경해야함
     
 }
```
###값 타입의 비교 [***중요!! *** 임베디드 타입의 비교도 아래와 같다]   
>• 값 타입: 인스턴스가 달라도 그 안에 값이 같으면 같은 것으로 봐야 함    
>> cf) 인스턴스의 참조 값을 비교, == 사용 [동일성(identity) 비교]      
>>   인스턴스의 값을 비교, equals() 사용 [동등성(equivalence) 비교]     

>• 값 타입은 a.equals(b)를 사용해서 동등성 비교를 해야 함    
>• 값 타입의 equals() 메소드를 적절하게 재정의(주로 모든 필드 사용)하여 사용하여 equals 비교를 한다     
>> => 재정의 하지 않은 equals 기본이 == 비교이기 때문의 재정의하여 사용 해야 함	     
[**getter로 만들도록 체크하여 사용. 직접 필드에 접근하면 프록시일때 접근이 안됨.       
> 그래서 jpa에서는 use getters during code generation 체크 후 equals 만들기! ]     
=> equals 오버라이드 재정의 시, hashcode 메소드도 함께 오버라이드 해야 함    

----

###값 타입 컬렉션     
> > *** 중요!! *** (결론)   
> 값 타입 컬렉션은 사용하지 말고, 일대다 관계(or 다대일 양방향을 쓰면 업데이트 쿼리 없앨수있음.)를 고려 [이유. 수정 시 모두 delete 후 다시 insert]       
•값 타입 컬렉션은 정말 단순할때만 사용. 추적할 필요없고 값이 바껴도 업데이트 칠 필요 없을때 사용    
>>> ex ) selectbox에 치킨,피자 멀티 셀렉트 가능하도록 체크박스 체크하고 싶을때..

• 엔티티를 컬렉션에 넣어서 쓰는게 아니라, 값 타입을 컬렉션에 넣어서 쓰는 것. (set, list...등의 컬렉션)      
• 값 타입을 하나 이상 저장할 때 사용     
• @ElementCollection, @CollectionTable 사용       
• 데이터베이스는 컬렉션을 같은 테이블에 저장할 수 없다, 컬렉션을 저장하기 위한 별도의 테이블이 필요함 [한 테이블에 저장 x]      
• 값 타입 컬렉션은 영속성 전이(Cascade) + 고아 객체 제거 기능을 필수로 가진다고 볼 수 있다     
• ex) 
```java
@Entity
public class Member{

     @Id @GeneratedValue
     privte Long id;

     @Embedded
     private Address homeAddress;
     
     @ElementCollection
     @CollectionTable(name = "FAVORITE_FOOT", 
         joinColumns = @JoinColumn(name = "MEMBER_ID")
     ) 
     @Column(name = "FOOD_NAAE")				//컬럼이 두개여서 예외적으로 클래스 생성하지 않고 이렇게 사용할 수 있음
     private Set<String> favoriteFoods = new HashSet<>();
     
     @ElementCollection
     @CollectionTable(name = "ADDRESS", 
         joinColumns = @JoinColumn(name = "MEMBER_ID")
     ) 
     private List<Address> adressHistory = new ArrayList<>();
 }

 @Embeddable
 public class Address{

     private String city;
     private String street;	

     //기본생성자 구현
     public Address(){
     }
 }	
```
•값 타입 컬렉션 저장	 [컬렉션의 값만 변경 되도 jpa가 알아서 저장해줌]     
- 값 타입 컬렉션을 따로 persist 하지 않아도 member를 persist할때 들어감. 값 타입 컬렉션의 라이프 사이클은 엔티티 Member에 의존함.     
  Member에서 값을 체인지 하면 자동으로 업데이트 됨. cascade와 비슷.[영속성 전이(Cascade) + 고아 객체 제거 기능을 필수로 가짐]     
  ex) 
  ```java
   @Transactional{
      Member member = new Member();
      member.setUsername("member1");
      member.setHomeAddress(new Address("homeCity","street");
      
      member.getFavoriteFoods().add("치킨");
      member.getFavoriteFoods().add("족발");
      
      member.getAddressHistory().add(new Address("old1","street1"));
      member.getAddressHistory().add(new Address("old2","street1"));
      
      em.persist(member);		
   }
  ```
  •값 타입 컬렉션 조회 : 지연로딩      
  •값 타입 수정 : 불변으로, 통으로 바꿔서 수정해야 함.      
   ex)
   ```java
   @Transactional{

  	Member member = new Member();
  	member.setUsername("member1");
  	member.setHomeAddress(new Address("homeCity","street");

  	member.getFavoriteFoods().add("치킨");
  	member.getFavoriteFoods().add("족발");

  	member.getAddressHistory().add(new Address("old1","street1"));
  	member.getAddressHistory().add(new Address("old2","street1"));
  	
  	em.persist(member);	
  	
  	em.flush();
  	em.clear();
  	
  	Member findMember = em.find(Member.class, member.getId());

  	findMember.getFavoriteFoods().remove("치킨");
  	findMember.getFavoriteFoods().add("한식");

  	//기본적으로 컬렉션에서 remove 대상 찾을때 equals를 사용함.이렇게 사용하려면 꼭 equals, hashcode 오버라이딩 필요[컬렉션 다룰때 주로 의미가 있음]		
  	findMember.getAddressHistory().remove(new Address("old1","street1"));       //삭제		
  	findMember.getAddressHistory().add(new Address("newnew","street1"));		//추가
  	
  	//트랜잭션 커밋
  }
  ```
  => 결과 : MEMBER_ID에 대한 ADDRESS를 모두 delete 하고, 새로 inset를 2번 한다.   
  [데이터를 모두 지우고,, 최종 컬렉션에 남아있는 것만 insert]     
  delete from Address where MEMBER_ID = ?    
  insert into Address (MEMBER_ID, city, street) values( ?,"old2","street1")      
  insert into Address (MEMBER_ID, city, street) values( ?,"newnew","street1")    

###값 타입 컬렉션의 제약사항    
• 값 타입은 엔티티와 다르게 식별자 개념이 없다.     
• 값은 변경하면 추적이 어렵다.      
> • ***중요!!*** 
 >>-값 타입 컬렉션에 변경 사항이 발생하면, 주인 엔티티와 연관된 모든 데이터를 삭제하고, 값 타입 컬렉션에 있는 현재 값을 모두 다시 저장한다.       
 >>-값 타입 컬렉션을 매핑하는 테이블은 모든 컬럼을 묶어서 기본키를 구성해야 함: null 입력X, 중복 저장X        
>>- 위와 같이 구성하지 않을 경우 값타입 컬렉션 대신 일대다 관계를 고려!!!     
      
###값 타입 컬렉션 대안
• 실무에서는 상황에 따라 값 타입 컬렉션 대신에 엔티티로 만들어 일대다 관계를 고려    
• 일대다 관계를 위한 엔티티를 만들고, 여기에서 값 타입을 사용      
• 영속성 전이(Cascade) + 고아 객체 제거를 사용해서 값 타입 컬렉션 처럼 사용     
• ex) 
```java
@Entity
public class Member{

     @Id @GeneratedValue
     privte Long id;

     @Embedded
     private Address homeAddress;
     
     @OneToMany(cascade = CascadeType.ALL, orphanRemoval = true)
     @JoinColumn(name= "MEMBER_ID")
     private List<AddressEntity> adressHistory = new ArrayList<>();
 }
 
 @Entity
 @Table(name="ADDRESS")
 public class AddressEntity{

     @Id @GeneratedValue
     private Long id;
     private Address address;	//값 타입
     
     //getter..setter..기본생성자.생성자..
 }	
 
 @Embeddable			
 public class Address{

     private String city;
     private String street;	

     //기본생성자 구현
     public Address(){
     }
}		
```

###엔티티 타입의 특징
• 식별자O   
• 생명 주기 관리  
• 공유  

### 값 타입의 특징
  • 식별자X       
  • 생명 주기를 엔티티에 의존         
  • 공유하지 않는 것이 안전(복사해서 사용)    
  • 불변 객체로 만드는 것이 안전    
  • 실무에서 잘 사용 X
> - 값 타입은 정말 값 타입이라 판단될 때만 사용
> - 엔티티와 값 타입을 혼동해서 엔티티를 값 타입으로 만들면 안됨
> - 식별자가 필요하고, 지속해서 값을 추적, 변경해야 한다면 그것은 값 타입이 아닌 엔티티
---------------------------------------------------------------------
##10.객체지향 쿼리 언어

###JPA는 다양한 쿼리 방법을 지원
• JPQL [Java Persistence Query Language]   **      
• JPA Criteria (실무 사용x)    
• QueryDSL  **    
• 네이티브 SQL     
• JDBC API 직접 사용, MyBatis, SpringJdbcTemplate 함께 사용      


### //JPQL 관련 내용 시작
### JPQL 소개
• 가장 단순한 조회 방법       
• EntityManager.find()     
• 객체 그래프 탐색(a.getB().getC())     

###JPQL
• JPA를 사용하면 엔티티 객체를 중심으로 개발      
• 문제는 검색 쿼리    
• 검색을 할 때도 테이블이 아닌 엔티티 객체를 대상으로 검색     
• 모든 DB 데이터를 객체로 변환해서 검색하는 것은 불가능      
• 애플리케이션이 필요한 데이터만 DB에서 불러오려면 결국 검색 조건이 포함된 SQL이 필요      
• JPA는 SQL을 추상화한 JPQL이라는 객체 지향 쿼리 언어 제공      
• SQL과 문법 유사, SELECT, FROM, WHERE, GROUP BY, HAVING, JOIN 지원      
• JPQL은 엔티티 객체를 대상으로 쿼리    
SQL은 데이터베이스 테이블을 대상으로 쿼리      
• 테이블이 아닌 객체를 대상으로 검색하는 객체 지향 쿼리    
• SQL을 추상화해서 특정 데이터베이스 SQL에 의존X     
• JPQL을 한마디로 정의하면 객체 지향 SQL      
• JPQL은 객체지향 쿼리 언어다.따라서 테이블을 대상으로 쿼리하는 것이 아니라 엔티티 객체를 대상으로 쿼리한다.     
• ** JPQL은 SQL을 추상화해서 특정데이터베이스 SQL에 의존하지 않는다.      
• *** JPQL은 결국 매핑정보랑 방언이랑 조합이 되서 SQL로 변환되어 실행된다.      
• ex)  
```java
//검색						  
//아래에서 Member는 엔티티를 가리킴, m도 Member 자체를 가지고 온다는 뜻
String jpql = "select m From Member m where m.name like ‘%hello%'";
List<Member> result = em.createQuery(jpql, Member.class).getResultList()
```

###JPQL 문법
• select m from Member as m where m.age > 18    
• 엔티티와 속성은 대소문자 구분O (Member, age)      
• JPQL 키워드는 대소문자 구분X (SELECT, FROM, where)      
• 엔티티 이름 사용, 테이블이나 클래스 이름이 아님(Member), 기본적으로 name 지정안하면 class명  => @Entity(name = "member") <- 보통 name으로 클래스명이랑 다르게 선언안함      
• 별칭은 필수(m) (as는 생략가능)     
>• TypeQuery: 반환 타입이 명확할 때 사용		=> 기본적으로 엔티티를 타입정보로 준다.    
>• Query: 반환 타입이 명확하지 않을 때 사용          
>• ex)
>```java
>TypedQuery<Member> query = em.createQuery("SELECT m FROM Member m", Member.class);
>TypedQuery<String> query = em.createQuery("SELECT m FROM Member m", String.class);
>Query query = em.createQuery("SELECT m.username, m.age from Member m");
>```

###결과 조회 API
- query.getResultList(): 결과가 하나 이상일 때, 리스트 반환      
  => 결과가 없으면 빈 리스트 반환      
- query.getSingleResult(): 결과가 정확히 하나, 단일 객체 반환    
  => (표준스펙)    
  결과가 없으면: javax.persistence.NoResultException     
  둘 이상이면: javax.persistence.NonUniqueResultException     
  => Spring Data Jpa 의 경우에는 이셉션을 발생하지 않고 null을 반환하거나 optional을 반환한다.    
  •파라미터 바인딩 - 이름 기준, 위치 기준       
  => 위치 기준으로 사용하지 말고 아래 예시와 같이 **이름기준으로 사용      
  ex) 메소드 체이닝 없이 쓸수 있음
  ```java
  Member result = em.createQuery("SELECT m FROM Member m where m.username = :username", Member.class);
  .setParameter("username", "member1")
  .getSingleResult();
   ```

###프로젝션
- SELECT 절에 조회할 대상을 지정하는 것    
- 프로젝션 대상: 엔티티, 임베디드 타입, 스칼라 타입(숫자, 문자등 기본 데이터 타입)    
```SQL
- SELECT m FROM Member m -> 엔티티 프로젝션      
- SELECT m.team FROM Member m -> 엔티티 프로젝션    
- SELECT m.address FROM Member m -> 임베디드 타입 프로젝션      
- SELECT m.username, m.age FROM Member m -> 스칼라 타입 프로젝션     
- DISTINCT로 중복 제거  [ex) SELECT distinct m.username, m.age FROM Member m   : username distinct하여 조회]      
```

###프로젝션 - 여러 값 조회      
• SELECT m.username, m.age FROM Member m     
> 1. Query 타입으로 조회         
> 2. Object[] 타입으로 조회
>   ex) 
> ```java 
>   List<Object[]> result  = em.createQuery("SELECT m.username, m.age from Member m")
>   .getResultList();			
>   Object[] result = resultList.get(0);
>   System.out.println("username="+result[0]+", age="+result[1]);
> ```
>3. new 명령어로 조회
>- 단순 값을 DTO로 바로 조회
>  ex) 
> ```java
>  List<MemberDTO> result  = em.createQuery("SELECT new jpql.MemberDTO(m.username, m.age) from Member m", MemberDTO.class)
>  .getResultList();			
>  MemberDTO memberDTO = result.get(0);
>  System.out.println("username="+memberDTO.getUserName());
> ```
>- 패키지 명을 포함한 전체 클래스 명 입력
>- 순서와 타입이 일치하는 생성자 필요

###페이징 API
- JPA는 페이징을 다음 두 API로 추상화     
- setFirstResult(int startPosition) : 조회 시작 위치(0부터 시작)      
- setMaxResults(int maxResult) : 조회할 데이터 수      
  ex) //페이징 쿼리
  ```java
  String jpql = "select m from Member m order by m.name desc";
  List<Member> resultList = em.createQuery(jpql, Member.class)
  .setFirstResult(10)
  .setMaxResults(20)
  .getResultList();
  ```

###조인
```sql
- 내부 조인 : SELECT m FROM Member m [INNER] JOIN m.team t      
- 외부 조인 : SELECT m FROM Member m LEFT [OUTER] JOIN m.team t    
- 세타 조인[cross join. 카타시안 곱]: select count(m) from Member m, Team t where m.username = t.name    
- ON절 사용 가능(JPA 2.1부터 지원)     
   1. 조인 대상 필터링      
   2. 연관관계 없는 엔티티 외부 조인(하이버네이트 5.1부터)     
```

###서브쿼리 지원함수
- [NOT] EXISTS (subquery): 서브쿼리에 결과가 존재하면 참
- {ALL | ANY | SOME} (subquery)
  => ALL 모두 만족하면 참
  ANY, SOME: 같은 의미, 조건을 하나라도 만족하면 참
- [NOT] IN (subquery): 서브쿼리의 결과 중 하나라도 같은 것이 있으면 참

###JPA 서브 쿼리 한계 *** 중요!!! ***
>- (표준스펙) JPA는 WHERE, HAVING 절에서만 서브 쿼리 사용 가능
>- SELECT 절도 가능(하이버네이트에서 지원)
>- ** 중요!! ** FROM 절의 서브 쿼리는 현재 JPQL에서 불가능
>- ** 중요!! ** 조인으로 풀 수 있으면 풀어서 해결

###JPQL 타입 표현
- 문자: ‘HELLO’, ‘She’’s’    
- 숫자: 10L(Long), 10D(Double), 10F(Float)     
- Boolean: TRUE, FALSE     
- ENUM: jpabook.MemberType.Admin (패키지명 포함!!)    
- 엔티티 타입: TYPE(m) = Member (상속 관계에서 사용)      
   
###SQL과 문법이 같은 식(BETWEEN ~and ~, = , <= , is null, exist, 등)      

###조건식 - CASE 식     
```sql
- case t.name when '팀A' then '인센티브110%'  when '팀B' then '인센티브120%' else '인센티브105%' end    

- COALESCE: 하나씩 조회해서 null이 아니면 반환      
  => 사용자 이름이 없으면 "이름 없는 회원"을 반환     
  ex) select coalesce(m.username,'이름 없는 회원') from Member m      

- NULLIF: 두 값이 같으면 null 반환, 다르면 첫번째 값 반환        
  => 사용자 이름이 ‘관리자’면 null을 반환하고 나머지는 본인의 이름을 반환     
  ex) select NULLIF(m.username, '관리자') from Member m      
```

###JPQL 기본 함수
- CONCAT, SUBSTRING, TRIM, LOWER, UPPER, LENGTH, LOCATE, ABS, SQRT, MOD    
  => 하이버네이트는 'a' || 'b'로  concat과 같은 기능을 제공한다.     
  => locate('de','abcdegf') -> return 4      
- SIZE, INDEX(JPA 용도)      
  => SIZE : 컬렉션의 크기 반환    
  => INDEX : @OrderColumn을 함께 쓸 때 사용하는데, 실무에서 사용하지 X    

###사용자 정의 함수 호출
- 데이터베이스에서 정의되어있는 사용자 정의 함수를 호출 하기 위해서는 하이버네이트는 사용전 방언에 추가해야 한다.     
- 사용하는 DB 방언을 상속받고, 벤더들이 제공하지 않는 함수는 직접 사용자 정의 함수를 등록한다.    
  ex) H2Dialect 소스코드 열어보면 등록하는 방법 확인 가능. 각 db에 맞도록 등록하면 됨      
  ```java
  public class MyH2Dialect extends H2Dialect{   

  	public MyH2Dialect(){
  		regissterFunction("group_concat", new StandardSQLFunction("group_concat", StandardBasicTypes.STRING));
  	}
  }

  query = "select function('group_concat', i.name) from Item i"
   ```
###JPQL - 경로표현식
```sql
- .(점)을 찍어 객체 그래프를 탐색하는 것     
  ex)    
     select m.username -> 상태 필드     
     from Member m      
     join m.team t -> 단일 값 연관 필드    
     join m.orders o -> 컬렉션 값 연관 필드    
     where t.name = '팀A'      
     
- 상태 필드(state field): 단순히 값을 저장하기 위한 필드(ex: m.username), 경로 탐색의 끝, 또 점을 찍어서 탐색X     

- 연관 필드(association field): 연관관계를 위한 필드      
   1. 단일 값 연관 필드: @ManyToOne, @OneToOne, 대상이 엔티티(ex: m.team)      
      => 단일 값 연관 경로 : 묵시적 내부 조인(inner join) 발생, 탐색O      
      ex) select m.team.name From Member m		**조인하여 쿼리가 나감. 묵시적 내부조인이 발생하게 이런식으로 쓰지 말기    
         =>  m.team, m.team.name 탐색 가능     
      
   2. 컬렉션 값 연관 필드:@OneToMany, @ManyToMany, 대상이 컬렉션(ex: m.orders)     
      => 컬렉션 값 연관 경로 : 묵시적 내부 조인(inner join) 발생, 탐색X     
      ex) select t.members From Team t 			**이렇게 잘 안씀     
         => t.members에서 더이상 탐색 불가      
            > FROM 절에서 명시적 조인을 통해 별칭을 얻으면 별칭을 통해 탐색 가능      
```

>>> ***중요!! (결론)*** 실무에서 묵시적 조인 절대 사용하지 말고, 명시적 조인을 사용한다!! ***

### 명시적 조인, 묵시적 조인
• 명시적 조인: join 키워드 직접 사용   
 > select m from Member m join m.team t   

• 묵시적 조인: 경로 표현식에 의해 묵시적으로 SQL 조인 발생 (항상 내부 조인), 조인이 일어나는 상황 파악 어려움     
 >select m.team from Member m

###JPQL - 페치 조인(fetch join)		**** 중요!! ****
- SQL 조인 종류X   
- JPQL에서 성능 최적화를 위해 제공하는 기능   
- 연관된 엔티티나 컬렉션을 SQL 한 번에 함께 조회하는 기능(한방쿼리)    
- join fetch 명령어 사용     
- 페치 조인 ::= [ LEFT [OUTER] | INNER ] JOIN FETCH 조인경로     

###엔티티 페치 조인
• 회원을 조회하면서 연관된 팀도 함께 조회(SQL 한 번에)     
• 즉시로딩과 똑같지만, 쿼리로 내가 원하는 대로 즉시 명시적으로 동적인 타이밍을 정할 수 있음.      
• SQL을 보면 회원 뿐만 아니라 팀(T.*)도 함께 SELECT     
> [JPQL]       
   select m from Member m join fetch m.team     
   ==    
   [SQL]    
   SELECT M.*, T.* FROM MEMBER M       
   INNER JOIN TEAM T ON M.TEAM_ID=T.ID 

•  ex) 페치 조인 사용코드    
```java
String jpql = "select m from Member m join fetch m.team";

List<Member> members = em.createQuery(jpql, Member.class).getResultList();

for (Member member : members) {
//페치 조인으로 회원과 팀을 함께 조회해서 지연 로딩X, 즉시로딩O, 한방쿼리
System.out.println("username = " + member.getUsername() + ", " + "teamName = " + member.getTeam().name());
}		
=> 그렇다면, 패치 조인과 일반조인이 같다고 생각할 수 있으나!! 절!대! 아님!!!
```

###페치 조인 VS 일반 조인 차이 	
> [강의 cf : 객체지향 쿼리 언어2-중급문법 > 페치조인1- 기본 > 26분~]

###일반 조인 (지연 로딩)
- 연관 엔티티에 join을 하게되면 Select 대상의 엔티티는 영속화하여 가져오지만, 조인의 대상은 영속화하여 가져오지 않는다.     
- 연관 엔티티가 검색 조건에 포함되고, 조회의 주체가 검색 엔티티뿐일 때 사용하면 좋다.       
- 일반 조인 실행 시 연관된 엔티티를 함께 조회해주지 않음         

###페치 조인 (즉시 로딩)
- 연관 엔티티에 fetch join을 하게되면 select 대상의 엔티티뿐만 아니라 조인의 대상까지 영속화하여 가져온다.    
- 객체 그래프 SQL을 한번에 조회, 페치조인을 사용할때만 연관된 엔티티도 함께 조회한다.      
- 연관 엔티티까지 select의 대상일 때, "N+1의 문제를 해결!!"하여 가져올 수 있는 좋은 방법이다.     
- 페치 조인을 사용할 때만 연관된 엔티티도 함께 조회(즉시 로딩)     
- 페치 조인은 객체 그래프를 SQL 한번에 조회하는 개념    
- 페치 조인을 사용할 경우 지연로딩으로 셋팅을 해도(team이 lazy로 설정됨) lazy는 무시되고 패치조인이 우선이다.      
  cf)https://cobbybb.tistory.com/18    
  cf)https://blog.naver.com/PostView.nhn?blogId=qjawnswkd&logNo=222078705093     

###컬렉션 페치 조인      
• 컬렉션을 가진다는 것은 1:N 관계라는 것이고 이 때 join하여 db조회 시 데이터가 뻥튀기 됨. (참고: 다대일 관계일 때는 뻥튀기 안됨)      
 >> ** 중요!! ** 즉, 반드시 distinct 옵션으로 중복 객체를 제거해야 한다    

> • 컬렉션 패치조인은 1개만 사용 ** 중요!! **      
>>>why? 컬렉션을 패치조인하면 1:N으로 데이터가 증가되는데 컬렉션을 또 패치조인하면 1:N:M 관계로 데이터가 많아져서 부정합하게 조회될 수 있음    

• 일대다 관계, 컬렉션 페치 조인     
>[JPQL]      
   select t    
   from Team t join fetch t.members    
   where t.name = '팀A'     
==    
[SQL]     
SELECT T.*, M.*      
FROM TEAM T    
INNER JOIN MEMBER M ON T.ID=M.TEAM_ID     
WHERE T.NAME = '팀A'     

• ex) 컬렉션 페치 조인 사용 코드      
```java
String jpql = "select t from Team t join fetch t.members where t.name = '팀A'"   
        
List<Team> teams = em.createQuery(jpql, Team.class).getResultList();

for(Team team : teams) {
    System.out.println("teamname = " + team.getName() + ", team = " + team);
   for (Member member : team.getMembers()) {
      //페치 조인으로 팀과 회원을 함께 조회해서 지연 로딩 발생 안함
      System.out.println("-> username = " + member.getUsername()+ ", member = " + member);
   }
}
```
• cf) https://velog.io/@neity16/4-%EC%8A%A4%ED%94%84%EB%A7%81-%EB%B6%80%ED%8A%B8%EC%99%80-JPA-%ED%99%9C%EC%9A%A9-7-API-%EA%B0%9C%EB%B0%9C-%EA%B3%A0%EA%B8%89-2-%EC%BB%AC%EB%A0%89%EC%85%98-%EC%A1%B0%ED%9A%8C-%EC%B5%9C%EC%A0%81%ED%99%94

###페치 조인과 DISTINCT   
• SQL의 DISTINCT는 중복된 결과를 제거하는 명령       
• JPQL의 DISTINCT 2가지 기능 제공    
1. SQL에 DISTINCT를 추가 (조인한 데이터가 다르므로 SQL 결과에서 중복제거 실패)    
2. 애플리케이션에서 엔티티 중복 제거. (같은 식별자를 가진 Team 엔티티 제거)    

###페치 조인의 특징과 한계
>• ** 중요!! ** 페치 조인 대상에는 별칭을 줄 수 없다.  => 하이버네이트는 가능, 가급적 실무에서 별칭(as)를 사용X     
>• ** 중요!! ** 둘 이상의 컬렉션은 페치 조인 할 수 없다. [1:N:M..]  
> 
>• ** 중요!! ** 컬렉션을 페치 조인하면 페이징 API(setFirstResult, setMaxResults)를 사용하면 안된다!!    
>>- 일대일, 다대일 같은 단일 값 연관 필드들은 페치 조인해도 페이징 가능!!     
>>- 하이버네이트는 경고 로그를 남기고 메모리에서 페이징(매우 위험) - db에서 페이징 x    
>>- 컬렉션일때 페이징이 안되니까, 이 때 @BatchSize를 이용하면 N+1개 문제 해결 가능    
>>- dto를 사용해서도 가능하지만 고려해야할 것들이 있음     
> 
>  • 엔티티에 직접 적용하는 글로벌 로딩 전략보다 우선함     
>>     -> @OneToMany(fetch = FetchType.LAZY) //글로벌 로딩 전략      
>  • 실무에서 글로벌 로딩 전략은 모두 지연 로딩!!! ***
> 
>  • 최적화가 필요한 곳은 페치 조인 적용 (70~80% 페치 조인 적용하면 성능 향상)    
>  • 모든 것을 페치 조인으로 해결할 수는 없음     
>  • 페치 조인은 객체 그래프를 유지할 때 사용하면 효과적 
> 
>  • 여러 테이블을 조인해서 엔티티가 가진 모양이 아닌 전혀 다른 결과를 내야 하면,     
>     페치 조인 보다는 일반 조인을 사용하고 필요한 데이터들만 조회해서 DTO로 반환하는 것이 효과적      
  => 통계 같은 경우 sql로도..가능..     
  
###JPQL - 다형성 쿼리
1.TYPE      
• 조회 대상을 특정 자식으로 한정     
• 예) Item 중에 Book, Movie를 조회해라      
> [JPQL]          
select i from Item i    
where type(i) IN (Book, Movie)      
==        
[SQL]    
select i from i      
where i.DTYPE in (‘B’, ‘M’)   

2.TREAT(JPA 2.1)
• 자바의 타입 캐스팅과 유사     
• 상속 구조에서 부모 타입을 특정 자식 타입으로 다룰 때 사용    
• FROM, WHERE, SELECT(하이버네이트 지원) 사용    
• 예) 부모인 Item과 자식 Book이 있다.      
>[JPQL]     
select i from Item i    
where treat(i as Book).auther = ‘kim’     
==       
[SQL]    
select i.* from Item i     
where i.DTYPE = ‘B’ and i.auther = ‘kim      

###엔티티 직접 사용 - 기본 키 값
• JPQL에서 엔티티를 직접 사용하면 SQL에서 해당하는 엔티티의 기본키 값을 사용하여 조회한다      
>[JPQL]     
select count(m.id) from Member m //엔티티의 아이디를 사용    
select count(m) from Member m //엔티티를 직접 사용
==    
[SQL](JPQL 둘다 아래 SQL과 같이 실행됨)    
select count(m.id) as cnt from Member m      

• 1.엔티티를 파라미터로 전달            
>String jpql = “select m from Member m where m = :member”;      
>List resultList = em.createQuery(jpql).setParameter("member", member) .getResultList();      

• 2.식별자를 직접 전달      
>String jpql = “select m from Member m where m.id = :memberId”;    
>List resultList = em.createQuery(jpql).setParameter("memberId", memberId) .getResultList();     

• => 엔티티를 파라미터로 전달하거나 식별자를 직접 전달한 JPQL에 대한 실행된 SQL은 아래와 같다    
>select m.* from Member m where m.id=?     

*엔티티 직접 사용 - 외래 키 값     
• ex)    
```java
Team team = em.find(Team.class, 1L);      
String qlString = “select m from Member m where m.team = :team”;     
List resultList = em.createQuery(qlString).setParameter("team", team) .getResultList();      

String qlString = “select m from Member m where m.team.id = :teamId”;
List resultList = em.createQuery(qlString).setParameter("teamId", teamId) .getResultList();

=>실행된 SQL
select m.* from Member m where m.team_id=?
```

###JPQL - Named 쿼리[정적 쿼리]     
• 미리 정의해서 이름을 부여해두고 사용하는 JPQL , 정적쿼리      
• 어노테이션, XML에 정의     
=>  XML이 항상 우선권을 가진다, 애플리케이션 운영 환경에 따라 다른 XML을 배포할 수 있다. 
• 애플리케이션 로딩 시점에 초기화 후 재사용     
> ** 중요 ** 애플리케이션 로딩 시점에 쿼리를 검증    
>=> spring data jpa를 사용한다면, 인터페이스 메소드 위에 @Query("select ~~")로 네임드쿼리를 사용할 수 있도록 제공한다.

###JPQL - 벌크연산
• 재고가 10개 미만인 모든 상품의 가격을 10% 상승하려면?    
• JPA 변경 감지 기능으로 실행하려면 너무많은 SQL실행      
• ex) 변경된 데이터가 100건이라면 100번의 UPDATE SQL 실행            
   1. 재고가 10개 미만인 상품을 리스트로 조회한다.    
   2. 상품 엔티티의 가격을 10% 증가한다.      
   3. 트랜잭션 커밋 시점에 변경감지가 동작한다.    

• 쿼리 한 번으로 여러 테이블 로우 변경(엔티티)     
• executeUpdate()의 결과는 영향받은 엔티티 수 반환      
• UPDATE, DELETE 지원     
• INSERT(insert into .. select, 하이버네이트 지원)      
ex) 
```java
String qlString = "update Product p set p.price = p.price * 1.1 where p.stockAmount < :stockAmount";
int resultCount = em.createQuery(qlString)
.setParameter("stockAmount", 10)
.executeUpdate();
```

###벌크 연산 주의
> 벌크 연산은 영속성 컨텍스트를 무시하고 데이터베이스에 직접 쿼리함. 즉, db에만 반영되고 영속성컨텍스트에는 저장되지 않는다.     
> ** 중요!!** [해결책] :      
>> 1. (영속성 컨텍스트에 값 넣지 말고) 벌크 연산을 먼저 실행    
>> 2. 벌크연산 수행 후 영속성 컨텍스트 초기화 [em.clear();]
```java
ex)
Member member1 = new Member();     
member1.setUsername("1");
member1.setAge(0);
em.persist(member1);

Member member2 = new Member();
member2.setUsername("1");
member2.setAge(0);
em.persist(member2);

//Flush
int result = em.createQuery("update Member set m.age =20").executeUpdate();

//createQuery가 날라간 이 시점에 db에 있는 모든 나이들은 20으로 업데이트 되었지만, [영속성컨텍스트를 무시하기 때문에 트랜잭션 커밋되지 않아도 반영됨]
//find로 영속성 컨텍스트에 있는 값을 조회하면 모두 0임. 싱크가 맞지 않는 현상 발생.    [Member findMember = em.find(Member.class, member1.getId())]
//em.clear()로 영속성 컨텍스트 초기화 반드시 필요

em.clear();

tx.commit();

//=> spring data jpa에서 @Modifying 어노테이션으로 벌크연산이 제공 되는데 이 어노테이션에 영속성 컨텍스트 초기화가 되도록 설정되어있다.
```
### // -- JPQL 관련 내용 끝

------------------------------
###Criteria
• 너무 복잡하고 실용성이 없다. 실무에서 안씀    
• Criteria 대신에 QueryDSL 사용 권장    

###QueryDSL
• ** 실무 사용 권장     
• 문자가 아닌 자바코드로 JPQL을 작성할 수 있음    
• JPQL 빌더 역할      
• *** 컴파일 시점에 문법 오류를 찾을 수 있음 [Q파일이 자동으로 생성됨]    
• 동적쿼리 작성 편리함, 단순하고 쉬움     
• ex)
```java
JPAFactoryQuery query = new JPAQueryFactory(em);    
QMember m = QMember.member;      
List<Member> list = query.selectFrom(m)      
.where(m.age.gt(18))    
.orderBy(m.name.desc())    
.fetch();      
//=> jpql 문법을 알면 QueryDSL은 문서보면 금방 활용 가능
```

###네이티브 SQL      
• JPA가 제공하는 SQL을 직접 사용하는 기능      
• JPQL로 해결할 수 없는 특정 데이터베이스에 의존적인 기능 : 예) 오라클 CONNECT BY, 특정 DB만 사용하는 SQL 힌트     
• ex)    
```java
String sql =  "SELECT ID, AGE, TEAM_ID, NAME FROM MEMBER WHERE NAME = 'kim'";    
List<Member> resultList = em.createNativeQuery(sql, Member.class).getResultList();
```
###JDBC 직접 사용, SpringJdbcTemplate 등
• JPA를 사용하면서 JDBC 커넥션을 직접 사용하거나, 스프링 JdbcTemplate, 마이바티스등을 함께 사용 가능     
• 단 영속성 컨텍스트를 적절한 시점에 강제로 플러시 필요    
>> *** 중요!! *** JPA를 우회해서 SQL을 실행하기 직전에 영속성 컨텍스트 수동 플러시 [em.flush()]    
cf) flush는 트랜잭션 커밋되거나 qury가 날라갈때 flush됨      

-----

cf) 강의 커뮤니티에 좋은 질답이 있어 첨부.. 아래 스크린 샷이 문제 되면 알려주세요!    
출처: 인프런, 자바 ORM 표준 JPA 프로그래밍 - 기본편
![A](imgs/cfscreenshot.PNG)

