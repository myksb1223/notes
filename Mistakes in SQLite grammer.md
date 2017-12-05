## Mistakes in SQLite grammer

After I changed `unique_id` field type to `String`, I got some errors.
  - I couldn't read anything from the table. I spended lots of time to solve this problem.

This source code before I solved this problem shows in below.
```objective-c
NSString *sqlStatementNS = [[NSString alloc] initWithString:[NSString stringWithFormat:@"UPDATE timeTable SET isMain = ? WHERE unique_id = %@", find]];
```

In SQLite, `String` value must be covered by single quota.

So, I changed the code like this.
```objective-c
NSString *sqlStatementNS = [[NSString alloc] initWithString:[NSString stringWithFormat:@"UPDATE timeTable SET isMain = ? WHERE unique_id = '%@''", find]];
```

`''` is really important in SQLite.
