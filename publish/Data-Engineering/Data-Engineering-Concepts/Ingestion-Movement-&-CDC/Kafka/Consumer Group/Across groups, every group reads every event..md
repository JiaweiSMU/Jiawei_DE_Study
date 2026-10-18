### Example:
The topic `orders` has one partition with five events:
```
offset:   0      1      2      3      4
event:   [o101] [o102] [o103] [o104] [o105]
```
Two **groups** read this topic:
- the **Redshift loader**, which runs in group `redshift-loader`
- the **fraud check**, which runs in group `fraud-check`
#### What happens
> [!NOTE] Bookmark
> It refers to the `offset` (e.g. 1, 2, 3) the consumer will read next
- The Redshift loader reads o101, o102 and o103. Its bookmark moves to 3.
- The fraud check starts reading. It ==does **not** start at 3==. It has its own bookmark, which starts at 0, so it reads o101 too.
Later the positions look like this:
```
offset:          0      1      2      3      4
event:          [o101] [o102] [o103] [o104] [o105]
                                      ▲              ▲
                        redshift-loader is here     fraud-check is here (finished)
```
The fraud check has read all five events. The Redshift loader has read three and will read o104 and o105 next. ==**Both end up processing every order.**==

