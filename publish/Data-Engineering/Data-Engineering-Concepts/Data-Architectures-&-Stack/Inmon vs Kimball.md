
|                           | [[Inmon]]                                     | [[Kimball]]                                        |
| ------------------------- | --------------------------------------------- | -------------------------------------------------- |
| Central model             | [[Normalized]] (3NF)                          | Dimensional (star schemas)                         |
| Where integration happens | In the central warehouse, once                | In the conformed dimensions, during the load       |
| Marts                     | Derived from the warehouse                    | Are the warehouse                                  |
| Build order               | Enterprise model first ("top-down")           | One business process at a time ("bottom-up")       |
| First usable report       | Months or more                                | Weeks                                              |
| History                   | Timestamped rows in the normalized centre     | Slowly changing dimensions with surrogate keys     |
| Adding a new source       | Extend the normalized model; marts unaffected | May require rebuilding and backfilling fact tables |
1. Inmon
2. Kimball
![[Pasted image 20261003232909.png]]