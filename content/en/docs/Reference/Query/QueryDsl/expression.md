---
title: "Query Expressions"
linkTitle: "Expressions"
weight: 200
description: >
---

## Overview {#overview}

This page covers the components of an expression, including declarations, operators, and functions.

- [Declarations]({{< relref "#declaration" >}})
- [Comparison operators]({{< relref "#comparison-operator" >}})
- [Logical operators]({{< relref "#logical-operator" >}})
- [Arithmetic operators]({{< relref "#arithmetic-operator" >}})
- [Mathematical functions]({{< relref "#mathematical-function" >}})
- [String functions]({{< relref "#string-function" >}})
- [Aggregate functions]({{< relref "#aggregate-function" >}})
- [Conditional expressions]({{< relref "#conditional-expression" >}})
- [Scalar subqueries]({{< relref "#scalar-subquery" >}})
- [literals]({{< relref "#literal" >}})

## Declarations {#declaration}

In the Query DSL, for example, you can pass a lambda expression representing
the search criteria to the `where` function.

```kotlin
QueryDsl.from(a).where { a.addressId eq 1 }
```

We call such lambda expressions declarations.
All declarations are defined as typealias in the `org.komapper.core.dsl.expression` package.

AssignmentDeclaration
: Used with the `values` and `set` functions.

HavingDeclaration
: Used with the `having` function.

OnDeclaration
: Used with the `on` function.

WhenDeclaration
: Used with the `When` function.

WhereDeclaration
: Used with the `where` function.

These declarations are composable.

### plus {#declaration-composition-plus}

The `+` operator constructs a new declaration that executes its operands in sequence:

```kotlin
val w1: WhereDeclaration = {
    a.addressId eq 1
}
val w2: WhereDeclaration = {
    a.version eq 1
}
val w3: WhereDeclaration = w1 + w2 // Use of the `+` operator
val query: Query<List<Address>> = QueryDsl.from(a).where(w3)
val list: List<Address> = db.runQuery { query }
/*
select t0_.ADDRESS_ID, t0_.STREET, t0_.VERSION from ADDRESS as t0_ where t0_.ADDRESS_ID = ? and t0_.VERSION = ?
*/
```

The `+` operator is available in all declarations.

### and {#declaration-composition-and}

The `and` function constructs a new declaration that concatenates its receiver and argument with the AND predicate:

```kotlin
val w1: WhereDeclaration = {
    a.addressId eq 1
}
val w2: WhereDeclaration = {
    a.version eq 1
    or { a.version eq 2 }
}
val w3: WhereDeclaration = w1.and(w2) // Use of the `and` function
val query: Query<List<Address>> = QueryDsl.from(a).where(w3)
val list: List<Address> = db.runQuery { query }
/*
select t0_.ADDRESS_ID, t0_.STREET, t0_.VERSION from ADDRESS as t0_ where t0_.ADDRESS_ID = ? and (t0_.VERSION = ? or (t0_.VERSION = ?))
*/
```

The `and` function can be applied to Having, When, and Where declarations.

### or {#declaration-composition-or}

The `or` function constructs a new declaration that concatenates its receiver and argument with the OR predicate:

```kotlin
val w1: WhereDeclaration = {
    a.addressId eq 1
}
val w2: WhereDeclaration = {
    a.version eq 1
    a.street eq "STREET 1"
}
val w3: WhereDeclaration = w1.or(w2) // Use of the `or` function
val query: Query<List<Address>> = QueryDsl.from(a).where(w3)
val list: List<Address> = db.runQuery { query }
/*
select t0_.ADDRESS_ID, t0_.STREET, t0_.VERSION from ADDRESS as t0_ where t0_.ADDRESS_ID = ? or (t0_.VERSION = ? and t0_.STREET = ?)
*/
```

The `or` function can be applied to Having, When, and Where declarations.

## Comparison operators {#comparison-operator}

Comparison operators are available in the Having, On, When, and Where [Declarations]({{< relref "#declaration" >}}).

If `null` is passed as an argument to a comparison operator, the operator is not evaluated.
That is, the corresponding SQL will not be generated:

```kotlin
val nullable: Int? = null
val query = QueryDsl.from(a).where { a.addressId eq nullable }
```

Thus, when the above `query` is executed, the following SQL will be issued:

```sql
select t0_.ADDRESS_ID, t0_.STREET, t0_.VERSION from ADDRESS as t0_
```

