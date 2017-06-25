# Spring-MVC
homework5
前端控制器流程处理关键方法（doDispatch）源码解析：
protected void doDispatch(HttpServletRequest request, HttpServletResponse response) throws Exception {
HttpServletRequest processedRequest = request;
HandlerExecutionChain mappedHandler = null;
boolean multipartRequestParsed = false;

WebAsyncManager asyncManager = WebAsyncUtils.getAsyncManager(request);

try {
ModelAndView mv = null;
Exception dispatchException = null;

try {
// 检查并且包装multipart/form-data请求
processedRequest = checkMultipart(request);
multipartRequestParsed = processedRequest != request;

// 根据请求去匹配HandlerMapping，并且通过HandlerMapping返回HandlerExecutionChain
mappedHandler = getHandler(processedRequest, false);
if (mappedHandler == null || mappedHandler.getHandler() == null) {
noHandlerFound(processedRequest, response);
return;
}

// 根据Handler匹配HandlerAdapter
HandlerAdapter ha = getHandlerAdapter(mappedHandler.getHandler());

// 验证HTTP头部last-modified
String method = request.getMethod();
boolean isGet = "GET".equals(method);
if (isGet || "HEAD".equals(method)) {
long lastModified = ha.getLastModified(request, mappedHandler.getHandler());
if (logger.isDebugEnabled()) {
String requestUri = urlPathHelper.getRequestUri(request);
logger.debug("Last-Modified value for [" + requestUri + "] is: " + lastModified);
}
if (new ServletWebRequest(request, response).checkNotModified(lastModified) && isGet) {
return;
}
}

// 执行前置通知
if (!mappedHandler.applyPreHandle(processedRequest, response)) {
return;
}

try {
// 解析方法参数，注入方法参数，并且执行Handler；最后返回ModelAndView；
mv = ha.handle(processedRequest, response, mappedHandler.getHandler());
}
finally {
if (asyncManager.isConcurrentHandlingStarted()) {
return;
}
}

// 设置默认视图
applyDefaultViewName(request, mv);
// 执行后置通知
mappedHandler.applyPostHandle(processedRequest, response, mv);
}
catch (Exception ex) {
dispatchException = ex;
}
// 处理返回结果：根据视图名匹配视图解析器，返回视图；最后将视图输出；
processDispatchResult(processedRequest, response, mappedHandler, mv, dispatchException);
}
catch (Exception ex) {
// 执行完成通知
triggerAfterCompletion(processedRequest, response, mappedHandler, ex);
}
catch (Error err) {
// 执行完成通知
triggerAfterCompletionWithError(processedRequest, response, mappedHandler, err);
}
finally {
// 处理异步通知
if (asyncManager.isConcurrentHandlingStarted()) {
// Instead of postHandle and afterCompletion
mappedHandler.applyAfterConcurrentHandlingStarted(processedRequest, response);
return;
}
// 清除multipart/form-data资源
if (multipartRequestParsed) {
cleanupMultipart(processedRequest);
}
}
}
