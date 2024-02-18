# (Korean) Amazon ECS 및 AWS Fargate의 .NET 워크로드 - 모듈 3

[개발자 센터 / AWS 기반 .NET / Amazon ECS 및 AWS Fargate의 .NET 워크로드 - 모듈 3](https://aws.amazon.com/ko/developer/language/net/badges-and-training/ecs-fargate/module-three/) 에 잘 설명된 핸즈온랩을 직접 수행하는데 도움되는 정보를 담았습니다.

---
**NOTE**

아래 내용은 2024년 2월 직접 실습하면서 정리한 내용입니다. 이후 변경되는 내용을 반영하지 못할 수 있으니 이 점 참고하여 살펴보시기 바랍니다.

(Views are my own - below content is based on my self hands-on lab experience and might not reflect further changes.)

---

## 기본 정보 및 폴더 설명

본 내용 실습은 .NET 6 기반입니다. 본인이 실습할 때 설치되었던 `dotnet` 버전은 `6.0.419` 입니다.

- [data](data/): DynamoDB에 추가할 테이블 데이터 (Seattle 데이터 포함)
- [WeatherAPI](WeatherAPI/): WeatherAPI 프로젝트
- [WeatherSite](WeatherSite/): WeatherSite 프로젝트

## WeatherAPI 프로젝트

- 컨트롤러 코드까지 완성 후 `dotnet run` 또는 Visual Studio Code에서 실행하니 런타임 오류가 발생하였습니다.
- 이를 해결하기 위해 다음과 같이 코드를 수정하였더니 정상 동작하였습니다.
    - [WeatherAPI/Controllers/WeatherForecastController.cs](https://github.com/ianychoi/aws-ecs-fargate-dotnet-module3-supplementary/commit/b04ef2bbf31db0cbc0dd0114d16a527ac6cb20b0)
- 발생한 오류 메시지는 아래를 참고하세요.
    ```
    An unhandled exception occurred while processing the request.
    InvalidOperationException: Unable to resolve service for type 'Microsoft.Extensions.Logging.ILogger' while attempting to activate 'WeatherAPI.Controllers.WeatherForecastController'.
    Microsoft.Extensions.DependencyInjection.ActivatorUtilities.GetService(IServiceProvider sp, Type type, Type requiredBy, bool isDefaultParameterRequired)

    Stack Query Cookies Headers Routing
    InvalidOperationException: Unable to resolve service for type 'Microsoft.Extensions.Logging.ILogger' while attempting to activate 'WeatherAPI.Controllers.WeatherForecastController'.
    Microsoft.Extensions.DependencyInjection.ActivatorUtilities.GetService(IServiceProvider sp, Type type, Type requiredBy, bool isDefaultParameterRequired)
    lambda_method3(Closure , IServiceProvider , object[] )
    Microsoft.AspNetCore.Mvc.Controllers.ControllerActivatorProvider+<>c__DisplayClass7_0.<CreateActivator>b__0(ControllerContext controllerContext)
    Microsoft.AspNetCore.Mvc.Controllers.ControllerFactoryProvider+<>c__DisplayClass6_0.<CreateControllerFactory>g__CreateController|0(ControllerContext controllerContext)
    Microsoft.AspNetCore.Mvc.Infrastructure.ControllerActionInvoker.Next(ref State next, ref Scope scope, ref object state, ref bool isCompleted)
    Microsoft.AspNetCore.Mvc.Infrastructure.ControllerActionInvoker.InvokeInnerFilterAsync()
    Microsoft.AspNetCore.Mvc.Infrastructure.ResourceInvoker.<InvokeFilterPipelineAsync>g__Awaited|20_0(ResourceInvoker invoker, Task lastTask, State next, Scope scope, object state, bool isCompleted)
    Microsoft.AspNetCore.Mvc.Infrastructure.ResourceInvoker.<InvokeAsync>g__Awaited|17_0(ResourceInvoker invoker, Task task, IDisposable scope)
    Microsoft.AspNetCore.Mvc.Infrastructure.ResourceInvoker.<InvokeAsync>g__Awaited|17_0(ResourceInvoker invoker, Task task, IDisposable scope)
    Microsoft.AspNetCore.Routing.EndpointMiddleware.<Invoke>g__AwaitRequestTask|6_0(Endpoint endpoint, Task requestTask, ILogger logger)
    Microsoft.AspNetCore.Authorization.AuthorizationMiddleware.Invoke(HttpContext context)
    Swashbuckle.AspNetCore.SwaggerUI.SwaggerUIMiddleware.Invoke(HttpContext httpContext)
    Swashbuckle.AspNetCore.Swagger.SwaggerMiddleware.Invoke(HttpContext httpContext, ISwaggerProvider swaggerProvider)
    Microsoft.AspNetCore.Diagnostics.DeveloperExceptionPageMiddleware.Invoke(HttpContext context)

    Show raw exception details
    System.InvalidOperationException: Unable to resolve service for type 'Microsoft.Extensions.Logging.ILogger' while attempting to activate 'WeatherAPI.Controllers.WeatherForecastController'.
    at Microsoft.Extensions.DependencyInjection.ActivatorUtilities.GetService(IServiceProvider sp, Type type, Type requiredBy, Boolean isDefaultParameterRequired)
    at lambda_method3(Closure , IServiceProvider , Object[] )
    at Microsoft.AspNetCore.Mvc.Controllers.ControllerActivatorProvider.<>c__DisplayClass7_0.<CreateActivator>b__0(ControllerContext controllerContext)
    at Microsoft.AspNetCore.Mvc.Controllers.ControllerFactoryProvider.<>c__DisplayClass6_0.<CreateControllerFactory>g__CreateController|0(ControllerContext controllerContext)
    at Microsoft.AspNetCore.Mvc.Infrastructure.ControllerActionInvoker.Next(State& next, Scope& scope, Object& state, Boolean& isCompleted)
    at Microsoft.AspNetCore.Mvc.Infrastructure.ControllerActionInvoker.InvokeInnerFilterAsync()
    --- End of stack trace from previous location ---
    at Microsoft.AspNetCore.Mvc.Infrastructure.ResourceInvoker.<InvokeFilterPipelineAsync>g__Awaited|20_0(ResourceInvoker invoker, Task lastTask, State next, Scope scope, Object state, Boolean isCompleted)
    at Microsoft.AspNetCore.Mvc.Infrastructure.ResourceInvoker.<InvokeAsync>g__Awaited|17_0(ResourceInvoker invoker, Task task, IDisposable scope)
    at Microsoft.AspNetCore.Mvc.Infrastructure.ResourceInvoker.<InvokeAsync>g__Awaited|17_0(ResourceInvoker invoker, Task task, IDisposable scope)
    at Microsoft.AspNetCore.Routing.EndpointMiddleware.<Invoke>g__AwaitRequestTask|6_0(Endpoint endpoint, Task requestTask, ILogger logger)
    at Microsoft.AspNetCore.Authorization.AuthorizationMiddleware.Invoke(HttpContext context)
    at Swashbuckle.AspNetCore.SwaggerUI.SwaggerUIMiddleware.Invoke(HttpContext httpContext)
    at Swashbuckle.AspNetCore.Swagger.SwaggerMiddleware.Invoke(HttpContext httpContext, ISwaggerProvider swaggerProvider)
    at Microsoft.AspNetCore.Diagnostics.DeveloperExceptionPageMiddleware.Invoke(HttpContext context)
    ```

## WeatherSite 프로젝트

- `dotnet build` 를 통해 빌드하니 다음과 같은 오류가 발생하였습니다.
    - ``WeatherSite/Pages/Index.cshtml.cs(13,9): error CS0305: Using the generic type 'IEnumerable<T>' requires 1 type arguments``
    - ``WeatherSite/Pages/Index.cshtml.cs(28,12): error CS4001: Cannot await 'method group'``
- 이를 해결하기 위해 다음과 같이 코드를 수정하였더니 정상 동작하였습니다.
    - [WeatherSite/Pages/Index.cshtml.cs](https://github.com/ianychoi/aws-ecs-fargate-dotnet-module3-supplementary/commit/0b37237b6c5a28d65fab319d88e19614fa77b29a)

## 기타

- [AWS Deploy Tool for .NET](https://aws.github.io/aws-dotnet-deploy/) 사용을 위해서는 Node.js가 필요하며, Node.js 버전이 낮은 경우 정상 동작하지 않을 수 있습니다. 자세한 내용은 아래 메시지를 참고하세요.
    ```
    Install Node.js 14.17.0 or later and restart your IDE/Shell. The latest Node.js LTS version is recommended. This deployment option uses the AWS CDK, which requires Node.js version higher than your current installation (10.16.0). 
    You can install the missing Node.js dependency from: https://nodejs.org/en/download/


    For more information, please visit our troubleshooting guide https://aws.github.io/aws-dotnet-deploy/troubleshooting-guide/.
    If you are still unable to solve this issue and believe this is an issue with the tooling, please cut a ticket https://github.com/aws/aws-dotnet-deploy/issues/new/choose.
    ```
- 설명 일부에 `미네소타` 로 언급되어 있는데 조금 더 정확하게는 미네소타 주에 있는 `미니애폴리스` 도시입니다.
- `dotnet aws deploy` 에서 배포 옵션 등에 대한 세부 순서 및 목록 개수가 조금 다를 수 있으니 명령 실행 후 출력되는 내용을 잘 살펴보고 실습 가이드에 따라 잘 진행하시면 됩니다.
- 실습을 모두 마쳤으면 사용하지 않는 AWS 리소스를 할당 해제하는 것이 중요합니다. 이 때, WeatherSite 배포 삭제를 위해서는 `dotnet aws delete-deployment WeatherSite` 와 같이 실행해야 합니다. `WeatherAPI` 로 잘못 입력하지 않도록 유의해주세요.

## 다음 단계

- 실습을 모두 마친 이후에는 [기술 평가](https://aws.amazon.com/ko/developer/language/net/badges-and-training/ecs-fargate/module-four/)를 통해 `Amazon ECS 및 AWS Fargate에서의 .NET 워크로드`에 대한 디지털 배지를 받아보시기를 추천합니다!