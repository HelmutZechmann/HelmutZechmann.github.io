---
layout: post
title: Processing Nested Data In Hadoop
tags: [avro, hive, pig, nested]
---

In traditional relational database systems data structures always should follow the first normal form. The first normal form demands that each attribute of an entity only contains atomic values. This is usually achieved by distributing data among multiple tables. Joins are being used to retrieve information from multiple tables.

While relational databases have excellent support for joining multiple datasets efficiently, in big data systems such as *apache hadoop* joins usually are very expensive, inefficient operations. Because of this limitation it often makes sense to store data in a semi-structured manner that does not follow the the first normal form.

This article shows how to store and process semi-structured data using data attributes of the types `map` and `list` in the hadoop ecosystem. For illustration purposes we use a data structure that contains annotations about apps. The data structure has the following attributes:

* `name`: the name of the app
* `attributes`: a map for storing arbitrary key value pairs
* `tags`: a list of tags

## A Schema For Semi-structured Data

The [apache avro](http://avro.apache.org/) project provides a data format for storing semi-structured data. The avro format is supported by all major projects in the hadoop ecosystem. It offers support for [list types](http://avro.apache.org/docs/1.7.5/spec.html#Arrays) as well as support for [maps](http://avro.apache.org/docs/1.7.5/spec.html#Maps).

The following snippet defines an avro schema for our example data structure:

```
{
  "namespace": "example.avro",
  "type": "record",
  "name": "app",
  "fields": [
    {"name": "name", "type": "string"},
    {"name": "attributes", "type": {"type": "map", "values": "string"}},
    {"name": "tags", "type": {"type": "array", "items": "string"}}
  ]
}
```

The keys of an avro map have the type `string`. In our example the values also are strings.
The data type for lists is called `array`. The list of tags is also of type string, it may have arbitrary length.

## Nested Data In Hive

The apache hive project supports mapping avro data to tables (see [hive avro docs](https://cwiki.apache.org/confluence/display/Hive/AvroSerDe)).
We can simply declare a table that uses our avro schema for the definition of the table structure.

### Create Table

The following hive statement creates the table for our app data:

```
CREATE TABLE apps
  ROW FORMAT SERDE
  'org.apache.hadoop.hive.serde2.avro.AvroSerDe'
  STORED AS INPUTFORMAT
  'org.apache.hadoop.hive.ql.io.avro.AvroContainerInputFormat'
  OUTPUTFORMAT
  'org.apache.hadoop.hive.ql.io.avro.AvroContainerOutputFormat'
  TBLPROPERTIES (
    'avro.schema.url'='hdfs:///demo/app.avsc');
```
Once the table is created, we can inspect the table structure:

```
DESCRIBE apps;

+-----------------------+-----------------------+-----------------------+
|       col_name        |       data_type       |        comment        |
+-----------------------+-----------------------+-----------------------+
| name                  | string                | from deserializer     |
| attributes            | map<string,string>    | from deserializer     |
| tags                  | array<string>         | from deserializer     |
+-----------------------+-----------------------+-----------------------+
```

### Put Data Into The Table

For demonstration purposes we use an existing avro file created with a small java application and put in into the hdfs folder that holds the data of our new hive table:

```
hdfs dfs -put avro-demo/nested.avro /user/hive/warehouse/demo.db/apps;
```

To inspect the data we can issue a simple `SELECT` statement:

```
SELECT * FROM apps;
+------------------+------------------------------------------+------------------------------------------+
|       name       |                attributes                |                   tags                   |
+------------------+------------------------------------------+------------------------------------------+
| Angry Birds      | {"category":"game","publisher":"rovio"}  | ["entertainment","casual"]               |
| Microsoft Excel  | {"category":"office"}                    | ["office","spreadsheet","productivity"]  |
+------------------+------------------------------------------+------------------------------------------+
```

As you can see the sample data contains two records. The first record has two attributes and two tags, the second record has only one attribute an three tags.

### Query Nested Data

Hive provides some special functions for working with complex data types. The following table of functions is taken from the [hive documentation](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+UDF#LanguageManualUDF-CollectionFunctions):

<table>
    <tr>
        <th>Return Type</th>
        <th>Name(Signature)</th>
        <th>Description</th>
    </tr>
    <tr>
        <td>array&lt;K&gt;</td>
        <td>map_keys(Map&lt;K, V&gt;)</td>
        <td>Returns an unordered array containing the keys of the input map.</td>
    </tr>
    <tr>
        <td>array&lt;T&gt;</td>
        <td>sort_array(Array&lt;T&gt;)</td>
        <td>Sorts the input array in ascending order according to the natural ordering of the array elements and returns it (as of version 0.9.0).</td>
    </tr>
    <tr>
        <td>array&lt;V&gt;</td>
        <td>map_values(Map&lt;K, V&gt;)</td>
        <td>Returns an unordered array containing the values of the input map.</td>
    </tr>
    <tr>
        <td>boolean</td>
        <td>array_contains(Array&lt;T&gt;, value)</td>
        <td>Returns TRUE if the array contains value.</td>
    </tr>
    <tr>
        <td>int</td>
        <td>size(Array&lt;T&gt;)</td>
        <td>Returns the number of elements in the map type.</td>
    </tr>
</table>

### Query Arrays In Hive

The following example demonstrates how to query our data for records with the tag *office* using the `array_contains` function:

```
SELECT *
FROM apps
WHERE array_contains(tags, "office");

+------------------+------------------------+------------------------------------------+
|       name       |       attributes       |                   tags                   |
+------------------+------------------------+------------------------------------------+
| Microsoft Excel  | {"category":"office"}  | ["office","spreadsheet","productivity"]  |
+------------------+------------------------+------------------------------------------+
```

### Query Maps In Hive

The second example finds all apps that are known to be published by a publisher called *rovio*. Therefore we use the `[]`-operator for accessing map entries.

```
SELECT *
FROM apps
WHERE attributes['publisher'] = 'rovio';

+--------------+------------------------------------------+-----------------------------+
|     name     |                attributes                |            tags             |
+--------------+------------------------------------------+-----------------------------+
| Angry Birds  | {"category":"game","publisher":"rovio"}  | ["entertainment","casual"]  |
+--------------+------------------------------------------+-----------------------------+
```

## Nested Data In Pig

Access to the hive-mapped data is not limited to hive. This section shows how to access our data using pig. Therefore we leverage the [pig hcatalog loader](https://cwiki.apache.org/confluence/display/Hive/HCatalog+LoadStore), especially the support for for handling [complex types](https://cwiki.apache.org/confluence/display/Hive/HCatalog+LoadStore#HCatalogLoadStore-ComplexTypes). Our avro list gets loaded into a pig tuple, avro maps are loaded into pig maps.

The following interactive pig session illustrates this. First we start a pig session with hcatalog access enabled:

```
pig -useHCatalog
```

In the next step we load our example data and inspect it:

```
grunt> APPS = LOAD 'demo.apps' USING org.apache.hcatalog.pig.HCatLoader();
grunt> describe APPS;
APPS: {name: chararray,attributes: map[],tags: {innertuple: (innerfield: chararray)}}
grunt> dump APPS;
(Angry Birds,[category#game,publisher#rovio],{(entertainment),(casual)})
(Microsoft Excel,[category#office],{(office),(spreadsheet),(productivity)})
```

### Query Arrays In Pig

In our first pig based analysis we find again all apps that are tagged with *office*. Therefore we use the `flatten` function to convert the *tags*-bag to tuples:

```
grunt>APPS_FLAT = FOREACH APPS GENERATE name, attributes, flatten(tags) AS tag;
grunt> OFFICE_APPS = FILTER APPS_FLAT BY tag == 'office';
grunt> DUMP OFFICE_APPS;
(Microsoft Excel,[category#office],office)
```

### Query Maps In Pig

In the second pig example we query our data again for apps published by *rovio*:

```
grunt> ROVIO_APPS = FILTER APPS BY attributes#'publisher' == 'rovio';
grunt> dump ROVIO_APPS;
(Angry Birds,[category#game,publisher#rovio],{(entertainment),(casual)})
```

## Conclusion

This article showed the basic concepts of processing nested data based on the avro file format with hive and pig. Of course there are also other file formats (e.g. [apache parquet](http://parquet.apache.org/)) that support nested data.

On the processing side there are also many other tools (e.g. [apache drill](https://drill.apache.org/)) with build in support for nested data structures.
