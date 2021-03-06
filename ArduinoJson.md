[ArduinoJson](https://github.com/bblanchon/ArduinoJson) - это эффективная библиотека для встраиваемых систем. 

Она была разработана для Arduino, но вы можете использовать её и на других платформах, в любом C++ проекте.

## Подключение библиотеки в среде Arduino

1. В меню Скетч -> Подключить библиотеку выберите "Управлять библиотеками";
2. В поле фильтра введите "ArduinoJson";
3. Кликните по появившемуся полю, выберите последнюю версию и нажмите "Установка".

## Использование библиотеки

Что бы использовать возможности библиотеки в скетче добавьте следующую линию в начало программы:

`#include <ArduinoJson.h>`

### Получаем и распознаем команду

```c++
String inputString = ""; // строка для хранения полученных данных

void setup() {
  // инициализируем serial:
  Serial.begin(9600);
  // резервируем 200 байт для inputString:
  inputString.reserve(200);
}

void loop() {
  while (Serial.available()) {
    // получаем байт
    char inChar = (char)Serial.read();
    // если полученный байт - это символ новой строки,
    // значит команда получена
    if (inChar == '\n') {
      // вызываем функцию, которая распознает и выполнит команду
      parseCommand();
      // очищаем строку
      inputString = "";
    } else {
      // добавляем полученный символ в inputString
      inputString += inChar;
    }
  }
}

/*
  Функция распознает и выполняет полученную команду.
*/
void parseCommand() {

  // пул памяти для команды
  StaticJsonBuffer<100> commandBuffer;

  // ссылка на Json объект команды
  JsonObject& command = commandBuffer.parseObject(inputString);
  
  // извлекаем значение "cmd"
  String cmd = command["cmd"];
  
  // в зависимости от полученной команды 
  // выполняем определенные действия
  if (cmd == "команда_1") {
    // действия при получении команды_1
  } else if (cmd == "команда_2") {
    // действия при получении команды_2 
  } else if (cmd == "команда_ползунка") {
     // получаем значение ползунка
     int value = command["params"]["value"];
     // действия ...
  } else if (cmd == "команда_джойстика") {
     int speed = command["params"]["speed"];
     int angle = command["params"]["angle"];
     // действия ...
  }

  // "команда_1","команда_2", "команда_ползунка" и "команда_джойстика" - это
  //  строки указанные в поле "команда" при добавлении виджета в приложении
}
```

### Формируем команду и отправляем в приложение

```c++

void setup() {
  // инициализируем serial:
  Serial.begin(9600);
}

void loop() {
  sendLedOn(); // отправляем команду для включения виджета "Светодиод"
  sendDisplayData (); // получаем данные с датчика и отправляем в приложение
}

/*
  Функция формирует и отправляет команду 
  включения виджета "Светодиод" в приложение.
*/
void sendLedOn () {
  // пул памяти для команды
  StaticJsonBuffer<100> commandBuffer;

  // ссылка на Json объект команды
  JsonObject& command = commandBuffer.createObject();

  // добавляем значение "cmd"
  // это значение должно соответствовать сроке, указанной 
  // в поле "Команда" при добавлении виджета в приложении
  command["cmd"] = "команда_включения";

   // отправляем команду
  command.printTo(Serial);

  // отправляем символ новой строки
  Serial.print('\n');
}

/*
  Функция формирует команду для виджета "Дисплей",
  записывает в команду значение с датчика и
  посылает команду в приложение.
*/
void sendDisplayData () {
  // пул памяти для команды
  StaticJsonBuffer<100> commandBuffer;

  // пул памяти для параметров команды
  StaticJsonBuffer<50> paramsBuffer;

  // ссылка на Json объект команды
  JsonObject& command = commandBuffer.createObject();

  // ссылка на Json объект параметров
  JsonObject& params = paramsBuffer.createObject();

  // получаем данные с датчика
  int sensorValue = constrain (analogRead(A0), 320, 910);
  sensorValue = map(sensorValue, 320, 910, 0, 100);

  // добавляем значение датчика в объект параметров
  // это значение будет показано на дисплее в приложении
  params["text"] = sensorValue;
  
  // добавляем значение "cmd"
  // это значение должно соответствовать сроке, указанной 
  // в поле "Команда" при добавлении виджета в приложении
  command["cmd"] = "команда_дисплея";

  // добавляем параметры в объект команды
  command["params"] = params;

  // отправляем команду
  command.printTo(Serial);

  // отправляем символ новой строки
  Serial.print('\n');
}
```
### Альтернативный способ сформировать и отправить команду

Вы также можете вручную сформировать и отправить команду без использования ArduinoJson.
Будьте внимательны, указывая JSON строки!

```c++
void setup() {
  // инициализируем serial:
  Serial.begin(9600);
}

void loop() {
  sendLedOn(); // отправляем команду для включения виджета "Светодиод"
  sendDisplayData (); // получаем данные с датчика и отправляем в приложение
}

/*
  Функция формирует и отправляет команду 
  включения виджета "Светодиод" в приложение.
*/
void sendLedOn () {
  // вручную формируем и отправляем команду
  String command = "команда";
  Serial.print ("{\"cmd\":\""+command+"\"}\n");
}

/*
  Функция формирует команду для виджета "Дисплей",
  записывает в команду значение с датчика и
  посылает команду в приложение.
*/
void sendDisplayData () {
  // получаем данные с датчика
  int sensorValue = constrain (analogRead(A0), 320, 910);
  sensorValue = map(sensorValue, 320, 910, 0, 100);
  
  // вручную формируем и отправляем команду
  String command = "команда";
  Serial.print ("{\"cmd\":\""+command+"\", \"params\": { \"text\":\""+sensorValue+"\"}}\n");
}
```

***

[Подробно о ArduinoJson](https://github.com/bblanchon/ArduinoJson/wiki)