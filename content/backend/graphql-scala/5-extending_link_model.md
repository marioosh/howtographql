---
title: Extending a Link model
pageTitle: "Add Date to the link model, make the DB and Sangria to understand this type"
description: "In this chapter Link model will get additional field to store date and time. You will learn how to handle such custom types and use scalars."
---

### Goal for this chapter

To align to the Schema we proposed at the beginning, we have to extend a `Link` model with additional field: `createdAt`. This field will store date and time. The problem is that `H2` database understand only timestamp, similar to Sangria - it also supports only basic types. Our goal is to store it in database and present in out in human friendly format.


###  
