DispatcherServlet.getHandler(request) 获取处理器

```java
protected HandlerExecutionChain getHandler(HttpServletRequest request) throws Exception {
   if (this.handlerMappings != null) {
      for (HandlerMapping mapping : this.handlerMappings) {
        //从众多的HandlerMapping中找到匹配的HandlerExecutionChain
         HandlerExecutionChain handler = mapping.getHandler(request);
         if (handler != null) {
            return handler;
         }
      }
   }
   return null;
}
```

```java
public final HandlerExecutionChain getHandler(HttpServletRequest request) throws Exception {
		Object handler = getHandlerInternal(request);
		---
        //根据HandlerMethod获取HandlerExecutionChain
		HandlerExecutionChain executionChain = getHandlerExecutionChain(handler, request);
		---
		return executionChain;
	}
```



```java
protected HandlerMethod getHandlerInternal(HttpServletRequest request) throws Exception {
  //从request中获取请求路径
   String lookupPath = getUrlPathHelper().getLookupPathForRequest(request);
   this.mappingRegistry.acquireReadLock();
   try {
     //根据请求路径找匹配的HandlerMethod
      HandlerMethod handlerMethod = lookupHandlerMethod(lookupPath, request);
      return (handlerMethod != null ? handlerMethod.createWithResolvedBean() : null);
   }
   finally {
      this.mappingRegistry.releaseReadLock();
   }
}
```



```java
protected HandlerExecutionChain getHandlerExecutionChain(Object handler, HttpServletRequest request) {
  //将HandlerMethod包装成HandlerExecutionChain
   HandlerExecutionChain chain = (handler instanceof HandlerExecutionChain ?
         (HandlerExecutionChain) handler : new HandlerExecutionChain(handler));
   String lookupPath = this.urlPathHelper.getLookupPathForRequest(request);
  //在HandlerExecutionChain添加HandlerInterceptor
   for (HandlerInterceptor interceptor : this.adaptedInterceptors) {
      if (interceptor instanceof MappedInterceptor) {
         MappedInterceptor mappedInterceptor = (MappedInterceptor) interceptor;
         if (mappedInterceptor.matches(lookupPath, this.pathMatcher)) {
            chain.addInterceptor(mappedInterceptor.getInterceptor());
         }
      }
      else {
         chain.addInterceptor(interceptor);
      }
   }
   return chain;
}
```