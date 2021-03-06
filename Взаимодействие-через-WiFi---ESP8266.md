В данной статье описан механизм взаимодействия между мобильным приложением и устройством с подключенным WiFi модулем. В качестве WiFi модуля используем ESP8266.

Взаимодействие осуществляется локально (не через Интернет). Взаимодействие посредством WiFi через Интернет будет реализовано и описано в отдельной статье.

# Настройка ESP8266

Вообще, возможно 2 варианта использования модуля:

1. Использование совместно с платой Arduino, которая будет управлять модулем по UART.
1. Написание собственной прошивки для ESP8266 и его использование как самоятоятельного устройства.

В данной статье рассматривается первый вариант.

## Подключение 

Для настройки модуля используем Arduino в качестве простого USB-to-Serial TTL конвертора.
* Зальем пустой скетч. 

```c++
void setup() { }
void loop() { }
```

* Далее замкнем RESET на GND (просто соединим их проводком).  Это изолирует процессор от I/O пинов.
* Подключим ESP8266, при этом выводы TX, RX устройства нужно подключить **напрямую**, без перекрещивания!

Теперь мы можем общаться напрямую с ESP8266 через стандартный монитор порта Arduino. В настройках монитора необходимо выбрать NL & CR. Скорость зависит от конкретной модели и прошивки ESP8266. 

## AT команды

Для настройки модуля используются AT команды. Пишем в окне монитора AT и отправляем. Результат должен быть ОК.  
```c++
AT
OK
```

В противном случае попробуйте поменять скорость подключения на одну из следующих: 9600, 57600 или 115200.

Если AT возвращает OK, вы можете задать желаемую скорость следующей командой:
```c++
AT+CIOBAUD=115200
```

Для взаимодействия через WiFi, смартфон и устройство, над которым производится управление, должны находится в одной локальной сети. 

Используя EPS8266 имеем два варианта:

1. Подключить ESP8266 и смартфон к одной WiFi сети (которую раздает, например, роутер).
1. Перевести ESP8266 в режим точки доступа и подключится к ней со смартфона.

## Подключение ESP8266 и смартфона к одной WiFi сети

Смартфон подключается к WiFi сети через стандартный интерфейс операционной системы. 

Для подключения ESP8266 к WiFi сети выполняем следующие AT команды:

* Устанавливаем режим работы ESP8266 в STA (Station)
```c++
AT+CWMODE=1
```

* Получаем список доступных WiFi сетей
```c++
AT+CWLAP
```
Пример результата выполнения команды:
```c++
+CWLAP:(4,"KrioNet",-74,"14:cc:20:aa:ba:3c",6,-9)
+CWLAP:(3,"RIO",-67,"14:cc:20:98:f7:82",8,-12)
+CWLAP:(3,"UN1RF",-87,"14:cc:20:98:e0:46",10,-4)
+CWLAP:(3,"HomeNet",-82,"08:60:6e:e2:3a:08",11,40)
```
* Подключаемся к необходимой WiFi cети
```c++
AT+CWJAP="SSID","PASSWORD"
```
Подключение длится 2-5 секунд. В случае успешного выполнения появится OK.

Пример:
```c++
AT+CWJAP="KrioNet","12345678"
WIFI CONNECTED
WIFI GOT IP

OK
```

## Подключение к ESP8266 работающей в режиме точки доступа

* Переведем ESP8266 в режим точки доступа (Soft-AP)
```c++
AT+CWMODE=2
```

* Создаем свою точку доступа
```c++
AT+CWSAP="SSID","PASSWORD",CHANNEL,SECURITY
```
Пример:
```c++
AT+CWSAP="ESP8266","12345678",11,4

OK
```

Подключаемся к созданной WiFi точке доступа со смартфона, используя стандартный интерфейс операционной системы.

## IP адресс ESP8266 

Для подключения к модулю из приложения необходим его IP адрес. Узнать его можно следующей командой:
```c++
AT+CIFSR
```

## Пример скетча для Arduino 

Теперь можно подключить модуль к RX и TX как обычно, то есть **не** напрямую. Либо можно использовать библиотеку SoftwareSerial, если необходимо оставить штатный UART свободным.

