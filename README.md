# 일러두기

This repo is for translating [ddd-by-examples library repo](https://github.com/ddd-by-examples/library) in Korean.  
I don't have any copyrights with this code and information.  
If there's a problem with the copyrights I'll close this repo ASAP.

이 repo는 [ddd-by-examples의 library repo](https://github.com/ddd-by-examples/library)를 fork하여 한국어로 번역하는 repo입니다.  
저에게는 해당 코드 및 지식에 대한 저작권이 없으며 문제 시 이 repo는 닫도록 하겠습니다.

DDD를 공부하고 실천하고자 노력하시는 모든 분들에게 도움이 되었으면 좋겠습니다.

---

# 목차

1. [About](#about)
2. [도메인 설명](#도메인-설명)
3. [General assumptions](#general-assumptions)  
    3.1 [Process discovery](#process-discovery)  
    3.2 [프로젝트 구조와 아키텍쳐](#프로젝트-구조와-아키텍쳐)    
    3.3 [Aggregates](#aggregates)  
    3.4 [Events](#events)  
    3.4.1 [Events in Repositories](#events-in-repositories)   
    3.5 [ArchUnit](#archunit)  
    3.6 [Functional thinking](#functional-thinking)  
    3.7 [No ORM](#no-orm)  
    3.8 [Architecture-code gap](#architecture-code-gap)  
    3.9 [Model-code gap](#model-code-gap)   
    3.10 [Spring](#spring)  
    3.11 [Tests](#tests)  
4. [How to contribute](#how-to-contribute)
5. [References](#references)

## About
[원문](https://github.com/ddd-by-examples/library#about)

이 프로젝트는 실제 [비지니스 요구사항](#도메인-설명)들 기반의 도서관(library) 프로젝트입니다.
우리는 Domain Driven Design, Behavior-Driven Development, Event Storming, User Story Mapping과 강하게 연결된 기술들을 사용합니다.

## 도메인 설명
[원문](https://github.com/ddd-by-examples/library#domain-description)

공공 도서관은 고객들이 여러 도서관 지점들에서 책을 예약할 수 있게 합니다.  
예약 가능한(available) 책들은 오직 한 명의 고객에 의해서만 예약 될 수 있습니다.  
책들은 유포 중(circulating)이거나 제한(restricted)되고, 회수(retrieval)나 사용료가 있을 수 있습니다.  
예약이 제한된 책은 오직 연구원인 고객들에 의해서만 예약이 될 수 있습니다.  
일반 고객은 책 예약이 5권으로 제한되어있지만, 연구원인 고객은 예약 권 수가 무제한으로 허용됩니다.  
기한이 없는(open-ended) 책 예약은 고객이 책을 대여할 때까지 활성화되고, 그 때 완료가 됩니다.  
기한이 있는(closed-ended) 책 예약은 예약이 요청된 이후로 정해진 일자가 지나기 전까지 완료가 되지 않고 만료가 됩니다  
(A closed-ended book hold that is not completed within a fixed number of days after it was requested will expire).  
이에 대한 확인는 하루의 시작에 만료되는 예약의 일일 시트를 확임함으로써 이뤄집니다.  
오직 연구원인 고객이 기한이 없는 예약을 요청할 수 있습니다.  
어느 고객이더라도 책을 2번 이상 연체 대여한 경우 같은 도서관 지점에 예약을 하려고 하면 도서관 지점은 예약을 거절합니다.  
책은 최대 60일 동안 대여할 수 있습니다.  
연체에 대한 확인은 일일 연체 대여 시트를 확인함으로서 이뤄집니다.  
고객은 고객 프로필 확인을 통해 그/그녀의 예약, 대여 등과 상호작용(interact)합니다.  
고객 프로필은 일일 시트와 같이 생겼지만, 정보는 한 명의 고객에게 제한되며 꼭 일일(daily)일 필요는 없습니다.  
현재는 고객이 현재 (취소나 만료된 것이 아닌)예약들과 현재 (연체를 포함하여)대여를 확인할 수 있습니다.  
또한, 그/그녀는 책을 예약하고 예약을 취소할 수 있습니다.  

실제로는 어떻게 고객이 빌릴 수 있는 책들이 어떤 것인지 아냐구요?  
도서관은 자체 책 카탈로그를 가지고 있고 이는 책들은 특정 책 인스턴스들과 함께 더해질 수 있습니다.  
책의 특정 책 인스턴스는 오직 카탈로그 내에 존재하는 ISBN과 일치하는 책이 있는 경우에만 추가될 수 있습니다.  
책은 항상 비어있지 않은 제목과 가격을 갖고 있습니다.  
책 인스턴스를 더할 때, 우리는 책이 유포 중인지 제한되있는지를 결정해야합니다.  
이는 우리가 같은 ISBN의 책이지만 유포된 책 인스턴스와 제한된 책 인스턴스를 갖을 수 있게 합니다 (예를 들면, Restricted로 보관하고 싶은 저자가 서명한 책이 있는 경우).

## General assumptions

### Process discovery

The first thing we started with was domain exploration with the help of Big Picture EventStorming.
The description you found in the previous chapter, landed on our virtual wall:    
![Event Storming Domain description](docs/images/eventstorming-domain-desc.png)   
The EventStorming session led us to numerous discoveries, modeled with the sticky notes:  
![Event Storming Big Picture](docs/images/eventstorming-big-picture.jpg)   
During the session we discovered following definitions:  
![Event Storming Definitions](docs/images/eventstorming-definitions.png)    

This made us think of real life scenarios that might happen. We discovered them described with the help of
the **Example mapping**:  
![Example mapping](docs/images/example-mapping.png)  

This in turn became the base for our *Design Level* sessions, where we analyzed each example:  
![Example mapping](docs/images/eventstorming-design-level.jpg)  

Please follow the links below to get more details on each of the mentioned steps:
- [Big Picture EventStorming](./docs/big-picture.md)
- [Example Mapping](docs/example-mapping.md)
- [Design Level EventStorming](docs/design-level.md)

### 프로젝트 구조와 아키텍쳐
[원문](https://github.com/ddd-by-examples/library#project-structure-and-architecture)

제일 처음에, 너무 프로젝트를 복잡하게 하지 않기 위해, 우리는 각 바운디드 컨텐스트를 분리된 패키지에 할당하기로 결정했습니다. 이는 시스템이 모듈형 모듈리스(modular monolith)임을 뜻합니다.  
아무런 장애물이 없긴하지만, 컨텍스트들을 maven 모듈에 넣거나 결국 마이크로 서비스에 넣습니다.  
(There are no obstacles, though, to put contexts into maven modules or finally into microservices)  
바운디드 컨텍스들은 아키텍쳐 면에서 (다른 컨텍스트들 사이에서) 자율성을 가져야합니다.  
그러므로, 컨텍스트를 캡슐화하는 각 모듈은 문제의 복잡성에 맞춰 모듈 자신의 로컬 아키텍쳐를 가지고 있습니다.  
컨텍스트의 경우, 우리가 진정한 비지니스 로직(**빌려주기(lending)**)을 식별한 곳에서 우린 현실의 (프로젝트의 목적을 위해)단순화된 추상화인 도메인 모델을 도입하고 헥사고날 아키텍쳐를 활용했습니다.  
컨텍스트의 경우, Event Storming 동안 복잡한 도메인 로직이 부족하다 판단되었기 대문에, 우리는 CRUD 형식의 로컬 아키텍쳐를 적용했습니다.

![Architecture](docs/images/architecture-big-picture.png) 

우리가 헥사고날 아키텍쳐에 대해 얘기하자면, 이는 우리가 도메인과 애플리케이션 로직을 프레임워크(와 인프라스트럭쳐)와 분리하도록 합니다.  
이러한 접근을 통해 우린 무엇을 얻을까요?  
먼저, 우린 애플리케이션의 가장 중요한 부분 - **비지니스 로직** - 에 아무런 의존성 스텁을 필요로 하지 않고 단위 테스트를 할 수 있습니다.  
둘째로, 우리는 우리 스스로 코어 기능을 깰 걱정없이 인프라스트럭트 레이어를 수정할 기회를 만듭니다.  
인프라스트럭쳐 레이어에서는 우리는 훌륭한 테스트를 지원하는 아마도 가장 성숙하고 강력한 애플리케이션 프레임워크인 Spring 프레임워크를 집중적으로 사용할 것입니다.  
우리가 어떻게 Spring을 사용하는지에 대한 더 많은 정보는 [여기](#spring)에서 확인합니다.  
우리가 미리 언급했듯이, 아키텍쳐는 Event Storming 세션을 통해 이끌어졌습니다(the architecture was driven by Event Storming sessions).  
식별된 컨텍스트들과 컨텍스트들의 복잡성 외에도, 우린 또한 읽기와 쓰기 모델을 분리하는(CQRS) 결정을 할 수 있습니다.  
예제로 **고객 프로필(Patron Profiles)** 과 **일일 시트(Daily Sheets)** 를 살펴볼 수 있습니다.

### Aggregates
Aggregates discovered during Event Storming sessions communicate with each other with events. There is
a contention, though, should they be consistent immediately or eventually? As aggregates in general
determine business boundaries, eventual consistency sounds like a better choice, but choices in software
are never costless. Providing eventual consistency requires some infrastructural tools, like message broker
or event store. That's why we could (and did) start with immediate consistency.

> Good architecture is the one which postpones all important decisions

... that's why we made it easy to change the consistency model, providing tests for each option, including
basic implementations based on **DomainEvents** interface, which can be adjusted to our needs and
toolset in future. Let's have a look at following examples:

* Immediate consistency
    ```groovy
    def 'should synchronize Patron, Book and DailySheet with events'() {
        given:
            bookRepository.save(book)
        and:
            patronRepo.publish(patronCreated())
        when:
            patronRepo.publish(placedOnHold(book))
        then:
            patronShouldBeFoundInDatabaseWithOneBookOnHold(patronId)
        and:
            bookReactedToPlacedOnHoldEvent()
        and:
            dailySheetIsUpdated()
    }
    
    boolean bookReactedToPlacedOnHoldEvent() {
        return bookRepository.findBy(book.bookId).get() instanceof BookOnHold
    }
    
    boolean dailySheetIsUpdated() {
        return new JdbcTemplate(datasource).query("select count(*) from holds_sheet s where s.hold_by_patron_id = ?",
                [patronId.patronId] as Object[],
                new ColumnMapRowMapper()).get(0)
                .get("COUNT(*)") == 1
    }
    ```
   _Please note that here we are just reading from database right after events are being published_
   
   Simple implementation of the event bus is based on Spring application events:
    ```java
    @AllArgsConstructor
    public class JustForwardDomainEventPublisher implements DomainEvents {
    
        private final ApplicationEventPublisher applicationEventPublisher;
    
        @Override
        public void publish(DomainEvent event) {
            applicationEventPublisher.publishEvent(event);
        }
    }
    ```

* Eventual consistency
    ```groovy
    def 'should synchronize Patron, Book and DailySheet with events'() {
        given:
            bookRepository.save(book)
        and:
            patronRepo.publish(patronCreated())
        when:
            patronRepo.publish(placedOnHold(book))
        then:
            patronShouldBeFoundInDatabaseWithOneBookOnHold(patronId)
        and:
            bookReactedToPlacedOnHoldEvent()
        and:
            dailySheetIsUpdated()
    }
    
    void bookReactedToPlacedOnHoldEvent() {
        pollingConditions.eventually {
            assert bookRepository.findBy(book.bookId).get() instanceof BookOnHold
        }
    }
    
    void dailySheetIsUpdated() {
        pollingConditions.eventually {
            assert countOfHoldsInDailySheet() == 1
        }
    }
    ```
    _Please note that the test looks exactly the same as previous one, but now we utilized Groovy's
    **PollingConditions** to perform asynchronous functionality tests_

    Sample implementation of event bus is following:
    
    ```java
    @AllArgsConstructor
    public class StoreAndForwardDomainEventPublisher implements DomainEvents {
    
        private final JustForwardDomainEventPublisher justForwardDomainEventPublisher;
        private final EventsStorage eventsStorage;
    
        @Override
        public void publish(DomainEvent event) {
            eventsStorage.save(event);
        }
    
        @Scheduled(fixedRate = 3000L)
        @Transactional
        public void publishAllPeriodically() {
            List<DomainEvent> domainEvents = eventsStorage.toPublish();
            domainEvents.forEach(justForwardDomainEventPublisher::publish);
            eventsStorage.published(domainEvents);
        }
    }
    ```

To clarify, we should always aim for aggregates that can handle a business operation atomically
(transactionally if you like), so each aggregate should be as independent and decoupled from other
aggregates as possible. Thus, eventual consistency is promoted. As we already mentioned, it comes
with some tradeoffs, so from the pragmatic point of view immediate consistency is also a choice.
You might ask yourself a question now: _What if I don't have any events yet?_. Well, a pragmatic
approach would be to encapsulate the communication between aggregates in a _Service-like_ class,
where you could call proper aggregates line by line explicitly.

### Events
Talking about inter-aggregate communication, we must remember that events reduce coupling, but don't remove
it completely. Thus, it is very vital to share(publish) only those events, that are necessary for other
aggregates to exist and function. Otherwise there is a threat that the level of coupling will increase
introducing **feature envy**, because other aggregates might start using those events to perform actions
they are not supposed to perform. A solution to this problem could be the distinction of domain events
and integration events, which will be described here soon.  

### Events in Repositories 
Repositories are one of the most popular design pattern. They abstract our domain model from data layer. 
In other words, they deal with state. That said, a common use-case is when we pass a new state to our repository,
so that it gets persisted. It may look like so:

```java
public class BusinessService {
   
    private final PatronRepository patronRepository;
    
    void businessMethod(PatronId patronId) {
        Patron patron = patronRepository.findById(patronId);
        //do sth
        patronRepository.save(patron);
    }
}
```

Conceptually, between 1st and 3rd line of that business method we change state of our Patron from A to B. 
This change might be calculated by dirty checking or we might just override entire Patron state in the database. 
Third option is _Let's make implicit explicit_ and actually call this state change A->B an **event**. 
After all, event-driven architecture is all about promoting state changes as domain events.

Thanks to this our domain model may become immutable and just return events as results of invoking a command like so:

```java
public BookPlacedOnHold placeOnHold(AvailableBook book) {
      ...
}
```

And our repository might operate directly on events like so:

```java
public interface PatronRepository {
     void save(PatronEvent event) {
}
```

### ArchUnit

One of the main components of a successful project is technical leadership that lets the team go in the right
direction. Nevertheless, there are tools that can support teams in keeping the code clean and protect the
architecture, so that the project won't become a Big Ball of Mud, and thus will be pleasant to develop and
to maintain. The first option, the one we proposed, is [ArchUnit](https://www.archunit.org/) - a Java architecture
test tool. ArchUnit lets you write unit tests of your architecture, so that it is always consistent with initial
vision. Maven modules could be an alternative as well, but let's focus on the former.

In terms of hexagonal architecture, it is essential to ensure, that we do not mix different levels of
abstraction (hexagon levels):
```java 
@ArchTest
public static final ArchRule model_should_not_depend_on_infrastructure =
    noClasses()
        .that()
        .resideInAPackage("..model..")
        .should()
        .dependOnClassesThat()
        .resideInAPackage("..infrastructure..");
```      
and that frameworks do not affect the domain model  
```java
@ArchTest
public static final ArchRule model_should_not_depend_on_spring =
    noClasses()
        .that()
        .resideInAPackage("..io.pillopl.library.lending..model..")
        .should()
        .dependOnClassesThat()
        .resideInAPackage("org.springframework..");
```    

### Functional thinking
When you look at the code you might find a scent of functional programming. Although we do not follow
a _clean_ FP, we try to think of business processes as pipelines or workflows, utilizing functional style through
following concepts.

_Please note that this is not a reference project for FP._

#### Immutable objects
Each class that represents a business concept is immutable, thanks to which we:
* provide full encapsulation and objects' states protection,
* secure objects for multithreaded access,
* control all side effects much clearer. 

#### Pure functions
We model domain operations, discovered in Design Level Event Storming, as pure functions, and declare them in
both domain and application layers in the form of Java's functional interfaces. Their implementations are placed
in infrastructure layer as ordinary methods with side effects. Thanks to this approach we can follow the abstraction
of ubiquitous language explicitly, and keep this abstraction implementation-agnostic. As an example, you could have
a look at `FindAvailableBook` interface and its implementation:

```java
@FunctionalInterface
public interface FindAvailableBook {

    Option<AvailableBook> findAvailableBookBy(BookId bookId);
}
```

```java
@AllArgsConstructor
class BookDatabaseRepository implements FindAvailableBook {

    private final JdbcTemplate jdbcTemplate;

    @Override
    public Option<AvailableBook> findAvailableBookBy(BookId bookId) {
        return Match(findBy(bookId)).of(
                Case($Some($(instanceOf(AvailableBook.class))), Option::of),
                Case($(), Option::none)
        );
    }  

    Option<Book> findBy(BookId bookId) {
        return findBookById(bookId)
                .map(BookDatabaseEntity::toDomainModel);
    }

    private Option<BookDatabaseEntity> findBookById(BookId bookId) {
        return Try
                .ofSupplier(() -> of(jdbcTemplate.queryForObject("SELECT b.* FROM book_database_entity b WHERE b.book_id = ?",
                                      new BeanPropertyRowMapper<>(BookDatabaseEntity.class), bookId.getBookId())))
                .getOrElse(none());
    }  
} 
```
    
#### Type system
_Type system - like_ modelling - we modelled each domain object's state discovered during EventStorming as separate
classes: `AvailableBook`, `BookOnHold`, `CheckedOutBook`. With this approach we provide much clearer abstraction than
having a single `Book` class with an enum-based state management. Moving the logic to these specific classes brings
Single Responsibility Principle to a different level. Moreover, instead of checking invariants in every business method
we leave the role to the compiler. As an example, please consider following scenario: _you can place on hold only a book
that is currently available_. We could have done it in a following way:
```java
public Either<BookHoldFailed, BookPlacedOnHoldEvents> placeOnHold(Book book) {
  if (book.status == AVAILABLE) {  
      ...
  }
}
```
but we use the _type system_ and declare method of following signature
```java
public Either<BookHoldFailed, BookPlacedOnHoldEvents> placeOnHold(AvailableBook book) {
      ...
}
```  
The more errors we discover at compile time the better.

Yet another advantage of applying such type system is that we can represent business flows and state transitions
with functions much easier. As an example, following functions:
```
placeOnHold: AvailableBook -> BookHoldFailed | BookPlacedOnHold
cancelHold: BookOnHold -> BookHoldCancelingFailed | BookHoldCanceled
``` 
are much more concise and descriptive than these:
```
placeOnHold: Book -> BookHoldFailed | BookPlacedOnHold
cancelHold: Book -> BookHoldCancelingFailed | BookHoldCanceled
```
as here we have a lot of constraints hidden within function implementations.

Moreover if you think of your domain as a set of operations (functions) that are being executed on business objects
(aggregates) you don't think of any execution model (like async processing). It is fine, because you don't have to.
Domain functions are free from I/O operations, async, and other side-effects-prone things, which are put into the
infrastructure layer. Thanks to this, we can easily test them without mocking mentioned parts. 

#### Monads
Business methods might have different results. One might return a value or a `null`, throw an exception when something
unexpected happens or just return different objects under different circumstances. All those situations are typical
to object-oriented languages like Java, but do not fit into functional style. We are dealing with this issues
with monads (monadic containers provided by [Vavr](https://www.vavr.io)):
* When a method returns optional value, we use the `Option` monad:

    ```java
    Option<Book> findBy(BookId bookId) {
        ...
    }
    ```

* When a method might return one of two possible values, we use the `Either` monad:

    ```java
    Either<BookHoldFailed, BookPlacedOnHoldEvents> placeOnHold(AvailableBook book) {
        ...
    }
    ```

* When an exception might occur, we use `Try` monad:

    ```java
    Try<Result> placeOnHold(@NonNull PlaceOnHoldCommand command) {
        ...
    }
    ```

Thanks to this, we can follow the functional programming style, but we also enrich our domain language and
make our code much more readable for the clients.

#### Pattern Matching
Depending on a type of a given book object we often need to perform different actions. Series of if/else or switch/case statements
could be a choice, but it is the pattern matching that provides the most conciseness and flexibility. With the code
like below we can check numerous patterns against objects and access their constituents, so our code has a minimal dose
of language-construct noise:
```java
private Book handleBookPlacedOnHold(Book book, BookPlacedOnHold bookPlacedOnHold) {
    return API.Match(book).of(
        Case($(instanceOf(AvailableBook.class)), availableBook -> availableBook.handle(bookPlacedOnHold)),
        Case($(instanceOf(BookOnHold.class)), bookOnHold -> raiseDuplicateHoldFoundEvent(bookOnHold, bookPlacedOnHold)),
        Case($(), () -> book)
    );
}
```

### (No) ORM
[원문](https://github.com/ddd-by-examples/library#no-orm)

`mvn dependency:tree`를 실행하면 어떤 JPA 구현체도 찾을 수 없을 겁니다.  
우린 (Hibernate 같은) ORM 솔루션들이 매우 강력하고 유용하다고 생각함에도 불구하고, 그것들을 사용하지 않기로 하였습니다, 우리가 그것의 기능들을 사용하지 않을 것처럼 말이죠.  
어떤 기능들이냐구요?  
Lazy loading, caching, dirty checking 입니다. 우리가 왜 그것들이 필요하죠?  
우린 SQL 쿼리 위에서(over SQL queries) 더 많은 제어권를 갖고, 우리 스스로 객체와 관련된 임피던스 부정합을(impedance mismatch) 최소화하길 원합니다.  
더 나아가, 변하지 않는 것들을 보호하기 위해 필요되는 것만큼의 작은 데이터를 가지는(containing as little data as it is required to protect the invariants) 상대적으로 크기가 작은 애그리거트들 덕분에, 우린 lazy loading 메커니즘 또한 필요하지 않습니다.  
헥사고날 아키텍쳐를 통해 우리는 도메인과 영속성 모델을 분리하고 독립적으로 이것들을 테스트할 수 있는 능력을 가집니다.  
더 나아가, 우리는 또한 다른 애그리거트들에 다른 영속성 전략을 소개할수 있습니다.  
이 프로젝트에서, 우리는 단순한(plain) SQL 쿼리들과 `JdbcTemplate`을 활용하고 (전에 언급했던 JPA와 관련된 오버헤드로부터 자유로운) Spring Data JDBC라 불리는 새롭고 매우 유망한(promising) 프로젝트를 사용할 것입니다.  
아래 repository의 예제를 살펴보세요:

```java
interface PatronEntityRepository extends CrudRepository<PatronDatabaseEntity, Long> {

    @Query("SELECT p.* FROM patron_database_entity p where p.patron_id = :patronId")
    PatronDatabaseEntity findByPatronId(@Param("patronId") UUID patronId);

}
```

동시에 우린 단순한 SQL 쿼리들과 `JdbcTemplate`을 사용하여 애그리거트를 영속화하는 다른 방식을 제안할 수 있습니다.

```java
@AllArgsConstructor
class BookDatabaseRepository implements BookRepository, FindAvailableBook, FindBookOnHold {

    private final JdbcTemplate jdbcTemplate;

    @Override
    public Option<Book> findBy(BookId bookId) {
        return findBookById(bookId)
                .map(BookDatabaseEntity::toDomainModel);
    }

    private Option<BookDatabaseEntity> findBookById(BookId bookId) {
        return Try
                .ofSupplier(() -> of(jdbcTemplate.queryForObject("SELECT b.* FROM book_database_entity b WHERE b.book_id = ?",
                                     new BeanPropertyRowMapper<>(BookDatabaseEntity.class), bookId.getBookId())))
                .getOrElse(none());
    }
    
    ...
}
```

_주의하세요: 애그리거트를 위한 다른 영속화 구현체들을 고를 수 있는 재량이 있더라도, 앱/팀 내에서 하나의 옵션을 계속 가져가는 것을(stick) 추천합니다_

### Architecture-code gap
We put a lot of attention to keep the consistency between the overall architecture (including diagrams)
and the code structure. Having identified bounded contexts we could organize them in modules (packages, to
be more specific). Thanks to this we gain the famous microservices' autonomy, while having a monolithic
application. Each package has well defined public API, encapsulating all implementation details by using
package-protected or private scopes.

Just by looking at the package structure:

```
└── library
    ├── catalogue
    ├── commons
    │   ├── aggregates
    │   ├── commands
    │   └── events
    │       └── publisher
    └── lending
        ├── book
        │   ├── application
        │   ├── infrastructure
        │   └── model
        ├── dailysheet
        │   ├── infrastructure
        │   └── model
        ├── librarybranch
        │   └── model
        ├── patron
        │   ├── application
        │   ├── infrastructure
        │   └── model
        └── patronprofile
            ├── infrastructure
            ├── model
            └── web
```
you can see that the architecture is screaming that it has two bounded contexts: **catalogue**
and **lending**. Moreover, the **lending context** is built around five business objects: **book**,
**dailysheet**, **librarybranch**, **patron**, and **patronprofile**, while **catalogue** has no subpackages,
which suggests that it might be a CRUD with no complex logic inside. Please find the architecture diagram
below.

![Component diagram](docs/c4/component-diagram.png)

Yet another advantage of this approach comparing to packaging by layer for example is that in order to 
deliver a functionality you would usually need to do it in one package only, which is the aforementioned
autonomy. This autonomy, then, could be transferred to the level of application as soon as we split our
_context-packages_ into separate microservices. Following this considerations, autonomy can be given away
to a product team that can take care of the whole business area end-to-end.

### Model-code gap
In our project we do our best to reduce _model-code gap_ to bare minimum. It means we try to put equal attention
to both the model and the code and keep them consistent. Below you will find some examples.

#### Placing on hold
![Placing on hold](docs/images/placing_on_hold.jpg)

Starting with the easiest part, below you will find the model classes corresponding to depicted command and events:

```java
@Value
class PlaceOnHoldCommand {
    ...
}
```
```java
@Value
class BookPlacedOnHold implements PatronEvent {
    ...
}
```
```java
@Value
class MaximumNumberOfHoldsReached implements PatronEvent {
    ...    
}
```
```java
@Value
class BookHoldFailed implements PatronEvent {
    ...
}
```

We know it might not look impressive now, but if you have a look at the implementation of an aggregate,
you will see that the code reflects not only the aggregate name, but also the whole scenario of `PlaceOnHold` 
command handling. Let us uncover the details:

```java
public class Patron {

    public Either<BookHoldFailed, BookPlacedOnHoldEvents> placeOnHold(AvailableBook book) {
        return placeOnHold(book, HoldDuration.openEnded());
    }
    
    ...
}    
```

The signature of `placeOnHold` method screams, that it is possible to place a book on hold only when it
is available (more information about protecting invariants by compiler you will find in [Type system section](#type-system)).
Moreover, if you try to place available book on hold it can **either** fail (`BookHoldFailed`) or produce some events -
what events?

```java
@Value
class BookPlacedOnHoldEvents implements PatronEvent {
    @NonNull UUID eventId = UUID.randomUUID();
    @NonNull UUID patronId;
    @NonNull BookPlacedOnHold bookPlacedOnHold;
    @NonNull Option<MaximumNumberOfHoldsReached> maximumNumberOfHoldsReached;

    @Override
    public Instant getWhen() {
        return bookPlacedOnHold.when;
    }

    public static BookPlacedOnHoldEvents events(BookPlacedOnHold bookPlacedOnHold) {
        return new BookPlacedOnHoldEvents(bookPlacedOnHold.getPatronId(), bookPlacedOnHold, Option.none());
    }

    public static BookPlacedOnHoldEvents events(BookPlacedOnHold bookPlacedOnHold, MaximumNumberOfHoldsReached maximumNumberOfHoldsReached) {
        return new BookPlacedOnHoldEvents(bookPlacedOnHold.patronId, bookPlacedOnHold, Option.of(maximumNumberOfHoldsReached));
    }

    public List<DomainEvent> normalize() {
        return List.<DomainEvent>of(bookPlacedOnHold).appendAll(maximumNumberOfHoldsReached.toList());
    }
}
```

`BookPlacedOnHoldEvents` is a container for `BookPlacedOnHold` event, and - if patron has 5 book placed on hold already -
`MaximumNumberOfHoldsReached` (please mind the `Option` monad). You can see now how perfectly the code reflects
the model.

It is not everything, though. In the picture above you can also see a big rectangular yellow card with rules (policies)
that define the conditions that need to be fulfilled in order to get the given result. All those rules are implemented 
as functions **either** allowing or rejecting the hold:

![Restricted book policy](docs/images/placing-on-hold-policy-restricted.png)
```java
PlacingOnHoldPolicy onlyResearcherPatronsCanHoldRestrictedBooksPolicy = (AvailableBook toHold, Patron patron, HoldDuration holdDuration) -> {
    if (toHold.isRestricted() && patron.isRegular()) {
        return left(Rejection.withReason("Regular patrons cannot hold restricted books"));
    }
    return right(new Allowance());
};
```

![Overdue checkouts policy](docs/images/placing-on-hold-policy-overdue.png)

```java
PlacingOnHoldPolicy overdueCheckoutsRejectionPolicy = (AvailableBook toHold, Patron patron, HoldDuration holdDuration) -> {
    if (patron.overdueCheckoutsAt(toHold.getLibraryBranch()) >= OverdueCheckouts.MAX_COUNT_OF_OVERDUE_RESOURCES) {
        return left(Rejection.withReason("cannot place on hold when there are overdue checkouts"));
    }
    return right(new Allowance());
};
```

![Max number of holds policy](docs/images/placing-on-hold-policy-max.png)

```java
PlacingOnHoldPolicy regularPatronMaximumNumberOfHoldsPolicy = (AvailableBook toHold, Patron patron, HoldDuration holdDuration) -> {
    if (patron.isRegular() && patron.numberOfHolds() >= PatronHolds.MAX_NUMBER_OF_HOLDS) {
        return left(Rejection.withReason("patron cannot hold more books"));
    }
    return right(new Allowance());
};
```

![Open ended hold policy](docs/images/placing-on-hold-policy-open-ended.png)

```java
PlacingOnHoldPolicy onlyResearcherPatronsCanPlaceOpenEndedHolds = (AvailableBook toHold, Patron patron, HoldDuration holdDuration) -> {
    if (patron.isRegular() && holdDuration.isOpenEnded()) {
        return left(Rejection.withReason("regular patron cannot place open ended holds"));
    }
    return right(new Allowance());
};
```

#### Spring
Spring Framework seems to be the most popular Java framework ever used. Unfortunately it is also quite common
to overuse its features in the business code. What you find in this project is that the domain packages
are fully focused on modelling business problems, and are free from any DI, which makes it easy to
unit-test it which is invaluable in terms of code reliability and maintainability. It does not mean,
though, that we do not use Spring Framework - we do. Below you will find some details:
- Each bounded context has its own independent application context. It means that we removed the runtime
coupling, which is a step towards extracting modules (and microservices). How did we do that? Let's have
a look:
    ```java
    @SpringBootConfiguration
    @EnableAutoConfiguration
    public class LibraryApplication {
    
        public static void main(String[] args) {
            new SpringApplicationBuilder()
                    .parent(LibraryApplication.class)
                    .child(LendingConfig.class).web(WebApplicationType.SERVLET)
                    .sibling(CatalogueConfiguration.class).web(WebApplicationType.NONE)
                    .run(args);
        }
    }
    ```
- As you could see above, we also try not to use component scan wherever possible. Instead we utilize
`@Configuration` classes where we define module specific beans in the infrastructure layer. Those
configuration classes are explicitly declared in the main application class.

### Tests
Tests are written in a BDD manner, expressing stories defined with Example Mapping.
It means we utilize both TDD and Domain Language discovered with Event Storming. 

We also made an effort to show how to create a DSL, that enables to write
tests as if they were sentences taken from the domain descriptions. Please
find an example below:

```groovy
def 'should make book available when hold canceled'() {
    given:
        BookDSL bookOnHold = aCirculatingBook() with anyBookId() locatedIn anyBranch() placedOnHoldBy anyPatron()
    and:
        PatronEvent.BookHoldCanceled bookHoldCanceledEvent = the bookOnHold isCancelledBy anyPatron()

    when:
        AvailableBook availableBook = the bookOnHold reactsTo bookHoldCanceledEvent
    then:
        availableBook.bookId == bookOnHold.bookId
        availableBook.libraryBranch == bookOnHold.libraryBranchId
        availableBook.version == bookOnHold.version
}
``` 
_Please also note the **when** block, where we manifest the fact that books react to 
cancellation event_

## How to contribute

The project is still under construction, so if you like it enough to collaborate, just let us
know or simply create a Pull Request.


## How to Build

### Requirements

* Java 11
* Maven

### Quickstart

You can run the library app by simply typing the following:

```console
$ mvn spring-boot:run
...
...
2019-04-03 15:55:39.162  INFO 18957 --- [           main] o.s.b.a.e.web.EndpointLinksResolver      : Exposing 2 endpoint(s) beneath base path '/actuator'
2019-04-03 15:55:39.425  INFO 18957 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat started on port(s): 8080 (http) with context path ''
2019-04-03 15:55:39.428  INFO 18957 --- [           main] io.pillopl.library.LibraryApplication    : Started LibraryApplication in 5.999 seconds (JVM running for 23.018)

```

### Build a Jar package

You can build a jar with maven like so:

```console
$ mvn clean package
...
...
[INFO] Building jar: /home/pczarkowski/development/spring/library/target/library-0.0.1-SNAPSHOT.jar
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
```

### Build with Docker

If you've already built the jar file you can run:

```console
docker build -t spring/library .
```

Otherwise you can build the jar file using the multistage dockerfile:

```console
docker build -t spring/library -f Dockerfile.build .
```

Either way once built you can run it like so:

```console
$ docker run -ti --rm --name spring-library -p 8080:8080 spring/library
```

### Production ready metrics and visualization
To run the application as well as Prometheus and Grafana dashboard for visualizing metrics you can run all services:

```console
$ docker-compose up
```

If everything goes well, you can access the following services at given location:
* http://localhost:8080/actuator/prometheus - published Micrometer metrics
* http://localhost:9090 - Prometheus dashboard
* http://localhost:3000 - Grafana dashboard

In order to see some metrics, you must create a dashboard. Go to `Create` -> `Import` and select attached `jvm-micrometer_rev8.json`. File has been pulled from 
`https://grafana.com/grafana/dashboards/4701`.

Please note application will be run with `local` Spring profile to setup some initial data.

## References

1. [Introducing EventStorming](https://leanpub.com/introducing_eventstorming) by Alberto Brandolini
2. [Domain Modelling Made Functional](https://pragprog.com/book/swdddf/domain-modeling-made-functional) by Scott Wlaschin
3. [Software Architecture for Developers](https://softwarearchitecturefordevelopers.com) by Simon Brown
4. [Clean Architecture](https://www.amazon.com/Clean-Architecture-Craftsmans-Software-Structure/dp/0134494164) by Robert C. Martin
5. [Domain-Driven Design: Tackling Complexity in the Heart of Software](https://www.amazon.com/Domain-Driven-Design-Tackling-Complexity-Software/dp/0321125215) by Eric Evans
