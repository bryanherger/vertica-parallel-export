# vertica-parallel-export
ParallelExport UDTF to export data from Vertica to flat files on all nodes

Derived from the HPE SaaS Marketplace offering and updated for Vertica 10.x

There's a great writeup here: https://www.dbjungle.com/exporting-vast-amounts-of-data-using-parallel-export-for-hpe-vertica/

Or simply "make install" and look at the README.txt for the original directions.

Some tests using a 123M row table, run on 3-node cluster on AWS with EBS storage for local exports
(TL;DR local export slowest, ParallelExport faster, EXPORT TO PARQUET fastest):
```
dbadmin=> select version();
Vertica Analytic Database v10.1.0-2
dbadmin=> create table dump1090_5days as select * from dump1090_aug2020 where date_generated between '2020-08-01' and '2020-08-05';
Time: First fetch (0 rows): 31593.426 ms. All rows formatted: 31593.440 ms
dbadmin=> select min(date_generated), max(date_generated), count(*) from dump1090_5days ;
2020-08-01|2020-08-05|123254356
dbadmin=> \o /tmp/5days.csv
dbadmin=> select * from dump1090_5days ;
Time: First fetch (1000 rows): 21.003 ms. All rows formatted: 274404.259 ms
dbadmin=> select ParallelExport(* using parameters path='/tmp/ParEx_test.csv.${nodeName}', separator=',') over (partition auto) from dump1090_5days;
Time: First fetch (3 rows): 184831.105 ms. All rows formatted: 184831.129 ms
dbadmin=> EXPORT TO PARQUET(directory = 's3://bryanh-vertica-poc/dump1090/5days/') AS SELECT * FROM dump1090_5days;
Time: First fetch (1 row): 115138.771 ms. All rows formatted: 115138.784 ms
```

