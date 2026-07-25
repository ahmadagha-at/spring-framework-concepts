# Part 1: Inversion of Control and Dependency Injection

Inversion of Control and Dependency Injection are the central ideas behind Spring Core. They describe who creates application objects and who connects those objects to their dependencies.

## The problem with manual object creation

Consider a service that needs a repository:

```java
public class UserService {

    private final UserRepository repository = new UserRepository();
}
```

`UserService` creates its own dependency. This creates several problems:

- the service is tightly coupled to one repository implementation
- replacing the repository requires changing the service
- isolated unit testing becomes harder
- object creation is spread throughout the application

The class is responsible for both its business behavior and the construction of its collaborators.

## Inversion of Control

Inversion of Control, or IoC, moves responsibility for creating and connecting objects away from the application classes.

With Spring, the application declares which objects are required. The Spring IoC container creates those objects and supplies their dependencies.

```text
Without IoC:
Application object → creates its dependencies

With IoC:
Spring container → creates objects and injects their dependencies
```

The control is “inverted” because the class no longer controls the construction of its object graph.

## Dependency Injection

Dependency Injection is the mechanism Spring commonly uses to implement IoC.

Instead of constructing a repository, the service declares that it needs one:

```java
@Service
public class UserService {

    private final UserRepository repository;

    public UserService(UserRepository repository) {
        this.repository = repository;
    }
}
```

Spring performs the following work:

1. discovers or registers `UserRepository`
2. creates the repository bean
3. discovers `UserService`
4. finds its constructor dependency
5. supplies the repository when creating the service

The service depends on a usable repository, but does not decide how that repository is constructed.

## Constructor injection

Constructor injection is generally the preferred approach for required dependencies:

```java
@Service
public class OrderService {

    private final PaymentClient paymentClient;
    private final OrderRepository orderRepository;

    public OrderService(
            PaymentClient paymentClient,
            OrderRepository orderRepository
    ) {
        this.paymentClient = paymentClient;
        this.orderRepository = orderRepository;
    }
}
```

Its advantages include:

- required dependencies are explicit
- fields can be `final`
- the object cannot be created in an incomplete state
- tests can construct the class without starting Spring
- circular dependencies are exposed early

When a Spring bean has one constructor, `@Autowired` is normally unnecessary.

## Setter injection

Setter injection can be useful for optional or replaceable dependencies:

```java
@Service
public class ReportService {

    private Formatter formatter;

    @Autowired
    public void setFormatter(Formatter formatter) {
        this.formatter = formatter;
    }
}
```

However, required dependencies are less obvious and the object can temporarily exist without them. Constructor injection is usually clearer for mandatory collaborators.

## Field injection

Field injection is concise:

```java
@Autowired
private UserRepository repository;
```

It is generally discouraged for application code because dependencies are hidden, fields cannot easily be `final`, and plain unit tests require reflection or a Spring context.

## Depend on abstractions

Dependency Injection becomes especially useful when classes depend on interfaces:

```java
public interface NotificationSender {
    void send(String message);
}
```

```java
@Component
public class EmailNotificationSender
        implements NotificationSender {

    @Override
    public void send(String message) {
        // send email
    }
}
```

```java
@Service
public class NotificationService {

    private final NotificationSender sender;

    public NotificationService(NotificationSender sender) {
        this.sender = sender;
    }
}
```

The service does not depend directly on email delivery. A different implementation can be supplied for another environment or test.

## IoC is broader than DI

IoC is the general design principle. Dependency Injection is one technique used to achieve it.

Spring also controls other parts of the application, including:

- bean creation and destruction
- proxy creation for transactions and security
- event delivery
- configuration and resource access
- web request routing through specialized modules

## Key takeaway

IoC determines who controls object creation. Dependency Injection determines how an object receives its collaborators.

```text
IoC → Spring controls object creation
DI  → Spring supplies object dependencies
```

Together, they make application components more modular, replaceable, and testable.

## References

- [Spring Framework: Introduction to the IoC Container and Beans](https://docs.spring.io/spring-framework/reference/core/beans/introduction.html)
- [Spring Framework: Dependency Injection](https://docs.spring.io/spring-framework/reference/core/beans/dependencies/factory-collaborators.html)
- [GeeksforGeeks: Spring Tutorial](https://www.geeksforgeeks.org/advance-java/spring/)
