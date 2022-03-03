---
id: 0kc4vsu1sr96bp65q3a2m8t
title: VirtualFile Stats
desc: ''
updated: 1646216220368
created: 1646216220368
---

##### VIRTUAL FILE STATISTICS
A common trap in my experience, when using wait statistics as a primary source of
troubleshooting data, is that most SQL Servers will demonstrate signs of what looks
like a disk I/O bottleneck. Unfortunately, the wait statistics don't tell you what
is causing the I/O to occur, and it's easy to misdiagnose the root cause.
This is why an examination of the virtual file statistics, alongside the wait statistics,
is almost always recommended. The virtual file statistics are exposed through the 
sys.dm_io_virtual_file_stats function which, when passed a file_id (and possibly database_id),
will provide cumulative physical I/O statistics, the number of reads and writes on each data
file, and the number of reads and writes on each log file, for the various databases
in the instance, from which can be calculated the ratio of reads to writes.
This also shows the number of I/O stalls and the stall time associated with
the requests, which is the total amount of time sessions have waited for I/O
to be completed on the file.

```SQL
SELECT DB_NAME(vfs.database_id) AS database_name ,
vfs.database_id ,
vfs.FILE_ID ,
io_stall_read_ms / NULLIF(num_of_reads, 0) AS avg_read_latency ,
io_stall_write_ms / NULLIF(num_of_writes, 0) AS avg_write_latency ,
io_stall / NULLIF(num_of_reads + num_of_writes, 0) AS avg_total_latency ,
num_of_bytes_read / NULLIF(num_of_reads, 0) AS avg_bytes_per_read ,
num_of_bytes_written / NULLIF(num_of_writes, 0) AS avg_bytes_per_write ,
vfs.io_stall ,
vfs.num_of_reads ,
vfs.num_of_bytes_read ,
vfs.io_stall_read_ms ,
vfs.num_of_writes ,
vfs.num_of_bytes_written ,
vfs.io_stall_write_ms ,
size_on_disk_bytes / 1024 / 1024. AS size_on_disk_mbytes ,
physical_name
FROM sys.dm_io_virtual_file_stats(NULL, NULL) AS vfs
JOIN sys.master_files AS mf ON vfs.database_id = mf.database_id
AND vfs.FILE_ID = mf.FILE_ID
ORDER BY avg_total_latency DESC
```
