### Overview
* EJBs are server-side components that encapsulate business logic and take care of transactions and security.
* A session bean may have the following traits:
..* Stateless: The session bean contains no conversational state between methods, and any
instance can be used for any client. It is used to handle tasks that can be concluded with
a single method call.
..* Stateful: The session bean contains conversational state, which must be retained across
methods for a single user. It is useful for tasks that have to be done in several steps.
..* Singleton: A single session bean is shared between clients and supports concurrent access.
The container will make sure that only one instance exists for the entire application.
* EJB container provides core services common to many enterprise applications such as the following:
..* Remote client communication: Without writing any complex code, an EJB client (another EJB, a user interface, a batch process, etc.) can invoke methods remotely via standard protocols.
..* Dependency injection: The container can inject several resources into an EJB (JMS destinations
and factories, datasources, other EJBs, environment variables, etc.) as well as any POJO thanks to CDI.
..* State management: For stateful session beans, the container manages their state transparently. You can maintain state for a particular client, as if you were developing a desktop application.
..* Pooling: For stateless beans and MDBs, the container creates a pool of instances that can be shared by multiple clients. Once invoked, an EJB returns to the pool to be reused instead of being destroyed.
..* Component life cycle: The container is responsible for managing the life cycle of each component.
..* Messaging: The container allows MDBs to listen to destinations and consume messages without too much JMS plumbing.
..* Transaction management: With declarative transaction management, an EJB can use annotations to inform the container about the transaction policy it should use. The container takes care of the commit or the rollback.
..* Security: Class or method-level access control can be specified on EJBs to enforce user and role authorization
..* Concurrency support: Except for singletons, where some concurrency declaration is needed, all the other types of EJB are thread-safe by nature. You can develop high-performance applications without worrying about thread issues.
..* Interceptors: Cross-cutting concerns can be put into interceptors, which will be invoked automatically by the container.
..* Asynchronous method invocation: Since EJB 3.1, it’s now possible to have asynchronous calls without involving messaging.
* Containers give EJBs a set of service. On the other hand, EJBs cannot create or manage threads, access files using
java.io, create a ServerSocket, load a native library, or use the AWT (Abstract Window Toolkit)or Swing APIs to
interact with the user.
*  Comparison Between EJB Lite and Full EJB
|Feature|EJB Lite|Full EJB 3.2|
|--- | --- | ---|
|Session beans (stateless, stateful, singleton)| Yes|Yes|
|No-interface view |Yes| Yes|
|Local interface| Yes |Yes|
|Interceptors| Yes |Yes|
|Transaction support Yes |Yes|
|Security |Yes |Yes|
|Embeddable API| Yes| Yes|
|Asynchronous calls |No| Yes|
|MDBs| No| Yes|
|Remote interface| No |Yes|
|JAX-WS web services |No| Yes|
|JAX-RS web services |No |Yes|
|Timer service |No| Yes|
|RMI/IIOP interoperability |No |Yes|


