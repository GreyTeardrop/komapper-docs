---
title: "UPDATEクエリ"
linkTitle: "更新"
weight: 30
description: 更新のためのクエリ
---

## 概要 {#overview}

UPDATEクエリは`QueryDsl`の`update`とそれに続く関数を呼び出して構築します。

クエリ実行時にキーが重複した場合、`org.komapper.core.UniqueConstraintException`がスローされます。

## single {#single}

エンティティ1件を更新するには`single`を呼び出します。

```kotlin
val address: Address = ..
val query: Query<Address> = QueryDsl.update(a).single(address)
/*
update ADDRESS set STREET = ?, VERSION = ? + 1 where ADDRESS_ID = ? and VERSION = ?
*/
```

このクエリを実行した場合の戻り値は更新されたデータを表す新しいエンティティです。

下記のマッピング定義に応じて、発行されるSQLにも新しいエンティティにも適切な値が反映されます。

- `@KomapperId`
- `@KomapperVersion`
- `@KomapperUpdatedAt`

クエリ実行時に楽観的排他制御が失敗した場合、`org.komapper.core.OptimisticLockException`がスローされます。

クエリ実行時にエンティティが見つからなかった場合、`org.komapper.core.EntityNotFoundException`がスローされます。

## batch {#batch}

バッチでエンティティ複数件を更新するには`batch`を呼び出します。

```kotlin
val address1: Address = ..
val address2: Address = ..
val address3: Address = ..
val query: Query<List<Address>> = QueryDsl.update(a).batch(address1, address2, address3)
/*
update ADDRESS set STREET = ?, VERSION = ? + 1 where ADDRESS_ID = ? and VERSION = ?
update ADDRESS set STREET = ?, VERSION = ? + 1 where ADDRESS_ID = ? and VERSION = ?
update ADDRESS set STREET = ?, VERSION = ? + 1 where ADDRESS_ID = ? and VERSION = ?
*/
```

このクエリを実行した場合の戻り値は更新されたデータを表す新しいエンティティのリストです。

下記のマッピング定義に応じて、発行されるSQLにも新しいエンティティにも適切な値が反映されます。

- `@KomapperVersion`
- `@KomapperUpdatedAt`

クエリ実行時に楽観的排他制御が失敗した場合、`org.komapper.core.OptimisticLockException`がスローされます。

クエリ実行時にエンティティが見つからなかった場合、`org.komapper.core.EntityNotFoundException`がスローされます。

## include {#include}

[single]({{< relref "#single" >}}) や [batch]({{< relref "#batch" >}})
の実行で特定のプロパティのみを更新対象に含めるには事前に`include`関数を呼び出します。

```kotlin
val department: Department = ..
val query: Query<Department> = QueryDsl.update(d).include(d.departmentName).single(department)
/*
update DEPARTMENT set DEPARTMENT_NAME = ?, VERSION = ? + 1 where DEPARTMENT_ID = ? and VERSION = ?
*/
```

`include`関数の呼び出しに関係なく、 以下のアノテーションで注釈されたプロパティは常に更新対象に含まれます。

- `@KomapperVersion`
- `@KomapperUpdatedAt`


## exclude {#exclude}

[single]({{< relref "#single" >}}) や [batch]({{< relref "#batch" >}})
の実行で特定のプロパティを更新対象から除外するには事前に`exclude`関数を呼び出します。

```kotlin
val department: Department = ..
val query: Query<Department> = QueryDsl.update(d).exclude(d.departmentName).single(department)
/*
update DEPARTMENT set DEPARTMENT_NO = ?, LOCATION = ?, VERSION = ? + 1 where DEPARTMENT_ID = ? and VERSION = ?
*/
```

`exclude`関数の呼び出しに関係なく、 以下のアノテーションで注釈されたプロパティは常に更新対象に含まれます。

- `@KomapperVersion`
- `@KomapperUpdatedAt`

## set {#set}

任意のプロパティに更新データをセットするには`set`関数にラムダ式を渡します。

ラムダ式の中では`eq`関数を使って値を設定できます。

```kotlin
val query: Query<Long> = QueryDsl.update(a).set {
  a.street eq "STREET 16"
}.where {
  a.addressId eq 1
}
/*
update ADDRESS as t0_ set STREET = ? where t0_.ADDRESS_ID = ?
*/
```

`eqIfNotNull`を使って値が`null`でない場合にのみ値を設定することもできます。

```kotlin
val query: Query<Long> = QueryDsl.update(e).set {
  e.managerId eqIfNotNull managerId
  e.employeeName eq "test"
}.where {
  e.employeeId eq 1
}
```

