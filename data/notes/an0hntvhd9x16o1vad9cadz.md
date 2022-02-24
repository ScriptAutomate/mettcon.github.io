
### to research
- SQLOS Cooperative scheduling vs. Win preemptive scheduling

###  mcsa 70/765


### troubleshooting
[troublshoooting sql server](https://assets.red-gate.com/community/books/troubleshooting-sql-server-accidental-dba.pdf)   ^k7i59zrcxn1o
Zusammenfassung  

If you collect and examine individually five separate pieces of performance data, it's
possible that each could send you down a separate path. Viewed as a group, they will
likely lead you down the sixth, and correct, path to resolving the issue. If there is one
take-away from this chapter, as well as this book, it should be that focusing on a single
piece of information alone can often lead to an incorrect diagnosis of a problem.  

#### Defining a Troubleshooting Methodology
Fairly early on in any analysis, I'll take a look at the wait statistics, in the sys.dm_os_
wait_stats Dynamic Management View (DMV), to identify any major resource waits in
the system, at the operating system level.  

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