Основные моменты скетча: 
* Запускаем TCP сервер (в нашем примере порт 8888).
* Отправляем и принимаем данные с помощью AT команд.

Наконец, скетч:
```c++
#include <ArduinoJson.h>

// строка для хранения полученных данных
String inputString = "";

// так как помимо данных, полученных от приложения
// мы получаем AT команды ESP8266 нам нужен специальный флаг
boolean cmdFlag = false;

// для примера подключим светодиод к пину 6,
// будем менять его яркость
const int ledBluePin = 6;

void setup() {
  // скорость порта здесь и в ESP8266 должны соответствовать
  Serial.begin(115200);

  pinMode(ledBluePin, OUTPUT);

  // запускаем TCP сервер на ESP8266, что бы
  // иметь возможность подключится к ней со смартфона
  startTcpServer();

  // отправим команду для виджета "Светодиод"
  // в подтверждение успешной инициализации
  // и просто для примера
  sendLedOn();
}


void loop() {

  // в этом цикле обрабатываем получение команд от приложения
  while (Serial.available()) {
    // получаем байт
    char inChar = (char)Serial.read();
    // если полученный байт - это символ новой строки,
    // значит, команда получена
    if (inChar == '\n') {
      // вызываем функцию, которая распознает и выполнит команду
      parseCommand();
      // очищаем строку
      inputString = "";
      cmdFlag = false;
    } else {

      // если получен символ '{', значит, AT команды ESP8266
      // были получены и сейчас идет получение команды от приложения
      if (inChar == '{') {
        cmdFlag = true;
      }

      // если получаем команду приложения (а не AT команду ESP8266)
      // добавляем полученный символ в inputString
      if (cmdFlag == true) {
        inputString += inChar;
      }
    }
  }
}


/*
  Функция запускает TCP сервер на ESP8266
*/
void startTcpServer () {
  Serial.println("AT+CIPMUX=1");
  delay(100);
  Serial.println("AT+CIPSERVER=1,8888");
  delay(100);
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

  // в этом примере мы получаем команду от виджета "полузнок"
  // и выставляем яркость светодиода в соответсвии с положением ползунка
  if (cmd == "seek") {
    // получаем значение ползунка
    int value = command["params"]["value"];
    int outputValue = map(value, 0, 100, 0, 255);

    // выставляем яркость светодиода в соответствии со значением
    analogWrite(ledBluePin, outputValue);

    // отправляем выставленное значение в приложение в качестве подтверждения
    sendDisplayData(String(outputValue));
  }

}


/*
  Функция формирует и отправляет команду
  включения виджета "Светодиод" в приложение.
*/
void sendLedOn () {
  // вручную формируем команду
  String command = "ledOn";
  String jsonCommand = "{\"cmd\":\"" + command + "\"}\n";

  // формируем AT команду для ESP8266
  String sendATCommand = "AT+CIPSEND=0,";
  sendATCommand.concat(jsonCommand.length());

  // отпправляем AT команду для ESP8266
  // используем println (sendATCommand + '\r\n')
  Serial.println(sendATCommand);
  delay(100);

  // отправляем команду включения светодиода
  // используем print (символ перевода строки указали в jsonCommand)
  Serial.print(jsonCommand);
}


/*
  Функция формирует команду для виджета "Дисплей",
  записывает в команду значение value и
  посылает команду в приложение.
*/
void sendDisplayData (String value) {

  // вручную формируем и отправляем команду
  String command = "display";
  String jsonCommand = "{\"cmd\":\"" + command + "\", \"params\": { \"text\":\"" + value + "\"}}\n";

  // формируем AT команду для ESP8266
  String sendATCommand = "AT+CIPSEND=0,";
  sendATCommand.concat(jsonCommand.length());

  // отпправляем AT команду для ESP8266
  // используем println (sendATCommand + '\r\n')
  Serial.println(sendATCommand);
  delay(100);

  // отправляем команду включения светодиода
  // используем print (символ перевода строки указали в jsonCommand)
  Serial.print(jsonCommand);
}
```
