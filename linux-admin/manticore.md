# Manticore
Manticore Search is a high-performance, open-source search engine designed to offer fast and precise full-text searches, along with other search-related functionalities. It originated as a fork of the Sphinx Search engine and has evolved to include features like full-text search, real-time indexing, and support for various data sources such as MySQL, PostgreSQL, ODBC, and CSV files. Manticore Search is known for its cost-efficiency, scalability, and SQL-like query language, making it a viable alternative to other search engines like Elasticsearch. It also supports advanced features such as replication, load balancing, and data backup, catering to the needs of modern, data-driven applications​.
## Terms
#### Data types
#### Full-text fields and attributes
Manticore's data types can be split into two categories: full-text fields and attributes.
#### Full-text fields

- can be indexed with natural language processing algorithms, therefore can be searched for keywords
- cannot be used for sorting or grouping
- original document's content can be retrieved
- original document's content can be used for highlighting

Full-text fields are represented by the data type text. All other data types are called "attributes".

Text fields: These are the primary data types used for full-text search. The content of text fields is processed, tokenized, and indexed to enable full-text search capabilities. Text fields are used to store the main body of content you wish to search through, such as article contents, product descriptions, etc.

Text is passed through an analyzer pipeline that converts the text to words, applies morphology transformations, etc. Eventually, a full-text table (a special data structure that enables quick searches for a keyword) gets built from that text.

#### Attributes
Attributes are non-full-text values associated with each document that can be used to perform non-full-text filtering, sorting and grouping during a search.

It is often desired to process full-text search results based not only on matching document ID and its rank, but also on a number of other per-document values. For example, one might need to sort news search results by date and then relevance, or search through products within a specified price range, or limit a blog search to posts made by selected users, or group results by month. To do this efficiently, Manticore enables not only full-text fields, but also additional attributes to be added to each document. These attributes can be used to filter, sort, or group full-text matches, or to search only by attributes.

The attributes, unlike full-text fields, are not full-text indexed. They are stored in the table, but it is not possible to search them as full-text.

A good example for attributes would be a forum posts table. Assume that only the title and content fields need to be full-text searchable - but that sometimes it is also required to limit search to a certain author or a sub-forum (i.e., search only those rows that have some specific values of author_id or forum_id); or to sort matches by post_date column; or to group matching posts by month of the post_date and calculate per-group match counts.
```sql
CREATE TABLE forum(title text, content text, author_id int, forum_id int, post_date timestamp);
```
This example shows running a full-text query filtered by author_id, forum_id and sorted by post_date.
```sql
select * from forum where author_id=123 and forum_id in (1,3,7) order by post_date desc
```

#### Character data types
string|text [stored|attribute] [indexed]

- indexed - full-text indexed (can be used in full-text queries)
- stored - stored in a docstore (stored on disk, not in RAM, lazy read)
- attribute - makes it a string attribute (can sort/group by it)

Specifying at least one property overrides all the default ones (see below), i.e., if you decide to use a custom combination of properties, you need to list all the properties you want.

- Integer Types: Attributes can be of various integer types (signed or unsigned), including BIGINT, which is useful for storing large numbers. Integer attributes can be used for filtering, sorting, and grouping in search queries.
- Floating-Point: FLOAT is supported for decimal numbers, allowing for more precision in numerical data that requires fractional values.
- Boolean: Although not a distinct type in Manticore, booleans can be represented using integer types, with 0 for false and 1 for true.
- String: Manticore supports string attributes for non-full-text query operations like filtering and grouping. String attributes are not tokenized and indexed in the same way as text fields. Unlike full-text fields, string attributes (just string or string/text attribute) are stored as they are received and cannot be used in full-text searches. Instead, they are returned in results, can be used in the WHERE clause for comparison filtering or REGEX, and can be used for sorting and aggregation. In general, it's not recommended to store large texts in string attributes, but use string attributes for metadata like names, titles, tags, keys.
- Multi-Value Attributes (MVA): MVAs allow an attribute to have a set of values for each document. This is useful for representing many-to-one relationships, such as tags or categories for an article.
```sql
CREATE TABLE products(title text, product_codes multi);
select * from products where any(product_codes)=3
select least(product_codes) l from products order by l asc
```
- JSON: Manticore Search supports indexing JSON attributes, enabling the storage and querying of structured data. This allows for complex data structures to be indexed and searched.

