***DK Gadget Controller*** is using **commands** for communicating with devices.

Commands are simple JSON objects.
During communication with the device, commands are separated by new line symbol '\n' (0x0A).

## Outgoing commands

Commands that app is sent to the device during interactions with widgets are called outgoing commands (or output commands).

***

For widgets like "***Button***" and "***Switch***" commands are presents itself strings like:
```json
{"cmd": "some_command"}
```
* "some_command" - string that set during widget adding in the app.

***

For widget "***SeekBar***":
```json
{
    "cmd": "some_command",
    "params": {
        "value": "x"    
    }
}
```
* "some_command" - string that set during widget adding in the app;
* "x" - value in a range from 0 to 100 depending on SeekBar state.

***

For widget "***Joystick***":
```json
{
    "cmd": "some_command",
    "params": {
        "speed": "x",
        "angle": "y"   
    }
}
```
* "some_command" - string that set during widget adding in the app;
* "x" - value in a range from 0 to 100 depending on distance between finger and Joystick center
* "y" - value in a range from 0 to 360 depending on Joystick state.

## Incoming commands

Commands that app receives from the device are called incoming commands (or input commands).

***
 
For widget "***Led***":
```json
{"cmd": "some_command"}
```
* "some_command" - string that set during widget adding in the app.

***

For widget "***Display***":
```json
{
    "cmd": "some_command",
    "params": {
        "text": "some_text"    
    }
}
```
* "some_command" - string that set during widget adding in the app;
* "some_text" - text for showing on display (on app screen).

## ArduinoJson

ArduinoJson - library that helps device:
* parse received command;
* prepare command for sending to app.

[Using ArduinoJson with DK Gadget Controller](https://github.com/ikrio/GadgetController/wiki/Using-ArduinoJson-with-DK-Gadget-Controller)