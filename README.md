# logstash-tcpdump
configure logstash with tcpdump for geoip

https://www.youtube.com/watch?v=vPRzCoFMl4I

### ref for 1 
#https://github.com/og-devops/logstash-tcpdump/blob/master/input-geoip.conf
1) configure logstash with input-geoip.conf 
note: filter is based on the command ran on router i.e. "tcpdump -nS -i eth1.2 -s0 -tttt". adjust accordingly to the tcpdump command you use. output in this example sends to elasticsearch running locally.

2) log into your openwrt router

### ref for 3 -4
#https://github.com/og-devops/logstash-tcpdump/blob/master/tcpdump.locally.on.openwrt.txt
3) generate the tcpdump file
tcpdump -nS -i eth1.2 -s0 -tttt > tcpdump.sample.txt

4) download tcpdump.sample.txt file from router 
scp root@192.168.0.1:/etc/config/tcpdump.sample.txt /home/unitelife/Desktop/geoiptest.txt

### ref for 5
#https://github.com/og-devops/logstash-tcpdump/blob/master/stdout.cmd.txt
5) run logstash to parse tcpdump.sample.txt and sends/creates index with auto mapping on elastic stack 
./logstash --path.logs /var/log/logstash/ --path.data /var/log/logstash -f /etc/logstash/conf.d/input-geoip.conf


### ref for 6 - 7
#https://github.com/og-devops/logstash-tcpdump/blob/master/update.mappings.via.devtools.txt
6) go to kibana dashboard and search for newly created index.
GET /logstash-tcpdump/_mapping

7) copy the mapping and add to:
PUT /logstash-tcpdump/
{
(insert mapping results here and clean up any trailing "}")  <===
}

8) edit the following
change:
"location" : {
  "properties" : {
    "lat" : {
      "type" : "float"
    },
    "lon" : {
      "type" : "float"
    }
  }
},

to:
"location" : { "type" : "geo_point"},

9) delete index "logstash-tcpdump" on kibana so we can create a new index with the updated mapping for geo_point

10) click PUT /logstash-tcpdump/ arrow on kibana dev tools to send info again and create new index with updated mapping for geo_point

11) verify mapping for locations contains geo_point by running: 
GET /logstash-tcpdump/_mapping

12) Create index pattern "logstash-tcpdump"

13) re-run steps 2-4 to generate new tcpdump data and ingest them through logstash to send to elastic seasrch

14) nav to maps and add layer with the newly added geo_point mapping for locations
