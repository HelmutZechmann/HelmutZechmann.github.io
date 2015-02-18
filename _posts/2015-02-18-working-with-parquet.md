---
layout: post
title: Working with parquet files.
---

For working with parquet files you can use the [parquet-tools](https://github.com/Parquet/parquet-mr/tree/master/parquet-tools) from the parquet project.

If you want to use them locally, make sure to enable the *local* profile during the build:

```bash
cd parquet-tools && mvn clean package -Plocal 
```

Supported commands are

* ```head``` -n <number of records>: Display the first n records
* ```cat```: Cat the whole file
* ```dump```: Dump file contents
* ```meta```: Show metadata information for the file
* ```schema```: Display the  file schema

