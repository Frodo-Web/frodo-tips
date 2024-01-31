# Manticore
Manticore Search is a high-performance, open-source search engine designed to offer fast and precise full-text searches, along with other search-related functionalities. It originated as a fork of the Sphinx Search engine and has evolved to include features like full-text search, real-time indexing, and support for various data sources such as MySQL, PostgreSQL, ODBC, and CSV files. Manticore Search is known for its cost-efficiency, scalability, and SQL-like query language, making it a viable alternative to other search engines like Elasticsearch. It also supports advanced features such as replication, load balancing, and data backup, catering to the needs of modern, data-driven applications​.
## Terms
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

A Real-time table is a main type of table in Manticore, allowing you to add, update, and delete documents with immediate availability of the changes. The settings for a Real-time Table can be defined in a configuration file or online using CREATE/UPDATE/DELETE/ALTER commands.

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
```
### Replication