### eq {#comparison-operator-eq}

```kotlin
QueryDsl.from(a).where { a.addressId eq 1 }
/*
select t0_.ADDRESS_ID, t0_.STREET, t0_.VERSION from ADDRESS as t0_ where t0_.ADDRESS_ID = ?
*/
```

### notEq {#comparison-operator-noteq}

```kotlin
QueryDsl.from(a).where { a.addressId notEq 1 }
/*
select t0_.ADDRESS_ID, t0_.STREET, t0_.VERSION from ADDRESS as t0_ where t0_.ADDRESS_ID <> ?
*/
```

### less {#comparison-operator-less}

```kotlin
QueryDsl.from(a).where { a.addressId less 1 }
/*
select t0_.ADDRESS_ID, t0_.STREET, t0_.VERSION from ADDRESS as t0_ where t0_.ADDRESS_ID < ?
*/

```

### lessEq {#comparison-operator-lesseq}

```kotlin
QueryDsl.from(a).where { a.addressId lessEq 1 }
/*
select t0_.ADDRESS_ID, t0_.STREET, t0_.VERSION from ADDRESS as t0_ where t0_.ADDRESS_ID <= ?
*/
```

### greater {#comparison-operator-greater}

```kotlin
QueryDsl.from(a).where { a.addressId greater 1 }
/*
select t0_.ADDRESS_ID, t0_.STREET, t0_.VERSION from ADDRESS as t0_ where t0_.ADDRESS_ID > ?
*/
```

### greaterEq {#comparison-operator-greatereq}

```kotlin
QueryDsl.from(a).where { a.addressId greaterEq 1 }
/*
select t0_.ADDRESS_ID, t0_.STREET, t0_.VERSION from ADDRESS as t0_ where t0_.ADDRESS_ID >= ?
*/
```

### isNull {#comparison-operator-isnull}

```kotlin
QueryDsl.from(e).where { e.managerId.isNull() }
/*
select t0_.EMPLOYEE_ID, t0_.EMPLOYEE_NO, t0_.EMPLOYEE_NAME, t0_.MANAGER_ID, t0_.HIREDATE, t0_.SALARY, t0_.DEPARTMENT_ID, t0_.ADDRESS_ID, t0_.VERSION from EMPLOYEE as t0_ where t0_.MANAGER_ID is null
*/
```
### isNotNull {#comparison-operator-isnotnull}

```kotlin
QueryDsl.from(e).where { e.managerId.isNotNull() }
/*
select t0_.EMPLOYEE_ID, t0_.EMPLOYEE_NO, t0_.EMPLOYEE_NAME, t0_.MANAGER_ID, t0_.HIREDATE, t0_.SALARY, t0_.DEPARTMENT_ID, t0_.ADDRESS_ID, t0_.VERSION from EMPLOYEE as t0_ where t0_.MANAGER_ID is not null
*/
```

### like {#comparison-operator-like}

```kotlin
QueryDsl.from(a).where { a.street like "STREET 1_" }.orderBy(a.addressId)
/*
select t0_.ADDRESS_ID, t0_.STREET, t0_.VERSION from ADDRESS as t0_ where t0_.STREET like ? escape ? order by t0_.ADDRESS_ID asc
*/
```

### notLike {#comparison-operator-notlike}

```kotlin
QueryDsl.from(a).where { a.street notLike "STREET 1_" }.orderBy(a.addressId)
/*
select t0_.ADDRESS_ID, t0_.STREET, t0_.VERSION from ADDRESS as t0_ where t0_.STREET not like ? escape ? order by t0_.ADDRESS_ID asc
*/
```

### startsWith {#comparison-operator-startswith}

```kotlin
QueryDsl.from(a).where { a.street startsWith "STREET 1" }.orderBy(a.addressId)
/*
select t0_.ADDRESS_ID, t0_.STREET, t0_.VERSION from ADDRESS as t0_ where t0_.STREET like ? escape ? order by t0_.ADDRESS_ID asc
*/
```

### notStartsWith {#comparison-operator-notstartswith}

```kotlin
QueryDsl.from(a).where { a.street notStartsWith "STREET 1" }.orderBy(a.addressId)
/*
select t0_.ADDRESS_ID, t0_.STREET, t0_.VERSION from ADDRESS as t0_ where t0_.STREET not like ? escape ? order by t0_.ADDRESS_ID asc
*/
```

