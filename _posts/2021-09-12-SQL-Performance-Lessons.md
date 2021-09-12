At work, I recently was tasked with improving the performance of our database queries. 
The motivation was that we had recently experienced a number of outages in a short timespan, after having had no outages at all in the year I had been working there.
We were able to observe that our database cpu utilization was maxed out. Using AWS RDS Performance Insights, we were able to see that a few queries were extremely non-performant.


objectives with writing this
- handling the nebulous task of "optimize"
- determining the source of slowness
- usefulness of adding indexes (opine on how we were a sql noob)
- how i tested/verified changes (load testing)
- pitfall of orm / code patterns vs sql patterns

- sql was generated and was hard to read - so kind of a pain in the ass to pin down the source
-   well-intentioned choice to hack in "comments" into the sql using a dummy "select where 'table_name' is not null" - resulted in catastrophic performance blowup lol

- some very simple cases were just adding indexes

- some more involved cases were joins and figuring out why the joins didn't do well
-   left anti-join can't be optimized
-   orm was building generic queries b/c of how were using it... so you'd end up with a full-table scan in a sub-query inside an anti-join
-   solution was to refactor the code so that the filters were applied early











