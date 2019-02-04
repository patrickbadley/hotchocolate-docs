---
id: tutorial-01-gettingstarted
title: Getting Started
---

In this section, you will setup the project for your GraphQL server and implement your first GraphQL query. In the end, we’ll talk theory for a bit and learn about the GraphQL schema.

## Creating the project

The quickest way to create a GraphQL server with _Hot Chocolate_ is to use our server template.

But in this tutorial you will build a _Hot Chocolate_ GraphQL server from the ground up in order to learn how the components interconnect.

Open your terminal, navigate to a location of your choice and run the following commands:

```bash
mkdir twitter-service
cd twitter-service
mkdir src
cd src
```

This creates a directory for our GraphQL server project and adds the `src` directory which will hold the code files.
Next up you will have to create a solution file and a server project.

In your terminal run the following command to create the solution file and create a minimal ASP.net server project:

```bash
dotnet new web -n Server

cd ..
dotnet new sln -n Demo
dotnet sln add src/Server
```

Now that we have created our server project let's add _Hot Chocolate_.

Get back in your terminal and run the following command to add the _HotChocolate.AspNetCore_ package:

```bash
dotnet add src/Server package HotChocolate.AspNetCore
dotnet restore
```

The _HotChocolate.AspNetCore_ package contains the ASP.net core middleware for _Hot Chocolate_ and also includes the _Hot Chocolate_ query engine as a dependency. This package basically provides all we need to get started.

## Configuring the GraphQL server

So, in order to configure our server let us open the `src/Server/Startup.cs` and rewrite the `ConfigureServices` method like the following:

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddGraphQL(c =>
    {
        // we will add our config here later
    });
}
```

`AddGraphQL` adds a schema to our services which makes it available to our middleware.

In order to create a valid schema we need to have at least a query type. So, let's introduce a new class called `Query` that shall contain one Property `Hello` which shall return the `string` `world`.

Create a new file `src/Server/Query.cs` and add the following code:

```csharp
namespace Server
{
    public class Query
    {
        public string Hello => "World";
    }
}
```

Now lets register this type as our query type with the GraphQL schema.

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddGraphQL(c =>
    {
        c.RegisterQueryType<Query>();
    });
}
```

With this we have finally a valid schema that we could now serve up with our middleware. In order to run our server we now have to register the GraphQL middleware with the ASP.net application builder.

For that replace the `Configure` method in `src/Server/Startup.cs` with the following code:

```csharp
public void Configure(IApplicationBuilder app, IHostingEnvironment env)
{
    if (env.IsDevelopment())
    {
        app.UseDeveloperExceptionPage();
    }

    app.UseGraphQL();
}
```

`app.UseGraphQL()` adds our middleware to the server and will by default use the schema that we have setup with our dependency injection.

In order to write queries and execute them it would be practical if our server also serves up _Playground_. Playground is a GraphiQL base UI developed by Prisma.

_Hot Chocolate_ provides packages for _GraphiQL_ and _Playground_. In our case we will add _Playground_.

Return to your terminal and run the following commands:

```bash
dotnet add src/Server package HotChocolate.AspNetCore.Playground
dotnet restore
```

Now that we have added the _Playground_ package we need to add the _Playground_ middleware to our server. For that we have to add one more line of code to our `src/Server/Startup.cs`.

```csharp
public void Configure(IApplicationBuilder app, IHostingEnvironment env)
{
    if (env.IsDevelopment())
    {
        app.UseDeveloperExceptionPage();
    }

    app.UseGraphQL();
    app.UsePlayground();
}
```

## Testing the server

Since we have now a UI that can execute queries against our server lets start our server and fire up our first GraphQL query against it:

```bash
dotnet watch --project src/Server/ run
```

The `watch` command will compile and the run your server. Every time a code file changes the `watch` command will recompile your server so we do not have to worry about compiling all the time. The server will per default be hosted on `http://127.0.0.1:5000`. Playground will be served under `http://127.0.0.1:5000/playground`.

Since every thing should be running now lets open a browser and navigate to `http://127.0.0.1:5000/playground`.

/// Playground image

Lets write our first query.

```GraphQL
{
    hello
}
```

After you have added the above query to the left panel of playground click the big play button in the middle and our GraphQL server should return the following result.

```json
{
  "data": {
    "hello": "World"
  }
}
```

### Root Types

In order to understand what GraphQL requests actually mean and how GraphQL works we should have a look at how a schema is really structured.
GraphQL has three well-known root types `Query`, `Mutation` and `Subscription`.

Each of those root types represent a way in which you can access their data.

#### Query
A query type for example is for requesting data, the server expects that query fields do not have side-effects. Basically, query fields are read-only operations where you query your data source. The GraphQL server is allowed to execute query fields in parallel.

#### Mutation

The mutation type is for providing functionality to modify your data source. The GraphQL server guaranties you that root fields will be executed serially. This is important since you could have a query like the following:

```graphql
mutation {
  createUser(name: "foo") {
    id
    name
  }
  addUserToGroup(userName: "foo", groupName: "bar") {
    name
  }
}
```

The mutation type is basically through which you can add, update or delete your data.

#### Subscription

The subscription type represents the events that a server exposes. Like query you cosume data through subscriptions rather than mutating them. But in contrast to query you subscribe to a certain subscription and than receive data asynchronously. W

GraphQL queries describe the request that you want to execute as well as the data that you want to consume.