/**************************************************************

This Sketch posts temperature data from a DS18B20 one-wire
temperature sensor to a Cosm datastream at 35-second intervals.
It uses a WiFly RN-XV wifi module from Roving Networks, which
has been set to auto-connect to a specified wireless network
automatically. The RN-XV will reboot approximately every 35
minutes.

The WiFly RN-XV is mounted to an official Arduino Wireless SD
Shield. For more information about this node, along with wiring
diagrams, visit:

http://www.mentalmunition.com/2013/02/sensor-journalism-building-wifi.html

Written by Matthew Schroyer of DroneJournalism.org to power
sensor nodes for journalists.

Originally used to post senor data to the Cosm feed at:
https://cosm.com/feeds/92154

**************************************************************/

#include <WiFlyHQ.h>
#include <OneWire.h>
#include <DallasTemperature.h>
#include <avr/wdt.h>

WiFly wifly;

#define ONE_WIRE_BUS 2
OneWire oneWire(ONE_WIRE_BUS);
DallasTemperature sensors(&oneWire);

#define APIKEY         "apikey" // cosm api key
#define FEEDID         012345 // feed ID
#define USERAGENT      "NodeJournalismFeed" // user agent is the project name

const char server[] = "api.cosm.com";

int rebootCounter = 0;

void terminal();

void setup() {
  Serial.begin(9600);
  wifly.begin(&Serial, NULL); // WiFly uses hardware serial.
  sensors.begin();
  wdt_enable(WDTO_8S); // Watchdog timer set for eight seconds
}

void loop() {
  wdt_reset();
  delayWatchdog();
  sensors.requestTemperatures();
  float tempC = sensors.getTempCByIndex(0);
  float tempF = DallasTemperature::toFahrenheit(tempC);
  sendData(tempF);
  rebootCounter++;
  
  if (rebootCounter > 60) {
    wifly.reboot();
    rebootCounter = 0;
  }
}

/*****************************************************
This function delays data transmission for 35 seconds.
Also resets the watchdog timer every five seconds.
*****************************************************/
void delayWatchdog() {
  int i = 0;
  int watchdogInterval = 5*1000;
  while (i<7) {
  delay(watchdogInterval);
  wdt_reset();
  i++;
  }
}
  

/*********************************************************
This function makes an HTTP connection to the Cosm server.
*********************************************************/
void sendData(int tempData) {
    wdt_reset(); // Watchdog reset
    wifly.open(server, 80);
    wifly.print("PUT /v2/feeds/");
    wifly.print(FEEDID);
    wifly.println(".csv HTTP/1.1");
    wifly.println("Host: api.cosm.com");
    wifly.print("X-ApiKey: ");
    wifly.println(APIKEY);
    wifly.print("User-Agent: ");
    wifly.println(USERAGENT);
    wifly.print("Content-Length: ");

    // calculate the length of the sensor reading in bytes:
    // 12 bytes for "temperature," + number of digits of the data:
    int thisLength = 12 + getLength(tempData);
    wifly.println(thisLength);

    // last pieces of the HTTP PUT request:
    wifly.println("Content-Type: text/csv");
    wifly.println("Connection: close");
    wifly.println();
    wifly.print("temperature,");
    wifly.println(tempData);
    
    // Finally, close the WiFly.
    wifly.close();
}


// This method calculates the number of digits in the
// sensor reading.  Since each digit of the ASCII decimal
// representation is a byte, the number of digits equals
// the number of bytes:

int getLength(int someValue) {
  // there's at least one byte:
  int digits = 1;
  // continually divide the value by ten, 
  // adding one to the digit count for each
  // time you divide, until you're at 0:
  int dividend = someValue /10;
  while (dividend > 0) {
    dividend = dividend /10;
    digits++;
  }
  // return the number of digits:
  return digits;
}

/***********************************************
Connect the WiFly serial to the serial monitor.
Essential for sending commands to the WiFly.
***********************************************/
void terminal()
{
    while (1) {
  if (wifly.available() > 0) {
	    Serial.write(wifly.read());
	}
	if (Serial.available() > 0) {
	    wifly.write(Serial.read());
	}
    }
}