#### Row-wise and columnar attribute storages
Manticore supports two types of attribute storages:
- row-wise - traditional storage available in Manticore Search out of the box
- columnar - provided by Manticore Columnar Library

As can be understood from their names, they store data differently. The traditional row-wise storage:
- stores attributes uncompressed
- all attributes of the same document are stored in one row close to each other
- rows are stored one by one
- accessing attributes is basically done by just multiplying the row ID by the stride (length of a single vector) and getting the requested attribute from the calculated memory location. It gives very low random access latency.
- attributes have to be in memory to get acceptable performance, otherwise due to the row-wise nature of the storage Manticore may have to read from disk too much unneeded data which is in many cases suboptimal.

With the columnar storage:
- each attribute is stored independently of all other attributes in its separate "column"
- storage is split into blocks of 65536 entries
- the blocks are stored compressed. This often allows storing just a few distinct values instead of storing all of them like in the row-wise storage. High compression ratio allows reading from disk faster and makes the memory requirement much lower
- when data is indexed, storage scheme is selected for each block independently. For example, if all values in a block are the same, it gets "const" storage and only one value is stored for the whole block. If there are less than 256 unique values per block, it gets "table" storage and stores indexes to a table of values instead of the values themselves
- search in a block can be early rejected if it's clear the requested value is not present in the block.

#### How to switch between the storages
The traditional row-wise storage is the default, so if you want everything to be stored in a row-wise fashion, you don't need to do anything when you create a table.

To enable the columnar storage you need to:

- specify engine='columnar' in CREATE TABLE to make all attributes of the table columnar. Then, if you want to keep a specific attribute row-wise, you need to add engine='rowwise' when you declare it. For example:
```sql
create table tbl(title text, type int, price float engine='rowwise') engine='columnar'
```
- specify engine='columnar' for a specific attribute in CREATE TABLE to make it columnar. For example:
```sql
create table tbl(title text, type int, price float engine='columnar');

or

create table tbl(title text, type int, price float engine='columnar') engine='rowwise';
```
- in the plain mode you need to list attributes you want to be columnar in columnar_attrs.
#### Percolate
In Manticore Search, the concept of "percolate" is used in a somewhat different context compared to traditional search operations.
#### Percolate Queries
Percolate queries, in the context of Manticore Search, refer to a kind of reverse search operation. Instead of the usual search process where you have a query and you look for documents that match this query, with percolate queries, you have a document, and you look for queries that match this document. This is useful in scenarios where you want to alert or trigger actions based on incoming documents that match certain predefined criteria or queries.
#### Percolate Table
A percolate table in Manticore Search is a special type of table designed to store these percolate queries. Just like a regular table stores data records, a percolate table stores queries that are used for matching against incoming documents. When a new document comes in, it can be "percolated" through this table to find any stored queries that the document matches. It is used for prospective searches, or "search in reverse." 
#### Percolate Table Schemas
The schema of a percolate table in Manticore Search defines the structure and format of the queries that can be stored within it. This includes the types of fields that queries can match against, the types of matching or querying operations that can be used (like match phrases, boolean operations, etc.), and potentially other metadata related to the queries. The schema helps ensure that the percolate table is organized and can efficiently handle the percolation process for incoming documents.

In essence, percolate functionality in Manticore Search allows for a dynamic and inverted approach to searching, where the documents become the input for queries stored within a specialized table, enabling a wide range of use cases such as real-time monitoring, alerting, and filtering of incoming data streams based on complex criteria.

```sql
CREATE TABLE products(title text, meta json) type='pq';
```

#### Real-Time (RT) table
In Manticore Search, a "real-time" (RT) table is a type of table that allows for on-the-fly data indexing without the need for a predefined data source or a delayed indexing process. Real-time tables are designed to support immediate data ingestion, enabling you to add, update, or delete documents in the index instantly, making the changes searchable right away. This is in contrast to traditional indexing methods that require batch processing of data and rebuilding of the index to reflect new or updated content.

Real-time mode requires no table definition in the configuration file. However, the data_dir directive in the searchd section is mandatory. Index files are stored inside the data_dir.

A Real-time table is a main type of table in Manticore, allowing you to add, update, and delete documents with immediate availability of the changes. The settings for a Real-time Table can be defined in a configuration file or online using CREATE/UPDATE/DELETE/ALTER commands.

