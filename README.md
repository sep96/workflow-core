# Workflow Core

[![Build status](https://ci.appveyor.com/api/projects/status/xnby6p5v4ur04u76?svg=true)](https://ci.appveyor.com/project/danielgerlag/workflow-core)

Workflow Core is a light weight workflow engine targeting .NET Standard.  Think: long running processes with multiple tasks that need to track state.  It supports pluggable persistence and concurrency providers to allow for multi-node clusters.

## Documentation

See [Full Documentation here.](https://github.com/danielgerlag/workflow-core/wiki)

## Fluent API

Define your workflows with the fluent API.

```c#
public class MyWorkflow : IWorkflow
{
    public void Build(IWorkflowBuilder<MyData> builder)
    {    
        builder
           .StartWith<Task1>()
           .Then<Task2>()
           .Then<Task3>;
    }
}
```

### Sample use cases

New user workflow
```c#
public class MyData
{
	public string Email { get; set; }
	public string Password { get; set; }
	public string UserId { get; set; }
}

public class MyWorkflow : IWorkflow
{
    public void Build(IWorkflowBuilder<MyData> builder)
    {    
        builder
            .StartWith<CreateUser>()
                .Input(step => step.Email, data => data.Email)
                .Input(step => step.Password, data => data.Password)
                .Output(data => data.UserId, step => step.UserId);
           .Then<SendConfirmationEmail>()
               .WaitFor("confirmation", data => data.UserId)
           .Then<UpdateUser>()
               .Input(step => step.UserId, data => data.UserId);
    }
}
```

Resilient service orchestration

```c#
public class MyWorkflow : IWorkflow
{
    public void Build(IWorkflowBuilder<MyData> builder)
    {    
        builder
            .StartWith<CreateCustomer>()
            .Then<PushToSalesforce>()
                .OnError(WorkflowErrorHandling.Retry, TimeSpan.FromMinutes(10))
            .Then<PushToERP>()
                .OnError(WorkflowErrorHandling.Retry, TimeSpan.FromMinutes(10));
    }
}
```

## Persistence

Since workflows are typically long running processes, they will need to be persisted to storage between steps.
There are several persistence providers available as separate Nuget packages.

* MemoryPersistenceProvider *(Default provider, for demo and testing purposes)*
* [MongoDB](src/providers/WorkflowCore.Persistence.MongoDB)
* [SQL Server](src/providers/WorkflowCore.Persistence.SqlServer)
* [PostgreSQL](src/providers/WorkflowCore.Persistence.PostgreSQL)
* [Sqlite](src/providers/WorkflowCore.Persistence.Sqlite)
* Redis *(coming soon...)*

## Extensions

* [User (human) workflows](src/extensions/WorkflowCore.Users)
* [HTTP wrapper for Workflow Host Service](src/extensions/WorkflowCore.WebAPI)


## Samples

[Hello World](src/samples/WorkflowCore.Sample01)

[Multiple outcomes](src/samples/WorkflowCore.Sample06)

[Passing Data](src/samples/WorkflowCore.Sample03)

[Events](src/samples/WorkflowCore.Sample04)

[Deferred execution & re-entrant steps](src/samples/WorkflowCore.Sample05)

[Looping](src/samples/WorkflowCore.Sample02)

[Exposing a REST API](src/samples/WorkflowCore.Sample07)

[Human(User) Workflow](src/samples/WorkflowCore.Sample08)


## Authors

* **Daniel Gerlag** - *Initial work*

## Ports

[Node.js](https://github.com/danielgerlag/workflow-es)

[Ruby](https://github.com/danielgerlag/workflow_rb)

## License

This project is licensed under the MIT License - see the [LICENSE.md](LICENSE.md) file for details

