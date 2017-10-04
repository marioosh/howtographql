---
title: Extending a Link model
pageTitle: "Add Date to the link model, make the DB and Sangria to understand this type"
description: "In this chapter Link model will get additional field to store date and time. You will learn how to handle such custom types and use scalars."
---

### Goal for this chapter

To align to the Schema we proposed at the beginning, we have to extend a `Link` model with additional field: `createdAt`. This field will store date and time. The problem is that `H2` database understand only timestamp, similar to Sangria - it also supports only basic types. Our goal is to store it in database and present in out in human friendly format.


### Extend a Link model

The type for storing date and time is `akka.http.scaladsl.model.DateTime`. It fits to our example because it has implemented ISO Format converters working in both sides.

<Instruction>

Change the content of `Link.scala`:

```scala
akka.http.scaladsl.model.DateTime

case class Link(id: Int, url: String, description: String, createdAt: DateTime)
```

</Instruction>

### Fix the databases

In `DbSchema` we're storing few links in database, we have to add additional field now.

<Instruction>

Add `createdAt` field in `Link` models for populated example data in `DBSchema`. Change `databaseSetup` function into following code:

```scala
Links ++= Seq(
      Link(1, "http://howtographql.com", "Awesome community driven GraphQL tutorial", DateTime(2017,9,12)),
      Link(2, "http://graphql.org", "Official GraphQL webpage",DateTime(2017,10,1)),
      Link(3, "https://facebook.github.io/graphql/", "GraphQL specification",DateTime(2017,10,2))
    )
```

</Instruction>

But `H2` doesn't know how to store such type in database, we have to help understand this as well.

<Instruction>

Add column mapper for our `DateTime` type in `DBSchema` object.

```scala
implicit val dateTimeColumnType = MappedColumnType.base[DateTime, Timestamp](
    dt => new Timestamp(dt.clicks),
    ts => DateTime(ts.getTime)
)
```

</Instruction>

This mapper will convert `DateTime` into used internally by database `Long` type.

The last thing is to add `createdAt` column definition in table declaration.

<Instruction>

Add following code inside `LinksTable` class. Remove current `*` function.

```scala
def createdAt = column[DateTime]("CREATED_AT")

def * = (id, url, description, createdAt) <> ((Link.apply _).tupled, Link.unapply)
```

</Instruction>


### Define custom scalar for DateTime
