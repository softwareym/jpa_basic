# JPA 활용1 강의 들으면서 유용한 내용 메모

* 실전! 스프링 부트와 JPA 활용1 - 웹 애플리케이션 개발
---

### gradle - boot + spring data jpa + queryDsl 조합

### <실무에서 사용하지 말기>
>•@ManyToMany x      
>=>  @ManyToMany 는 편리한 것 같지만, 중간 테이블( CATEGORY_ITEM )에 컬럼을 추가할 수 없고, 세밀하게 쿼리를 실행하기 어렵기 때문에 실무에서 사용하기에는 한계가 있다.  
>>중간 엔티티( CategoryItem ) 를 만들고 @ManyToOne , @OneToMany 로 매핑해서 사용하자. 정리하면 대다대 매핑을 일대다, 다대일 매핑으로 풀어내서 사용하자.        

>•@Setter x      
>=> @Getter만 사용하고, @Setter 를 제거하고, 생성자에서 값을 모두 초기화해서 변경 불가능한 클래스를 만들자.          
>JPA 스펙상 엔티티나 임베디드 타입( @Embeddable )은 자바 기본 생성자(default constructor)를 public 또는 protected 로 설정해야 한다.        
>public 으로 두는 것 보다는 protected 로 설정하는 것이 그나마 더 안전하다.     
>>> JPA가 이런 제약을 두는 이유는 JPA 구현 라이브러리가 객체를 생성할 때 리플랙션 같은 기술을 사용할 수 있도록 지원해야 하기 때문이다.      
```java
@Embeddable
@Getter
public class Address {
     private String city;
     private String street;
     private String zipcode;
     
     protected Address() {
     
     }
     
     public Address(String city, String street, String zipcode) {
     this.city = city;
     this.street = street;
     this.zipcode = zipcode;
     }
}
```

>•@Enumerated 이넘 타입 사용시 OrdinaryType 절대 사용 x (나중에 중간에 추가되면 순서가 얽힘), String으로 사용하자         
>>=>	자바 enum 타입을 엔티티 클래스의 속성으로 사용할 수 있다.          
>>@Enumerated 애노테이션에는 두 가지 EnumType이 존재한다.      
>>>-EnumType.ORDINAL : enum 순서 값을 DB에 저장        
>>>-EnumType.STRING : enum 이름을 DB에 저장       

### 모든 연관관계는 지연로딩으로 설정!
>-즉시로딩( EAGER )은 예측이 어렵고, 어떤 SQL이 실행될지 추적하기 어렵다. 특히 JPQL을 실행할 때 N+1 문제가 자주 발생한다.     
(ex-member 테이블을 읽어들일때 연관된 모든 테이블을 다 읽어들임)       
>>=>한건 조회할때는 eager 사용해도 되긴하는데, JPQL [select o From order o;] 실행 시 > SQL [select * from order 100+1(order)]과 같게 되어 100번 읽어 들이는 꼴..     
>>그래서 사용안하는게 낫다.eager 대신 lazy와 fetch join을 함께 사용      
>
>-실무에서 모든 연관관계는 지연로딩( LAZY )으로 설정해야 한다.     
> [-> order에 지연로딩을 하면 order테이블 읽을때 order만 읽음]      
>
>-연관된 엔티티를 함께 DB에서 조회해야 하면, fetch join 또는 엔티티 그래프 기능을 사용한다.      
>
>-@XToOne(OneToOne, ManyToOne) 관계는 기본이 즉시로딩이므로 직접 지연로딩으로 설정해야 한다.        
>>> ** (결론) @XToMany는 기본이 지연로딩(LAZY)이라 그대로 두면 되는데, @XToOne인 경우에는 무!조!건! 즉시로딩(EAGER)이 기본이라 지연로딩(LAZY)으로 설정을 해줘야함       
@ManyToOne, @OnToMany(fetch=LAZY)       

### 컬렉션은 필드에서 초기화 하자.
-컬렉션은 필드에서 바로 초기화 하는 것이 안전하다. null 문제에서 안전하다.           
하이버네이트는 엔티티를 영속화 할 때, 컬랙션을 감싸서 하이버네이트가 제공하는 내장 컬렉션으로 변경한다.      
만약 getOrders() 처럼 임의의 메서드에서 컬력션을 잘못 생성하면 하이버네이트 내부 메커니즘에 문제가 발생할 수 있다. 따라서 필드레벨에서 생성하는 것이 가장 안전하고, 코드도 간결하다.        

