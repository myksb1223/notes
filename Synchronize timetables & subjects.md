## Synchronize ClassUp Timetables(Schedules) with Subjects(Classes)

Before we write this documents, it needs to define some words meaning.
Timetable is same with schedule.
Subject is same with class.
Because we use words timetable and subject when it is first developed, database table or codes are using these words. But we use schedule and class when we expain some models and relationship each other.

### Goal
- We have been running two database systems. One is Amazon **`Aurora`** is known as relational database MySQL. The other is Amazon **`DynamoDb`** is known as NoSQL.
- Because of data backup and performance, we created same tables in each databases. But now we do not use **`Aurora`** tables, so we decide to remove these tables.
- There is the most important reason. **`Aurora`** is so slow as subject datas increase. When we read something in subjects table, computer always freeze up or it returns data too slowly. So we moved all datas to **`DynamoDb`** and have been running both systems. Now we make a decision removing  **`Aurora`** tables.

***
### `unique_id` Schema
- Timetable
  - unique_id : (user_id)_(timestamp)
- Subject
  - unique_id : (user_id)_(timestamp)

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

After finishing upgrade database, we synchronize all datas to server. We can know what datas need synchronization because `server_timestamp` is nil or null. We also use this way when user create timetable and subject.

    When user create timetable and subject, `server_timestamp` is also nil or null.
    So when first synchronize, `update_timestamp` and `device_timestamp` are also nil or null.
    Server database already has these datas.
    So we have to check whether datas is already created in server.

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

##### Subject
After that, we change subject add function when users search subjects. In previous verison, users have to wait the processing finished. But now nothing waits to do. All functions do in main controller or activity.

There are three processes we have to change. One is adding subject. Another is editing subject. the other is deleting subject.

These processes are also same with subject search.

`status` field will be changed to 2 in delete process and 1 in edit process.

##### Timetable
In previous version, timetables are not synchronized to server. So changed datas are not saved in server. Many users have asked us why timetable infomation is not same with before. But now all information can be saved.

Timetable is also processed same ways in subject. But there is one thing we have to work. That is timetable background synchronization.

We will save a background to Amazon `S3`. So we have to add the function of that.

The function was writed down in `Send image to S3`.

***
### Server
Users make many schedules relate with their semesters. users also listen classes. So legacy server database table has users, timetables, subjects, enrollments.

In Legacy server, create subjects, create enrollments and delete enrollments function have been divided. But now all functions is integrated.

#### Legacy
```ruby
class TimeTablesController
  def create
  end
end

class SubjectsController
  def create
  end
end

class EnrollmentsController
  def create
  end

  def destroy
  end
end
```

#### Now
```ruby
class TimeTablesController
  def post_timetables
  end
end

class SubjectsController
  def post_subjects
    // This function contains subject create, enrollment create, destroy.
  end
end
```

Although functions are integrated, database tables are still divided. So `post_subjects` is more complicated than others.

- First, we have to parse datas inputted in server.
- Second, create subject using some of parsed datas. If datas' `update_timestamp` and `device_timestamp` are nil, get original data using `unique_id` in parsed datas and change some parsed values to original values.
- Third, create enrollment using datas remained. If datas' `update_timestamp` and `device_timestamp` are nil, do same upper case.
- At last, get all datas that `server_timestamp` is larger than `recent_timestamp` in parsed data from database.

We will save all things changed by users. So `enrollment` table in `DynamoDb` needs lots of fields more.(days, end time, each day for location)

When getting subjects user enrolled, the datas some fields will be updated by enrollment datas.

In SubjectsController, post_subjects process contains function that create subject datas to `ElasticSearch`. Before we just use
```ruby
@subject.__elasticsearch__.index_document
```

But now we change
```ruby
$client.bulk body: body
```

In `Aurora`, primary key type is `INTEGER` in subjects table. So server subject model's id type also `INTEGER`. We have to do deprecate it.

So, we create new index `subjects_v2` in `ElasticSearch`. `_id` field and `subject_id` field type will be changed to `String`.

We have to add subject data to `ElasticSearch` when users create, edit or enroll subject. We use `bulk` api to process them.



UserTimetable `subject_ids` field will be removed.

***
### ElasticSearch
