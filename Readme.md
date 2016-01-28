### Introduction ###

Brutis is a tool designed to exercise memcache instances by providing reproducible performance data for comparison purposes. Brutis can be useful for sizing memcache clusters as well as testing changes to the system, the hardware, and/or the environment. Much like a dynamometer, the numbers Brutis produces are not as important as the differences in the numbers between changes to the system, the hardware, and/or the environment.

When sizing memcached clusters, Brutis can help by stressing a memcached cluster to see:
  * How many ops the cluster is capable of
  * How much load the network config can take
  * How many connections a memcache cluster is capable of handling (we’ve tested up to 100K connections)

With respect to instance (not cluster) sizing, Brutis can be used to simulate varying object and key sizes to see how many keys/objects can fit in a memcache instance before evictions start.

### How It Works ###

Brutis is made up of three components: The main executable, the client, and the collector.

The main executable serves as a wrapper for the client and collector. It handles forking and process control. The client is the actual memcache client program. The collector aggregates the stats from one or many client processes and outputs results to the screen or to a file.

For greater control, you can run the collector and the client as standalone programs.

Brutis exercises and measures the performance of one or more memcache instances by:

  1. Generating pseudo random data and md5 checksum
  1. Executing set operations with objects that encapsulates data and md5 checksum
  1. Executing get operations from the memcache servers
  1. Verifying data and md5 checksum match
  1. Reporting the results to a collector

### Requirements ###

Brutis requires PHP 5 or newer with the following compiled in:

```
 Sockets Support
 pcntl
```

PHP Libraries:
  * `pecl/memcache           >= 2.2.4 -` http://pecl.php.net/package/memcache
For JSON Output:
  * `pear/Services_JSON      >= 1.0.0 -` http://pear.php.net/package/Services_JSON




### Known Issues ###
  * Random batch gets can cause false misses if rand() picks the same key multiple times
  * Total stats may not work on 32Bit arch. Looking into big int to work around php's lack of uint64\_t

### Files ###
```
./license License info 
./readme This file
./brutis Main executable.
./lib/client Memcache client
./lib/collector Statistic collector
./lib/functions.php Common functions 
./lib/taskmanager.php Taskmanager class
```
### Usage ###
```
brutis [OPTIONS] -x [server,..]
```

### Options ###

Options may be given in any order.

```
-k {Max Keys}
	Maximum key id to iterate up to. Keys are a concatenation of $prefix and $key_id. Default is '1000000'.

-z {Key offset}
	Where key_id starts to iterate. Default is '0'.

-a {Set Pattern:Get Pattern}
	Valid patterns are S for sequential and R for Random. Default is 'S:R'.

-p {Key Prefix}
	Data to use for key prefix. Keys are a concatenation of $prefix and $key_id. Default is 'brutis-'.

-P
        Use persistent connections

-R {time}
        Time before forcing disconnect/reconnect of connection pool (for non-persistent connections)

-m {Multiplier}
       Number of connection pools to create. used for large connection count testing.

-i {Poll interval}
	Interval in seconds to send data to collector and write out to output file.

-b {Batch}
	Number of key's to batch into 1 get operation.
			
-c {Collector:Port}
	Host to connect or start collector on. Default is localhost's DNS name '<hostname>:9091'.

-D
        Disable starting of local collector. (for remote reporting)

-d
	Enable MD5 checksum.

-r {Set Ratio:Get Ratio}
	Ratio of sets to gets. Default '1:10'.

-n {Operations}
	Number of operations to perform before exiting.

-t {Time}
	Time to run operations in seconds before exiting. Default '172800'.

-f {Forks}
	Number of clients to fork. Default '1'.

-F {Format}
        Format of output file. CSV or JSON supported. Default CSV.

-s {Size}
	Size of object to generate for sets in bytes. Default 256.

-x {Server:tcp_port:udp_port,...}
	List of memcache servers to add to connection pool. UDP port should be 0 unless trying to enable UDP support. UDP support is only available with danga memcache library 3.x

-o {Output Filename}
	Filename to output raw stats to. Causes collector to be invoked.

-v
	Enable Verbose output. Causes the collector to be invoked.

-h
	Display help.
```


### Exit Codes ###
```
Exit Code 0:	Success	
Exit Code 1:	Error
Exit Code 4:	Warning - Either Set failures or md5 checksum mismatch
```

