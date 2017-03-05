[ArduinoJson](https://github.com/bblanchon/ArduinoJson) - efficient JSON library for embedded systems. 

It has been written with Arduino in mind, but it isn't linked to Arduino libraries so you can use this library in any other C++ project. 

## Install a Library

1. Click to the "Sketch" menu and then Include Library > Manage Libraries
2. Type "ArduinoJson" in filter field;
3. Click on install and wait for the IDE to install the library.

## Using ArduinoJson with DK Gadget Controller

First, include library into project by adding the following line:

`#include <ArduinoJson.h>`

### Receive and parse command

```c++
String inputString = ""; // string for storing received data

void setup() {
  // initialize serial:
  Serial.begin(9600);
  // reserve 200 bytes for inputString:
  inputString.reserve(200);
}

void loop() {
  while (Serial.available()) {
    // get byte
    char inChar = (char)Serial.read();
    // if received symbol is a new line symbol 
    // then the command has been received completely
    if (inChar == '\n') {
      // call method that parse and execute the command
      parseCommand();
      // clean inputString
      inputString = "";
    } else {
      // add received symbol to inputString
      inputString += inChar;
    }
  }
}

/*
  Method parse and execute received command.
*/
void parseCommand() {

  // buffer for command
  StaticJsonBuffer<100> commandBuffer;

  // pointer to Json command object
  JsonObject& command = commandBuffer.parseObject(inputString);
  
  // get "cmd" value
  String cmd = command["cmd"];
  
  // depend on received command
  // perform actions
  if (cmd == "some_command_1") {
    // perform some actions as a result of receive some_command_1
  } else if (cmd == "some_command_2") {
    // perform some actions as a result of receive some_command_2
  } else if (cmd == "seek_bar_command") {
     // get seek bar value
     int value = command["params"]["value"];
     // actions ...
  } else if (cmd == "joystick_command") {
     int speed = command["params"]["speed"];
     int angle = command["params"]["angle"];
     // actions ...
  }

  // "some_command_1","some_command_2", "seek_bar_command" and "joystick_command" - strings that set
  //  in the field "command" during add widget in app.
}
```

### Prepare and send command to app

```c++

void setup() {
  // initialize serial:
  Serial.begin(9600);
}

void loop() {
  sendLedOn(); // send widget "LED" on command 
  sendDisplayData (); // get data from sensor and send to app
}

/*
  Method prepare command for "LED" widget
  and send command to app.
*/
void sendLedOn () {
  // buffer for command
  StaticJsonBuffer<100> commandBuffer;

  // pointer to Json command object
  JsonObject& command = commandBuffer.createObject();

  // initialize "cmd" value
  // this value should be the same that 
  // value set in the filed "command" during add widget in the app
  command["cmd"] = "led_on";

  // send command
  command.printTo(Serial);

  // send new line symbol
  Serial.print('\n');
}

/*
  Method prepare command for "Display" widget,
  add sensor value to command as parameter
  and send command to the app.
*/
void sendDisplayData () {
  // buffer for command
  StaticJsonBuffer<100> commandBuffer;

  // buffer for command parameters
  StaticJsonBuffer<50> paramsBuffer;

  // pointer to Json command object
  JsonObject& command = commandBuffer.createObject();

  // pointer to Json command parameters object
  JsonObject& params = paramsBuffer.createObject();

  // read data from sensor
  int sensorValue = constrain (analogRead(A0), 320, 910);
  sensorValue = map(sensorValue, 320, 910, 0, 100);

  // add sensor value to command parameters
  // this value would be shown on display in app
  params["text"] = sensorValue;
  
  // initialize "cmd" value
  // this value should be the same that 
  // value set in the filed "command" during add widget in the app
  command["cmd"] = "display_command";

  // add parameters to command
  command["params"] = params;

  // send command
  command.printTo(Serial);

  // send new line symbol
  Serial.print('\n');
}
```
### Alternative way to prepare and send commands

You also can prepare and send commands without ArduinoJson library by prepare strings yourself.

```c++
void setup() {
  // initialize serial:
  Serial.begin(9600);
}

void loop() {
  sendLedOn(); // send widget "LED" on command 
  sendDisplayData (); // get data from sensor and send to app
}

/*
  Method prepare command for "LED" widget
  and send command to app.
*/
void sendLedOn () {
  // prepare and send command
  String command = "led_on";
  Serial.print ("{\"cmd\":\""+command+"\"}\n");
}

/*
  Method prepare command for "Display" widget,
  add sensor value to command as parameter
  and send command to the app.
*/
void sendDisplayData () {
  // read data from sensor
  int sensorValue = constrain (analogRead(A0), 320, 910);
  sensorValue = map(sensorValue, 320, 910, 0, 100);
  
  // prepare and send command
  String command = "display_command";
  Serial.print ("{\"cmd\":\""+command+"\", \"params\": { \"text\":\""+sensorValue+"\"}}\n");
}
```

***

[More about ArduinoJson](https://github.com/bblanchon/ArduinoJson/wiki)