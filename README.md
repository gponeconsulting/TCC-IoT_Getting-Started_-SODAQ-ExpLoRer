# SODAQ ExpLoRer Guide

This guide is designed to help beginners set up a SODAQ ExpLoRer to connect to The Things Network.

## What you will need
To follow this guide you will need the following things:
- A SODAQ ExpLoRer which can be bought [here](https://shop.sodaq.com/explorerrn2903a-us.html)
- A micro usb connector which is included with the SODAQ ExpLoRer
- A computer to connect to the SODAQ ExpLoRer and write the code

You will also need to be in range of a Gateway connected to The Things Network which you can find out about [here](https://www.thethingsnetwork.org/community).

## Step 1 - Setting up the environment
To get started you will first need to install the Arduino IDE which can be downloaded [here](https://www.arduino.cc/en/software).
After downloading the appropriate option for your system, run the installer and complete the installation process.
Once the program is done installing, open the Arduino IDE.

Now that we are in the Arduino IDE, install the SODAQ ExpLoRer Board and some libraries that will be use throughout the guide.
- To do this, first navigate to the Preferences window by going to `File > Preferences`.
- Now, in the textbox named 'Additional Boards Manager URLs:' paste the following `http://downloads.sodaq.net/package_sodaq_samd_index.json`.
- Then click the Ok button

This will allow you to find the package that includes the SODAQ ExpLoRer board.

- Next navigate to the boards manager by going to `Tools > Board: > Boards Manager...`
- Then using the search bar enter `SODAQ SAMD Boards` and install the package with the same name.
- Once the package is finished installing click the close button.

Now that we have installed the package, select your SODAQ ExpLoRer Board.

- To do so navigate and click `Tools > Board: > SODAQ SAMD (32-bits ARM Cortex-M0+) Boards > SODAQ ExpLoRer`

Now that you have selected the correct Board there are three Libraries to install.

- First navigate to the Library Manager by going to `Tools > Manage Libraries...`.

Next, much like installing the `SODAQ SAMD Boards` in the Boards Manager, Search for and install the following libraries.

- `Sodaq_RN2483`
- `Sodaq_wdt`
- `TheThingsNetwork`

Once all of the installs are complete, click close, as the environment is now finished setting up.

## Test that the SODAQ ExpLoRer is Working (Optional)
To test that the SODAQ ExpLoRer is connected properly and working complete the following steps:

- First, plug in your SODAQ ExpLoRa to your computer using the micro usb cable.
- Then Navigate to `Tools > Port:` and ensure that the port that has the SODAQ device is selected.
- Next, Navigate to `File > Examples > 01.Basics > Blink`

This will open up another window containing code that will make the LED on the SODAQ ExpLoRer Blink

- Click the arrow in the top left corner to upload the code

After a short time the code will finish uploading and the LED on the SODAQ ExpLoRer will begin to turn on and off.


## Step 2 - Getting your device EUI

The device EUI is a code that will be used to identify the device and will be needed when adding the device to The Things Network.
- First, plug in your SODAQ ExpLoRa to your computer using the micro usb cable.
- Then Navigate to `Tools > Port:` and ensure that the port that has the SODAQ device is selected.
- Next, Replace the text in the main section of the Arduino IDE with the following code
```
/**
 * Works with:
 * SODAQ Mbili
 * SODAQ Autonomo
 * SODAQ One
 * SODAQ Explorer
 *
 */

#define CONSOLE_STREAM SERIAL_PORT_MONITOR

#if defined(ARDUINO_SODAQ_EXPLORER)
#define LORA_STREAM Serial2
#else
#define LORA_STREAM Serial1
#endif

#define LORA_BAUD 57600
#define DEBUG_BAUD 57600

void setup() {
  // put your setup code here, to run once:
  // Enable LoRa module
  #if defined(ARDUINO_SODAQ_AUTONOMO)
  pinMode(BEE_VCC, OUTPUT);
  digitalWrite(BEE_VCC, HIGH); //set input power BEE high
  #endif

  //wait forever for the Serial Monitor to open
  while(!CONSOLE_STREAM);

  //Setup streams
  CONSOLE_STREAM.begin(DEBUG_BAUD);
  LORA_STREAM.begin(LORA_BAUD);

  #ifdef LORA_RESET
  //Hardreset the RN module
  pinMode(LORA_RESET, OUTPUT);
  digitalWrite(LORA_RESET, HIGH);
  delay(100);
  digitalWrite(LORA_RESET, LOW);
  delay(100);
  digitalWrite(LORA_RESET, HIGH);
  delay(1000);

  // empty the buffer
  LORA_STREAM.end();
  #endif
  LORA_STREAM.begin(57600);

  // get the Hardware DevEUI
  CONSOLE_STREAM.println("Get the hardware serial, sending \"sys get hweui\\r\\n\", expecting \"xxxxxxxxxxxxxxxx\", received: \"");
  delay(100);
  LORA_STREAM.println("sys get hweui");
  delay(100);

  char buff[16];
  memset(buff, 0, sizeof(buff));

  LORA_STREAM.readBytesUntil(0x20, buff, sizeof(buff));
  CONSOLE_STREAM.print(buff);

  CONSOLE_STREAM.println();
}

void loop() {
  // put your main code here, to run repeatedly:
}
```

- Once you have copied the code into the Arduino IDE Click the arrow in the upper left corner to upload the code to the SODAQ ExpLoRer
- After the code has finished uploading open the serial monitor by navigating to `Tools > Serial Monitor`
- Finally, Copy the 16-character EUI code from the Serial Monitor.

## Step 3 - Sign Up on The Things Network
Now that  you have your SODAQ ExpLoRer EUI code, it can be added  to The Things Network by following the steps below.

1. Create an account at [The Things Network](https://account.thethingsnetwork.org/register) or sign in if you already have an account
2. Go to the console by clicking on the profile icon and clicking the console option.
3. Select applications
4. Select add application
5. Fill out the application ID with a unique name, add a Description and press `add application`.
6. Press the `register device` button in the devices section.
7. Enter a unique name for the device ID
8. Enter the EUI code you copied from the previous step into the `Device EUI` section
9. Click `Register`
Your device will now be registered and is ready to connect



## Step 4 - Connecting The SODAQ ExpLoRer to The Things Network
Now that the device is registered on The Things Network all that is left to do is connect to the network.
- In the Arduino IDE Navigate to `File > Examples > TheThingsNetwork > SendOTAA` which will open a new window.
- In the `Example Code` section of your device page on The Things Network will be two codes called the `appEui` and the `appKey`, Copy these codes into their respective fields in the Arduino IDE .
```
// Set your AppEUI and AppKey
const char *appEui = "0000000000000000";
const char *appKey = "00000000000000000000000000000000";
```
- change the values of the variables `loraSerial`, `debugSerial` and `freqPlan` to the following.
```
#define loraSerial Serial2
#define debugSerial SerialUSB

// Replace REPLACE_ME with TTN_FP_EU868 or TTN_FP_US915
#define freqPlan TTN_FP_AU915
```
-  Click the arrow in the upper left corner to upload the code to the SODAQ ExpLoRer

After waiting for the code to upload you can now open the Serial Monitor through `Tools > Serial Monitor` and see the output. If everything went well it should post a Successful transmission every 10 seconds which you will also be able to see on the things network website in the device data section.  


## Step 5 - Customising your message
Right now, the example code is sending one byte of information to The Thing Network to tell if the LED is on or off. This can be changed to a number of different things like the output of different sensors.
For now, the output will be changed from the state of the LED to the state of the onboard button located on the opposite side of the board from the USB port.

This can be done by adding `pinMode(BUTTON, INPUT);` to the setup function to initialize the pushbutton pin as an input.
Then changing the payload to be `payload[0] = (digitalRead(BUTTON) == HIGH) ? 1 : 0;`.

When run this will make the SODAQ ExpLoRer send a 1 if the button is not being pressed and a 0 if the button is being pressed.

Below is the final Code for the setup and loop functions:
```
void setup()
{
  loraSerial.begin(57600);
  debugSerial.begin(9600);

  // Wait a maximum of 10s for Serial Monitor
  while (!debugSerial && millis() < 10000)
    ;
    
  pinMode(BUTTON, INPUT);

  debugSerial.println("-- STATUS");
  ttn.showStatus();

  debugSerial.println("-- JOIN");
  ttn.join(appEui, appKey);
}

void loop()
{
  debugSerial.println("-- LOOP");

  // Prepare payload of 1 byte to indicate LED status
  byte payload[1];
  payload[0] = (digitalRead(BUTTON) == HIGH) ? 1 : 0;

  // Send it off
  ttn.sendBytes(payload, sizeof(payload));

  delay(10000);
}
```