#### Table types and modes

| Table type    | RT mode       | Plain mode |
| ------------- | ------------- | -----------|
| Real-time     | supported     |  supported |
| Plain         | not supported |  supported |
| Percolate     | supported     |  supported |
| Distributed   | supported     |  supported |
| Template      | not supported |  supported |

##### Chunks
A Real-time Table is comprised of one or multiple plain tables called chunks. There are two types of chunks:
- multiple disk chunks - These are stored on disk and have the same structure as a Plain Table.
- single ram chunk - This is stored in memory and is used as an accumulator of changes.

The size of the RAM chunk is controlled by the rt_mem_limit setting. Once this limit is reached, the RAM chunk is flushed to disk in the form of a disk chunk. If there are too many disk chunks, they can be merged into one for better performance using the OPTIMIZE command or automatically.

##### Features
Real-time tables support various features, including:
- Dynamic indexing. You can insert, update, or delete documents in real time, making them immediately searchable.
- Flexible schema. The schema of an RT table, which defines the structure of the indexed data, can include various field types and attributes, similar to what you might define in a traditional database table.
- Full-text search capabilities. RT tables support Manticore Search's full-text search features, allowing for complex querying and searching within the indexed data.
- Attributes for filtering and sorting. Besides the full-text fields, RT tables can include attributes (such as integers, timestamps, strings, etc.) that can be used for filtering, sorting, and grouping search results.
- Transaction support. Changes to RT tables can be done within transactions, providing atomicity for batches of operations.

Real-time tables are particularly useful in scenarios where the dataset is dynamic, and the application requires the search index to be updated in real-time as the underlying data changes. This makes RT tables suitable for applications such as live dashboards, real-time analytics, and any other use case where immediate searchability of fresh data is critical.

```sql
CREATE TABLE products(title text, price float) morphology='stem_en';
```

#### Plain table
Plain table is a basic element for non-percolate searching. It can be defined only in a configuration file using the Plain mode, and is not supported in the RT mode. It is typically used in conjunction with a source to process data from the external storage and can later be attached to a real-time table.

Plain table is a mostly immutable data structure and a basic element used by real-time tables. Plain table stores a set of documents, their common dictionary and indexation settings. One real-time table can consist of multiple plain tables (chunks), but besides that Manticore provides direct access to building plain tables using tool indexer. It makes sense when your data is mostly immutable, therefore you don't need a real-time table for that.

Here's an example of a plain table configuration and a source for fetching data from a MySQL database:
```
source source {
  type             = mysql
  sql_host         = localhost
  sql_user         = myuser
  sql_pass         = mypass
  sql_db           = mydb
  sql_query        = SELECT id, title, description, category_id  from mytable
  sql_attr_uint    = category_id
  sql_field_string = title
 }

table tbl {
  type   = plain
  source = source
  path   = /path/to/table
 }
```
#### Real-time mode vs plain mode

Manticore Search works in two modes:

- Real-time mode (RT mode). This is a default one and allows to manage your data schema imperatively:

        allows managing your data schema online using SQL commands CREATE/ALTER/DROP TABLE and their equivalents in non-SQL clients
  
        in the configuration file you need to define only server-related settings including data_dir
  
- Plain mode allows to define your data schemas in a configuration file, i.e. provides declarative kind of schema management. It makes sense in three cases:

        when you only deal with plain tables
  
        or when your data schema is very stable and you don't need replication (as it's available only in the RT mode)
  
        when you have to make your data schema portable (e.g. for easier deployment of it on a new server)

#### Data tokenization
Manticore doesn't store text as is for performing full-text searching on it. Instead, it extracts words and creates several structures that allow fast full-text searching. From the found words, a dictionary is built, which allows a quick look to discover if the word is present or not in the index. In addition, other structures record the documents and fields in which the word was found (as well as the position of it inside a field). All these are used when a full-text match is performed.

The process of demarcating and classifying words is called tokenization. The tokenization is applied at both indexing and searching, and it operates at the character and word level.

On the character level, the engine allows only certain characters to pass. This is defined by the charset_table. Anything else is replaced with a whitespace (which is considered the default word separator). The charset_table also allows mappings, such as lowercasing or simply replacing one character with another. Besides that, characters can be ignored, blended, defined as a phrase boundary. 

