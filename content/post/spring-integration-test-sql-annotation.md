---
title: "Spring Integration Test @Sql Annotation and the Separation of the Statements"
date: 2019-09-19T10:39:31+03:00
lastmod: 2019-09-19T10:39:31+03:00
draft: false
keywords: ["spring", "spring integration test", "integration test", "jdbc", "flyway", "spring boot integration test", "DEFAULT_STATEMENT_SEPARATOR", "@Sql", "spring boot"]
description: "Spring integration test Sql annotation. How to separate individual SQL statements."
tags: ["spring", "sql", "spring framework", "sql server"]
categories: ["programming", "spring", "java", "tips"]
author: "Aziz Unsal"

# You can also close(false) or open(true) something for this content.

# P. S.comment can only be closed
comment: true
toc: true
autoCollapseToc: false
postMetaInFooter: false
hiddenFromHomePage: false

# You can also define another contentCopyright.e.g.contentCopyright: "This is another copyright."

contentCopyright: false
reward: false
mathjax: false
mathjaxEnableSingleDollar: false
mathjaxEnableAutoNumber: false

# You unlisted posts you might want not want the header or footer to show

hideHeaderAndFooter: false

# You can enable or disable out-of-date content warning for individual post.

# Comment this out to use the global config.
#enableOutdatedInfoWarning: false

flowchartDiagrams:
  enable: false
  options: ""

sequenceDiagrams: 
  enable: false
  options: ""

---

The `@Sql` annotation is a powerful Spring-specific annotation which you can use while creating integration tests. I've been using it for quite a while, it's easy to use and powerful.

Recently I came across a situation in which I had to insert a row into a database table before the integration test performed and of course I used `@Sql` annotation.
<!--more-->

Common usage of the `@Sql` annotation looks like following code block.

``` java
@Test
@Sql(scripts = "/cameras/sql/create_camera.sql")
public void testMethod {
    // execute code that relies on the test data
}
```

The detailed information can be found in [Spring Testing documentation](https://docs.spring.io/spring/docs/current/spring-framework-reference/testing.html#spring-testing-annotation-sql).

Here are the statements that I used to create a new row inside the provided SQL file, `create_cameras.sql` ; 

``` sql
DECLARE @camera_id bigint = NEXT VALUE FOR [dbo].[hibernate_sequence];
INSERT INTO [dbo].[camera] (id, ip_address, status, transfer_type, frame_rate)
  VALUES (@camera_id, '192.0.0.8', 'PENDING', 'RTSP', 0);
```

When the tests were executed on the CI server, I surprised that one of the integration tests was failed. And the failure was in the unit which I changed its SQL file as above.

The same statement runs smoothly on [Flyway](https://flywaydb.org/) migration scripts which I used to populate the database with some initial data, but it failed in Spring Framework integration test context.

The problem was;

```
org.springframework.jdbc.datasource.init.ScriptStatementFailedException:
Failed to execute SQL script statement #5 of class path resource [cameras/sql/create_cameras.sql]:
INSERT INTO [dbo].[camera] (id, ip_address, status, transfer_type, frame_rate)
  VALUES (@camera_id, '192.0.0.8', 'PENDING', 'RTSP', 0);
nested exception is com.microsoft.sqlserver.jdbc.SQLServerException: Must declare the scalar variable "@camera_id".
```

The important part of the error message is **must declare the scalar variable "@camera_id".**

Actually I did!

The Problem
--

After my investigation, I found that Spring Framework uses `;` char to separate individual statements within the SQL script unlike Flyway's [strategy](https://flywaydb.org/documentation/database/sqlserver#sql-script-syntax). In brief, Flyway uses `GO` for the statement separator by default.

You can read `org.springframework.test.context.jdbc.SqlConfig` JavaDoc [here](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/test/context/jdbc/SqlConfig.html#separator--).

This is the line that I looked for; 

> Implicitly defaults to "; " if not specified and falls back to "\n" as a last resort.

Conclusion
---

I modified the script like below and it worked perfectly.

``` sql
DECLARE @camera_id bigint = NEXT VALUE FOR [dbo].[hibernate_sequence]
INSERT INTO [dbo].[camera] (id, ip_address, status, transfer_type, frame_rate)
  VALUES (@camera_id, '192.0.0.8', 'PENDING', 'RTSP', 0)
;
```

And, yes the only thing that I changed was to remove `;` s from at the end of each statement and put a single `;` to the end of the block!

Done.
