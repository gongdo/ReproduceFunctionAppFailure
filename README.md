# ReproduceFunctionAppFailure
Reproduces Function App failure when references Microsoft.Extensions.Logging.Abstraction

## Problem
I want to integrate logging system with `Microsoft.Extensions.Logging.ILogger` in Azure Function App.

But the Function App throws an exception for any request(both POST and GET).
> Exception while executing function: Test -> Method not found: 'System.Net.Http.HttpResponseMessage System.Net.Http.HttpRequestMessageExtensions.CreateResponse(System.Net.Http.HttpRequestMessage, System.Net.HttpStatusCode, !!0)'.

It can be reproduced by reference `Microsoft.Extensions.Logging.Abstraction`.
No other code is needed. Just reference the package, it will broke your Function App.
It can be also reproduced by reference other library that references `Microsoft.Extensions.Logging.Abstraction`.

Funny thing is, `Microsoft.NET.Sdk.Functions (1.0.2)` has reference of `Microsoft.Extensions.Logging.Abstraction (2.0.0)` implicitly via `Microsoft.Azure.WebJobs (2.1.0-beta1)`, but it won't be a problem unless you reference to `Logging.Abstraction` explicitly.

- Request
```
GET http://localhost:7071/api/test?name=gongdo HTTP/1.1

Host: localhost:7071
```

- Response
```
HTTP/1.1 500 Internal Server Error
Content-Type: application/json; charset=utf-8
Server: Microsoft-HTTPAPI/2.0
Date: Tue, 29 Aug 2017 04:44:35 GMT
Content-Length: 4288

{"id":"963cf0ef-2a8a-41aa-826d-11cf5acc13f0","requestId":"3bf437de-b551-4100-96e0-f7550854a2ad","statusCode":500,"errorCode":0,"message":"Exception while executing function: Test -> Method not found: 'System.Net.Http.HttpResponseMessage System.Net.Http.HttpRequestMessageExtensions.CreateResponse(System.Net.Http.HttpRequestMessage, System.Net.HttpStatusCode, !!0)'.","errorDetails":"Microsoft.Azure.WebJobs.Host.FunctionInvocationException : Exception while executing function: Test ---> System.MissingMethodException : Method not found: 'System.Net.Http.HttpResponseMessage System.Net.Http.HttpRequestMessageExtensions.CreateResponse(System.Net.Http.HttpRequestMessage, System.Net.HttpStatusCode, !!0)'.\r\n   at async LoggingFunctionApp.Test.Run(HttpRequestMessage req,TraceWriter log)\r\n   at System.Runtime.CompilerServices.AsyncTaskMethodBuilder`1.Start[TStateMachine](TStateMachine& stateMachine)\r\n   at LoggingFunctionApp.Test.Run(HttpRequestMessage req,TraceWriter log)\r\n   at lambda_method(Closure ,Test ,Object[] )\r\n   at Microsoft.Azure.WebJobs.Host.Executors.TaskMethodInvoker`2.InvokeAsync(TReflected instance,Object[] arguments)\r\n   at async Microsoft.Azure.WebJobs.Host.Executors.FunctionInvoker`2.InvokeAsync[TReflected,TReturnValue](Object[] arguments)\r\n   at async Microsoft.Azure.WebJobs.Host.Executors.FunctionExecutor.InvokeAsync(IFunctionInvoker invoker,ParameterHelper parameterHelper,CancellationTokenSource timeoutTokenSource,CancellationTokenSource functionCancellationTokenSource,Boolean throwOnTimeout,TimeSpan timerInterval,IFunctionInstance instance)\r\n   at async Microsoft.Azure.WebJobs.Host.Executors.FunctionExecutor.ExecuteWithWatchersAsync(IFunctionInstance instance,ParameterHelper parameterHelper,TraceWriter traceWriter,ILogger logger,CancellationTokenSource functionCancellationTokenSource)\r\n   at async Microsoft.Azure.WebJobs.Host.Executors.FunctionExecutor.ExecuteWithLoggingAsync(??)\r\n   at async Microsoft.Azure.WebJobs.Host.Executors.FunctionExecutor.ExecuteWithLoggingAsync(??) \r\n   End of inner exception\r\n   at System.Runtime.ExceptionServices.ExceptionDispatchInfo.Throw()\r\n   at async Microsoft.Azure.WebJobs.Host.Executors.FunctionExecutor.ExecuteWithLoggingAsync(??)\r\n   at async Microsoft.Azure.WebJobs.Host.Executors.FunctionExecutor.TryExecuteAsync(IFunctionInstance functionInstance,CancellationToken cancellationToken)\r\n   at Microsoft.Azure.WebJobs.Host.Executors.ExceptionDispatchInfoDelayedException.Throw()\r\n   at async Microsoft.Azure.WebJobs.JobHost.CallAsync(??)\r\n   at async Microsoft.Azure.WebJobs.Script.ScriptHost.CallAsync(String method,Dictionary`2 arguments,CancellationToken cancellationToken)\r\n   at async Microsoft.Azure.WebJobs.Script.WebHost.WebScriptHostManager.HandleRequestAsync(FunctionDescriptor function,HttpRequestMessage request,CancellationToken cancellationToken)\r\n   at async Microsoft.Azure.WebJobs.Script.WebHost.Controllers.FunctionsController.ProcessRequestAsync(HttpRequestMessage request,FunctionDescriptor function,CancellationToken cancellationToken)\r\n   at async Microsoft.Azure.WebJobs.Script.WebHost.Controllers.FunctionsController.<>c__DisplayClass3_0.<ExecuteAsync>b__0(??)\r\n   at async Microsoft.Azure.WebJobs.Extensions.Http.HttpRequestManager.ProcessRequestAsync(HttpRequestMessage request,Func`3 processRequestHandler,CancellationToken cancellationToken)\r\n   at async Microsoft.Azure.WebJobs.Script.WebHost.Controllers.FunctionsController.ExecuteAsync(HttpControllerContext controllerContext,CancellationToken cancellationToken)\r\n   at async System.Web.Http.Dispatcher.HttpControllerDispatcher.SendAsync(HttpRequestMessage request,CancellationToken cancellationToken)\r\n   at async System.Web.Http.Dispatcher.HttpControllerDispatcher.SendAsync(HttpRequestMessage request,CancellationToken cancellationToken)\r\n   at async Microsoft.Azure.WebJobs.Script.WebHost.Handlers.SystemTraceHandler.SendAsync(HttpRequestMessage request,CancellationToken cancellationToken)\r\n   at async Microsoft.Azure.WebJobs.Script.WebHost.Handlers.WebScriptHostHandler.SendAsync(HttpRequestMessage request,CancellationToken cancellationToken)\r\n   at async System.Web.Http.HttpServer.SendAsync(HttpRequestMessage request,CancellationToken cancellationToken)"}
```

## Solution Setup

### LoggingFunctionApp
- Created by Azure Function App Project Template
- NuGet
  - Function App (Microsoft.NET.Sdk.Functions 1.0.2)
  - Microsoft.Extensions.Logging.Abstraction
- Project
  - LoggingLibraryDummy461
  - LoggingLibraryDummyCore
- Function (Test.cs)
  - A default Function App with Http Request Trigger.

### LoggingLibraryDummy461
- Created by Windows Classic Desktop / `Class Library` / `.NET 461`
- NuGet
  - Microsoft.Extensions.Logging.Abstraction

### LoggingLibraryDummyCore
- Created by `.NET Core` / `Class Library` / `.NET Standard 2.0`
- NuGet
  - Microsoft.Extensions.Logging.Abstraction
