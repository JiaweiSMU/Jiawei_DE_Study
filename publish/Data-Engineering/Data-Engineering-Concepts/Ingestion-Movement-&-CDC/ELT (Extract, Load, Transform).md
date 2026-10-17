Data is extracted from the source system and ==loaded as-is== data before being transformed.
- Raw data is generally kept, making it easy to redo the transformation whenever logic changes
- Data ==Schema does not have to be pre-defined== ([[Schema On Read]]) until the data is being **queried**.
- It is mainly used in a [[Data Lake]]