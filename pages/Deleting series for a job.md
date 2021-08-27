- #[[prometheus]]
- [[Deleting time series from Prometheus]]
-
-
  ```
  curl -X POST -g 'http://localhost:9090/api/v1/admin/tsdb/delete_series?match[]={job="crushftp"}'
  ```