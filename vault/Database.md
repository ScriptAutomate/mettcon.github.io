---
id: an0hntvhd9x16o1vad9cadz
title: Database
desc: ''
updated: 1646216558575
created: 1645719623400
stub: false
---

### to research
- SQLOS Cooperative scheduling vs. Win preemptive scheduling

###  mcsa 70/765



### troubleshooting
[troublshoooting sql server](https://assets.red-gate.com/community/books/troubleshooting-sql--accidental-dba.pdf)   ^k7i59zrcxn1o
Zusammenfassung  server

If you collect and examine individually five separate pieces of performance data, it's
possible that each could send you down a separate path. Viewed as a group, they will
likely lead you down the sixth, and correct, path to resolving the issue. If there is one
take-away from this chapter, as well as this book, it should be that focusing on a single
piece of information alone can often lead to an incorrect diagnosis of a problem.  

<!-- Insert explanation about timespans of Statistics to analyze -->
#### Defining a Troubleshooting Methodology
Fairly early on in any analysis, I'll take a look at the wait statistics, in the sys.dm_os_
wait_stats Dynamic Management View (DMV), to identify any major resource waits in
the system, at the operating system level.  






##### Performance Counters
One of the challenges with querying the raw performance counter data directly is that
some of the performance counters are cumulative ones, increasing in value as time
progresses, and analysis of the data requires capturing two snapshots of the data and then
calculating the difference between the snapshots. The query in Listing 1.5 performs the
snapshots and calculations automatically, allowing the output to be analyzed directly.
There are other performance counters, not considered in Listing 1.5, which have a 32
secondary, associated base counter by which the main counter has to be divided to arrive
at its actual value.

```SQL
DECLARE @CounterPrefix NVARCHAR(30)
SET @CounterPrefix = CASE WHEN @@SERVICENAME = 'MSSQLSERVER'
                              THEN 'SQLServer:'
                          ELSE 'MSSQL$' + @@SERVICENAME + ':'
                     END ;
-- Capture the first counter set
SELECT CAST(1 AS INT) AS collection_instance ,
     [OBJECT_NAME] ,
     counter_name ,
     instance_name ,
     cntr_value ,
     cntr_type ,
     CURRENT_TIMESTAMP AS collection_time
INTO #perf_counters_init
FROM sys.dm_os_performance_counters
WHERE ( OBJECT_NAME = @CounterPrefix + 'Access Methods'
     AND counter_name = 'Full Scans/sec'
     )
     OR ( OBJECT_NAME = @CounterPrefix + 'Access Methods'
     AND counter_name = 'Index Searches/sec'
     )
     OR ( OBJECT_NAME = @CounterPrefix + 'Buffer Manager'
     AND counter_name = 'Lazy Writes/sec'
     )
     OR ( OBJECT_NAME = @CounterPrefix + 'Buffer Manager'
     AND counter_name = 'Page life expectancy'
     )
     OR ( OBJECT_NAME = @CounterPrefix + 'General Statistics'
     AND counter_name = 'Processes Blocked'
     )
     OR ( OBJECT_NAME = @CounterPrefix + 'General Statistics'
     AND counter_name = 'User Connections'
     )
     OR ( OBJECT_NAME = @CounterPrefix + 'Locks'
     AND counter_name = 'Lock Waits/sec'
     )
     OR ( OBJECT_NAME = @CounterPrefix + 'Locks'
     AND counter_name = 'Lock Wait Time (ms)'
     )
     OR ( OBJECT_NAME = @CounterPrefix + 'SQL Statistics'
AND counter_name = 'SQL Re-Compilations/sec'
)
OR ( OBJECT_NAME = @CounterPrefix + 'Memory Manager'
AND counter_name = 'Memory Grants Pending'
)
OR ( OBJECT_NAME = @CounterPrefix + 'SQL Statistics'
AND counter_name = 'Batch Requests/sec'
)
OR ( OBJECT_NAME = @CounterPrefix + 'SQL Statistics'
AND counter_name = 'SQL Compilations/sec'
)

-- Wait on Second between data collection
WAITFOR DELAY '00:00:01'

-- Capture the second counter set
SELECT    CAST(2 AS INT) AS collection_instance ,
          OBJECT_NAME ,
          counter_name ,
          instance_name ,
          cntr_value ,
          cntr_type ,
          CURRENT_TIMESTAMP AS collection_time
INTO    #perf_counters_second
FROM      sys.dm_os_performance_counters
WHERE     ( OBJECT_NAME = @CounterPrefix + 'Access Methods'
          AND counter_name = 'Full Scans/sec'
          )
          OR ( OBJECT_NAME = @CounterPrefix + 'Access Methods'
          AND counter_name = 'Index Searches/sec'
          )
          OR ( OBJECT_NAME = @CounterPrefix + 'Buffer Manager'
          AND counter_name = 'Lazy Writes/sec'
          )
          OR ( OBJECT_NAME = @CounterPrefix + 'Buffer Manager'
          AND counter_name = 'Page life expectancy'
          )
          OR ( OBJECT_NAME = @CounterPrefix + 'General Statistics'
          AND counter_name = 'Processes Blocked'
          )
          OR ( OBJECT_NAME = @CounterPrefix + 'General Statistics'
          AND counter_name = 'User Connections'
          )
