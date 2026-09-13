### Using AS
e.g:
```
1. select name **AS full_name**
2. select * from users **AS u**
```

### Using Expressions in SELECT
Expression: Combination of values, operators & functions for the database to evaluate to produce a result

e.g.:
```
select 
	**price + tax** AS total_price
	quantity * unit_price AS total_cost
from orders
```