### contains {#comparison-operator-contains}

```kotlin
QueryDsl.from(a).where { a.street contains "T 1" }.orderBy(a.addressId)
/*
select t0_.ADDRESS_ID, t0_.STREET, t0_.VERSION from ADDRESS as t0_ where t0_.STREET like ? escape ? order by t0_.ADDRESS_ID asc
*/
```

### notContains {#comparison-operator-notcontains}

```kotlin
QueryDsl.from(a).where { a.street notContains "T 1" }.orderBy(a.addressId)
/*
select t0_.ADDRESS_ID, t0_.STREET, t0_.VERSION from ADDRESS as t0_ where t0_.STREET not like ? escape ? order by t0_.ADDRESS_ID asc
*/
```

### endsWith {#comparison-operator-endswith}

```kotlin
QueryDsl.from(a).where { a.street endsWith "1" }.orderBy(a.addressId)
/*
select t0_.ADDRESS_ID, t0_.STREET, t0_.VERSION from ADDRESS as t0_ where t0_.STREET like ? escape ? order by t0_.ADDRESS_ID asc
*/
```

### notEndsWith {#comparison-operator-notendswith}

```kotlin
QueryDsl.from(a).where { a.street notEndsWith "1" }.orderBy(a.addressId)
/*
select t0_.ADDRESS_ID, t0_.STREET, t0_.VERSION from ADDRESS as t0_ where t0_.STREET not like ? escape ? order by t0_.ADDRESS_ID asc
*/
```

### between {#comparison-operator-between}

```kotlin
QueryDsl.from(a).where { a.addressId between 5..10 }.orderBy(a.addressId)
/*
select t0_.ADDRESS_ID, t0_.STREET, t0_.VERSION from ADDRESS as t0_ where t0_.ADDRESS_ID between ? and ? order by t0_.ADDRESS_ID asc
*/
```

### notBetween {#comparison-operator-notbetween}

```kotlin
QueryDsl.from(a).where { a.addressId notBetween 5..10 }.orderBy(a.addressId)
/*
select t0_.ADDRESS_ID, t0_.STREET, t0_.VERSION from ADDRESS as t0_ where t0_.ADDRESS_ID not between ? and ? order by t0_.ADDRESS_ID asc
*/
```

### inList {#comparison-operator-inlist}

```kotlin
QueryDsl.from(a).where { a.addressId inList listOf(9, 10) }.orderBy(a.addressId.desc())
/*
select t0_.ADDRESS_ID, t0_.STREET, t0_.VERSION from ADDRESS as t0_ where t0_.ADDRESS_ID in (?, ?) order by t0_.ADDRESS_ID desc
*/
```

The `inList` operator also accepts a subquery.

```kotlin
QueryDsl.from(e).where {
    e.addressId inList {
        QueryDsl.from(a)
            .where {
                e.addressId eq a.addressId
                e.employeeName like "%S%"
            }.select(a.addressId)
    }
}
/*
select t0_.EMPLOYEE_ID, t0_.EMPLOYEE_NO, t0_.EMPLOYEE_NAME, t0_.MANAGER_ID, t0_.HIREDATE, t0_.SALARY, t0_.DEPARTMENT_ID, t0_.ADDRESS_ID, t0_.VERSION from EMPLOYEE as t0_ where t0_.ADDRESS_ID in (select t1_.ADDRESS_ID from ADDRESS as t1_ where t0_.ADDRESS_ID = t1_.ADDRESS_ID and t0_.EMPLOYEE_NAME like ? escape ?)
*/
```

### notInList {#comparison-operator-notinlist}

```kotlin
QueryDsl.from(a).where { a.addressId notInList (1..9).toList() }.orderBy(a.addressId)
/*
select t0_.ADDRESS_ID, t0_.STREET, t0_.VERSION from ADDRESS as t0_ where t0_.ADDRESS_ID not in (?, ?, ?, ?, ?, ?, ?, ?, ?) order by t0_.ADDRESS_ID asc
*/
```

The `notInList` operator also accepts a subquery.