ex)
```java
@Getter
@Entity
public class Member{

    @Id @GeneratedValue
    @Column(name="member_id")
    private Long id;
    
    @OneToMany(mappedBy="member", cascade = CasecadeType.ALL)		//=> cascade = CasecadeType.ALL를 추가하면 Member를 저장하면 List<Order>에 한번에 저장한다. (원래는 cascade = CasecadeType.ALL이 없으면 orders의 엔티티만큼 반복해서 저장함)
    private List<Order> orders = new ArrayList<>();			//=>이렇게 필드에서 바로 초기화하여 사용하고. 가급적 절대 변경하지 않는 것이 제일 안전하다

    @OneToOne(fetch = LAZY, cascade = CasecadeType.ALL)
    private Delivery delivery					//=>모든엔티티는 각각 퍼시스트 하고 싶으면 각각 넣어줘야 하는데 cascade = CasecadeType.ALL를 설정함으로써 Member를 넣으면 delivery에도 같이 넣어짐
}
...

Member member = new Member();
System.out.println(member.getOrders().getClass());
em.persist(team);
System.out.println(member.getOrders().getClass());  

...

//출력 결과
class java.util.ArrayList
class org.hibernate.collection.internal.PersistentBag

```

### 테이블, 컬럼명 생성 전략
-스프링 부트에서 하이버네이트 기본 매핑 전략을 변경해서 실제 테이블 필드명은 다름          
cf)

https://docs.spring.io/spring-boot/docs/2.1.3.RELEASE/reference/htmlsingle/#howtoconfigure-hibernate-naming-strategy        
http://docs.jboss.org/hibernate/orm/5.4/userguide/html_single/Hibernate_User_Guide.html#naming      
-하이버네이트 기존 구현: 엔티티의 필드명을 그대로 테이블의 컬럼명으로 사용( SpringPhysicalNamingStrategy )      

-스프링 부트 신규 설정 (엔티티(필드) 테이블(컬럼))     
1. 카멜 케이스 언더스코어(memberPoint member_point)
2. .(점) _(언더스코어)
3. 대문자 소문자

-적용 2 단계
1. 논리명 생성: 명시적으로 컬럼, 테이블명을 직접 적지 않으면 ImplicitNamingStrategy 사용
spring.jpa.hibernate.naming.implicit-strategy : 테이블이나, 컬럼명을 명시하지 않을 때 논리명 적용,
2. 물리명 적용: spring.jpa.hibernate.naming.physical-strategy : 모든 논리명에 적용됨, 실제 테이블에 적용 (username usernm 등으로 회사 룰로 바꿀 수 있음)

-스프링 부트 기본 설정       
>spring.jpa.hibernate.naming.implicit-strategy:	org.springframework.boot.orm.jpa.hibernate.SpringImplicitNamingStrategy     
>spring.jpa.hibernate.naming.physical-strategy:	org.springframework.boot.orm.jpa.hibernate.SpringPhysicalNamingStrate

### 연관관계 메소드 [양방향일때 쓰면 좋음. 한쪽에만 코드 넣고 사용할 수 있음]
ex)
```java
public class Order{
    ....
    private LocalDateTime orderDate;
    ....

    public void setMember(Member member){
        this.member = member;
        member.getOrders().add(this);
    }

    public void addOrderItem(OrderItem orderItem){
        orderItems.add(orderItem);
        orderItem.setOrder(this);
    }

    public void setDelivery(Delivery delivery){
        this.delivery = delivery;
        delivery.setOrder(this);
    }
}
```

-------------
#### *test 영역만 따로 db설정 가능
#### *test에 yml이 있으면 main에 있는 yml은 무시되고 동작
#### *테스트는 h2 inmemory로 처리함
#### *스프링부트는 별도 설정이 없으면 메모리 모드로 돌려버림

