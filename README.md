# homebridge-http-humidity-sensor

This [Homebridge](https://github.com/nfarina/homebridge) plugin can be used integrate your humidity sensor which has a 
http api into HomeKit.

## Installation

First of all you need to have [Homebridge](https://github.com/nfarina/homebridge) installed. Refer to the repo for 
instructions.  
Then run the following command to install `homebridge-http-humidity-sensor`

```
sudo npm install -g homebridge-http-humidity-sensor
```

## Updating the humidity in HomeKit

The _'CurrentRelativeHumdity'_ characteristic from the _'HumiditySensor'_ service has the permission to `notify` the 
HomeKit controller of state changes. 
`homebridge-http-humidity-sensor` supports two ways to send humidity changes to HomeKit.

#### The 'pull' way:

The 'pull' way is probably the easiest to set up and supported in every scenario. `homebridge-http-humidity-sensor` 
requests the humidity of the sensor in an specified interval (pulling) and sends the value to HomeKit.  
Look for `pullInterval` in the list of configuration options if you want to configure it.

#### The 'push' way:

When using the 'push' concept the http device itself sends the updated value itself to `homebridge-http-humidity-sensor` 
whenever the value changes. This is more efficient as the new value is updated instantly and 
`homebridge-http-humidity-sensor` does not need to make needless requests when the value didn't actually change. 
However because the http device needs to actively notify the `homebridge-http-humidity-sensor` plugin there is more 
work needed to implement this method into your http device.  
How to implement the protocol into your http device can be read in the chapter [**Notification Server**](#notification-server)


## Configuration

The configuration can contain the following properties:
* `name` \<string\> **required**: Defines the name which is later displayed in HomeKit
* `getUrl` \<string |  [urlObject](#urlobject)\> **required**: Defines the url 
(and other properties when using an urlObject) to query the current relative humidity from the sensor. It currently expects the http 
server to return an integer ranging from 0-100 leaving out any html markup.
* `pullInterval` \<integer\> **optional**: The property expects an interval in **milliseconds** in which the plugin 
pulls updates from your http device. For more information read [pulling updates](#the-pull-way).  

Below are two example configurations. One is using a simple string url and the other is using a simple urlObject.  
Both configs can be used for a basic plugin configuration.
```json
{
    "accessories": [
        {
          "accessory": "HTTP-HUMIDITY",
          "name": "Humidity Sensor",
          
          "getUrl": "http://localhost/api/getHumidity"
        }   
    ]
}
```
```json
{
    "accessories": [
        {
          "accessory": "HTTP-HUMIDITY",
          "name": "Humidity Sensor",
          
          "getUrl": {
            "url": "http://localhost/api/getHumidity",
            "method": "GET"
          }
        }   
    ]
}
```

#### UrlObject

A urlObject can have the following properties:
* `url` \<string\> **required**: Defines the url pointing to your http server
* `method` \<string\> **optional** \(Default: **"GET"**\): Defines the http method used to make the http request
* `body` \<string\> **optional**: Defines the body sent with the http request
* `auth` \<object\> **optional**: If your http server uses basic authentication you can specify your credential in this 
object. When defined the object must contain the following properties:
    * `username` \<string\>
    * `password` \<string\>
* `headers` \<object\> **optional**: Using this object you can define any http headers which are sent with the http 
request. The object must contain only string key value pairs.  
  
Below is an example of an urlObject containing all properties:
```json
{
  "url": "http://example.com:8080",
  "method": "GET",
  "body": "exampleBody",
  
  "auth": {
    "username": "yourUsername",
    "password": "yourPassword"
  },
  
  "headers": {
    "Content-Type": "text/html"
  }
}
```

## Notification Server

`homebridge-http-humidity-sensor` can be used together with 
[homebridge-http-notification-server](https://github.com/Supereg/homebridge-http-notification-server) in order to receive
updates when the state changes at your external program. For details on how to implement those updates and how to 
install and configure `homebridge-http-notification-server`, please refer to the 
[README](https://github.com/Supereg/homebridge-http-notification-server) of the repository.

Down here is an example on how to configure `homebridge-http-humidity-sensor` to work with your implementation of the 
`homebridge-http-notification-server`.

```json
{
    "accessories": [
        {
          "accessory": "HTTP-HUMIDITY",
          "name": "Humidity Sensor",
          
          "notificationID": "my-humidity-sensor",
          "notificationPassword": "superSecretPassword",
          
          "getUrl": "http://localhost/api/getHumidity"
        }   
    ]
}
```

* `notificationID` is an per Homebridge instance unique id which must be included in any http request.  
* `notificationPassword` is **optional**. It can be used to secure any incoming requests.

To get more details about the configuration have a look at the 
[README](https://github.com/Supereg/homebridge-http-notification-server).

**Available characteristics (for the POST body)**

Down here are all characteristics listed which can be updated with an request to the `homebridge-http-notification-server`

* `characteristic` "CurrentRelativeHumidity": expects an int `value` in a range of 0-100