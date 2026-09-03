- Spring Boot doesn't use the word "middleware" (that's an Express/Django term), but it has the equivalent via Filters and Interceptors.
- Filter/Interceptor → knows about the HTTP request (URL, headers, body, session).  They are web-layer concerns.
- Here's how the three compare:
  
```Client → [Filter] → DispatcherServlet → [Interceptor] → Controller → [AOP wraps method] → Response``` 

||	Filter|	Interceptor|	AOP Aspect|
|-|-|-|-|
|Layer|	Servlet (before Spring)|	Spring MVC (controller level)|	Any Spring bean method|
|Interface|	jakarta.servlet.Filter|	HandlerInterceptor|	@Aspect + @Around|
|Sees|	Raw HTTP request/response|	Request + which controller method is being called|	Method name, params, return value|
|Scope|	ALL requests (incl.  static files)|	Only Spring MVC controller requests|	Any matched method (service, repo, util…)|
|Can access Spring beans?|	❌ (limited)|	✅	| ✅ |
|Typical use |	CORS, GZIP, auth token parsing, logging|	Role checks, metrics, model/view modification|	Transactions, caching, call counting, logging|
|Example - Log how long a method took| Only for HTTP endpoints as it only sees web requests| Only for controller methods as it only fires on Spring MVC handlers| Any method anywhere as Pointcut can target execution(* com.myapp.service.*.*(..))|
|Example - Add a custom header to every response|  ✅ as direct access to HttpServletResponse | Partially | ❌ |



Rule of Thumb
|Need|	Use|
|-|-|
|Modify raw HTTP request/response (headers, compression, CORS)|	Filter
|Logic tied to which controller method is being called (auth, audit)|	Interceptor
|Logic around any Spring bean method (transactions, metrics, your @CountCalls)|	AOP Aspect



