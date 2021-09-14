- talk about my experience
- distill the more generalizable takeaways

At work, I recently was tasked with improving the performance of our database queries. 
The motivation was that we had recently experienced a number of outages in a short timespan, after having had no outages at all in the year I had been working there.
We were able to observe that our database cpu utilization was maxed out. Using AWS RDS Performance Insights (PI), we were able to see that a few queries were extremely non-performant.

PI shows you the average number of rows scanned per query, so it was easy to see that this wasn't strictly a problem of increased traffic; we knew upfront that our queries were bad. 

A problem though is that we didn't know exactly where those queries came from. Our app was written in scala, using [slick](https://scala-slick.org/) as our ORM (slick's landing page claims that it's not an ORM, but I believe the term covers slick's functionality pretty well and it's a widely familiar term, so I'll be using it). The problem is that the queries that slick builds typically are hard to read:

**TODO better example**
```sql
select x2.x3, x2.x4, x2.x5 from (select x5.x6 as x3, x7.x8, x12.x16 as x5) x2
...
```

So it wasn't immediately clear which codepaths were producing these queries. The process of investigation was basically looking at the human-readable table names in the query and using our own knowledge of the codebase to narrow down which functions make use of these tables. 

As a bit of a tangent: Given this non-optimal process, we wanted to find a better way to label our queries. Unfortunately, slick doesn't support comments. So we tried to hack around it by introducing a dummy `select where "my_query_label" is not null` into our statements. This had the undesired effect of catastrophically blowing up our performance. My understanding of why is that these dummy clauses were being inserted deep into nested queries which involved left anti-joins which are not indexable. Therefore every row in the table was being scanned. We rolled back the change, but didn't find a great solution for this. 

Some queries were easily identifable. Many of these were easily fixed by adding a simple index. 

For the rest, we were able to narrow them down to a few functions, and we found that the functions all called a helper function. 

**TODO** wasn't a major problem that the `query` was doing stuff too? like it wasn't just a base query but was doing extra stuff too?

```scala
code examples here
```

The problem is how the helper query is being inserted. Note that it doesn't take any parameters. So if you were to run it on its own, it would scan its whole table.
And that's essentially what happened. The `filter` happens _after_ the join and it's not optimized by the query planner presumably because it's involved in a left anti-join. So you end up having to scan against the entire table, instead of just against the small subset you care about.

We fixed this by instead of generating this "genearlized" query from the helper function, the helper function should take parameters to filter by,
and will generate a query with a `where` clause. This way, if an outer query has to scan it, then the scan is limited to a much smaller subset.

```scala
show code example of fix here
```

Another problem we ran into involved some abstractions we wrote.

```scala
shorthand query and baseQuery stuff
```

We have tables where we soft-delete records. "Deleting" really means changing a column `active` from `true` to `false` for example. 
Obviously, a programmer unfamiliar with this could easily include the deactivated rows in their queries if they forget to `filter(_.active)`.
So we had a pattern of overriding the `query` like `override def query = query.filter(_.active)` so that someone less familiar wouldn't have to remember to
filter every time.

This has unfortunate side effect however. When slick generates sql, the rough idea is that `MyTable.baseQuery` gets translated to `my_table`.
It's just the table name. However, if you override `query` - if you do any filtering at all - then you're actually generating a sql statement like so:
`(select * from my_table where active)` which is fundamentally different when writing a join:

```sql
select * from tbl1 join tbl2 on tbl1.x = tbl2.y
```

if you have good indexes, this join will be quite efficient.

However, if you use the overridden query, then you end up executing the inner statement. This was bad in our case because `active` has just two values, so it's low cardinality, and is therefore not helped much by indexing (imagine most of your data is active, then you essentially are doing an almost full table scan).

So we want only active records, but we want the less efficient query to happen later. To accomplish that we need to filter for `active` during/after the join, not before. So we couldn't use our `query` pattern for this and instead had to remember to do the join using `baseQuery`, and to also remember to filter for `active`. 

**TODO note the 180k vs 4 rows example**

So given these changes, our performance improved drastically. Instead of scanning on average 50k rows, we were now scanning about 10. 
The biggest issue/takeaway I got from this was that patterns we value in code like abstraction and code reuse can actually be counter-productive in sql. 
I grew up allergic to sql and databases, always using an ORM. My experience with rails and django also made it so that I didn't even have to write DDL either. I think this allowed me to develop a misunderstanding of how things worked for a long time. I've come to appreciate the idea that ORMs are not silver bullets, and for hotpaths, you will inevitably get closer and closer to writing raw sql.

While I appreciate the desire for raw sql in extreme scenarios, slick gives us such a huge benefit over raw sql strings. It's stupidly easy to typo your sql, but slick is powerfully type-checked. It's good to be aware of the pitfalls, but it's still an extremely useful tool for most cases. 

**TODO section on load testing**