```kotlin
QueryDsl.from(e).where {
    e.addressId notInList {
        QueryDsl.from(a).where {
            e.addressId eq a.addressId
            e.employeeName like "%S%"
        }.select(a.addressId)
    }
}
/*
select t0_.EMPLOYEE_ID, t0_.EMPLOYEE_NO, t0_.EMPLOYEE_NAME, t0_.MANAGER_ID, t0_.HIREDATE, t0_.SALARY, t0_.DEPARTMENT_ID, t0_.ADDRESS_ID, t0_.VERSION from EMPLOYEE as t0_ where t0_.ADDRESS_ID not in (select t1_.ADDRESS_ID from ADDRESS as t1_ where t0_.ADDRESS_ID = t1_.ADDRESS_ID and t0_.EMPLOYEE_NAME like ? escape ?)
*/
```

### inList2 {#comparison-operator-inlist2}

```kotlin
QueryDsl.from(a).where { a.addressId to a.version inList2 listOf(9 to 1, 10 to 1) }.orderBy(a.addressId.desc())
/*
select t0_.ADDRESS_ID, t0_.STREET, t0_.VERSION from ADDRESS as t0_ where (t0_.ADDRESS_ID, t0_.VERSION) in ((?, ?), (?, ?)) order by t0_.ADDRESS_ID desc
*/
```

The `inList2` operator also accepts a subquery.

```kotlin
QueryDsl.from(e).where {
    e.addressId to e.version inList2 {
        QueryDsl.from(a)
            .where {
                e.addressId eq a.addressId
                e.employeeName like "%S%"
            }.select(a.addressId, a.version)
    }
}
/*
select t0_.EMPLOYEE_ID, t0_.EMPLOYEE_NO, t0_.EMPLOYEE_NAME, t0_.MANAGER_ID, t0_.HIREDATE, t0_.SALARY, t0_.DEPARTMENT_ID, t0_.ADDRESS_ID, t0_.VERSION from EMPLOYEE as t0_ where (t0_.ADDRESS_ID, t0_.VERSION) in (select t1_.ADDRESS_ID, t1_.VERSION from ADDRESS as t1_ where t0_.ADDRESS_ID = t1_.ADDRESS_ID and t0_.EMPLOYEE_NAME like ? escape ?)
*/
```

### notInList2 {#comparison-operator-notinlist2}

```kotlin
QueryDsl.from(a).where { a.addressId to a.version notInList2 listOf(9 to 1, 10 to 1) }.orderBy(a.addressId)
/*
select t0_.ADDRESS_ID, t0_.STREET, t0_.VERSION from ADDRESS as t0_ where (t0_.ADDRESS_ID, t0_.VERSION) not in ((?, ?), (?, ?)) order by t0_.ADDRESS_ID asc
*/
```

The `notInList2` operator also accepts a subquery.

```kotlin
QueryDsl.from(e).where {
    e.addressId to e.version notInList2 {
        QueryDsl.from(a).where {
            e.addressId eq a.addressId
            e.employeeName like "%S%"
        }.select(a.addressId, a.version)
    }
}
/*
select t0_.EMPLOYEE_ID, t0_.EMPLOYEE_NO, t0_.EMPLOYEE_NAME, t0_.MANAGER_ID, t0_.HIREDATE, t0_.SALARY, t0_.DEPARTMENT_ID, t0_.ADDRESS_ID, t0_.VERSION from EMPLOYEE as t0_ where (t0_.ADDRESS_ID, t0_.VERSION) not in (select t1_.ADDRESS_ID, t1_.VERSION from ADDRESS as t1_ where t0_.ADDRESS_ID = t1_.ADDRESS_ID and t0_.EMPLOYEE_NAME like ? escape ?)
*/
```

### exists {#comparison-operator-exists}

```kotlin
QueryDsl.from(e).where {
    exists {
        QueryDsl.from(a).where {
            e.addressId eq a.addressId
            e.employeeName like "%S%"
        }
    }
}
/*
select t0_.EMPLOYEE_ID, t0_.EMPLOYEE_NO, t0_.EMPLOYEE_NAME, t0_.MANAGER_ID, t0_.HIREDATE, t0_.SALARY, t0_.DEPARTMENT_ID, t0_.ADDRESS_ID, t0_.VERSION from EMPLOYEE as t0_ where exists (select t1_.ADDRESS_ID, t1_.STREET, t1_.VERSION from ADDRESS as t1_ where t0_.ADDRESS_ID = t1_.ADDRESS_ID and t0_.EMPLOYEE_NAME like ? escape ?)
*/
```

### notExists {#comparison-operator-notexists}