これらのクエリを実行した場合の戻り値は更新された件数です。

以下のマッピング定義を持つプロパティについて明示的に`eq`を呼び出さない場合、発行されるSQLに自動で値が設定されます。
明示的に`eq`を呼び出した場合は明示した値が優先されます。

- `@KomapperVersion`
- `@KomapperUpdatedAt`

## where {#update-where}

任意の条件にマッチする行を更新するには`where`を呼び出します。

```kotlin
val query: Query<Long> = QueryDsl.update(a).set {
  a.street eq "STREET 16"
}.where {
  a.addressId eq 1
}
/*
update ADDRESS as t0_ set STREET = ? where t0_.ADDRESS_ID = ?
*/
```

デフォルトではWHERE句の指定は必須でありWHERE句が指定されない場合は例外が発生します。
意図的に全件更新を認める場合は`options`を呼び出して`allowMissingWhereClause`に`true`を設定します。

```kotlin
val query: Query<Long> = QueryDsl.update(e).set {
    e.employeeName eq "ABC"
}.options { 
    it.copy(allowMissingWhereClause = true)
}
```

このクエリを実行した場合の戻り値は更新された件数です。

## returning

以下の関数の後続で`returning`関数を呼び出すことで、更新された値を取得できます。

- single
- set

`single`関数の後続で`returning`関数を呼び出す例です。

```kotlin
val address: Address = ..
val query: Query<Address?> = QueryDsl.update(a).single(address).returning()
/*
update ADDRESS set STREET = ?, VERSION = ? + 1 where ADDRESS_ID = ? and VERSION = ? returning ADDRESS_ID, STREET, VERSION
*/
```

`returning`関数にプロパティを指定することで取得対象のカラムを限定できます。

```kotlin
val query: Query<Int?> = QueryDsl.update(a).single(address).returning(a.addressId)
/*
update ADDRESS set STREET = ?, VERSION = ? + 1 where ADDRESS_ID = ? and VERSION = ? returning ADDRESS_ID
*/
```

```kotlin
val query: Query<Pair<Int?, String?>?> = QueryDsl.update(a).single(address).returning(a.addressId, a.street)
/*
update ADDRESS set STREET = ?, VERSION = ? + 1 where ADDRESS_ID = ? and VERSION = ? returning ADDRESS_ID, STREET
*/
```

```kotlin
val query: Query<Triple<Int?, String?, Int?>?> = QueryDsl.update(a).single(address).returning(a.addressId, a.street, a.version)
/*
update ADDRESS set STREET = ?, VERSION = ? + 1 where ADDRESS_ID = ? and VERSION = ? returning ADDRESS_ID, STREET, VERSION
*/
```

{{< alert color="warning" title="Warning" >}}
`returning`関数は次のDialectでのみサポートされています。
- Oracle Database
- PostgreSQL
- SQL Server
{{< /alert >}}

## options

クエリの挙動をカスタマイズするには`options`を呼び出します。
ラムダ式のパラメータはデフォルトのオプションを表します。
変更したいプロパティを指定して`copy`メソッドを呼び出してください。

```kotlin
val address: Address = ..
val query: Query<Address> = QueryDsl.update(a).single(address).options {
    it.copy(
      queryTimeoutSeconds = 5
    )
}
```

指定可能なオプションには以下のものがあります。

allowMissingWhereClause
: 空のWHERE句を認めるかどうかです。デフォルトは`false`です。

escapeSequence
: LIKE句に指定されるエスケープシーケンスです。デフォルトは`null`で`Dialect`の値を使うことを示します。

batchSize
: バッチサイズです。デフォルトは`null`です。

disableOptimisticLock
: 楽観的ロックを無効化するかどうかです。デフォルトは`false`です。この値が`true`のときWHERE句にバージョン番号が含まれません。

queryTimeoutSeconds
: クエリタイムアウトの秒数です。デフォルトは`null`でドライバの値を使うことを示します。

suppressLogging
: SQLのログ出力を抑制するかどうかです。デフォルトは`false`です。

suppressOptimisticLockException
: 楽観的ロックの取得に失敗した場合に`OptimisticLockException`のスローを抑制するかどうかです。デフォルトは`false`です。

suppressEntityNotFoundException
: エンティティが見つからない場合に`EntityNotFoundException`のスローを抑制するかどうかです。デフォルトは`false`です。

[executionOptions]({{< relref "../../database-config/#executionoptions" >}})
の同名プロパティよりもこちらに明示的に設定した値が優先的に利用されます。
