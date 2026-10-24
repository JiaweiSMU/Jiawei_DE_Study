Column oriented [[Data Lake File Format]].

It uses the `Column Layout` but in a single ==immutable== file
```sql
File
├── Row group 1   (a horizontal slice of rows, ~128 MB by default)
│   ├── Column chunk: order_id   → pages (~1 MB each)
│   ├── Column chunk: region     → pages
│   └── Column chunk: amount     → pages
├── Row group 2
│   └── ...
└── Footer (schema, byte offset of every column chunk, min/max per chunk)
```
A Parquet file splits a table three times. Example: a table with 200 rows.
- **Row Group:** a set of rows.
  - `e.g.`: Row Group 1 holds rows 1-100, Row Group 2 holds rows 101-200.
- **Column Chunk:** the values of one column, for the rows in one row group.
  - `e.g.`: In Row Group 1, the `amount` column chunk holds the `amount` values for rows 1-100.
- **Page:** a smaller piece of one column chunk. Each page is compressed on its own.
  - `e.g.`: The `amount` column chunk in Row Group 1 is cut into 4 pages: `amount` values for rows 1-25, 26-50, 51-75, 76-100.
```
Row Group 1 (rows 1-100)
├── order_id chunk → Page 1 (rows 1-25), Page 2 (26-50), Page 3 (51-75), Page 4 (76-100)
├── region chunk   → Page 1, Page 2, ...
└── amount chunk   → Page 1, Page 2, ...
Row Group 2 (rows 101-200)
└── ...
```