At the word level, the base setting is the min_word_len which defines the minimum word length in characters to be accepted in the index. A common request is to match singular with plural forms of words. For this, morphology processors can be used. 

Going further, we might want a word to be matched as another one because they are synonyms. For this, the word forms feature can be used, which allows one or more words to be mapped to another one.

Very common words can have some unwanted effects on searching, mostly because of their frequency they require lots of computing to process their doc/hit lists. They can be blacklisted with the stop words functionality. This helps not only in speeding up queries but also in decreasing the index size.
#### Distributed table
Manticore allows for the creation of distributed tables, which act like regular plain or real-time tables, but are actually a collection of child tables used for searching. When a query is sent to a distributed table, it is distributed among all tables in the collection. The server then collects and processes the responses to sort and recalculate values of aggregates, if necessary.

Distributed tables can be composed of any combination of tables, including:
- Local storage tables (plain table and Real-Time)
- Remote tables
- A combination of local and remote tables
- Percolate tables (local, remote, or a combination)
- Single local and multiple remote tables, or any other combination

Mixing percolate and template tables with plain and real-time tables is not recommended.

```
table foo {
    type = distributed
    local = bar
    local = bar1, bar2
    agent = 127.0.0.1:9312:baz
    agent = host1|host2:tbl
    agent = host1:9301:tbl1|host2:tbl2 [ha_strategy=random retry_count=10]
    ...
}
```
Or
```sql
CREATE TABLE distributed_index type='distributed' local='local_index' agent='127.0.0.1:9312:remote_index'
```
#### Manticore cluster
Manticore Search is a highly distributed system that provides all the necessary components to create a highly available and scalable database for search. This includes:
- distributed table for sharding
- Mirroring for high availability
- Load balancing for scalability
- Replication for data safety
### Overview

By default Manticore is waiting for your connections on:
- port 9306 for MySQL clients
- port 9308 for HTTP/HTTPS connections
- port 9312 for connections from other Manticore nodes and clients based on Manticore binary API

Connect to Manticore
```
mysql -h0 -P9306
```

Let's now create a table called "products" with 2 fields and add few documents to the table:
```sql
create table products(title text, price float) morphology='stem_en';
insert into products(title,price) values ('Crossbody Bag with Tassel', 19.85), ('microfiber sheet set', 19.99), ('Pet Hair Remover Glove', 7.99);
```
Note that it is possible to omit creating a table with an explicit create statement. For more information, see Auto schema.

Let's find one of the documents. The query we will use is 'remove hair'. As you can see, it finds a document with the title 'Pet Hair Remover Glove' and highlights 'Hair remover' in it, even though the query has "remove", not "remover". This is because when we created the table, we turned on using English stemming (morphology "stem_en").
```sql
select id, highlight(), price from products where match('remove hair');
..


+---------------------+-------------------------------+----------+
| id                  | highlight()                   | price    |
+---------------------+-------------------------------+----------+
| 1513686608316989452 | Pet <strong>Hair Remover</strong> Glove | 7.990000 |
+---------------------+-------------------------------+----------+
1 row in set (0.00 sec)
```

Let's assume we now want to update the document - change the price to 18.5. This can be done by filtering by any field, but normally you know the document id and update something based on that.
Let's now delete all documents with price lower than 10.
```sql
update products set price=18.5 where id = 1513686608316989452;
delete from products where price < 10;
```