### Collector Output ###
```
Brutis                                                           40 Connections

Type        Ops/sec    Hits/sec  Misses/sec   Fails/sec     Latency      MB/sec
-------------------------------------------------------------------------------
Sets:      6,739.89         ---         ---        0.00    0.000550        1.65
Gets:     67,378.85   66,913.86      464.99        0.00    0.000518       15.61
Totals:   74,118.74   66,913.86      464.99        0.00    0.001068       17.25
-------------------------------------------------------------------------------

                              SETS                             GETS
-------------------------------------------------------------------------------
Host             Ops/sec     Latency  MB/sec      Ops/sec     Latency  MB/sec
-------------------------------------------------------------------------------
la-cn1          3,345.94    0.000550    0.82    33,453.43    0.000523    7.75
la-cn2          3,393.94    0.000549    0.83    33,925.42    0.000514    7.86
```


### File Output ###
File output is in colon delimited format. A new line is added at every poll interval set at the command line. default is 2 seconds.

Formula:
```
t2 - t1 = elapsed_time
column_value/elapsed_time = value_per_sec
```

To calculate operations or transactions per second stats from the below format. You need to subtract the 2nd line's timestamps from the 1st line's timestamp. This will give you the elapsed time in seconds. Then divide the value from the column you wish to calculate by the elapsed time to calculate the per second stat.


```
#timestamp:sets:set_latency:gets:get_latency:hits:misses:set_fails:md5_fails:operations:set_transferred:get_transferred:clients
START
1240530481.23:61:0.01:6086:0.79:5947:139:0:0:6147:31232:1366878:1
1240530485.23:121:0.01:12109:0.92:11843:266:0:0:12230:61952:2684941:1
1240530487.23:60:0.01:6003:0.92:5889:114:0:0:6063:30720:1333141:1
1240530489.23:60:0.01:5942:0.92:5824:118:0:0:6002:30720:1321096:1
1240530491.23:60:0.01:6056:0.92:5910:146:0:0:6116:30720:1344118:1
1240530493.23:60:0.01:5971:0.92:5843:128:0:0:6031:30720:1328520:1
1240530495.23:60:0.01:5974:0.92:5834:140:0:0:6034:30720:1324654:1
1240530496.16:0:0.00:0:0.00:0:0:0:0:0:0:0:1
END
```

  * To calculate totals for gets, sets, hits, misses, set\_fails, or md5\_fails, you must add all values in the column you wish to know the total for.


### Exit Summary ###
```
Total Operations:	       48623
Total Sets:		         482
Total Gets:		       48141
Total Hits:		       47090
Total Misses:		        1051
Total Set Fails:	           0
Total MD5 Fails:	           0
Total Keys:		        2175
Total Transferred:	    10950132
```

### Examples Usage ###
```
Single client node, 20 forks, random Set test, single collector	
	cn-1: ./brutis -f 20 -x mc-1,mc-2,mc-3,mc-4 -a r:r -r 1:0 -v

Single client node, 20 forks, random Get test, single collector
	cn-1: ./brutis -f 20 -x mc-1,mc-2,mc-3,mc-4 -a r:r -r 0:1 -v

Single client node, 20 fork, random 1:10 set:get ratio, single collector
	cn-1: ./brutis -f 20 -x mc-1,mc-2,mc-3,mc-4 -a r:r -r 1:10 -v

Multi client node, 20 forks, random Set test, single collector
	cn-1: ./brutis -f 20 -x mc-1,mc-2,mc-3,mc-4 -a r:r -r 1:0 -v -c cn-1
	cn-2: ./brutis -f 20 -x mc-1,mc-2,mc-3,mc-4 -a r:r -r 1:0 -v -c cn-1
	cn-3: ./brutis -f 20 -x mc-1,mc-2,mc-3,mc-4 -a r:r -r 1:0 -v -c cn-1
	cn-4: ./brutis -f 20 -x mc-1,mc-2,mc-3,mc-4 -a r:r -r 1:0 -v -c cn-1

Multi client node, 20 forks, random Get test, 1 collecter per client node.
	cn-1: ./brutis -f 20 -x mc-1,mc-2,mc-3,mc-4 -a r:r -r 0:1 -v 	
	cn-2: ./brutis -f 20 -x mc-1,mc-2,mc-3,mc-4 -a r:r -r 0:1 -v
	cn-3: ./brutis -f 20 -x mc-1,mc-2,mc-3,mc-4 -a r:r -r 0:1 -v
	cn-4: ./brutis -f 20 -x mc-1,mc-2,mc-3,mc-4 -a r:r -r 0:1 -v

Multi client node, 20 forks, random 1:10 set:get ratio, single collector
	cn-1: ./brutis -f 20 -x mc-1,mc-2,mc-3,mc-4 -a r:r -r 1:10 -v -c cn-1
	cn-2: ./brutis -f 20 -x mc-1,mc-2,mc-3,mc-4 -a r:r -r 1:10 -v -c cn-1
	cn-3: ./brutis -f 20 -x mc-1,mc-2,mc-3,mc-4 -a r:r -r 1:10 -v -c cn-1
	cn-4: ./brutis -f 20 -x mc-1,mc-2,mc-3,mc-4 -a r:r -r 1:10 -v -c cn-1

Multi client node, 20 forks, mixed load/size test, single collector
	cn-1: ./brutis -f 20 -x mc-1,mc-2,mc-3,mc-4 -a r:s -r 1:100 -v -s 250 -c cn-1
	cn-2: ./brutis -f 20 -x mc-1,mc-2,mc-3,mc-4 -a s:r -r 1:10 -v -s 512 -c cn-1
	cn-3: ./brutis -f 20 -x mc-1,mc-2,mc-3,mc-4 -a s:r -r 1:50 -v -s 128 -c cn-1
	cn-4: ./brutis -f 20 -x mc-1,mc-2,mc-3,mc-4 -a r:s -r 1:1 -v -s 64 -c cn-1

Multi client node, 20 forks, High connection count, single collector
	cn-1: ./brutis -i 20 -x mc-1,mc-2,mc-3,mc-4 -m 20-a r:r -r 1:10 -v -c cn-1
	cn-2: ./brutis -f 20 -x mc-1,mc-2,mc-3,mc-4 -m 20 -a r:r -r 1:10 -v -c cn-1
	cn-3: ./brutis -f 20 -x mc-1,mc-2,mc-3,mc-4 -m 20 -a r:r -r 1:10 -v -c cn-1
	cn-4: ./brutis -f 20 -x mc-1,mc-2,mc-3,mc-4 -m 20 -a r:r -r 1:10 -v -c cn-1
```

