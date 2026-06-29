# PostgreSQL Table Check Commands

## Show All Tables

``` sql
\dt
```

## Show Database List

``` sql
\l
```

## Connect Database

``` sql
\c phalel_db
```

## Show Table Structure

``` sql
\d "Video"
\d "User"
\d "Teacher"
\d "Course"
\d "Lesson"
\d "Category"
\d "VideoFolder"
```

## Check Data

``` sql
SELECT * FROM "Video";
SELECT * FROM "Video" LIMIT 10;

SELECT * FROM "User";
SELECT * FROM "Teacher";
SELECT * FROM "Course";
SELECT * FROM "Lesson";
SELECT * FROM "Category";
SELECT * FROM "VideoFolder";
```

## Count Records

``` sql
SELECT COUNT(*) FROM "Video";
SELECT COUNT(*) FROM "User";
SELECT COUNT(*) FROM "Teacher";
SELECT COUNT(*) FROM "Course";
SELECT COUNT(*) FROM "Lesson";
SELECT COUNT(*) FROM "Category";
SELECT COUNT(*) FROM "VideoFolder";
```

## Pretty Output

``` sql
\x
SELECT * FROM "Video" LIMIT 1;
\x
```

## Exit

``` sql
\q
```