### searchd
```
The options available to searchd in all operating systems are:

    --help (-h for short) lists all of the parameters that can be used in your particular build of searchd.

    --version (-v for short) shows Manticore Search version information.

    --config <file> (-c <file> for short) tells searchd to use the specified file as its configuration.

    --stop is used to asynchronously stop searchd, using the details of the PID file as specified in the Manticore configuration file. Therefore, you may also need to confirm to searchd which configuration file to use with the --config option. Example:

    $ searchd --config /etc/manticoresearch/manticore.conf --stop

    --stopwait is used to synchronously stop searchd. --stop essentially tells the running instance to exit (by sending it a SIGTERM) and then immediately returns. --stopwait will also attempt to wait until the running searchd instance actually finishes the shutdown (eg. saves all the pending attribute changes) and exits. Example:

    $ searchd --config /etc/manticoresearch/manticore.conf --stopwait

Possible exit codes are as follows:

    0 on success

    1 if connection to running searchd server failed

    2 if server reported an error during shutdown

    3 if server crashed during shutdown

    --status command is used to query running searchd instance status using the connection details from the (optionally) provided configuration file. It will try to connect to running instance using the first found UNIX socket or TCP port from the configuration file. On success it will query for a number of status and performance counter values and print them. You can also use SHOW STATUS command to access the very same counters via SQL protocol. Examples:

    $ searchd --status
    $ searchd --config /etc/manticoresearch/manticore.conf --status

    --pidfile is used to explicitly force using a PID file (where the searchd process identification number is stored) despite any other debugging options that say otherwise (for instance, --console). This is a debugging option.

    $ searchd --console --pidfile

    --console is used to force searchd into console mode. Typically, Manticore runs as a conventional server application and logs information into log files (as specified in the configuration file). However, when debugging issues in the configuration or the server itself, or trying to diagnose hard-to-track-down problems, it may be easier to force it to dump information directly to the console/command line from which it is being called. Running in console mode also means that the process will not be forked (so searches are done in sequence) and logs will not be written to. (It should be noted that console mode is not the intended method for running searchd.) You can invoke it as:

    $ searchd --config /etc/manticoresearch/manticore.conf --console

    --logdebug, --logreplication, --logdebugv, and --logdebugvv options enable additional debug output in the server log. They differ by the logging verboseness level. These are debugging options and should not be normally enabled, as they can pollute the log a lot. They can be used temporarily on request to assist with complicated debugging sessions.

    --iostats is used in conjunction with the logging options (the query_log must have been activated in manticore.conf) to provide more detailed information on a per-query basis about the input/output operations carried out in the course of that query, with a slight performance hit and slightly bigger logs. The IO statistics don't include information about IO operations for attributes, as these are loaded with mmap. To enable it, you can start searchd as follows:

    $ searchd --config /etc/manticoresearch/manticore.conf --iostats

    --cpustats is used to provide actual CPU time report (in addition to wall time) in both query log file (for every given query) and status report (aggregated). It depends on clock_gettime() Linux system call or falls back to less precise call on certain systems. You might start searchd thus:

    $ searchd --config /etc/manticoresearch/manticore.conf --cpustats

    --port portnumber (-p for short) is used to specify the port that Manticore should listen on to accept binary protocol requests, usually for debugging purposes. This will usually default to 9312, but sometimes you need to run it on a different port. Specifying it on the command line will override anything specified in the configuration file. The valid range is 0 to 65535, but ports numbered 1024 and below usually require a privileged account in order to run.

    An example of usage:

    $ searchd --port 9313

    --listen ( address ":" port | port | path ) [ ":" protocol ] (or -l for short) Works as --port, but allows you to specify not only the port, but the full path, IP address and port, or Unix-domain socket path that searchd will listen on. In other words, you can specify either an IP address (or hostname) and port number, just a port number, or a Unix socket path. If you specify a port number but not the address, searchd will listen on all network interfaces. A Unix path is identified by a leading slash. As the last parameter, you can also specify a protocol handler (listener) to be used for connections on this socket. Supported protocol values are 'sphinx' and 'mysql' (MySQL protocol used since 4.1).

    --force-preread forbids the server from serving any incoming connection until prereading of table files completes. By default, at startup, the server accepts connections while table files are lazy-loaded into memory. This extends the behavior and makes it wait until the files are loaded.

    --index (--table) <table> (or -i (-t) <table> for short) forces this instance of searchd to only serve the specified table. Like --port, above, this is usually for debugging purposes; more long-term changes would generally be applied to the configuration file itself.

    --strip-path strips the path names from all the file names referenced from the table (stopwords, wordforms, exceptions, etc). This is useful for picking up tables built on another machine with possibly different path layouts.

    --replay-flags=<OPTIONS> switch can be used to specify a list of extra binary log replay options. The supported options are:
        accept-desc-timestamp, ignore descending transaction timestamps and replay such transactions anyway (the default behavior is to exit with an error).
        ignore-open-errors, ignore missing binlog files (the default behavior is to exit with an error).
        ignore-trx-errors, ignore any transaction errors and skip current binlog file (the default behavior is to exit with an error).
        ignore-all-errors, ignore any errors described above (the default behavior is to exit with an error).

    Example:

      $ searchd --replay-flags=accept-desc-timestamp

    --coredump is used to enable saving a core file or a minidump of the server on crash. Disabled by default to speed up of server restart on crash. This is useful for debugging purposes.

    $ searchd --config /etc/manticoresearch/manticore.conf --coredump

    --new-cluster bootstraps a replication cluster and makes the server a reference node with cluster restart protection. On Linux you can also run manticore_new_cluster. It will start Manticore in --new-cluster mode via systemd.

    --new-cluster-force bootstraps a replication cluster and makes the server a reference node bypassing cluster restart protection. On Linux you can also run manticore_new_cluster --force. It will start Manticore in --new-cluster-force mode via systemd.


searchd supports a number of signals:

    SIGTERM - Initiates a clean shutdown. New queries will not be handled, but queries that are already started will not be forcibly interrupted.
    SIGHUP - Initiates tables rotation. Depending on the value of seamless_rotate setting, new queries might be shortly stalled; clients will receive temporary errors.
    SIGUSR1 - Forces reopen of searchd log and query log files, allowing for log file rotation.


Environment variables

    MANTICORE_TRACK_DAEMON_SHUTDOWN=1 enables detailed logging while searchd is shutting down. It's useful in case of some shutdown problems, such as when Manticore takes too long to shut down or freezes during the shutdown process.
```

