# ESP32: Simple HTTP-server

This project is a basic HTTP-server capable of receiving http requests and responding accordingly. 
By default it implements the following HTTP verbs: GET, POST, PUT, PATCH, and DELETE so it can also be easily RESTfull servers.

This server uses the ESP32's [oficial development framework](http://esp-idf.readthedocs.io/en/latest/index.html) (ESP-IDF)

## Getting Started

These instructions will get you a copy of the project up and running on your local machine for development and testing purposes.

### Prerequisites

What things you need to install the software and how to install them

* The toolchain [Guide](http://esp-idf.readthedocs.io/en/latest/get-started/windows-setup.html)
* The latest ESP-IDF [Github](https://github.com/espressif/esp-idf)

### Running

Before compiling you should first specify the SSID and Password:
```
#define AP_TARGET_SSID "Your SSID"
#define AP_TARGET_PASSWORD "Your password"
```

By default the ESP32 is setup to use a dynamic IP. If you wish to use a static IP uncomment, and adjust the following settings:

```
#define DEVICE_IP          "192.168.178.4"
#define DEVICE_GW          "192.168.178.1"
#define DEVICE_NETMASK     "255.255.255.0"
```


If the ESP32 has sucesfully connected to the WIFI network you should something like:
```
I (275) wifi: mode : sta (30:ae:a4:8f:6b:a8)
I (761) wifi: n:4 0, o:1 0, ap:255 255, sta:4 0, prof:1
I (1420) wifi: state: init -> auth (b0)
I (1424) wifi: state: auth -> assoc (0)
I (1428) wifi: state: assoc -> run (10)
I (1460) wifi: connected with 'Your SSID', channel 4
I (1461) wifi: pm start, type: 1

I (3837) event: sta ip: 192.168.178.122, mask: 255.255.255.0, gw: 192.168.178.1
```


### configuration
 
#### Starting the server

Like all the examples using the ESP-IDF this server uses freeRTOS.
To start the http server you have to create a task that runs the function 'http_server
```
xTaskCreate(&http_server, "http_server", 2048, NULL, 5, NULL);		// Starts server at port 80
xTaskCreate(&http_server, "http_server", 2048, (void *) 123, 5, NULL);	// Starts server at port 123
```
Here pvParameters is abbused to pass the desired port to the function (normally this is a pointer, but this is converted back to int in 'http_server'). If pvParameters is left to be NULL it defaults to port 80.


#### Handling requests
When a request is received the payload is placed in a variable called 'payload'.
the following request below results in payload holding the string '/index.html?param1=value1&param2=value2'
```
GET /index.html?param1=value1&param2=value2 HTTP/1.1
Host: 192.168.178.122:404
Cache-Control: no-cache
```

As this is a minimal server the actual parsing of the string is not implemented (note that the cJSON library is suplied with the ESP_IDF, so parsing json can be implemented quite easily).


before sending data or properly terminate a connection, the ESP32 should always send a html header containing the http status code.
This is done with `netconn_write(conn, HDR_200, sizeof(HDR_200)-1, NETCONN_NOCOPY);`

The implemented http status codes are:
* HDR_200 'OK'
* HDR_201 'created'
* HDR_204 'No content'
* HDR_404 'Not found'
* HDR_405 'Method not allowed'
* HDR_409 'conflict'
* HDR_501 'Not implemented'

*the selection is based on the [REST API tutorial](http://www.restapitutorial.com/lessons/httpmethods.html) but can easily be extended.*

to reply with data (consecutive) calls to  `netconn_write(conn, "html code as string", sizeof("html code as string")-1, NETCONN_NOCOPY);` can be made.