### Release Notes ###
Version - 0.93
  * Fixed bug with not reporting when verbose not specified
  * Added dataset size to summary
  * Fixed sequential set/get bug causing misses
  * Fixed multiplier bug in sets exiting after 1st failure

Version - 0.92
  * Fixed bug with taskmanager library hitting maxruntime and killing collector
  * Fixed bug with collector starting when -v & -o not specified in client
  * Fixed disconnect errors in collector
  * [Issue 8](https://code.google.com/p/brutis/issues/detail?id=8): Fixed onShutdown() not being called in collector
  * Added JSON output format (experimental)

Version - 0.91
  * Changed connection count on collector to client connection count
  * Renamed collector3 to collector, removed old collectors
  * [Issue 7](https://code.google.com/p/brutis/issues/detail?id=7): Fixed latency bug
  * Collector 3 introduced
  * Removed dependency on Net\_Server & Net\_Socket
  * Aggregate stats per client hostname
  * Added connection Multiplier (Experimental)
  * Switch from Connect to addServer to use connection pooling
  * Added Persistent commandline option
  * Added Reconnect timer for non-persistent connection tests

Version - 0.90
  * Added collector2 and made default
  * Changed checksum to default to disabled
  * Added new exit summary in collector2
  * Fixed ratio rounding bug in set\_intervals()
  * Added latency
  * Fixed rounding bug in operations displayed by collector.
  * Changed random data to use chr(1-255).
  * Fixed bug in valid hosts not allowing IP addresses.


Version - 0.89
  * Rename to Brutis
  * Do not allow batch gets greater then max keys.
Version - 0.88
  * Added experimental support for danga version 3.x library and UDP.
Version - 0.86
  * Fixed bug with multiple of the same argument
  * added sigint handler to collector so it cleans up before exit on ctrl-c
  * Fixed total operations accounting in collector to be correct
  * Defined Exit codes
  * Fixed incorrect exitcode of 1024 from client
  * Removed collector limitation of running on all ip/ports
Version - 0.85
  * Added host validation
  * Many cleanups
Version - 0.83
  * Fixed MD5 not being added to transferred total
  * recognize localhost as well as hostname to know when to start collector
  * Removed offset from set\_interval
  * Cleaned up taskmanager formating
Version - 0.82
  * Added batch gets
  * Added Polling interval
  * Changed Gets to hits and added ops to collector for batch gets accounting
  * Removed 33 byte limitation with warning and disable of checksum
  * Added Readme file
Version - 0.81
  * Exit code 4 on only set failure or md5 checksum mismatch
Version - 0.80
  * Exit code 4 on set failure, md5 checksum mismatch, and misses
  * Introduced Access Patterns for set and get operations
  * Fixed bug to correctly display MB/Sec transferred with mixed object sizes
Version - pre 0.80
  * Undocumented