### Installation
```
sudo yum install https://repo.manticoresearch.com/manticore-repo.noarch.rpm
sudo yum install manticore manticore-extra

wget https://repo.manticoresearch.com/manticore-repo.noarch.deb
sudo dpkg -i manticore-repo.noarch.deb
sudo apt update
sudo apt install manticore manticore-columnar-lib

sudo systemctl start manticore

// Dirty mysql client installation
curl -L -O https://downloads.mysql.com/archives/get/p/23/file/mysql-community-client-8.2.0-1.el7.x86_64.rpm
rpm -ivh --nodeps --force mysql-community-client-8.2.0-1.el7.x86_64.rpm
mysql -h0 -P9306
```
### Replication

## Perfomance recommendations
- For the fastest search response time and ample memory availability, use row-wise attributes and lock them in memory using mlock. Additionally, use mlock for doclists/hitlists.
- If you prioritize can't afford lower performance after start and are willing to sacrifice longer startup time, use the --force-preread. option. If you desire faster searchd restart, stick to the default mmap_preread option.
- If you are looking to conserve memory, while still having enough memory for all attributes, skip the use of mlock. The operating system will determine what should be kept in memory based on frequent disk reads.
- If row-wise attributes do not fit into memory, opt for columnar attributes
- If full-text search performance is not a concern, and you wish to save memory, use access_doclists/access_hitlists=file

The default mode offers a balance of:
- mmap,
- Prereading non-columnar attributes,
- Seeking and reading columnar attributes with no preread,
- Seeking and reading doclists/hitlists with no preread.

This provides a decent search performance, optimal memory utilization, and faster searchd restart in most scenarios.

## Examples
### Percolate index
```sql
mysql> create table t(f text, j json) type='percolate';
mysql> insert into t(query,filters) values('abc', 'j.a=1');
mysql> call pq('t', '[{"f": "abc def", "j": {"a": 1}}, {"f": "abc ghi"}, {"j": {"a": 1}}]', 1 as query);
+---------------------+-------+------+---------+
| id                  | query | tags | filters |
+---------------------+-------+------+---------+
| 8215503050178035714 | abc   |      | j.a=1   |
+---------------------+-------+------+---------+
```
### Шардинг и распределённые таблицы (индексы)
```sql
create table user1(name text, email string, description text, age int, active int);
insert into user1(name) values('John');
create table user2 like user1;
insert into user2(name) values('Mary');

create table user type='distributed' local='user1' local='user2';
select * from user;
..
John
Mary
```
## Optimizations
Index Optimization

    Selective Indexing: Index only the necessary text fields and attributes to reduce index size and improve search performance.
    Batch Indexing: For Plain indexes, perform indexing in batches during off-peak hours to minimize the impact on search performance.
    Use RT Indexes Wisely: Real-time indexes are more flexible but can be less efficient than Plain indexes for static data sets. Use them where real-time updates are crucial.

Query Optimization

    Refine Queries: Simplify and refine queries to minimize complexity. Use filters and field weights wisely to speed up query processing.
    Avoid Large Offset: High offset values in pagination can slow down queries. Consider alternative approaches for deep pagination.
    Precompute Facets: If possible, precompute frequently used facets or aggregations to reduce the load on Manticore during query time.

