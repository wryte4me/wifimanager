# WiFi Manager 

## Table of Contents

1. [Overview](#overview)
2. [Pre-requisites](#pre-requisites)
3. [Variables and Functions](#variables-and-functions)
   - [WiFi Configuration Struct](#wifi-configuration-struct)
   - [Init WiFi Manager](#init-wifi-manager)
   - [Config WiFi](#config-wifi)
   - [Listen Config](#listen-config)
4. [How to use](#how-to-use)
   - [Code](#code)
   - [Using the config portal](#using-the-config-portal)
5. [Frequently Asked Questions](#frequently-asked-questions)
6. [Sample Full Code](#sample-full-code)

## *Overview*

This provides an easier implementation of the  [WiFi manager v.2.0.17 by tzapu](https://github.com/tzapu/WiFiManager). It only includes copy pasting three (3) user-defined functions and a struct to your arduino code. One of which, ```initWifiManager ()```, should be called in the ```setup ()```, and the other one should be called inside the ```loop ()``` function in your sketch.

## *Pre-requisites*

1. Push button circuit connected to a pin of your ESP-based Microcontroller. This code works only by using a push-button connected to a pin connected to GND using a pull-down resistor. You may refer to the schematic below.

 ![Circuit Diagram](/assets/images/circuitdiagram/wificonfig.png)

2. Install the wifi manager library by Tzapu either directly from the Arduino IDE Library Manager or by downloading the zip file from  [GitHub](https://github.com/tzapu/WiFiManager)

## Variables and Functions

* ```Struct ``` to store config wifi variables 
```cpp
// Define the WiFi configuration struct
struct configPortal {
  const String title;      // Title of the WiFi configuration portal
  const char *apSsid;      // Access Point SSID for the configuration portal
  const char *apPassWord;  // Access Point Password for the configuration portal
  const byte trigPin;      // Pin connected to the WiFi configuration button
  const int timeout;       // Timeout duration for the configuration portal
};

```
* ```initWifimanager``` to setup WiFi Manager 
```cpp
// Initialize WiFiManager with custom parameters
void initWifiManager(const configWiFi &_wifi) {
  WiFi.mode(WIFI_STA);              // Explicitly set the wifi mode to station

  Serial.println("\nStarting");

  pinMode(_wifi.configPin, INPUT);  // Set the pin mode of the push button switch to open config portal

  WiFiManager wm;
  wm.setTitle(_wifi.configPortalTitle);

  // Check if there is a saved WiFi network
  if (wm.getWiFiIsSaved()) {
    // Autoconnect to saved WiFi network
    Serial.println("Connected to:");
    Serial.println(wm.getWiFiSSID(true));
    wm.autoConnect();  
  } else {
    // Open config portal to setup WiFi network
    Serial.println("No saved WiFi AP found.");
    configWifi(_wifi);  
  }
}
```
* ```configWifi ``` to handle config portal processes
```cpp

// Configure WiFi connection
void configWiFi(const configPortal &_wifi) {

    wm.setConfigPortalTimeout(_wifi.timeout);
    Serial.print("Opening config portal: ");

    if (!wm.startConfigPortal(_wifi.apSsid, _wifi.apPassWord)) {
        Serial.println("Config portal timeout. Press WiFi configuration button to try again.");
        delay(3000);
        ESP.restart();
        delay(5000);
    }

    Serial.print("Connected to: ");
    Serial.println(wm.getWiFiSSID(true));

    delay(2000);
}

```

* ```listenConfig``` to open config portal
```cpp
// Check if the WiFi configuration button is pressed
void checkConfig(const configWiFi &_wifi) {
  if (digitalRead(_wifi.configPin) == 1) {
    configWifi(_wifi);
  } else {
    return;
  }
}
```


## How to use

### Code
1. Include library header files
    ```cpp
    #include <WiFiManager.h>
    #include <WiFi.h>
    ```
2. Create an instance of the WiFi Manager Library
    ```cpp
    WiFiManager wm; //Create an instance of the Wifi manager library
    ```
3. Create an instance of the ```Struct ```
    ```cpp
    // Create an instance of the struct
    configPortal yourDeviceNickname = {
        "Your config portal title",     // Config portal title
        "Your AP SSID",                 // AP SSID
        "Your AP Password",             // AP Password
        4,                              // Trigger Pin 
        360                             // timeout before closing the config portal in seconds
    };
    
    ```
4. Initiate wifi manager in ```setup ()```
   
   ```cpp
    void setup (){
    Serial.begin (115200);

    initWiFiManager (yourDeviceNickname);

    // Other codes here

    }
   ```

5. Call ```listenConfig ()``` in ```loop ()```
    ```cpp
    listenConfig (yourDeviceNickName);
    ```
6. Upload sketch to your device. Make sure to choose correct board and port before uploading.

### Using the config portal

1. Press the trigger button
2. Check the serial monitor for the IP address to open the config portal
2. On your wifi capable device, connect to your AP SSID and AP Password you have set in the code.
3. When prompted, tap or click in the Sign-in to WiFi network on your device.
4. Tap on configure wifi. All available wifi networks will show in the screen. Choose your wifi network and enter its password.
5. Your device should automatically connect to the wifi network and close the config portal.

## Frequently asked questions

1. **Why is my ESP32 not connecting to the WiFi network after configuration?**
   > Ensure that you have entered the correct WiFi SSID and password during the configuration. Check the serial monitor for any error messages that might help diagnose the issue.

2. **How can I reset the WiFi configuration and open the config portal again?**
   > Press the WiFi configuration button connected to the trigPin. This will trigger the config portal to open, allowing you to reconfigure the WiFi settings.

3. **What should I do if the config portal times out before I can enter my WiFi details?**
   > Increase the timeout duration in the configPortal struct to give yourself more time to complete the configuration process.


### Sample full code
```cpp
#include <WiFiManager.h>
#include <Arduino.h>
#include <WiFi.h>

// Define the WiFi configuration struct
struct configPortal {
  const String title;      // Title of the WiFi configuration portal
  const char *apSsid;      // Access Point SSID for the configuration portal
  const char *apPassWord;  // Access Point Password for the configuration portal
  const byte trigPin;      // Pin connected to the WiFi configuration button
  const int timeout;       // Timeout duration for the configuration portal
};

// Create an instance of the struct
configPortal yourDeviceNickname = {
  "Your config portal title",  // portalTitle
  "Your AP SSID",              // apSsid
  "Your AP Password",          // apPassWord
  4,                           // trigPin
  360                          // timeout
};

// Initialize WiFiManager with custom parameters
void initWiFiManager(const configPortal &_wifi) {
  WiFi.mode(WIFI_STA);  // Explicitly set the wifi mode to station
  Serial.println("\nStarting");

  pinMode(_wifi.trigPin, INPUT);  // Set the pin mode of the push button switch to open config portal

  WiFiManager wm;
  wm.setTitle(_wifi.title);

  // Check if there is a saved WiFi network
  if (wm.getWiFiIsSaved()) {
    Serial.println("Connected to:");
    Serial.println(wm.getWiFiSSID(true));
    wm.autoConnect();  // Autoconnect to saved WiFi network
  } else {
    Serial.println("No saved WiFi AP found.");
    configWiFi(_wifi);  // Open config portal to setup WiFi network
  }
}

// Configure WiFi connection
void configWiFi(const configPortal &_wifi) {
  WiFiManager wm;

  wm.setConfigPortalTimeout(_wifi.timeout);
  Serial.print("Opening config portal: ");

  if (!wm.startConfigPortal(_wifi.apSsid, _wifi.apPassWord)) {
    Serial.println("Config portal timeout. Press WiFi configuration button to try again.");
    delay(3000);
    ESP.restart();
    delay(5000);
  }

  Serial.print("Connected to: ");
  Serial.println(wm.getWiFiSSID(true));

  delay(2000);
}

// Check if the WiFi configuration button is pressed
void listenConfig(const configPortal &_wifi) {
  if (digitalRead(_wifi.trigPin) == 1) {
    configWiFi(_wifi);
  } else {
    return;
  }
}

void setup() {
  Serial.begin(115200);

  initWiFiManager(yourDeviceNickname);

  // Other codes here
}

void loop() {
  listenConfig(yourDeviceNickname);
}



```
