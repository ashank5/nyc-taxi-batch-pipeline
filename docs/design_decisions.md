# Design Decisions

## Why Delta Lake over plain Parquet?

Plain Parquet has no transaction log. Concurrent writes corrupt data silently.  
Delta Lake gives ACID guarantees, schema enforcement, and time travel — 
all critical for a production pipeline where reruns are common.

## Why replaceWhere instead of full overwrite?

A full overwrite on a 500MB dataset to fix one month's data is wasteful 
and risky. `replaceWhere` surgically touches only the affected partition, 
leaving all other months intact. This is how production pipelines handle 
late-arriving or corrected data.

## Why MERGE in Silver watermark mode?

When Bronze is re-run for a past month, Silver may receive overlapping 
records. MERGE handles this safely — existing rows are updated, new rows 
are inserted, and no duplicates accumulate. replaceWhere alone would 
miss updates to already-written records.

## Why five separate Gold tables instead of one wide table?

A single wide Gold table forces every consumer to filter columns they 
do not need, increasing scan cost. Five focused tables mean each 
consumer reads only what is relevant. This also makes each table's 
ownership and SLA independently manageable.

## Why cache Silver in Gold notebook?

Gold reads Silver five times — once per aggregation. Without caching, 
Spark re-reads and re-deserialises the Delta files on each action. 
One cache() call trades memory for five fewer disk reads on a shared 
Community Edition cluster.

## Why is there no Gold watermark table?

Gold is always triggered either by an explicit RUN_DATE or by reading 
Silver's watermark directly. Maintaining a separate Gold watermark would 
introduce a third state store with no benefit — Gold has no independent 
source to track.
