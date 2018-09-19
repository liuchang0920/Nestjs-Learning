# Nest 核心功能 —— Controller

什么是 `Controller`？语义化翻译就是 **控制器**，它负责处理传入的请求并将响应结果返回给客户端。

控制器的目的是接收应用程序的特定请求。其 **路由** 机制控制哪个控制器接收哪些请求。通常，每个控制器都有多个路由，不同的路由可以执行不同的操作。

## 如何将类定义为控制器？

在类声明上，定义 `@Controller()` 注解，即可将该类定义为控制器，在 `Nest` 中，几乎所有的注解都是以方法形式存在的，我们通过查看 `@Controller()` 注解的源码：

```typescript
export function Controller(prefix?: string): ClassDecorator {
  const path = isUndefined(prefix) ? '/' : prefix;
  return (target: object) => {
    Reflect.defineMetadata(PATH_METADATA, path, target);
  };
}
```

可以看出，控制器注解有一个可选的方法参数，该参数默认值为 `/`，其作用就是定义当前控制器类下所有路由方法的前缀，这样以来，就可以避免我们在定义路由时出现多余的相同前缀。

现在，让我们定义一个前缀为 `cats` 的控制器：

```typescript
import { Controller } from '@nestjs/common';

@Controller('cats')
export class CatsController { }
```

## 如何在类的方法上定义路由？

在类的方法声明上，定义 `@Get()`、`@Post()`、`@Put()`、`@Patch()`、`@Delete()`、`@Options()`、`@Head()`、`@All()`。这些注解都表示各自的 **HTTP** 请求方法。

现在，让我们在上述的 `CatsController` 类中定义实际开发中常用的几种路由映射：

```typescript
import { Controller, Get, Post, Body, Put, Param, Delete } from '@nestjs/common';

@Controller('cats')
export class CatsController {
  @Post()
  create(@Body() createCatDto) {
    return 'This action adds a new cat';
  }

  @Get()
  findAll(@Query() query) {
    return `This action returns all cats (limit: ${query.limit} items)`;
  }

  @Get(':id')
  findOne(@Param('id') id) {
    return `This action returns a #${id} cat`;
  }

  @Put(':id')
  update(@Param('id') id, @Body() updateCatDto) {
    return `This action updates a #${id} cat`;
  }

  @Delete(':id')
  remove(@Param('id') id) {
    return `This action removes a #${id} cat`;
  }
}

// createCatDto 和 updateCatDto 可以使用类来定义其结构：
export class CreateCatDto {
    id: number;
    name: string;
}

export class UpdateCatDto {
    name: string;
}
```

在定义路由时，方法参数中，我们使用到了 `@Body()`、`@Query()`、`@Param()` 这样的参数注解，他们是什么意思呢？用过 `express` 的同学都会经常接触到：`req`、`res`、`next`、`req.body`、`req.query`、`req.params`等等，诸如此类的特性，下面我们列举出，在 **Nest** 框架中，这些注解与 `express` 中的使用方法是如何对应的：

|Nest|Express|
|:-----------------------|:-------------------------------|
|@Request()              |req                             |
|@Response()             |res                             |
|@Next()                 |next                            |
|@Session()              |req.session                     |
|@Param(param?: string)  |req.params / req.params[param]  |
|@Body(param?: string)   |req.body / req.body[param]      |
|@Query(param?: string)  |req.query / req.query[param]    |
|@Headers(param?: string)|req.headers / req.headers[param]|

> Tips：Nest 底层默认使用了 `express` 作为 web 层的框架

### 通配符路由

Nest 也支持基于模式的路由。例如，星号用作通配符，将匹配任何字符组合。示例：

```typescript
@Get('ab*cd')
findAll() {
  return 'This route uses a wildcard';
}
```

以上路由路径将匹配ABCD、ab _ CD、abecd等。字符 `？`、`+`、`*`、`()` 是正则表达式对应项的子集。连字符(`-`)和点(`.`)由字符串路径按字面解释。

### 状态码(Status code)

如上所述，默认情况下，除 POST 请求的状态码是 **201** 以外，其余请求的状态码总是 **200**，但我们可以通过在方法声明上添加 `@HttpCode(...)` 注解来改变默认的状态码：

```typescript
@Post()
@HttpCode(204)
create() {
  return 'This action adds a new cat';
}
```

### 响应头(Headers)

若要指定自定义响应头，可以使用 `@Header()` 注解：

```typescript
@Post()
@Header('Cache-Control', 'none')
create() {
  return 'This action adds a new cat';
}
```