# Lab: positioning algorithms integration

The objective of this lab consists in integrating positioning algorithms developped during TDs into a positioning server.
To maintain compatibility with code produced during TDs, the server will run with flask, a light HTTP Python framework.

## Setup

Follow the [flask tutorial](https://flask.palletsprojects.com/en/1.0.x/quickstart/) to run a flask web application.
You can test your routes with the **curl** command to forge HTTP requests.

## Routes to implement

There are four main features to implement to the server, each one corresponding to a route:

- Handling RSSI sample packet from the probes
- Handling calibration start requests by a mobile device
- Handling calibration stop requests by a mobile device
- Handling positioning requests by a mobile device

To use the ORM mapping classes, fingerprint_value and calibrating_mobile, you must open the rssi.db file sqlite3 rssi.db and run the following SQL statements:

```sql
CREATE TABLE fingerprint_value(id integer primary key, loc_id integer not null, ap_id integer not null, foreign key(ap_id) references accesspoint(id), foreign key(loc_id) references location(id));
CREATE TABLE calibrating_mobile(mac_address text primary key, loc_id integer not null, foreign key(loc_id) references location(id));
```

## Handling RSSI from probes

This task is handled by route `/rssi`, where a probe request will be processed as follows: it is composed of a parameter ap whose value is the sending probe MAC address, and of a series of pairs XX:XX:XX:XX:XX:XX=-YY.YYYY where:

- the X's are the measured devices MAC addresses hexadecimal characters
- and the Y's are the avg RSSI value digits for the corresponding MAC address over the last second

You have to put these information in the sqlite3 database named **rssi.db** whose schema can be displayed from the sqlite3 prompt through the command .schema.

## Calibration start requests

This feature is routed to the path `/start_calibration`. A device must send its MAC address and its coordinates (x, y, z) to the server. Upon receiving these parameters, the positioning server moves samples from the sample table if they are less than 1 second old and their source address equals those of the device query. It also puts the location and the mobile device MAC address into table calibrating_mobile.

Then, the `/rssi` route also puts into fingerprint_value table all RSSI received for devices in calibrating_mobile table along with the AP MAC address who issued the RSSI and the corresponding location from the calibrating_mobile table.

You can test the calibration start request by using curl from a linux laptop or from a smartphone (then, use a browser instead of curl) with the following command:

`curl http://server_host:server_port/start_calibration?mac_addr=my_mac&x=my_x&y=my_y&z=my_z`

where:

- my_mac is your mobile device Wi-Fi interface card MAC address (see ifconfig or ip addr commands)
- my_x, my_y and my_z are the coordinates of your location when running the curl command.

## Calibration stop requests

This feature is routed to the path `/stop_calibration`. The server removes any entry whose mac_address matches the parameter mac_address from table calibrating_mobile.

You can test this feature with curl:

`curl http://server_host:server_port/stop_calibration?mac_addr=my_mac`

## Handling positioning requests

This route is called by a mobile device to get its location. It uses a positioning algorithm (GNSS-like for now, switch to fingerprinting when done in TD).

This feature is routed to the path `/locate`. It has a series of parameters, the first one is `mac_addr`, that identifies the mobile device.
The following ones are pairs of MAC addresses and RSSI values defining a RSSI sample.

Again, test this feature with curl (use proper values):

`curl http://server_host:server_port/locate?mac_addr=my_mac&mac1=aa:bb:cc:dd:ee:ff&rssi1=-45.2156&...&macN=aa:bb:cc:dd:ee:ff&rssiN=-54.5489`

It returns a location encoded in JSON.