> 참고: 주문 서비스의 주문과 주문 취소 메서드를 보면 비즈니스 로직 대부분이 엔티티에 있다.       
서비스 계층은 단순히 엔티티에 필요한 요청을 위임하는 역할을 한다.       
이처럼 엔티티가 비즈니스 로직을 가지고 객체 지향의 특성을 적극 활용하는 것을 도메인 모델 패턴(http://martinfowler.com/eaaCatalog/domainModel.html이라 한다.       
반대로 엔티티에는 비즈니스 로직이 거의 없고 서비스 계층에서 대부분의 비즈니스 로직을 처리하는 것을 트랜잭션 스크립트 패턴(http://martinfowler.com/eaaCatalog/transactionScript.html)이라 한다-sql

#### cf) javax validation
#### cf) thymeleaf.org references
#### cf) getter.setter/  model mapper..?  플러그인..?

#### cf)
>* 화면에 나오는 폼객체(데이터 전송하는 객체인 getter,setter있는 dto는 따로 있고)와        
>    비즈니스 로직을 가지고 있는 엔티티와는 달라야함. 각각 가지고 있어야함!
>>=> api 만들때는 외부에 entity를 절대 반환하면 안됨.       
>>   요청.응답 dto를 따로 가지고 있어야함.        
>> 그렇지 않으면 중간에 엔티티에 로직을 추가할 경우 api	스펙이 변함      
>> [참고 : JPA활용1 > 웹 계층 개발 > 회원 목록 조회 8분~]

-----
### 변경감지와 병합(merge)
-실무에서 merge 안씀      
-엔티티가 영속 상태로 관리가 되는데, 그 어떤 값을 변경하면.. jpa가 트랜잭션 커밋 시점에(@Transactional) 변경된 내용을 알아서 찾아서(flush) 디비에 값을 변경해줌        
=> 이것을 변경감지(dirty checking)라 함.             

-jpa로 디비에 한번 들어갔다 온 애는 준영속성 엔티티(영속성 컨텍스트가 더 이상 관리하지 않는 엔티티)       
> =>준영속성 엔티티는 jpa가 더이상 관리하지 않기 때문에, 값을 변경 하여도 트랜잭션 커밋 시점에 디비 값이 변경이 되지 않음.      
```java
@GetMapping("items/{itemId}/edit")
public String updateItemForm(@PathVariable("itemId") Long itemId, Model model) {
    Book item = (Book) itemService.findOne(itemId);

    //준영속성 엔티티(jpa가 더 이상 관리하지 않음)
    BookForm form = new BookForm();
    form.setId(item.getId());           //이미 jpa가 해당 디비에 들어갔다 나왔다는 증거..id를 가지고 나왔으니까... 준영속 엔티티(영속성 컨텍스트가 더 이상 관리하지 않는 엔티티)
    form.setName(item.getName());
    form.setPrice(item.getPrice());
    form.setStockQuantity(item.getStockQuantity());
    form.setAuthor(item.getAuthor());
    form.setIsbn(item.getIsbn());
    
    model.addAttribute("form", form);
    return "items/updateItemForm";
}
//컨트롤러에서 어설프게 엔티티를 생성하지 말자
//컨트롤러 단에서 엔티티를 사용하기 보다, 서비스층에서 변경감지 및 조회 등을 사용하자.(이유는 컨트롤러단에서는 @Transactional이 없기 때문에 영속성 컨텍스트 x )
```

### 영속성 컨텐스트란?       
-엔티티를 영구 저장하는 환경이라는 뜻이다.            
-애플리케이션과 데이터베이스 사이에서 객체를 보관하는 가상의 데이터베이스 같은 역할을 한다. 엔티티 매니저를 통해 엔티티를 저장하거나 조회하면 엔티티 매니저는 영속성 컨텍스트에 엔티티를 보관하고 관리한다.      
-em.persist(member); 엔티티 매니저를 사용해 회원 엔티티를 영속성 컨텍스트에 저장한다는 의미!       

### 영속성 컨텍스트의 특징             
-엔티티 매니저를 생성할 때 하나 만들어진다.       
-엔티티 매니저를 통해서 영속성 컨텍스트에 접근하고 관리할 수 있다.      
-cf ) https://velog.io/@neptunes032/JPA-%EC%98%81%EC%86%8D%EC%84%B1-%EC%BB%A8%ED%85%8D%EC%8A%A4%ED%8A%B8%EB%9E%80       
