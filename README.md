# Lab: positioning algorithms integration

The objective of this lab consists in integrating positioning algorithms developped during TDs into a positioning server.
To maintain compatibility with code produced during TDs, the server will run with flask, a light HTTP Python framework.

## Setup

Follow the [flask tutorial](https://flask.palletsprojects.com/en/1.0.x/quickstart/) to run a flask web application.
You can test your routes with the **curl** command to forge HTTP requests.

## Routes to implement

Several routes shall be implemented to provide all required positioning server capabilities:

- Display the page for RSSI data file selection
- Loading RSSI data from a file
- Handling positioning requests by a mobile device (curl values from the test-data file).

## Selecting the RSSI file

This route loads a page with a file selection and a *Submit* button. Upon file choice and submission, the file is uploaded to the server and the route
responsible for data loading is called.

## Loading RSSI data

From the file uploaded as part of a multipart request, the content is imported into a memory data structure (see TD1)

## Handling positioning requests

This route is called by a mobile device to get its location. It uses a positioning algorithm (GNSS-like for now, switch to fingerprinting when done in TD).

This feature is routed to the path `/locate`. It has a series of parameters, the first one is `mac_addr`, that identifies the mobile device.
The following ones are pairs of MAC addresses and RSSI values defining a RSSI sample.

Again, test this feature with curl (use proper values):

curl http://server_host:server_port/locate?mac_addr=my_mac&mac1=aa:bb:cc:dd:ee:ff&rssi1=-45.2156&...&macN=aa:bb:cc:dd:ee:ff&rssiN=-54.5489

It returns a location encoded in JSON.
