# Dgraph Dev Envionment

[Devbox](https://www.jetify.com/docs/devbox/) includes [Process Compose](https://f1bonacc1.github.io/process-compose/), so I used to that to make it easy to fire up an instance of the [Dgraph Learning Environment](https://dgraph.io/docs/deploy/installation/single-host-setup/). There's a weird error where `docker rm dgraph` isn't always removing the stopped container, which makes it impossible to restart all service. #future-problems

## Creating a schema

Schema creation with Dgraph is simple: you just post the GraphQL schema to an admin URL and it's done.

```bash
curl -X POST localhost:8080/admin/schema --data-binary '@schema.graphql'
```

That raises some concerns:

- [?] How will I even manage schema migrations?
- [?] If the app is local-first, different people will be running different versions. How will I ensure changes are backward compatible?
- [?] How do I protect that URL so that _anyone_ can't push a schema migration?
- [?] How the heck to schema migrations even work with Dgraph...?

Playing around, I created a really simple test schema and pushed it to Dgraph.

```graphql
type Task {
  id: ID!
  content: String!
  createdAt: String!
  updatedAt: String
}
```

When inspecting the GraphQL API at http://localhost:8080/graphql
, I can see that—by convention—Dgraph generated the following (as well as a [heap of boilerplate](https://dgraph.io/docs/graphql/schema/dgraph-schema/)):

```graphql
type Task {
  id: ID!
  content: String!
  createdAt: String!
  updatedAt: String
}

type Mutation {
  addTask(input: [AddTaskInput!]!): AddTaskPayload
  updateTask(input: UpdateTaskInput!): UpdateTaskPayload
  deleteTask(filter: TaskFilter!): DeleteTaskPayload
}

type Query {
  getTask(id: ID!): Task
  queryTask(
    filter: TaskFilter
    order: TaskOrder
    first: Int
    offset: Int
  ): [Task]
  aggregateTask(filter: TaskFilter): TaskAggregateResult
}

type TaskAggregateResult {
  count: Int
  contentMin: String
  contentMax: String
  createdAtMin: String
  createdAtMax: String
  updatedAtMin: String
  updatedAtMax: String
}

input TaskFilter {
  id: [ID!]
  has: [TaskHasFilter]
  and: [TaskFilter]
  or: [TaskFilter]
  not: TaskFilter
}

enum TaskHasFilter {
  content
  createdAt
  updatedAt
}

input TaskOrder {
  asc: TaskOrderable
  desc: TaskOrderable
  then: TaskOrder
}

enum TaskOrderable {
  content
  createdAt
  updatedAt
}

input TaskPatch {
  content: String
  createdAt: String
  updatedAt: String
}

input TaskRef {
  id: ID
  content: String
  createdAt: String
  updatedAt: String
}

input UpdateTaskInput {
  filter: TaskFilter!
  set: TaskPatch
  remove: TaskPatch
}

type UpdateTaskPayload {
  task(filter: TaskFilter, order: TaskOrder, first: Int, offset: Int): [Task]
  numUids: Int
}
```