```kotlin
QueryDsl.from(e).where {
    notExists {
        QueryDsl.from(a).where {
            e.addressId eq a.addressId
            e.employeeName like "%S%"
        }
    }
}
/*
select t0_.EMPLOYEE_ID, t0_.EMPLOYEE_NO, t0_.EMPLOYEE_NAME, t0_.MANAGER_ID, t0_.HIREDATE, t0_.SALARY, t0_.DEPARTMENT_ID, t0_.ADDRESS_ID, t0_.VERSION from EMPLOYEE as t0_ where not exists (select t1_.ADDRESS_ID, t1_.STREET, t1_.VERSION from ADDRESS as t1_ where t0_.ADDRESS_ID = t1_.ADDRESS_ID and t0_.EMPLOYEE_NAME like ? escape ?)
*/
```

## Logical operators {#logical-operator}

Logical operators are available in the Having, On, When, and Where [Declarations]({{< relref "#declaration" >}}).

### and {#logical-operator-and}

Expressions in the declaration are implicitly concatenated using the AND operator.

```kotlin
QueryDsl.from(a).where {
    a.addressId greater 1
    a.street startsWith "S"
    a.version less 100
}
/*
select t0_.ADDRESS_ID, t0_.STREET, t0_.VERSION from ADDRESS as t0_ where t0_.ADDRESS_ID > ? and t0_.STREET like ? escape ? and t0_.VERSION < ?
*/
```

To explicitly concatenate expression using the AND operator, pass a lambda expression to the `and` function.

```kotlin
QueryDsl.from(a).where {
  a.addressId greater 1
  and {
    a.street startsWith "S"
    a.version less 100
  }
}
/*
select t0_.ADDRESS_ID, t0_.STREET, t0_.VERSION from ADDRESS as t0_ where t0_.ADDRESS_ID > ? and (t0_.STREET like ? escape ? and t0_.VERSION < ?)
*/
```

### or {#logical-operator-or}

To concatenate expressions using the OR operator, pass a lambda expression to the `or` function.

```kotlin
QueryDsl.from(a).where {
  a.addressId greater 1
  or {
    a.street startsWith "S"
    a.version less 100
  }
}
/*
select t0_.ADDRESS_ID, t0_.STREET, t0_.VERSION from ADDRESS as t0_ where t0_.ADDRESS_ID > ? or (t0_.STREET like ? escape ? and t0_.VERSION < ?)
*/
```

### not {#logical-operator-not}

To use the NOT operator, pass a lambda expression to the `not` function.

```kotlin
QueryDsl.from(a).where {
    a.addressId greater 5
    not {
        a.addressId greaterEq 10
    }
}.orderBy(a.addressId)
/*
select t0_.ADDRESS_ID, t0_.STREET, t0_.VERSION from ADDRESS as t0_ where t0_.ADDRESS_ID > ? and not (t0_.ADDRESS_ID >= ?) order by t0_.ADDRESS_ID asc
*/
```

## Arithmetic operators {#arithmetic-operator}

The following operators are available as arithmetic operators:

- `+`
- `-`
- `*`
- `/`
- `%`

These operators are defined in `org.komapper.core.dsl.operator`.

The following is an example of using the `+` operator:

```kotlin
QueryDsl.update(a).set {
    a.version eq (a.version + 10)
}.where {
    a.addressId eq 1
}
/*
update ADDRESS as t0_ set VERSION = (t0_.VERSION + ?) where t0_.ADDRESS_ID = ?
*/
```

## Mathematical functions {#mathematical-function}

The following function is available:

- random

This function is defined in `org.komapper.core.dsl.operator`.

The following is an example of using the `random` function:

```kotlin
QueryDsl.from(a).orderBy(random())
/*
select t0_.ADDRESS_ID, t0_.STREET, t0_.VERSION from ADDRESS as t0_ order by random() asc
*/
```

## String functions {#string-function}

The following functions are available as string functions:

- concat
- substring
- lower
- upper
- trim
- ltrim
- rtrim

These functions are defined in `org.komapper.core.dsl.operator`.

The following is an example of using the `concat` function:

```kotlin
QueryDsl.update(a).set {
  a.street eq (concat(concat("[", a.street), "]"))
}.where {
  a.addressId eq 1
}
/*
update ADDRESS as t0_ set STREET = (concat((concat(?, t0_.STREET)), ?)) where t0_.ADDRESS_ID = ?
*/
```

## Aggregate functions {#aggregate-function}