Configuration Tuning

    Memory Management: Adjust the memory settings, like mem_limit for indexing and max_children for search queries, to optimize memory usage without exceeding system limits.
    Index Settings: Tune index settings such as min_infix_len and min_word_len to balance between search flexibility and index size/performance.
    Merge Strategies: Regularly merge delta indexes into the main index (for Plain indexes) to keep the index size manageable and search performance optimal.

Schema Design

    Attribute Types: Choose the most efficient attribute types (e.g., use integers instead of strings when possible) to reduce memory usage and speed up filtering/sorting operations.
    Normalize Data: Normalize repetitive text data into attributes when possible to reduce index size and improve search performance.

Infrastructure and Hardware

    SSD Storage: Use SSDs for storing indexes to significantly improve read/write speeds compared to traditional HDDs.
    Adequate RAM: Ensure there's enough RAM to hold frequently accessed data in memory, reducing disk I/O.
    Load Balancing: Distribute search queries across multiple Manticore instances to balance the load and improve response times.

High Availability and Scaling

    Replication: Use Manticore's replication features to distribute data across multiple nodes, improving search performance and fault tolerance.
    Sharding: Break down large indexes into smaller, more manageable shards to distribute the data and improve search performance.

Monitoring and Maintenance

    Monitoring: Implement monitoring tools to track Manticore's performance, resource usage, and query times. Use this data to identify bottlenecks and areas for improvement.
    Regular Maintenance: Perform regular maintenance tasks such as optimizing indexes, cleaning up old data, and updating statistics to maintain optimal performance.

Advanced Features

    Per-Column Compression: Use per-column string compression to reduce the index size for string attributes, improving I/O performance.
    Hitless Rotation: Use seamless index rotation to update indexes without interrupting search operations, ensuring continuous availability.

Keep Up-to-Date

    Manticore Updates: Stay updated with the latest Manticore Search releases and updates, as they often include performance improvements, new features, and bug fixes.

Customization and Testing

    Custom Ranking: Customize ranking algorithms to suit your specific needs, which can sometimes lead to performance improvements.
    Benchmarking: Regularly benchmark your Manticore setup under various loads and query patterns to identify performance issues and validate optimization efforts.
## TODO

1. Understand the Basics of Search Engines

    Conceptual Foundation: Start with the basics of search engines, including how they index data, process queries, and rank results. Understanding these concepts will help you grasp how Manticore Search operates.
    Manticore Search Overview: Familiarize yourself with Manticore Search, its origins from Sphinx, and its specific features and advantages.

2. Install and Configure Manticore Search

    Installation: Follow the official documentation to install Manticore Search on your preferred platform (Linux, Docker, etc.).
    Configuration Basics: Learn how to configure Manticore Search, including setting up indexes, data sources, and other essential parameters.

3. Dive into Index Types

    Understand the differences between Real-Time (RT) indexes and Plain indexes, including their use cases, advantages, and limitations.
    Practice creating both RT and Plain indexes, and experiment with adding, updating, and deleting data in RT indexes.

4. Querying and Data Manipulation

    Query Language: Get comfortable with Manticore's querying language, which is similar to SQL. Practice writing queries to search, filter, and aggregate data.
    Advanced Features: Explore full-text search capabilities, attribute filtering, and faceted search.

5. Integration and Data Pipelines

    Learn how to integrate Manticore Search with existing applications and data sources. This may involve setting up data ingestion pipelines, using APIs, or connecting to databases.
    Explore Manticore's support for various data formats and sources, and practice setting up data synchronization.

6. Scaling and Performance Optimization

    Understand the scalability options available with Manticore Search, including sharding, replication, and load balancing.
    Learn about performance tuning, including index optimization, query optimization, and hardware considerations.

7. High Availability and Disaster Recovery

    Set up a high-availability configuration for Manticore Search to ensure service continuity.
    Plan and implement backup and recovery strategies for Manticore Search data and configurations.

8. Monitoring and Maintenance

    Implement monitoring solutions to track Manticore Search performance, resource usage, and operational health.
    Learn about routine maintenance tasks, including index rotation, cleanup, and performance audits.

9. Advanced Topics

    Explore advanced features like percolate queries, JSON attributes, and custom ranking algorithms.
    Stay updated with Manticore Search's release notes and community forums for new features and best practices.
