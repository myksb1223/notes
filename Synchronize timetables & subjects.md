## Synchronize ClassUp Timetables with Subjects

### Goal
- We have been running two database systems. One is Amazon **`Aurora`** is known as relational database MySQL. The other is Amazon **`DynamoDb`** is known as NoSQL.
- Because of data backup and performance, we created same tables in each databases. But now we do not use **`Aurora`** tables, so we decide to remove the tables.
- There is the most important reason. **`Aurora`** is so slow as subject datas increase. When we read something in subjects table, Computer always freeze up or it returns data too slowly. So we moved all datas to **`DynamoDb`** and have been running both systems. Now we make a decision removing  **`Aurora`** tables.
