# CASA0014 plant monitor
In our class, we have built a plant monitor that measures a plant's level of humidity, temperature and moisture. 
Put together with an Adafruit Feather Huzzah With ESP8266 board, we used a DHT22 sensor to obtain temperature and humidity values, and used a solid state soil sensor with two nails, which gave moisture readings depending on the resistance measured between the nails stuck into soil.

After building it, we published our data on the Internet through our department's MQTT server, set up a Raspberry Pi as a data store and further visualized the collected data through programs such as influxDB and Grafana.

<p><strong>Section 2: My InfluxDB customisations</strong></p>
<p>I made use of the Alert function in InfluxDB's UI to make custom alert statuses for each temperature, humidity and moisture reading recieved from the MQTT server. With that, I defined a range of values that would assign each new reading recieved a status of either <strong>crit</strong>, <strong>warn</strong> or <strong>ok</strong>.</p>

InfluxDB’s documentation of this function can be found [here](https://docs.influxdata.com/influxdb/cloud/monitor-alert/checks/create/).

<p>Firstly, I needed to <strong>create and configure a check</strong>, which <strong>indicates the dataset to monitor</strong>, <strong>defines check properties</strong> including the<strong> check interval </strong>and <strong>status message</strong>, and <strong>applies a specific status to each data point</strong>. In this project, I used a <strong>threshold check</strong> to assign a status to each value recieved based on whichever defined threshold they fit into. </p>

<p>To <strong>create a check</strong>, I selected <strong>Alerts</strong> from the <strong>navigation menu on the left</strong> , created a new check and named it.</p>

<p>In the <strong>1. Define Query section</strong>, I selected the <strong>bucket</strong>, <strong>measurement</strong>, <strong>field</strong> and <strong>tag sets</strong> that I wanted to monitor.</p>

<p> In my case, I selected the bucket mqtt-data, where the data from my plant monitor was stored, filtered for the dataset I was monitoring under plant-topics, and filtered for _value under field. </p>

<p>Next, in the <strong>2. Configure Check section</strong>, I selected the <strong>intervals for which the query would run</strong>, which configures how frequently the check is run. In the <strong>Configure a Check column</strong> in the<strong> bottom left</strong>, I scheduled the check to run once every minute with 0 offset so the check can label every new value that the MQTT server sends to InfluxDB once per minute. I left the <strong>Status Message Template column</strong> in the<strong> middle</strong> unchanged. In the <strong>Thresholds column</strong> on the bottom right, I clicked the <strong>pre-set status names</strong>, <strong>crit</strong>, <strong>warn</strong>, <strong>info</strong> or <strong>ok</strong>, to define my criteria for values to be tagged with that specific status.</p>

<p>I <strong>demarcated my threshold ranges</strong> with the <strong>When value drop-down list</strong>, in which I entered values for the thresholds I wanted to set for each status. Lastly, I saved and quickly created 2 more checks with the option to <strong>clone check</strong>, which I carried out by<strong> clicking the cog icon next to the check</strong> I recently created, and choosing <strong>clone</strong>.</p>

<p>In total, I created 3 different checks for the plant's humidity, temperature and moisture levels called Plant Humidity Check, Plant Temperature Check and Plant Moisture Check respectively, each with their own thresholds for assigning statuses to each different type of reading.</p>

![plant_monitor_check](https://user-images.githubusercontent.com/114293506/202552517-3c6e8165-0340-4b48-b6f7-4b193769b19c.png)
<p> <em>Fig 1. The configuration details of my Plant Humidity Check. (note. I changed the time in Schedule Every from 1h to 1m) </em></p>

<p><strong>Determining threshold values:</p></strong>
<p>I set those thresholds with respect to my plant, the Neon Pothos', needs, as well as my own plant caring style. Since Neon Pothos enjoys high levels of moisture, temperature and humidity, while I also  have the tendency to underwater instead of overwater my plants,  I configured each check to label low value readings with warning and critical statuses. I tagged plant monitor readings that indicated that my plant was in good condition with OK status, tagged readings that indicated that my plant might not be doing so well with warn status, and tagged readings that indicated that my plant needs care as soon as possible with crit status. </p>

<p><strong>Alert History:</p></strong>
<p>The<strong> output of these checks over time</strong> can be viewed in the tab <strong>Alert History</strong> in the <strong>Alerts tab</strong> in InfluxDB. It shows the <strong>time of the check</strong>, the <strong>check name</strong>, the <strong>result of the check</strong>, which in my case indicates <strong>the status of each value</strong>, and the <strong>status message</strong>.</p>

![plant_monitor_alert_history](https://user-images.githubusercontent.com/114293506/202552429-b300ff69-b759-45b9-ac7b-3d4b7917d1ae.png)
<p><em>Fig 2. The alert history of my plant monitor readings.</em></p>

InfluxDB also stores the check output in the <strong>statuses</strong> measurement in a <strong>system bucket</strong> called <strong>_monitoring</strong>. Statuses are further separated into <strong>fields</strong> and <strong>tags</strong>, in which the tag <strong>_level</strong> specifically stores the <strong>status level evaluated by the check</strong>, which can then be further filtered by ok, info, warn and crit. The developer documentation of this InfluxDB system bucket which describes its schema in more depth can be found [here](https://docs.influxdata.com/influxdb/cloud/reference/internals/system-buckets/). 
Importantly, filtering for the results of these checks with the various measurements collated in the bucket<strong> enables them to be visualised through Data Explorer</strong>.

![plant_monitor_data_explorer](https://user-images.githubusercontent.com/114293506/202552381-738fcb3f-2f4a-4b7e-a6a3-c1d18c0c8075.png)
<p><em>Fig 3. The display of humidity readings with an OK status from the bucket _monitoring, made through Data Explorer. The humidity values are shown because they are stored in the <strong>tag _source_measurement</strong>, which logs the original measurement queried by the check.</em></p>

<p>Supplementarily applying the InfluxDB Alerts functions to the data from my plant monitor helps me gain a clear overview of my plant’s condition over time. Since this function is easy to set up and has the potential to produce useful results, I think it is worth implementing in projects similar to the plant monitor. I hope this mini guide of the InfluxDB feature I added can help others intepret sensor values in a more human-friendly way.</p>

