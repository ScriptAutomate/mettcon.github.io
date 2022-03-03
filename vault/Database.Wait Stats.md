---
id: h71qqx0ag0p0j39n67olrji
title: Wait Stats
desc: 'Starting Point for SQL Troubleshooting'
updated: 1646217050070
created: 1646215087614
---

#### Wait Statistics: the Basis for Troubleshooting
As a part of the normal operations of SQL Server, a number of wait conditions exist
which are non-problematic in nature and generally expected on the server. These wait
conditions can generally be queried from the sys.dm_os_waiting_tasks DMV for the
system sessions.

```SQL
SELECT DISTINCT
     wt.wait_type
FROM sys.dm_os_waiting_tasks AS wt
JOIN sys.dm_exec_sessions AS s ON wt.session_id = s.session_id
WHERE s.is_user_process = 0
```

When looking at the wait statistics being tracked by SQL Server, it's important that these
wait types are eliminated from the analysis, allowing the more problematic waits in the
system to be identified. One of the things I do as a part of tracking wait information is to
maintain a script that filters out the non-problematic wait types.

```SQL
SELECT TOP 10
wait_type ,
max_wait_time_ms wait_time_ms ,
signal_wait_time_ms ,
wait_time_ms - signal_wait_time_ms AS resource_wait_time_ms ,
100.0 * wait_time_ms / SUM(wait_time_ms) OVER ( )
AS percent_total_waits ,
100.0 * signal_wait_time_ms / SUM(signal_wait_time_ms) OVER ( )
AS percent_total_signal_waits ,
100.0 * ( wait_time_ms - signal_wait_time_ms )
/ SUM(wait_time_ms) OVER ( ) AS percent_total_resource_waits
FROM sys.dm_os_wait_stats
WHERE wait_time_ms > 0 -- remove zero wait_time
AND wait_type NOT IN -- filter out additional irrelevant waits
( 'SLEEP_TASK', 'BROKER_TASK_STOP', 'BROKER_TO_FLUSH',
'SQLTRACE_BUFFER_FLUSH','CLR_AUTO_EVENT', 'CLR_MANUAL_EVENT',
'LAZYWRITER_SLEEP', 'SLEEP_SYSTEMTASK', 'SLEEP_BPOOL_FLUSH',
'BROKER_EVENTHANDLER', 'XE_DISPATCHER_WAIT', 'FT_IFTSHC_MUTEX',
'CHECKPOINT_QUEUE', 'FT_IFTS_SCHEDULER_IDLE_WAIT',
'BROKER_TRANSMITTER', 'FT_IFTSHC_MUTEX', 'KSOURCE_WAKEUP',
'LOGMGR_QUEUE', 'ONDEMAND_TASK_QUEUE',
'REQUEST_FOR_DEADLOCK_SEARCH', 'XE_TIMER_EVENT', 'BAD_PAGE_PROCESS',
'DBMIRROR_EVENTS_QUEUE', 'BROKER_RECEIVE_WAITFOR',
'PREEMPTIVE_OS_GETPROCADDRESS', 'PREEMPTIVE_OS_AUTHENTICATIONOPS',
'WAITFOR', 'DISPATCHER_QUEUE_SEMAPHORE', 'XE_DISPATCHER_JOIN',
'RESOURCE_QUEUE' )
ORDER BY wait_time_ms DESC
```

##### Major Wait Types
###### CXPACKET  
Often indicates nothing more than that certain queries are executing with parallelism;
CXPACKET waits in the server are not an immediate sign of problems, although they may
be the symptom of another problem, associated with one of the other high value wait
types in the instance.

###### SOS_SCHEDULER_YIELD
The tasks executing in the system are yielding the scheduler, having exceeded their
quantum, and are having to wait in the runnable queue for other tasks to execute.
This may indicate that the server is under CPU pressure. See Chapter 3 for more
information on this.

###### THREADPOOL
A task had to wait to have a worker bound to it, in order to execute. This could
be a sign of worker thread starvation, requiring an increase in the number of
CPUs in the server, to handle a highly concurrent workload, or it can be a sign of
blocking, resulting in a large number of parallel tasks consuming the worker threads
for long periods.

###### LCK_*
hese wait types signify that blocking is occurring in the system and that sessions
have had to wait to acquire a lock of a specific type, which was being held by anoth-
er database session. This problem can be investigated further using the information
in the sys.dm_db_index_operational_stats and techniques described in <!-- Link -->

###### PAGEIOLATCH_*, IO_COMPLETION, WRITELOG
These waits are commonly associated with disk I/O bottlenecks, though the root
cause of the problem may be, and commonly is, a poorly performing query that is
consuming excessive amounts of memory in the server. PAGEIOLATCH_* waits are
specifically associated with delays in being able to read or write data from the data-
base files. WRITELOG waits are related to issues with writing to log files. These waits
should be evaluated in conjunction with the virtual file statistics as well as Physical
Disk performance counters, to determine if the problem is specific to a single data-
base, file, or disk, or is instance wide.
###### PAGELATCH_*

Non-I/O waits for latches on data pages in the buffer pool. A lot of times
PAGELATCH_* waits are associated with allocation contention issues. One of
the best-known allocations issues associated with PAGELATCH_* waits occurs in
tempdb when the a large number of objects are being created and destroyed in
tempdb and the system experiences contention on the Shared Global Allocation
Map (SGAM), Global Allocation Map (GAM), and Page Free Space (PFS) pages in the
tempdb database.

###### LATCH_*
These waits are associated with lightweight short-term synchronization objects that
are used to protect access to internal caches, but not the buffer cache. These waits
can indicate a range of problems, depending on the latch type. Determining the
specific latch class that has the most accumulated wait time associated with it can
be found by querying the sys.dm_os_latch_stats DMV.

###### ASYNC_NETWORK_IO
This wait is often incorrectly attributed to a network bottleneck. In fact, the most
common cause of this wait is a client application that is performing row-by-row processing
of the data being streamed from SQL Server as a result set (client accepts one row,
processes, accepts next row, and so on). Correcting this wait type generally
requires changing the client-side code so that it reads the result set as fast as pos-
sible, and then performs processing.

After fixing any problem in the server, in order to validate that the problem has indeed
been fixed, the wait statistics being tracked by the server can be reset using the code

```sql
DBCC SQLPERF('sys.dm_os_wait_stats', clear)
```

One of the caveats associated with clearing the wait statistics on the server, is that it
will take a period of time for the wait statistics to accumulate to the point that you
know whether or not a specific problem has been addressed.
