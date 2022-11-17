# plantmonitor
In our class, we have built a plant monitor that measures a plant's level of humidity, temperature and moisture. 
Put together with an Adafruit Feather Huzzah With ESP8266 board, we used a DHT22 sensor to obtain temperature and humidity values, and used a solid state soil sensor with two nails, which gave moisture readings depending on the resistance measured between the nails stuck into soil.

After building it, we published our data on the Internet through our department's MQTT server, set up a Raspberry Pi as a data store and further visualized the collected data through programs such as influxDB and Grafana.


My InfluxDB customisations:

I made use of the functions in InfluxDB's UI to make custom alert statuses for each temperature, humidity and moisture reading received from the MQTT server. With that, I defined a range of values that would assign each new reading a status of either critical, warning or ok.

InfluxDB's tutorialon this function: https://docs.influxdata.com/influxdb/cloud/monitor-alert/checks/create/

Created and configured a check
Firstly, I needed to create and configure a check, which indicates the dataset to monitor, defines check properties including the check interval and status message, and applies a specific status to each data point. In this project, I used a threshold check to assign a status to each value recieved based on whichever defined threshold they fit into. 

To create a check, I selected Alerts in the navigation menu on the left , created a new check and named it.

In the 1.Define Query section, I selected the bucket, measurement, field and tags sets that I wanted to monitor. In my case, I selected the bucket mqtt-data, where the data from my plant monitor was stored, filtered for the dataset I was monitoring under plant-topics, and filtered for _value under field. 

Next, in the 2. Configure Check section, I selected the intervals for which the query would run, which configures how frequently the check runs. In the configure a check column in the bottom left, I scheduled the check to run once every minute with 0 offset so it can label every new value that the MQTT server sends to InfluxDB every one minute. 

I left the status message template column in the middle unchanged. In the thresholds column on the bottom right, I clicked the pre-set status names, crit, warn, info or ok, to define my criteria for values to be tagged with that specific status. I defined those threshold ranges with the When value drop-down list and entered my values for the type of thresholds I wanted to set. Lastly I saved and quickly created 2 more checks with the option to clone check, which I carried out by clicking the cog icon next to the check I just created and clicking clone.

In total, I created 3 different checks for the plant's humidity, temperature and moisture levels called Plant Humidity Check, Plant Temperature Check and Plant Moisture Check respectively, each with their own different thresholds that will assign readings with different statuses.

![Screenshot (6455)](https://user-images.githubusercontent.com/114293506/202510890-3541776f-98ac-4658-b417-f5759abea5b2.png)

Fig 1. The configuration details of my Plant Humidity Check (note. I changed the time in Schedule Every from 1h to 1m)

I set those thresholds according to my plant, the Neon Pothos’, needs, as well as my own plant caring style. Since Neon Pothos enjoys high levels of moisture, temperature and humidity, and I also tend to underwater rather than overwater my plants,  I configured each check to label low value readings with warning and critical statuses. I tagged plant monitor readings that indicated that my plant was in good condition with OK status, tagged readings that my plant might not be doing so well with Warning Status, and tagged readings that meant that my plant needs care as soon as possible with Critical Status. 

The log of these alerts over time can be viewed in the tab Alert History in the Alerts tab in InfluxDB, as seen in the image below.

![Screenshot (6458)](https://user-images.githubusercontent.com/114293506/202510598-0fe519d3-7448-40e5-ac11-f1c9e274a53d.png)

Fig 2. The alert history of my plant monitor readings

Additionaly, all of these alert statuses also go into another influxDB bucket called Monitoring, which allows me to visualise the data of these statuses through InfluxDB’s Data Explorer to get a quick overview of my plant’s condition over time.

