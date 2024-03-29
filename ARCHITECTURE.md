# ARCHITECTURE

This project is an example of architecture using new technologies and best practices.

The goal is to learn and share knowledge and use it as reference for new projects.

## PRINCIPLES and PATTERNS

* Clean Architecture
* Clean Code
* SOLID Principles
* KISS Principle
* DRY Principle
* Fail Fast Principle
* Common Closure Principle
* Common Reuse Principle
* Acyclic Dependencies Principle
* Mediator Pattern
* Result Pattern
* Folder-By-Feature Structure
* Separation of Concerns

## BENEFITS

* Simple and evolutionary architecture.
* Standardized and centralized flow for validation, log, security, return, etc.
* Avoid cyclical references.
* Avoid unnecessary dependency injection.
* Segregation by feature instead of technical type.
* Single responsibility for each request and response.
* Simplicity of unit testing.


## LAYERS

**Web:** Frontend and Backend.

**Application:** Flow control.

**Domain:** Business rules and domain logic.

**Model:** Data transfer objects.

**Database:** Data persistence.

## WEB

### FRONTEND

### Service

It is the interface between frontend and backend and has logic that does not belong in components.

```typescript
export class AppCustomerService { }
```

### Guard

It validates if a route can be activated.

```typescript
export class AppGuard implements CanActivate { }
```

### ErrorHandler

It provides a hook for centralized exception handling.

```typescript
export class AppErrorHandler implements ErrorHandler { }
```

### HttpInterceptor

It intercepts and handles an HttpRequest or HttpResponse.

```typescript
export class AppHttpInterceptor implements HttpInterceptor { }
```
### BACKEND

### Controller

It has no any logic, business rules or dependencies other than mediator.

```cs
[ApiController]
[Route("customers")]
public sealed class CustomerController : BaseController
{
    [HttpPost]
    public IActionResult Add(AddCustomerRequest request) => Mediator.HandleAsync<AddCustomerRequest, AddCustomerResponse>(request).ApiResult();

    [HttpDelete("{id:long}")]
    public IActionResult Delete(long id) => Mediator.HandleAsync(new DeleteCustomerRequest(id)).ApiResult();

    [HttpGet("{id:long}")]
    public IActionResult Get(long id) => Mediator.HandleAsync<GetCustomerRequest, GetCustomerResponse>(new GetCustomerRequest(id)).ApiResult();

    [HttpGet("grid")]
    public IActionResult Grid([FromQuery] GridCustomerRequest request) => Mediator.HandleAsync<GridCustomerRequest, GridCustomerResponse>(request).ApiResult();

    [HttpGet]
    public IActionResult List() => Mediator.HandleAsync<ListCustomerRequest, ListCustomerResponse>(new ListCustomerRequest()).ApiResult();

    [HttpPut("{id:long}")]
    public IActionResult Update(UpdateCustomerRequest request) => Mediator.HandleAsync(request).ApiResult();
}
```

## APPLICATION

It has only business flow, not business rules.

### Request

It has properties representing the request.

```cs
public sealed record AddCustomerRequest;
```

### Request Validator

It has rules for validating the request.

```cs
public sealed class AddCustomerRequestValidator : AbstractValidator<AddCustomerRequest> { }
```

### Response

It has properties representing the response.

```cs
public sealed record AddCustomerResponse;
```

### Handler

It is responsible for the business flow and processing a request to return a response.

It call factories, repositories, unit of work, services or mediator, but it has no business rules.

```cs
public sealed record AddCustomerHandler : IHandler<AddCustomerRequest, AddCustomerResponse>
{
    public async Task<Result<AddCustomerResponse>> HandleAsync(AddCustomerRequest request)
    {
        return Result<AddCustomerResponse>.Success(new AddCustomerResponse());
    }
}
```

### Factory

It creates a complex object.

Any change to object affects compile time rather than runtime.

```cs
public interface ICustomerFactory { }
```

```cs
public sealed class CustomerFactory : ICustomerFactory { }
```

## DOMAIN

It has no any references to any layer.

It has aggregates, entities, value objects and services.

### Aggregate

It defines a consistency boundary around one or more entities.

The purpose is to model transactional invariants.

One entity in an aggregate is the root, any other entities in the aggregate are children of the root.

```cs
public sealed class CustomerAggregate { }
```

### Entity

It has unique identity. Identity may span multiple bounded contexts and may endure beyond the lifetime.

Changing properties is only allowed through internal business methods in the entity, not through direct access to the properties.

```cs
public sealed class Customer : Entity { }
```

### Value Object

It has no identity and it is immutable.

It is defined only by the values ​​of its properties.

To update a value object, you must create a new instance to replace the old one.

It can have methods that encapsulate domain logic, but these methods must have no side effects on the state.

```cs
public sealed record ValueObject;
```

### Services

It performs domain operations and business rules.

It is stateless and has no operations that are not a part of an entity or value object.

```cs
public interface ICustomerService { }
```

```cs
public sealed class CustomerService : ICustomerService { }
```

## MODEL

It has properties to transport and return data.

```cs
public sealed record CustomerModel;
```

## DATABASE

It encapsulates data persistence.

### Context

It configures the connection and represents the database.

```cs
public sealed class Context : DbContext
{
    public Context(DbContextOptions options) : base(options) { }
}
```

### Entity Configuration

It configures the entity and its properties in the database.

```cs
public sealed class CustomerConfiguration : IEntityTypeConfiguration<Customer>
{
    public void Configure(EntityTypeBuilder<Customer> builder) { }
}
```

### Repository

It inherits from the generic repository and only implements specific methods.

```cs
public interface ICustomerRepository : IRepository<Customer> { }
```

```cs
public sealed class CustomerRepository : Repository<Customer>, ICustomerRepository
{
    public CustomerRepository(Context context) : base(context) { }
}
```
