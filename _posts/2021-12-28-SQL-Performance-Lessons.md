At a previous job, I had the pleasure of improving our web app's performance. 
The motivation was that we had recently experienced a number of outages in a short timespan, after having had no outages at all in the year I had been working there at that point.
We were able to observe that our database cpu utilization was maxed out. Using AWS RDS Performance Insights (PI), we were able to see that a few queries were extremely non-performant.

PI shows you the average number of rows scanned per query, so it was easy to see that this wasn't strictly a problem of increased traffic; we knew upfront that our queries were bad. 

A problem though is that we didn't know exactly where those queries came from. Our app was written in scala, using [slick](https://scala-slick.org/) as our ORM (slick's landing page claims that it's not an ORM, but I believe the term covers slick's functionality pretty well and it's a widely familiar term, so I'll be using it). The problem is that the queries that slick builds typically are hard to read:

```sql
select x2.x3, x2.x4, x2.x5 from (select x5.x6 as x3, x7.x8, x12.x16 as x5) x2
join (select id, status from thing_status) x15
...
```

(This isn't a literal example, but the illegibility should hopefully be clear)

So it wasn't immediately clear which codepaths were producing these queries. The process of investigation was basically looking at the human-readable table names in the query and using our own knowledge of the codebase to narrow down which functions make use of these tables. 

As a bit of a tangent: Given this non-optimal process, we wanted to find a better way to label our queries. Unfortunately, slick doesn't support comments. So we tried to hack around it by introducing a dummy `select where 'my_query_label' is not null` into our statements. This had the undesired effect of catastrophically blowing up our performance. My understanding of why is that these dummy clauses were being inserted deep into nested queries which involved left anti-joins which are not indexable. Therefore every row in the table was being scanned. We rolled back the change, but didn't find a great solution for this.
In the meantime though, this hack did help identify some functions.

Some of the queries were simple and easily identifable. Many of these were easily fixed by adding a simple index. 

For the rest, we were able to narrow them down to a few functions, and we found that the functions all called a helper function. 

```scala
def helperFunctionQuery() = {
    table1.query
    .join(table2.query)
    .on(table1.x === table2.y)
    .join(user.query)
    .on(_._2.userId === _.id)
    ...
}

def findStuffForUser(userId: UserId) = {
  helperFunctionQuery()
  .join(otherTable.query)
  .on(_._1.x === _.y)
  .filter(_._1._3.id === userId)
  .joinLeft(anotherTable.query)
  .on(_.xyz === _.abc)
  .filter(_._2.isEmpty)
}
```

This is an example, and the code may be difficult to read for those unfamiliar with scala, so the main idea is that ultimately we want to find "stuff" for a given user. We join some tables together and filter by the user's id. We then do a left anti-join against another table to eliminate some rows.
The important part is that the helper query doesn't do any filtering of its own. If it were to run on its own, it would do a large scan against much of the user table.

And that's essentially what happened. The `filter` happens _after_ the join. Because it's used with a left anti-join, which can't be optimized, we end up running the near-full-table scan in the subquery before filtering for the single user we care about.

We were able to fix this by parameterizing the helper function.\


Another problem we ran into involved some abstractions we wrote.

```scala
override def query = baseQuery.filter(_.active)
```

We have tables where we soft-delete records. "Deleting" really means changing a column `active` from `true` to `false` for example. 
Obviously, a developer unfamiliar with this fact could easily include the deactivated rows in their queries if they forget to `filter(_.active)`.
So we had a pattern of overriding the `query` so that someone less familiar wouldn't have to remember to filter every time.

This has an unfortunate side effect however. When slick generates sql, the rough idea is that `MyTable.baseQuery` gets translated to `my_table`.
It's just the table name. However, if you override `query` - if you do any filtering at all - then you're actually generating a sql statement like so:
`(select * from my_table where active)` which is fundamentally different when writing a join:

```sql
select * from tbl1 join tbl2 on tbl1.x = tbl2.y
```
If you have good indexes, this join will be quite efficient.

vs

```sql
select * from tbl1 join (select * tbl2 where active) on tbl1.x = tbl2.y
```

Here, we are joining against a nested query. This was bad in our case because `active` has just two values, so it's low cardinality, and is therefore not helped much by indexing (imagine most of your data is active, then you essentially are doing an almost full table scan).

So we want only active records, but we want the less efficient query to happen later. To accomplish that we need to filter for `active` during/after the join, not before. So we couldn't use our `query` pattern for this and instead had to remember to do the join using `baseQuery`, and to also remember to filter for `active`. 

So given these changes, our performance improved drastically. Instead of scanning on average 50k rows, we were now scanning about 10. In the most extreme example, we went from scanning 180k rows to 4.

The biggest issue/takeaway I got from this was that patterns we value in code like abstraction and code reuse can actually be counter-productive in sql. 
I grew up allergic to sql and databases, always using an ORM. My experience with rails and django also made it so that I didn't even have to write DDL either. I think this allowed me to develop a misunderstanding of how things worked for a long time. I've come to appreciate the idea that ORMs are not silver bullets, and for hotpaths, you will inevitably get closer and closer to writing raw sql.

While I appreciate the desire for raw sql in some scenarios, slick gave us such a huge benefit over raw sql strings. It's stupidly easy to typo your sql, but slick is powerfully type-checked. It's good to be aware of the pitfalls, but it's still an extremely useful tool for many cases. 