The following functions are available as aggregate functions:

- avg
- count
- sum
- max
- min

These functions are defined in `org.komapper.core.dsl.operator`.

The expression obtained by the aggregate function call is intended to be used with the `having` or `select` function:

```kotlin
QueryDsl.from(e)
    .groupBy(e.departmentId)
    .having {
        count(e.employeeId) greaterEq 4L
    }
    .orderBy(e.departmentId)
    .select(e.departmentId, count(e.employeeId))
/*
select t0_.DEPARTMENT_ID, count(t0_.EMPLOYEE_ID) from EMPLOYEE as t0_ group by t0_.DEPARTMENT_ID having count(t0_.EMPLOYEE_ID) >= ? order by t0_.DEPARTMENT_ID asc
*/
```

### avg {#aggregate-function-avg}

```kotlin
QueryDsl.from(a).select(avg(a.addressId))
/*
select avg(t0_.ADDRESS_ID) from ADDRESS as t0_
*/
```

### count {#aggregate-function-count}

To generate a SQL `count(*)`, call the `count` function with no arguments:

```kotlin
QueryDsl.from(a).select(count())
/*
select count(*) from ADDRESS as t0_
*/
```

It is possible to pass a metamodel property to the `count` function:

```kotlin
QueryDsl.from(a).select(count(a.street))
/*
select count(t0_.STREET) from ADDRESS as t0_
*/
```

### sum {#aggregate-function-sum}

```kotlin
QueryDsl.from(a).select(sum(a.addressId))
/*
select sum(t0_.ADDRESS_ID) from ADDRESS as t0_
*/
```

### max {#aggregate-function-max}

```kotlin
QueryDsl.from(a).select(max(a.addressId))
/*
select max(t0_.ADDRESS_ID) from ADDRESS as t0_
*/
```

### min {#aggregate-function-min}

```kotlin
QueryDsl.from(a).select(min(a.addressId))
/*
select min(t0_.ADDRESS_ID) from ADDRESS as t0_
*/
```
## Conditional expression {#conditional-expression}

The following functions and expressions are available as conditional expressions:

- coalesce
- case

These are defined in `org.komapper.core.dsl.operator`.

### coalesce {#conditional-expression-coalesce}

The following is an example of using the `coalesce` function:

```kotlin
QueryDsl.from(a).select(a.addressId, coalesce(a.street, literal("default")))
/*
select t0_.ADDRESS_ID, coalesce(t0_.STREET, 'default') from ADDRESS as t0_
*/
```

### CASE expressions {#case-expression}

To use a CASE expression, call the `case` function:

```kotlin
val caseExpression = case(
  When(
    {
      a.street eq "STREET 2"
      a.addressId greater 1
    },
    literal("HIT")
  )
) { literal("NO HIT") }
val list: List<Pair<String?, String?>> = db.runQuery {
  QueryDsl.from(a).where { a.addressId inList listOf(1, 2, 3) }
    .orderBy(a.addressId)
    .select(a.street, caseExpression)
}
/*
select t0_.street, case when t0_.street = ? and t0_.address_id > ? then 'HIT' else 'NO HIT' end from address as t0_ where t0_.address_id in (?, ?, ?) order by t0_.address_id asc
*/
```

## Scalar subqueries {#scalar-subquery}

A query that returns a scalar using an aggregate function is a scalar subquery.
The scalar subquery can be passed to the `select` function of another query:

```kotlin
val subquery = QueryDsl.from(e).where { d.departmentId eq e.departmentId }.select(count())
val query = QueryDsl.from(d)
    .orderBy(d.departmentId)
    .select(d.departmentName, subquery)
/*
select t0_.department_name, (select count(*) from employee as t1_ where t0_.department_id = t1_.department_id) from department as t0_ order by t0_.department_id asc
*/
```

## Literals {#literal}

To embed a value directly into SQL as a literal without binding variable, call the `literal` function.

These functions are defined in `org.komapper.core.dsl.operator`.

The `literal` function supports the following argument types:

- Boolean
- Int
- Long
- String

Here is an example of literal function usage:

```kotlin
QueryDsl.insert(a).values {
  a.addressId eq 100
  a.street eq literal("STREET 100")
  a.version eq literal(100)
}
/*
insert into ADDRESS (ADDRESS_ID, STREET, VERSION) values (?, 'STREET 100', 100)
*/
```