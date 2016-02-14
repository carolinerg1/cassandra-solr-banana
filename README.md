# cassandra-solr-banana
Visualizing DSE Cassandra Solr with Banana


Requirements
1. DataStax Enterprise (current DSE version: 4.8.4):  https://academy.datastax.com/downloads
2. Luciworks Banana:  https://github.com/lucidworks/banana



Setup
1. Install DSE:  https://docs.datastax.com/en/datastax_enterprise/4.8/datastax_enterprise/install/installTOC.html


2. Start DSE with Solr enabled
No service/Tarball installation:  $ bin/dse cassandra -s
or
Services/installer: set SOLR_ENABLED=1 in /etc/default/dse

http://docs.datastax.com/en/datastax_enterprise/4.8/datastax_enterprise/srch/srchInstall.html?scroll=srchInstall__srchStrtStp


3. Create Cassandra Keysapce and Table 

cqlsh> 
 CREATE KEYSPACE IF NOT EXISTS sensor WITH replication = { 'class': 'SimpleStrategy', 'replication_factor': '1'};

 CREATE TABLE IF NOT EXISTS sensor.daily (
 	stationid text,
	event_timestamp timestamp,
	metric text,
	location text,
	min int,
	max int,
	mean int,
	median int,
	percentile1 int,
	percentile5 int,
	percentile95 int,
	percentile99 int,
	total int,
	PRIMARY KEY (stationid, metric, event_timestamp)
);


4. Load Data (weather data file included in DSE demos)
cqlsh> COPY sensor.daily (stationid, metric, event_timestamp, location, max, mean, median, min, percentile1, percentile5, percentile95, percentile99, total) FROM '$DSE_HOME/demos/weather_sensors/resources/daily.csv' WITH HEADER='true';


5. Create Solr Index
$ bin/dsetool create_core sensor.daily generateResources=true reindex=true


6. Clone Banana (https://github.com/lucidworks/banana) to DSE's "resources" folder
Services/Installer:  /usr/share/dse/resources
No services/tarball: install_location/resources


7. Configure Banana

7.1 $DSE_HOME/resources/banana/src/config.js, change "banana_index" to "banana.dashboards"

7.2 Post schema.xml and solrconfig.xml

- Using solrconfig.xml from Step #5 above:
$ curl --data-binary @solrconfig.xml -H 'Content-type:text/xml; charset=utf-8' "http://localhost:8983/solr/resource/banana.dashboards/solrconfig.xml"

$ curl --data-binary @schema.xml -H 'Content-type:text/xml; charset=utf-8' "http://localhost:8983/solr/resource/banana.dashboards/schema.xml"

$ curl -X POST -H 'Content-type:text/xml; charset=utf-8' "http://localhost:8983/solr/admin/cores?action=CREATE&name=banana.dashboards"

(if you make changes to schema.xml: curl -X POST -H 'Content-type:text/xml; charset=utf-8' "http://localhost:8983/solr/admin/cores?action=CREATE&name=banana.dashboards")

7.3 Update $DSE_HOME/resources/tomcat/conf/server.xml, add the following inside the <Host> tags:

<Context docBase="../../banana/src" path="/banana" />


8. Delete $DSE_HOME/resources/tomcat/work


9. Restart DSE:  https://docs.datastax.com/en/datastax_enterprise/4.8/datastax_enterprise/startStop/refDseStartStopDse.html


10. Configure Banana Dashboard:  http://localhost:8983/banana
Upper right menu: New -> Timeseries Dashboard -> change Collection to "sensor.daily" 

Make sure Time Field is event_timestamp (should match name of column that contains time in table definition)







 
