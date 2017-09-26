---
title: Introduction
pageTitle: "Building a GraphQL Server with Scala Backend Tutorial"
description: "Learn how to build a GraphQL server with Scala & Sangria and best practices for filters, authentication, pagination and subscriptions."
question: "Does Schema defines where the responded data should be taken from?"
answers: ["Yes, datasources definition is a part of schema", "Optionally Schema could contains such information", "It's not a part of specification, but few implementations use it","No, it isn't a rensponsibilty of the GraphQL server" ]
correctAnswer: 3
---

### Motivation

Scala is quite popular nowadays and it's often chosen to deliver efficient and distributed systems. It leverages the Java VM, known mostly for its efficiency. Support of Functional Programming of Scala and power of Java make able to deliver applications fast in either develompent or runtime.

### What is GraphQL Server?

  The GraphQL server is a software responsible for parsing, validating and executing GraphQL queries/mutations/subscriptions. It's different from HTTP server because queries could be transported even in socket connection, but HTTP protocol is used in the most cases. Maybe it's a reason why GraphQL is often called successor of REST. I'd rather say it supplementor of REST because both can work together.

  In the next chapter you'll learn how to build your own GraphQL server using Scala and the following technologies:
  * [Scala](https://www.scala-lang.org/) Scala language
  * [Akka HTTP](http://doc.akka.io/docs/akka-http/current/scala/http) Web server to handle HTTP requests.
  * [Sangria](http://sangria-graphql.org/) A library for GraphQL execution
  * [Slick](http://slick.lightbend.com/) A Database query and access library.
  * [H2 Database](http://www.h2database.com/html/main.html) In-memory database.
  * [Graphiql](https://github.com/graphql/graphiql) A simple GraphQL console to play with.

  I assume you're familiar with GraphQL concepts, but if not, you can visit [GraphQL site](http://graphql.org/) to learn more about that.



### Schema-Driven Development

Schema in GraphQL is its heart. Everything is built around this. It defines what user can fetch for, or what kind of mutations is able to perform. GraphQL isn't a language you can use to fetch anything.... It's a language you can use to fetch anything *what Schema allows for*. On the other hand GraphQL Schema doesn't define where the data should be taken from. Server is responsbile for fetch, even from different sources like Database and another RESTful service.

In the following chapters I will teach you how to implement as similar as possible to following schema:

```graphql(nocopy)(https://github.com/howtographql/howtographql/blob/master/meta/structure.graphql)
type Query {
  allLinks(filter: LinkFilter, orderBy: LinkOrderBy, skip: Int, first: Int): [Link!]!
  _allLinksMeta: _QueryMeta!
}

type Mutation {
  signinUser(email: AUTH_PROVIDER_EMAIL): SigninPayload!
  createUser(name: String!, authProvider: AuthProviderSignupData!): User
  createLink(description: String!, url: String!, postedById: ID): Link
  createVote(linkId: ID, userId: ID): Vote
}

type Subscription {
  Link(filter: LinkSubscriptionFilter): LinkSubscriptionPayload
  Vote(filter: VoteSubscriptionFilter): VoteSubscriptionPayload
}

interface Node {
  id: ID!
}

type User implements Node {
  id: ID! @isUnique
  createdAt: DateTime!
  name: String!
  links: [Link!]! @relation(name: "UsersLinks")
  votes: [Vote!]! @relation(name: "UsersVotes")
  email: String @isUnique
  password: String
}

type Link implements Node {
  id: ID! @isUnique
  createdAt: DateTime!
  url: String!
  description: String!
  postedBy: User! @relation(name: "UsersLinks")
  votes: [Vote!]! @relation(name: "VotesOnLink")
}

type Vote implements Node {
  id: ID! @isUnique
  createdAt: DateTime!
  user: User! @relation(name: "UsersVotes")
  link: Link! @relation(name: "VotesOnLink")
}

input AuthProviderSignupData {
  email: AUTH_PROVIDER_EMAIL
}

input AUTH_PROVIDER_EMAIL {
  email: String!
  password: String!
}

input LinkSubscriptionFilter {
  mutation_in: [_ModelMutationType!]
}

input VoteSubscriptionFilter {
  mutation_in: [_ModelMutationType!]
}

input LinkFilter {
  OR: [LinkFilter!]
  description_contains: String
  url_contains: String
}

type SigninPayload {
  token: String
  user: User
}

type LinkSubscriptionPayload {
  mutation: _ModelMutationType!
  node: Link
  updatedFields: [String!]
}

type VoteSubscriptionPayload {
  mutation: _ModelMutationType!
  node: Vote
  updatedFields: [String!]
}

enum LinkOrderBy {
  createdAt_ASC
  createdAt_DESC
}

enum _ModelMutationType {
  CREATED
  UPDATED
  DELETED
}

type _QueryMeta {
  count: Int!
}

scalar DateTime
```
