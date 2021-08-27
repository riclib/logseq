contact:: [[JohanDeWulf]]

	- DONE Latest CrushFTP build can now be targeted by prometheus as a data source
	  later:: 1628023116566
	  now:: 1628023331055
	  done:: 1628056841591
	  id:: 610a27f6-7581-4ef3-96aa-f3fcb7b6049c
	- . For PoC pleasures, using the solidMon install on 409 server, would it be possible for you to scrape the endpoint(s) and represent the data in a dashboard ?
	- --- Feedback Ben ---
	   
	  Example config snippet included below.  This will need to be in your yaml file in the appropriate place.  Additional types of items can be exposed upon request...this is just meant to get you started and see into CrushFTP a little bit more.  Effectively, this is everything the dashboard sees...
	-
	  ``` yaml
	  - targets: ['192.168.1.11:9191']
	       metrics_path: "/"
	       params:
	           command: ['prometheusMetrics']
	           type: ['server_info,server_prefs']
	       basic_auth:
	         username: 'crush_monitor_prometheus'
	         password: "!#BmtH0123456"
	  ```
	- ==Translated== to prometheus config
	-
	  ``` yaml
	    - job_name: 'crushftp'
	      metrics_path: "/"
	      basic_auth:
	         username: crush_monitor_prometheus
	         password: later
	      relabel_configs:
	        - target_label: __param_command
	          replacement: prometheusMetrics
	        - target_label: __param_type
	          replacement: server_info
	      static_configs:
	        - targets:
	          - 192.168.1.11:9191
	  ```
	- Test server_info
	   via cURL
	  ``` shell
	   curl -u crush_monitor_prometheus -k "https://itsbebel00197.jnj.com:11443/?command=prometheusMetrics&type=server_info"
	  ```
	   returned result captured in attachment server_info.log
	   Test server_prefs
	   Via cURL
	  ``` shell
	   curl -u crush_monitor_prometheus -k "https://itsbebel00197.jnj.com:11443/?command=prometheusMetrics&type=server_prefs"
	  ```
	   returned result captured in attachment server_prefs.log