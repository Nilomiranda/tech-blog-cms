While building GraphQL apis, if you need to stay compliant with relay server specifications you can use this [relay-graphql library](https://github.com/graphql/graphql-relay-js) since it provides a bunch of helpers so you can avoid some boilerplate code, specially when dealing with pagination following the connection specifications.

However, you might encounter some problems if you try using this with NestJS GraphQL module since it's heavily focused on object oriented programming paradigms while graphql-relay is more generic and works around composing different functions.

The real problem is how you can specify the given entity connection return type so NestJS can properly work with your schema.

So what we can do is, define a generic function that will return a Connection class that implements Relay's connection interface

```typescript
import { Type } from "@nestjs/common";
import { Field, ObjectType } from "@nestjs/graphql";
import { Connection as RelayConnection, Edge as RelayEdge, PageInfo as RelayPageInfo } from "graphql-relay";

export function Connection<GraphQLObject>(GenericClass?: Type<GraphQLObject>) {
  @ObjectType({ isAbstract: true })
  abstract class IConnection implements RelayConnection<GraphQLObject> {
    @Field(() => [Edge], { nullable: false })
    edges: Array<RelayEdge<GraphQLObject>>

    @Field(() => PageInfo, { nullable: false })
    pageInfo: PageInfo;
  }

  return IConnection
}
```



Notice that this class `IConnection` is implementing RelayConnection that we are importing from "graphql-relay" package.	



We are defining that the field `edges` is of type `[Edge]`. So let's define it in our function:

```typescript
@ObjectType({ isAbstract: true })
abstract class Edge<GraphQLObject> implements RelayEdge<GraphQLObject> {
    @Field(() => GenericClass, { nullable: false })
    node: GraphQLObject

    @Field(() => String, { nullable: false })
    cursor: string
}
```

And the field `pageInfo` is returning `PageInfo`, so let's also define it:

```typescript
@ObjectType({ isAbstract: true })
class PageInfo implements RelayPageInfo {
    @Field(() => String, { nullable: true })
    startCursor: string;

    @Field(() => String, { nullable: true })
    endCursor: string;

    @Field(() => Boolean, { nullable: false })
    hasPreviousPage: boolean;

    @Field(() => Boolean, { nullable: false })
    hasNextPage: boolean;
}
```



Your function `Connection` should look like this now:



```typescript
export function Connection<GraphQLObject>(GenericClass?: Type<GraphQLObject>) {
    @ObjectType({ isAbstract: true })
    class PageInfo implements RelayPageInfo {
        @Field(() => String, { nullable: true })
        startCursor: string;

        @Field(() => String, { nullable: true })
        endCursor: string;

        @Field(() => Boolean, { nullable: false })
        hasPreviousPage: boolean;

        @Field(() => Boolean, { nullable: false })
        hasNextPage: boolean;
    }

    @ObjectType({ isAbstract: true })
    abstract class Edge<GraphQLObject> implements RelayEdge<GraphQLObject> {
        @Field(() => GenericClass, { nullable: false })
        node: GraphQLObject

        @Field(() => String, { nullable: false })
        cursor: string
    }

    @ObjectType({ isAbstract: true })
    abstract class IConnection implements RelayConnection<GraphQLObject> {
        @Field(() => [Edge], { nullable: false })
        edges: Array<RelayEdge<GraphQLObject>>

        @Field(() => PageInfo, { nullable: false })
        pageInfo: PageInfo;
    }

    return IConnection
}
```

You might be wondering, why not simply use generic and separated classes. Unfortunately typescript has some limitations that won't make this generic classes work as we would like. (You can read a bit more about it [here](https://typegraphql.com/docs/generic-types.html#how-to)).

Now, the only thing you would need to do, for each of your project's entities, anytime you need to implement pagination following the connection specficiations, is this:

You could create a `user.connection.ts` and define it like this:

```typescript
import { ObjectType } from "@nestjs/graphql";
import {Connection} from "../../_types/models/connection.model";
import { User } from "./user.model";

@ObjectType()
export class UserConnection extends Connection<User>(User) {
}

```

And in your resolver you'll be able to do something like this:

```typescript
import { connectionFromPromisedArray } from "graphql-relay";

/**
* ... rest of your code ...
*/

@Query(() => FinancialRecordConnection)
async users(@Args({ type: () => ConnectionArguments }) args: ConnectionArguments) {
	return connectionFromPromisedArray(this.usersService?.list(), args)
}
```

## Note:
In your function `Connection` if you get an error like this:

> Return type of exported function has or is using private name 'IConnection'.ts(4060)

You have two options:

1. Go to your tsconfig.json file and set `false` to `declarations` property.
2. Explicitly type the return of the `Connection` function with `any`.