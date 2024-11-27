```
|--- adapter (提供程序入口)
  |--- model-1
     |--- controller (web请求)
     |--- sheduler (定时任务)
   |--- model-2
|--- application (服务编排，组织domain层提供的service、数据库操作、外部调用等)
   |--- model-1
      |--- assembler (将前端的请求转换成后端entity、valueObject，或反向转成前端返回的对象。由容器管理，入参和出参的assembler区分开，职责单一、只做转换、不能有状态、防止并发)
      |--- service (编排domain层提供的service、api、repository。原则上来讲，service层之间不能互相调用)
      |--- strategy (策略模式相关)
      |--- viewobject (前端后端定义的请求对象...X1、...Z1)
      |--- handler (异步任务处理handler)
   |--- model-2
|--- domain
   |--- model-1
      |--- dto (调用三方接口的防腐dto，命名格式为 xxxReqDTO、xxxResDTO、xxxDTO)
      |--- entity (领域实体)
      |--- factory (工厂方法)
      |--- service (领域服务，一个方法放到领域A不合适，放到领域B不合适，抽出service)
      |--- support (依赖基础设施的抽象接口)
         |--- api (第三方网络调用api，根据业务功能去命名api，比如：CurrencyApi)
         |--- repository (数据库操作repository)
      |--- valueobject (值对象，注意：要体现具体的业务意义)
   |--- model-2
|--- infrastructure
   |--- annotation (注解)
   |--- aspect (切面)
   |--- config (config要求功能单一，不要一个配置类配置了很多个不同功能的bean)
   |--- constant (常量，需要分类)
   |--- enums (无业务意义的枚举值，比如渠道)
   |--- error (错误处理、错误码枚举、统一异常处理、自定义异常)
   |--- factory (基础设施层的工厂)
   |--- filter (过滤器，如果使用到第三方接口，直接调用xxxImpl接口，不需要防腐)
   |--- function (定义一些函数式接口)
   |--- invoker (第三方调用)
      |--- dto (命名无限制，同第三方系统)
      |--- feign (feign调用invoker，命令：xxxxFeignInvoker)
      |--- apidefine (api，命名：xxxxApiDefine)
      |--- message (行内报文invoker 命名：xxxxMessageInvoker)
   |--- supportimpl
      |--- converter (防腐，数据库PO和entity、valueObject，以及第三方防腐dto的转换)
         |--- xxxDtoConverter (与三方系统输出之间的转换，比如：bakCallbackDtoConverter)
         |--- xxxConverter (entity、valueObject 转换，比如：busFlowConverter)
      |--- api (domain层定义的api的实现类)
      |--- repository (domain层定义的Repository的实现类)
         |--- mybatisplus 
            |--- mapper (mapper)
            |--- po (数据库实体)
            |--- persistence (mybatis-plus生成的 service)
   |--- util
```