Scopes
===

CDI defines the following built-in scopes and even gives you 
extension points so you can create your own:

* Application scope (@ApplicationScoped): Spans for the entire duration of an application. 
The bean is created only once for the duration of the application and is discarded when the 
application is shut down. This scope is useful for utility or helper classes, or objects that store 
data shared by the entire application (but you should be careful about concurrency issues 
when the data have to be accessed by several threads).
* Session scope (@SessionScoped): Spans across several HTTP requests or several method 
invocations for a single user’s session. The bean is created for the duration of an HTTP session 
and is discarded when the session ends. This scope is for objects that are needed throughout 
the session such as user preferences or login credentials.
* Request scope (@RequestScoped): Corresponds to a single HTTP request or a method 
invocation. The bean is created for the duration of the method invocation and is discarded 
when the method ends. It is used for service classes or JSF backing beans that are only needed 
for the duration of an HTTP request.
* Conversation scope (@ConversationScoped): Spans between multiple invocations within 
the session boundaries with starting and ending points determined by the application. 
Conversations are used across multiple pages as part of a multistep workflow.
* Dependent pseudo-scope (@Dependent): The life cycle is same as that the client. A dependent 
bean is created each time it is injected and the reference is removed when the injection target 
is removed. This is the default scope for CDI. If a scope is not explicitly specified, then the bean belongs to the dependent pseudo-scope (@Dependent). Beans with this scope are never shared between different clients or different injection points. They are dependent on some other bean, which means their life cycle is bound to the life cycle of that bean.
