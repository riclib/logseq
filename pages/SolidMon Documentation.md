- # SolidMon Administration Guide
  collapsed:: true
	- ## Release Notes
	  collapsed:: true
		- 1.2.2
		  collapsed:: true
			- Fixes bug in
		- 1.2.1
		  collapsed:: true
			- Fixes Webresources bug
				- As part of the install you should download install the following components
				  
				  ```perl
				  # Components to install
				  alertmanager -> ../lib/alertmanager-0.19.0.linux-amd64/
				  blackbox_exporter -> ../lib/blackbox_exporter-0.18.0.linux-amd64/
				  grafana -> ../lib/grafana-7.5.2/
				  node_exporter -> ../lib/node_exporter-0.18.0.linux-amd64/
				  prometheus -> ../lib/prometheus-2.25.0.linux-amd64/
				  loki -> ../lib/loki-2.1.0/
				  ```
		- 1.2.0
		  collapsed:: true
			- First Release candidate
				- As part of the install you should download install the following components
				  
				  ```perl
				  # Components to install
				  alertmanager -> ../lib/alertmanager-0.19.0.linux-amd64/
				  blackbox_exporter -> ../lib/blackbox_exporter-0.18.0.linux-amd64/
				  grafana -> ../lib/grafana-7.5.2/
				  node_exporter -> ../lib/node_exporter-0.18.0.linux-amd64/
				  prometheus -> ../lib/prometheus-2.25.0.linux-amd64/
				  loki -> ../lib/loki-2.1.0/
				  ```
	- ## Install SolidMon
	  collapsed:: true
		- ### Download SolidMon
		  collapsed:: true
			- Download the latest (highest number) release from solidmon.com with sftp or scp. Use the password and username provided as part of your support contract
			  
			  ```shell
			  mkdir -p /apps/
			  cd /apps/
			  scp <user>@solidmon.com:dist/solidmon-X.X.X.tar.gz .
			  tar xzf loki_x.x.x.tar.gz
			  ```
			  
			    replace X.X.X by your chosen release
		- ### Download Components
		  collapsed:: true
			- Download latest versions of the following components to `/apps/solidmon/lib`
			  collapsed:: true
			  
			  > make sure to download linux amd64 tarballs. Extract and keep the name, deleting the downloaded file
			  
			  ```shell
			  cd /apps/solidmon/lib
			  wget "component URL"
			  tar xzf "downloaded file"
			  rm "downloaded file"
			  cd ../opt
			  ln -s ../lib/component-x.x.x "component"
			  ```
			- Grafana
			  collapsed:: true
			  
			  [Download Grafana](https://grafana.com/grafana/download)
			- Prometheus, AlertManager, Node Exporter, Blackbox Exporter
			  collapsed:: true
			  
			  [Download | Prometheus](https://prometheus.io/download/)
			- Loki
			  collapsed:: true
			  
			  Since there is no easily downloadable distribution of loki you can find it under the dist folder together with the solidmon install. you can download from there.
			  
			  ```other
			  cd /apps/solidmon/lib
			  sftp xxx@solidmon.com
			  ls dist/loki*
			  # choose the lastest version unless instructed otherwise
			  get loki_amd64-x.x.x.tar.gz
			  exit
			  tar xzf loki_amd64-x.x.x.tar.gz
			  cd ../opt
			  ln -s ../lib/loki-x.x.x loki
			  ```
		- ### Preinstalled components
		  collapsed:: true
		  
		  
		  Under opt there should already be links to the following components
		  
		  ```perl
		  # Shipped as part of solidmon
		  moog_gateway -> ../lib/moog_gateway-1.1.2
		  tsa_exporter -> ../lib/tsa_exporter-1.0.9
		  ```
		  
		    These components will already have been installed as part of the solidmon distribution. Again version number will vary.
		- ### Extra Components to install
		  collapsed:: true
		  As part of the install you should download and install any other components mentioned in the release notes
	- ## Install IS
	  collapsed:: true
		- Install a local IS under `/opt...` and deploy the packages that you find under `apps/solidmon/webm`.
		- Configure as a standard inframon install
		- start IS
		- ### Package install Order
		  collapsed:: true
		  
		  1.  InfraMonData
		  2.  InfraMonIS
		  3.  InfraMon
		  4.  SolidMon
	- ## Configuration
	  collapsed:: true
		- ### Configure Region
		  collapsed:: true
			- edit the `conf/prometheus.yml`
			  > replace the string “EMEA” by the region name
			  
			  ```javascript
			  relabel_configs:
			   - target_label: "federate"
			     replacement: "true"
			   - target_label: "region"
			     replacement: "EMEA"
			  ```
		- ### Configure loki discovery
		  collapsed:: true
		  
		  > replace [itsbebelsp03288](http://itsbebelsp03288:3100/loki/api/v1/push) by the address of server you're installing
		  
		  ```javascript
		  - target_label: __param_lokiaddress
		  replacement: "http://itsbebelsp03288:3100/loki/api/v1/push"
		  ```
		  _conf/prometheus.yml_
		- ### Set Access Rights in Integration Server
		  collapsed:: true
			- Create a solidmon collector user in your target integration server
				- Make sure the user is in a group "`SolidMonCollectors`"
				- Solidmon Collectors should have two ACLS:
					- local/administrators
					- local/SolidMonCollectors
			-
			  > this needs to be done for any integration server that will be monitored
		- ### Configure the IS Password
		  collapsed:: true
			-
			  * create a file called prometheus.conf and write without a newline the password for the use chosen in prometheus.yml
		- #### Configure moog endpoint
		  collapsed:: true
		  
		  > Usually this step isn't needed as the moog Gateway is running on the same server
		  
		  edit the `conf/alertmanager.yml`
		  
		  ```yaml
		  receivers:
		  - name: "none"
		  
		  - name: "email"
		  
		  - name: "Netcool"
		    webhook_configs:
		      - url: "http://localhost:7980/prometheus_webhook_event"
		        #  - url: 'http://localhost:5001'
		  ```
			- replace the url by url to moog endpoint
		- #### Configure backup folder
		  collapsed:: true
		  
		  Edit `/apps/solidmon/conf/backup.conf`
		  
		  ```swift
		  DIR=“/apps/solidmon/backups”
		  ```
		- #### Run the post install script
		  collapsed:: true
		  
		  This script schedules needed cron jobs bash
		  
		  ```swift
		  cd /apps/solidmon/bin
		  ./solidmon_postinstall.sh
		  ```
		  
		  if there was no crontab for user it will respond “no crontab for user”, you can safely ignore it.
		  
		  To check that log rotation was added to cron
		  
		  ```swift
		  crontab -l
		  ```
		  
		  should output:
		  
		  ```shell
		  30 2 * * * /usr/sbin/logrotate -s /apps/solidmon/conf/logrotate/status /apps/solidmon/conf/logrotate/solidmon.conf > /dev/null 2>&1
		  ```
	- ## Start SolidMon
	  collapsed:: true
	  
	  Start SolidMon (see: [Operating solidmon](craftdocs://open?spaceId=1a24b3d7-de0a-7d28-2205-b3c808be324d&blockId=CBAE08C2-2061-464D-9276-41B4EE76F371) )
	- ## Check that everything started
	  collapsed:: true
	  
	  you can use a browser or curl to see that all these ports are listening
	  
	  1.  Prometheus: host:9090
	  2.  Grafana: host:3000
	  3.  Alert Manager: host:9093 If any issues check the `/apps/solidmon/logs`
	- ## Configure Grafana
	  collapsed:: true
		- ### Change Grafana admin password
		  collapsed:: true
			- log in to grafana ( port 3000) with default password admin/admin
			- change admin password when requested
			  
			    Choose the SolidMon Dashboard
		- ### Set Grafana default dashboard
		  collapsed:: true
		  1. Star the Solidmon dashboard
		  2. Set the Solidmon dashboard as the default
		  3. Log out and login with your ldap passwords to test
		- ### Configure Grafana LDAP
		  collapsed:: true
		  
		  Edit the `conf/grafana/ldap.ini` file in collaboration with our consultants if needed.
	- ## Add the servers you want to monitor
	  
	  Using the inframon interface, add the servers you want monitored. Make sure to also add the integration server you installed solidmon itself on (localhost).
- # Operating solidmon
  collapsed:: true
	- ### Starting and Stopping
	- #### Starting solidmon
	  collapsed:: true
	  
	  ```shell
	  cd /apps/solidmon/bin
	  ./solidmon_start.sh
	  ```
	- #### Stopping Solidmon
	  collapsed:: true
		- To stop all solidmon processes::
		  collapsed:: true
		  
		  ```shell
		  cd /apps/solidmon/bin
		  ./solidmon_stop.sh
		  ```
	- #### Starting and stopping individual components
	  collapsed:: true
	   use the respective start or stop script in `/apps/solidmon/bin`
		- #### Reload config without restarting
		  collapsed:: true
			- The following command will force all components to reload config without having to restart them:
			  collapsed:: true
			  
			  ```shell
			  cd /apps/solidmon/bin
			  ./solidmon_reload.sh
			  ```
			  
			  Alternatively you can stop and start them using the respective scripts.
			  
			  > Reloading using the script has the advantage that it avoids downtime.
		- #### Start Scripts
		  collapsed:: true
			- `alertmanager_start.sh`
			- `databricks_start.sh`
			- `loki_start.sh`
			- `prom_start.sh`
			- `solidmon_start.sh`
			- `blackbox_start.sh`
			- g`rafana_start.sh`
			- `node_start.sh`
			- `promtail_start.sh`
			- `prom_history.sh`
		- #### Stop Scripts
		  collapsed:: true
			- `grafana_stop.sh`
			- `solidmon_stop.sh`
	- #### Updating individual components
	  collapsed:: true
	  
	  All open source components are accessed through softlinks in the opt folder.
	  Per example for solidmon 1.0:
	  
	  ```other
	  jnj@solidmon:~/src/opt$ ls -la
	  total 8
	  drwx------  2 jnj jnj 4096 May  3 16:55 .
	  drwxrwxr-x 11 jnj jnj 4096 May  3 16:01 ..
	  lrwxrwxrwx  1 jnj jnj   39 Nov 22 12:05 alertmanager -> ../lib/alertmanager-0.19.0.linux-amd64/
	  lrwxrwxrwx  1 jnj jnj   44 May  3 09:41 blackbox_exporter -> ../lib/blackbox_exporter-0.14.0.linux-amd64/
	  lrwxrwxrwx  1 jnj jnj   21 Mar  6 10:41 grafana -> ../lib/grafana-6.6.2/
	  lrwxrwxrwx  1 jnj jnj   18 Jan 26 16:23 loki -> ../lib/loki-1.3.0/
	  lrwxrwxrwx  1 jnj jnj   40 May  3 09:40 node_exporter -> ../lib/node_exporter-0.18.0.linux-amd64/
	  lrwxrwxrwx  1 jnj jnj   37 Oct  6  2019 prometheus -> ../lib/prometheus-2.13.0.linux-amd64/
	  ```
		- ## To update follow the following steps:
		  
		  1. Download tarball to lib folder
		  2. Extract to folder including version number
		  3. Stop the component you're updating
		  4. Change the softlink to point to the new version
		  
		  ```shell
		  # go to folder
		  cd /apps/solidmon/opt
		  
		  # Delete the old link
		  rm grafana
		  
		  # create a new softlink
		  ln -s ../lib/grafana-9.0.1 grafana
		  ```
		  
		  example of how to create a new soft link
- # Loading Base Data
  collapsed:: true
	-
	  > These Instructions apply only to SolidMon for Databricks
	- ## Input file should look like this: (text tab separated export from excel)
	  collapsed:: true
	  
	  ![Image.png](https://res.craft.do/user/full/1a24b3d7-de0a-7d28-2205-b3c808be324d/doc/7704E5FE-BEEB-465B-841B-7CB65194151A/506FA672-4E51-49F5-9306-2870C001FDF4_2/Image.png)
	  
	  which when loaded into vs code
	  
	  ![Image.png](https://res.craft.do/user/full/1a24b3d7-de0a-7d28-2205-b3c808be324d/doc/7704E5FE-BEEB-465B-841B-7CB65194151A/F9734EE5-CB49-43F0-AF63-B2264E6CC593_2/Image.png)
	- ## Do the following steps
	  collapsed:: true
	  
	  1.  Delete the header row
	  2.  replace: `\tONCE\t` → `\tonce\t`
	  3.  replace: `\tstream\t` → `\tstream\t`
	  4.  replace: `\tY\t` → `\tyes\t`
	  5.  Search: .`([^\t]+)\t([^\t]+)\t([^\t]+)\t([^\t]+)\t([^\t]+)\t([^\t]+)\t([^\t]+)\t(.+)`
		- replace: `databricks_stream_table_info{ table="$1", erp="$2", table_name="$3", active="$5", project="$8", project_code="$7", frequency="$6"} 1.0`
# Directory Structure
collapsed:: true
	- ## Folder Tree
	  
	  ```other
	  📦bin                   → start scripts are here
	   ┣ 📜solidmon_start.sh	→ main start script
	  📦conf                  → all configuration files
	   ┣ 📂alerts             → IS will generate alerts for prometheus here
	   ┣ 📂grafana            → config files for grafana
	   ┣ ┣ 📂provisioning     → provisioning settings for grafana
	   ┣ ┣ 📂dashboards       → dashboards automatically loaded by grafana
	   ┣ 📂logrotate          → logrotation uses this folder to store config and status
	   ┣ 📂rules              → IS will generate rules for prometheus here
	   ┣ 📂sd                 → file based service discovery. Automatically generated by IS
	   ┃ ┣ 📂ws
	   ┃ ┣ 📜broker.yml
	   ┃ ┗ 📜is.yml
	   ┣ 📂thresholds         → thresholds for alerts are generated here by IS
	   ┃ ┣ 📂broker
	   ┃ ┣ 📂is
	   ┃ ┣ 📂ws
	   ┣ 📜alertmanager.conf  → configuration for access to netcool endpoint
	   ┣ 📜alertmanager.yml   → general config for alert manager
	   ┣ 📜blackbox.yml       → blackbox exporter config
	   ┣ 📜loki.yml           → loki config
	   ┣ 📜positions.yaml     → promtail stores here how far it is into each file, in case of restart
	   ┣ 📜prometheus.yml     → prometheus config
	   ┣ 📜promtail.yml       → promtail config
	  📦data                  → databases are stored here
	   ┣ 📂grafana            → grafana db, stores dashboards and other grafana data
	   ┣ 📂loki               → database for log monitoring
	   ┣ 📂prometheus         → prometheus time series data
	  📦docs                  → solidmon documentation is here
	  📦lib                   → download new versions of components here
	   ┣ 📂grafana-6.6.2      → per example grafana 6.6.2 was downloaded with wget and extracted here
	   ┣ ...
	  📦log                   → log files are
	  📦opt                   → start scripts are here
	   ┣ 📂grafana            → softlink to current grafana version.
	   ┣ ...
	  ```