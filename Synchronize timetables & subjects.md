## Synchronize ClassUp Timetables with Subjects

### Goal
- We have been running two database systems. One is Amazon **`Aurora`** is known as relational database MySQL. The other is Amazon **`DynamoDb`** is known as NoSQL.
- Because of data backup and performance, we created same tables in each databases. But now we do not use **`Aurora`** tables, so we decide to remove the tables.
- There is the most important reason. **`Aurora`** is so slow as subject datas increase. When we read something in subjects table, Computer always freeze up or it returns data too slowly. So we moved all datas to **`DynamoDb`** and have been running both systems. Now we make a decision removing  **`Aurora`** tables.

***
### Devices
- First, add 4 columns in default tables.(server_timestamp, update_timestamp, device_timestamp, status)
  - `server_timestamp` is for checking whether a data is synchronized and shows created time.
  - `update_timestamp` shows updated time.
  - `device_timestamp` shows created time in device.
  - `status` shows stat of data.(0 is created, 1 is updated, 2 is deleted)
  - ###### We have our technology about synchronization(ClassUp Synchronize tech).
  - ###### This way(Best way to database upgrade) is more stable than just add column using SQL.

- Second, divide 3 cases.
  - When existing users update.
  - When new users sign up.
  - When existing users update after logout.

***
#### When Existing users update.

This case is fully complicated. Because we have to keep all datas, It is really difficult to upgrade database.

    But we know the best way to upgrade database!

After finishing upgrade database, we synchronize all datas to server. We can know what datas need synchronization because `server_timestamp` is nil or null.

```Objective-c
- (void)readUnprocessedTimetable {
  // Read timetable datas that server_timestamp is nil.
}

- (void)postTimetable {
  NSDictionary *timetable_dict = [dbAdapter readUnprocessedTimetable];
  // Connect to server using timetable_dict.
}

- (void)readUnprocessedSubject {
  // Read subject datas that server_timestamp is nil.
}

- (void)postSubject {
  NSDictionary *subject_dict = [dbAdapter readUnProcessedSubject];
  // Connect to server using subject_dict.
}

- (void)connectionDidFinishLoading {
  if(isFinishSyncTimetable) {
    [dbAdapter insertOrReplaceTimetables];
    [self postSubject];
  }
  else if(isFinishSyncSubject) {
    [dbAdapter insertOrReplaceSubjects];
  }
}
```
We just get 4 columns' datas from server. So we using `INSERT OR REPLACE INTO` query!

After that, we change subject add function when users use search.

### Server
