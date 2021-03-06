Spark Core Firmware
==========

Functions
=====

Cloud
---

### Spark.variable()

Expose a *variable* through the Spark Cloud so that it can be called with `GET /v1/devices/{DEVICE_ID}/{VARIABLE}`.

```C++
EXAMPLE USAGE
int temperature = 0;

void setup()
{
  Spark.variable("temperature", &temperature, INT);
  pinMode(A0, INPUT);
}

void loop()
{
  temperature = analogRead(A0);
}
```

COMPLEMENTARY API CALL

```json
# EXAMPLE REQUEST IN TERMINAL
# Core ID is 0123456789abcdef01234567
# Your access token is 1234123412341234123412341234123412341234
curl "https://api.spark.io/v1/devices/0123456789abcdef01234567/temperature?access_token=1234123412341234123412341234123412341234"
```


### Spark.function()

Expose a *function* through the Spark Cloud so that it can be called with `POST device/{FUNCTION}`.

Currently the application supports the creation of upto 4 different Spark functions.

```C++
SYNTAX TO REGISTER A SPARK FUNCTION
Spark.function("funcKey", funcName);
                  ^
                  |
     (max of 12 characters long)
```

In order to register a Spark function, the user provides the `funcKey`, which is the string name used to make a POST request and a `funcName`, which is the actual name of the function that gets called in the Spark app. The Spark function must always return an `INT` value of either `1` (success) or `-1` (failure).

The length of the `funcKey` is limited to a max of 12 characters.

A Spark function is set up to take one argument of the [String](http://arduino.cc/en/Reference/StringObject) datatype. This argument length is limited to a max of 64 characters.

```C++
EXAMPLE USAGE
int brewCoffee(String command);

void setup()
{
  //register the Spark function
  Spark.function("brew", brewCoffee);
}

void loop()
{
  //this loops forever
}

//this function automagically gets called upon a matching POST request
int brewCoffee(String command) 
{
  //look for the matching argument "coffee" <-- max of 64 characters long
  if(command == "coffee")
  {
    //do something here
    activateWaterHeater();
    activateWaterPump();
    return 1;
  }
  else return -1;
}
```

```json
COMPLEMENTARY API CALL
POST /v1/devices/{DEVICE_ID}/{FUNCTION}

# EXAMPLE REQUEST
curl https://api.spark.io/v1/devices/0123456789abcdef01234567/brew \
     -d access_token=1234123412341234123412341234123412341234 \
     -d "args=coffee"
```

The API request will be routed to the Spark Core and will run your brew function. The response will have a return_value key containing the integer returned by brew.


### Spark.event()

Send an *event* through the Spark Cloud that will be forwarded to registered callbacks and server-sent event streams.

This feature will allow the Core to generate an *event* based on a condition. For example, you could connect a motion sensor to the core and have a the Core generate an event whenever motion is detected. Unlike the Spark variable, you don't have to send out an API GET request.

This feature is currently under implementation.

<!-- TO DO -->
<!--
```C++
SYNTAX
Spark.event(event_name, event_result);

EXAMPLE USAGE
int motion_sensor = 0;
int sensor_value = 0;

void loop() {
  sensor_value = analogRead(motion_sensor);

  if (sensor_value == 1) {
    Spark.event("motion", "motion detected");
  }
}

COMPLEMENTARY API CALL
I guess this should be callback registration...?
```
-->

### Spark.connected()

Returns `true` when connected to the Spark Cloud, and `false` when disconnected to the Spark Cloud.

```C++
SYNTAX
Spark.connected();

RETURNS
boolean (true or false)

EXAMPLE USAGE
void setup() {
  Serial.begin(9600);
}

void loop() {
  if (Spark.connected()) {
    Serial.println("Connected!");
  }
  delay(1000);
}
```
<!-- TO DO 
### Spark.disconnect()

Disconnects the Spark Core from the Spark Cloud.

```C++
SYNTAX
Spark.disconnect()

EXAMPLE USAGE
Hmm, not sure what this one should look like...
```

NOTE: When the Core is disconnected, over-the-air updates are no longer possible. To re-enable over-the-air firmware updates, initiate a factory reset.



### Spark.connect()

Re-connects the Spark Core to the Spark Cloud after `Spark.disconnect()` is called.

```C++
SYNTAX
Spark.connect()
```

The Spark Core connects to the cloud by default, so it's not necessary to call `Spark.connect()` unless you have explicitly disconnected the Core.
-->

<!-- TO DO -->
<!-- Add example implementation here -->
<!--
### Spark.print()

Prints to the debug console in Spark's web IDE.
-->

<!-- TO DO -->
<!-- Add example implementation here -->

<!--
### Spark.println()

Prints to the debug console in Spark's web IDE, followed by a *newline* character.
-->
<!-- TO DO -->
<!-- Add example implementation here -->

Sleep
---

### Spark.sleep()

`Spark.sleep()` can be used to dramatically improve the battery life of a Spark-powered project by temporarily deactivating the Wi-Fi module, which is by far the biggest power draw.

```C++
SYNTAX
Spark.sleep(int seconds);
```

```C++
// EXAMPLE USAGE: Put the Core to sleep for 5 seconds
Spark.sleep(5);
// The Core LED will flash green during sleep
```

`Spark.sleep()` can also be used to put the entire Core into a *deep sleep* mode. In this particular mode, the Core shuts down the Wi-Fi chipset (CC3000) and puts the microcontroller in a stand-by mode.

```C++
SYNTAX
Spark.sleep(SLEEP_MODE_DEEP, int seconds);
```

```C++
// EXAMPLE USAGE: Put the Core into deep sleep for 60 seconds
Spark.sleep(SLEEP_MODE_DEEP,60);
// The Core LED will shut off during deep sleep
```
The Core will automatically *wake up* and reestablish the WiFi connection after the specified number of seconds.

In *standard sleep mode*, the Core current consumption is in the range of: **15mA to 30mA**

In *deep sleep mode*, the Core current consumption is around: **3.2 μA**

<!--
Spark.sleep(int millis, array peripherals);
-->

<!--
`Spark.sleep()` can also take an optional second argument, an `array` of other peripherals to deactivate. Deactivating unused peripherals on the micro-controller can take its power consumption into the micro-amps.
-->

<!-- TO DO -->
<!-- Add example implementation here -->

Input/Output
---

### pinMode()

`pinMode()` configures the specified pin to behave either as an input or an output. 

```C++
SYNTAX
pinMode(pin,mode);
```

`pinMode()` takes two arguments, `pin`: the number of the pin whose mode you wish to set and `mode`: `INPUT, INPUT_PULLUP, INPUT_PULLDOWN or OUTPUT.`

`pinMode()` does not return anything.

```C++
EXAMPLE USAGE
int button = D0;                       // button is connected to D0
int LED = D1;                          // LED is connected to D1 

void setup()
{
  pinMode(LED, OUTPUT);               // sets pin as output
  pinMode(button, INPUT_PULLDOWN);    // sets pin as input
}

void loop()
{
  while(digitalRead(button) == HIGH)  // blink the LED as long as the button is pressed
  {
    digitalWrite(LED, HIGH);          // sets the LED on
    delay(200);                       // waits for 200mS
    digitalWrite(LED, LOW);           // sets the LED off
    delay(200);                       // waits for 200mS
  }
}
```

### digitalWrite()

Write a `HIGH` or a `LOW` value to a digital pin.

```C++
SYNTAX
digitalWrite(pin, value);
```

If the pin has been configured as an OUTPUT with pinMode(), its voltage will be set to the corresponding value: 3.3V for HIGH, 0V (ground) for LOW.

`digitalWrite()` takes two arguments, `pin`: the number of the pin whose value you wish to set and `value`: `HIGH` or `LOW`.

`digitalWrite()` does not return anything.

```C++
EXAMPLE USAGE
int LED = D1;                       // LED is connected to D1

void setup()
{
  pinMode(LED, OUTPUT);             // sets pin as output
}

void loop()
{
  digitalWrite(LED, HIGH);          // sets the LED on
  delay(200);                       // waits for 200mS
  digitalWrite(LED, LOW);           // sets the LED off
  delay(200);                       // waits for 200mS
}
```

### digitalRead()

Reads the value from a specified digital `pin`, either `HIGH` or `LOW`.

```C++
SYNTAX
digitalRead(pin);
```

`digitalRead()` takes one argument, `pin`: the number of the digital pin you want to read.

`digitalRead()` returns `HIGH` or `LOW`.

```C++
EXAMPLE USAGE
int button = D0;                       // button is connected to D0
int LED = D1;                          // LED is connected to D1 
int val = 0;                           // variable to store the read value

void setup()
{
  pinMode(LED, OUTPUT);               // sets pin as output
  pinMode(button, INPUT_PULLDOWN);    // sets pin as input
}

void loop()
{
  val = digitalRead(button);          // read the input pin
  digitalWrite(LED, val);             // sets the LED to the button's value
}

```

### analogWrite()

Writes an analog value (PWM wave) to a pin. Can be used to light a LED at varying brightnesses or drive a motor at various speeds. After a call to analogWrite(), the pin will generate a steady square wave of the specified duty cycle until the next call to analogWrite() (or a call to digitalRead() or digitalWrite() on the same pin). The frequency of the PWM signal is approximately 500 Hz.

On the Spark Core, this function works on pins A0, A1, A4, A5, A6, A7, D0 and D1.

You do not need to call pinMode() to set the pin as an output before calling analogWrite().

The analogWrite function has nothing to do with the analog pins or the analogRead function. 

```C++
SYNTAX
analogWrite(pin, value);
```

`analogWrite()` takes two arguments, `pin`: the number of the pin whose value you wish to set and `value`: the duty cycle: between 0 (always off) and 255 (always on).

`analogWrite()` does not return anything.

```C++
EXAMPLE USAGE
int ledPin = D1;                // LED connected to digital pin D1
int analogPin = A0;             // potentiometer connected to analog pin A0
int val = 0;                    // variable to store the read value

void setup()
{
  pinMode(ledPin, OUTPUT);      // sets the pin as output
}

void loop()
{
  val = analogRead(analogPin);  // read the input pin
  analogWrite(ledPin, val/16);  // analogRead values go from 0 to 4095, analogWrite values from 0 to 255
  delay(10);
}
```

### analogRead()

Reads the value from the specified analog pin. The Spark Core has 8 channels (A0 to A7) with a 12-bit resolution. This means that it will map input voltages between 0 and 3.3 volts into integer values between 0 and 4095. This yields a resolution between readings of: 3.3 volts / 4096 units or, 0.0008 volts (0.8 mV) per unit.

```C++
SYNTAX
analogRead(pin);
```

`analogRead()` takes one argument `pin`: the number of the analog input pin to read from ('A0 to A7'.)

`analogRead()` returns an integer value ranging from 0 to 4095.

```C++
EXAMPLE USAGE
int ledPin = D1;                // LED connected to digital pin D1
int analogPin = A0;             // potentiometer connected to analog pin A0
int val = 0;                    // variable to store the read value

void setup()
{
  pinMode(ledPin, OUTPUT);      // sets the pin as output
}

void loop()
{
  val = analogRead(analogPin);  // read the input pin
  analogWrite(ledPin, val/16);  // analogRead values go from 0 to 4095, analogWrite values from 0 to 255
  delay(10);
}
```


Communication
===

Serial
-----

Used for communication between the Spark Core and a computer or other devices. The Core has two serial channels:  

`Serial:` This channel communicates through the USB port and when connected to a computer, will show up as a virtual COM port.  

`Serial1:` This channel is available via the Core's TX and RX pins. To use these pins to communicate with your personal computer, you will need an additional USB-to-serial adapter. To use them to communicate with an external TTL serial device, connect the TX pin to your device's RX pin, the RX to your device's TX pin, and the ground of your Core to your device's ground.

**NOTE:** Please take into account that the voltage levels on these pins runs at 0V to 3.3V and should not be connected directly to a computer's RS232 serial port which operates at +/- 12V and can damage the Core. 

### begin()

Sets the data rate in bits per second (baud) for serial data transmission. For communicating with the computer, use one of these rates: 300, 600, 1200, 2400, 4800, 9600, 14400, 19200, 28800, 38400, 57600, or 115200. You can, however, specify other rates - for example, to communicate over pins TX and RX with a component that requires a particular baud rate. 

```C++
SYNTAX
Serial.begin(speed);    // serial via USB port
Serial1.begin(speed);   // serial via TX and RX pins
```
`speed`: parameter that specifies the baud rate *(long)*

`begin()` does not return anything

```C++
EXAMPLE USAGE
void setup()
{
  Serial.begin(9600);   // open serial over USB
  Serial1.begin(9600);  // open serial over TX and RX pins

  Serial.println("Hello Computer");
  Serial1.println("Hello Serial 1");
}

void loop() {}
```

### end()

Disables serial communication, allowing the RX and TX pins to be used for general input and output. To re-enable serial communication, call `Serial1.begin()`.

```C++
SYNTAX
Serial1.end();
```

### available()

Get the number of bytes (characters) available for reading from the serial port. This is data that's already arrived and stored in the serial receive buffer (which holds 64 bytes).

```C++
EXAMPLE USAGE
void setup() 
{
  Serial.begin(9600);
  Serial1.begin(9600);

}

void loop() 
{
  // read from port 0, send to port 1:
  if (Serial.available()) 
  {
    int inByte = Serial.read();
    Serial1.print(inByte, BYTE);
  }
  // read from port 1, send to port 0:
  if (Serial1.available()) 
  {
    int inByte = Serial1.read();
    Serial.print(inByte, BYTE);
  }
}
```

### peek()

Returns the next byte (character) of incoming serial data without removing it from the internal serial buffer. That is, successive calls to peek() will return the same character, as will the next call to `read()`. 

```C++
SYNTAX
Serial.peek();
Serial1.peek(); 
```
`peek()` returns the first byte of incoming serial data available (or `-1` if no data is available) - *int*

### write()

Writes binary data to the serial port. This data is sent as a byte or series of bytes; to send the characters representing the digits of a number use the `print()` function instead.

```C++
SYNTAX
Serial.write(val);
Serial.write(str);
Serial.write(buf, len); 
```

*Parameters:*

- `val`: a value to send as a single byte
- `str`: a string to send as a series of bytes 
- `buf`: an array to send as a series of bytes
- `len`: the length of the buffer

`write()` will return the number of bytes written, though reading that number is optional.

```C++
EXAMPLE USAGE
void setup()
{
  Serial.begin(9600);
}

void loop()
{
  Serial.write(45); // send a byte with the value 45

   int bytesSent = Serial.write(“hello”); //send the string “hello” and return the length of the string.
}
```

### read()

Reads incoming serial data.

```C++
SYNTAX
Serial.read();
Serial1.read();
```
`read()` returns the first byte of incoming serial data available (or -1 if no data is available) - *int*

```C++
EXAMPLE USAGE
int incomingByte = 0;   // for incoming serial data

void setup() {
        Serial.begin(9600);     // opens serial port, sets data rate to 9600 bps
}

void loop() {

        // send data only when you receive data:
        if (Serial.available() > 0) {
                // read the incoming byte:
                incomingByte = Serial.read();

                // say what you got:
                Serial.print("I received: ");
                Serial.println(incomingByte, DEC);
        }
}
```
### print()

Prints data to the serial port as human-readable ASCII text. This command can take many forms. Numbers are printed using an ASCII character for each digit. Floats are similarly printed as ASCII digits, defaulting to two decimal places. Bytes are sent as a single character. Characters and strings are sent as is. For example:

- Serial.print(78) gives "78"
- Serial.print(1.23456) gives "1.23"
- Serial.print('N') gives "N"
- Serial.print("Hello world.") gives "Hello world." 

An optional second parameter specifies the base (format) to use; permitted values are BIN (binary, or base 2), OCT (octal, or base 8), DEC (decimal, or base 10), HEX (hexadecimal, or base 16). For floating point numbers, this parameter specifies the number of decimal places to use. For example:

- Serial.print(78, BIN) gives "1001110"
- Serial.print(78, OCT) gives "116"
- Serial.print(78, DEC) gives "78"
- Serial.print(78, HEX) gives "4E"
- Serial.println(1.23456, 0) gives "1"
- Serial.println(1.23456, 2) gives "1.23"
- Serial.println(1.23456, 4) gives "1.2346" 

### println()

Prints data to the serial port as human-readable ASCII text followed by a carriage return character (ASCII 13, or '\r') and a newline character (ASCII 10, or '\n'). This command takes the same forms as `Serial.print()`.

```C++
SYNTAX
Serial.println(val);
Serial.println(val, format);
```

*Parameters:*  

- `val`: the value to print - any data type
- `format`: specifies the number base (for integral data types) or number of decimal places (for floating point types) 

`println()` returns the number of bytes written, though reading that number is optional - `size_t (long)`

```C++
EXAMPLE
//reads an analog input on analog in A0, prints the value out.

int analogValue = 0;    // variable to hold the analog value

void setup() 
{
  // open the serial port at 9600 bps:
  Serial.begin(9600);
}

void loop() {
  // read the analog input on pin A0:
  analogValue = analogRead(A0);

  // print it out in many formats:
  Serial.println(analogValue);       // print as an ASCII-encoded decimal
  Serial.println(analogValue, DEC);  // print as an ASCII-encoded decimal
  Serial.println(analogValue, HEX);  // print as an ASCII-encoded hexadecimal
  Serial.println(analogValue, OCT);  // print as an ASCII-encoded octal
  Serial.println(analogValue, BIN);  // print as an ASCII-encoded binary

  // delay 10 milliseconds before the next reading:
  delay(10);
}
```

### flush()

Waits for the transmission of outgoing serial data to complete.

```C++
SYNTAX
Serial.flush();
Serial1.flush();
``` 

`flush()` neither takes a parameter nor returns anything

SPI
----
This library allows you to communicate with SPI devices, with the Spark Core as the master device.

![SPI](images/core-pin-spi.jpg)

### begin()

Initializes the SPI bus by setting SCK, MOSI, and SS to outputs, pulling SCK and MOSI low, and SS high. 

Note that once the pin is configured, you can't use it anymore as a general I/O, unless you call the SPI.end() method on the same pin. 

```C++
// SYNTAX
SPI.begin();
```

### end()

Disables the SPI bus (leaving pin modes unchanged). 

```C++
// SYNTAX
SPI.end();
```

### setBitOrder()

Sets the order of the bits shifted out of and into the SPI bus, either LSBFIRST (least-significant bit first) or MSBFIRST (most-significant bit first). 

```C++
// SYNTAX
SPI.setBitOrder(order);
```

Where, the parameter `order` can either be `LSBFIRST` or `MSBFIRST`. 

### setClockDivider()

Sets the SPI clock divider relative to the system clock. The available dividers  are 2, 4, 8, 16, 32, 64, 128 or 256. The default setting is SPI_CLOCK_DIV4, which sets the SPI clock to one-quarter the frequency of the system clock.

```C++
// SYNTAX
SPI.setClockDivider(divider) ;
```
Where the parameter, `divider` can be: 
```
SPI_CLOCK_DIV2
SPI_CLOCK_DIV4
SPI_CLOCK_DIV8
SPI_CLOCK_DIV16
SPI_CLOCK_DIV32
SPI_CLOCK_DIV64
SPI_CLOCK_DIV128 
SPI_CLOCK_DIV256 
```

### setDataMode()

Sets the SPI data mode: that is, clock polarity and phase. See the [Wikipedia article on SPI](http://en.wikipedia.org/wiki/Serial_Peripheral_Interface_Bus) for details. 

```C++
// SYNTAX
SPI.setClockDivider(mode) ;
```
Where the parameter, `mode` can be: 

```
SPI_MODE0
SPI_MODE1
SPI_MODE2
SPI_MODE3 
```

### transfer() 

Transfers one byte over the SPI bus, both sending and receiving. 

```C++
// SYNTAX
SPI.transfer(val);
```
Where the parameter `val`, can is the byte to send out over the SPI bus.

Wire
----

![I2C](images/core-pin-i2c.jpg)

This library allows you to communicate with I2C / TWI devices. On the Spark Core, D0 is the Serial Data Line (SDA) and D1 is the Serial Clock (SCL). Both of these pins runs at 3.3V logic but are tolerant to 5V.

### begin()

Initiate the Wire library and join the I2C bus as a master or slave. This should normally be called only once. 

```C++
// SYNTAX
Wire.begin();
Wire.begin(address);
```

Parameters: `address`: the 7-bit slave address (optional); if not specified, join the bus as a master. 

### requestFrom()

Used by the master to request bytes from a slave device. The bytes may then be retrieved with the `available()` and `read()` functions. 

If true, requestFrom() sends a stop message after the request, releasing the I2C bus.

If false, requestFrom() sends a restart message after the request. The bus will not be released, which prevents another master device from requesting between messages. This allows one master device to send multiple requests while in control.

The default value is true. 

```C++
// SYNTAX
Wire.requestFrom(address, quantity);
Wire.requestFrom(address, quantity, stop) ;
```

Parameters:

- `address`: the 7-bit address of the device to request bytes from
- `quantity`: the number of bytes to request
- `stop`: boolean. true will send a stop message after the request, releasing the bus. false will continually send a restart after the request, keeping the connection active. 

Returns: `byte` : the number of bytes returned from the slave device.

### beginTransmission()

Begin a transmission to the I2C slave device with the given address. Subsequently, queue bytes for transmission with the `write()` function and transmit them by calling `endTransmission()`. 

```C++
// SYNTAX
Wire.beginTransmission(address);
```

Parameters: `address`: the 7-bit address of the device to transmit to.

### endTransmission()

Ends a transmission to a slave device that was begun by `beginTransmission()` and transmits the bytes that were queued by `write()`. 

If true, `endTransmission()` sends a stop message after transmission, releasing the I2C bus.

If false, `endTransmission()` sends a restart message after transmission. The bus will not be released, which prevents another master device from transmitting between messages. This allows one master device to send multiple transmissions while in control.

The default value is true. 

```C++
Wire.endTransmission();
Wire.endTransmission(stop);
```

Parameters: `stop` : boolean.  
`true` will send a stop message, releasing the bus after transmission. `false` will send a restart, keeping the connection active. 

Returns: `byte`, which indicates the status of the transmission:

- 0: success
- 1: data too long to fit in transmit buffer
- 2: received NACK on transmit of address
- 3: received NACK on transmit of data
- 4: other error 

### write()

Writes data from a slave device in response to a request from a master, or queues bytes for transmission from a master to slave device (in-between calls to `beginTransmission()` and `endTransmission()`). 

```C++
// Syntax
Wire.write(value);
Wire.write(string);
Wire.write(data, length);
```
Parameters:

- `value`: a value to send as a single byte
- `string`: a string to send as a series of bytes
- `data`: an array of data to send as bytes
- `length`: the number of bytes to transmit 

Returns:  `byte`  

`write()` will return the number of bytes written, though reading that number is optional.

```C++
// EXAMPLE USAGE
byte val = 0;

void setup()
{
  Wire.begin(); // join i2c bus
}

void loop()
{
  Wire.beginTransmission(44); // transmit to device #44 (0x2c)
                              // device address is specified in datasheet
  Wire.write(val);             // sends value byte  
  Wire.endTransmission();     // stop transmitting

  val++;        // increment value
  if(val == 64) // if reached 64th position (max)
  {
    val = 0;    // start over from lowest value
  }
  delay(500);
}
```

### available()

Returns the number of bytes available for retrieval with `read()`. This should be called on a master device after a call to `requestFrom()` or on a slave inside the `onReceive()` handler. 

```C++
Wire.available();
```

Returns: The number of bytes available for reading. 

### read()

Reads a byte that was transmitted from a slave device to a master after a call to `requestFrom()` or was transmitted from a master to a slave. `read()` inherits from the `Stream` utility class. 

```C++
Wire.read() ;
```

Returns: The next byte received 

```C++
// EXAMPLE USAGE

void setup()
{
  Wire.begin();        // join i2c bus (address optional for master)
  Serial.begin(9600);  // start serial for output
}

void loop()
{
  Wire.requestFrom(2, 6);    // request 6 bytes from slave device #2

  while(Wire.available())    // slave may send less than requested
  {
    char c = Wire.read();    // receive a byte as character
    Serial.print(c);         // print the character
  }

  delay(500);
}
```

### onReceive()

Registers a function to be called when a slave device receives a transmission from a master. 

Parameters: `handler`: the function to be called when the slave receives data; this should take a single int parameter (the number of bytes read from the master) and return nothing, e.g.: `void myHandler(int numBytes) `

### onRequest() 

Register a function to be called when a master requests data from this slave device. 

Parameters: `handler`: the function to be called, takes no parameters and returns nothing, e.g.: `void myHandler() `

TCPServer
-----
### TCPServer

Create a server that listens for incoming connections on the specified port. 

```C++
// SYNTAX
TCPServer server = TCPServer(port);
```

Parameters: `port`: the port to listen on (`int`)

```C++
// EXAMPLE USAGE

// telnet defaults to port 23
TCPServer server = TCPServer(23);

void setup()
{
    // start listening for clients
    server.begin();
    
    Serial.begin(9600);

    delay(1000);

    Serial.println(Network.localIP());
    Serial.println(Network.subnetMask());
    Serial.println(Network.gatewayIP());
    Serial.println(Network.SSID());
}

void loop()
{
  // if an incoming client connects, there will be bytes available to read:
  TCPClient client = server.available();
  if (client == true) 
  {
      // read bytes from the incoming client and write them back
      // to any clients connected to the server:
      server.write(client.read());
  }
}
```

### begin()

Tells the server to begin listening for incoming connections. 

```C++
// SYNTAX
server.begin();
```

### available()

Gets a client that is connected to the server and has data available for reading. The connection persists when the returned client object goes out of scope; you can close it by calling `client.stop()`.

`available()` inherits from the `Stream` utility class. 

### write()

Write data to all the clients connected to a server. This data is sent as a byte or series of bytes. 

```C++
// Syntax
server.write(val);
server.write(buf, len); 
```

Parameters:

- `val`: a value to send as a single byte (byte or char)
- `buf`: an array to send as a series of bytes (byte or char)
- `len`: the length of the buffer

Returns: `byte`: `write()` returns the number of bytes written. It is not necessary to read this. 

### print()

Print data to all the clients connected to a server. Prints numbers as a sequence of digits, each an ASCII character (e.g. the number 123 is sent as the three characters '1', '2', '3'). 

```C++
// Syntax
server.print(data);
server.print(data, BASE) ;
```

Parameters: 

- `data`: the data to print (char, byte, int, long, or string)
- `BASE`(optional): the base in which to print numbers: BIN for binary (base 2), DEC for decimal (base 10), OCT for octal (base 8), HEX for hexadecimal (base 16). 

Returns:  `byte`:  `print()` will return the number of bytes written, though reading that number is optional

### println()

Print data, followed by a newline, to all the clients connected to a server. Prints numbers as a sequence of digits, each an ASCII character (e.g. the number 123 is sent as the three characters '1', '2', '3'). 

```C++
// Syntax

server.println();
server.println(data);
server.println(data, BASE) ;
```

Parameters: 

- `data` (optional): the data to print (char, byte, int, long, or string)
- `BASE` (optional): the base in which to print numbers: BIN for binary (base 2), DEC for decimal (base 10), OCT for octal (base 8), HEX for hexadecimal (base 16). 

TCPClient
-----

### TCPClient

Creates a client which can connect to a specified internet IP address and port (defined in the `client.connect()` function). 

```C++
// SYNTAX
TCPClient client;
```

```C++
// EXAMPLE USAGE

TCPClient client;
byte server[] = { 74, 125, 224, 72 }; // Google
void setup()
{
  Serial.begin(9600);
  delay(1000);
  Serial.println("connecting...");

  if (client.connect(server, 80)) 
  {
    Serial.println("connected");
    client.println("GET /search?q=unicorn HTTP/1.0");
    client.println();
  } 
  else 
  {
    Serial.println("connection failed");
  }
}

void loop()
{
  if (client.available()) 
  {
    char c = client.read();
    Serial.print(c);
  }

  if (!client.connected()) 
  {
    Serial.println();
    Serial.println("disconnecting.");
    client.stop();
    for(;;)
      ;
  }
}
```

### connected()

Whether or not the client is connected. Note that a client is considered connected if the connection has been closed but there is still unread data. 

```C++
// SYNTAX
client.connected();
```

Returns true if the client is connected, false if not. 

### connect()

Connects to a specified IP address and port. The return value indicates success or failure. Also supports DNS lookups when using a domain name. 

```C++
// SYNTAX

client.connect();
client.connect(ip, port);
client.connect(URL, port);
```

Parameters:

- `ip`: the IP address that the client will connect to (array of 4 bytes)
- `URL`: the domain name the client will connect to (string, ex.:"spark.io")
- `port`: the port that the client will connect to (`int`) 

Returns true if the connection succeeds, false if not. 

### write()

Write data to the server the client is connected to. This data is sent as a byte or series of bytes. 

```C++
// SYNTAX
client.write(val);
client.write(buf, len);
```

Parameters:

- `val`: a value to send as a single byte (byte or char)
- `buf`: an array to send as a series of bytes (byte or char)
- `len`: the length of the buffer 

Returns: `byte`: `write()` returns the number of bytes written. It is not necessary to read this value.

### print()

Print data to the server that a client is connected to. Prints numbers as a sequence of digits, each an ASCII character (e.g. the number 123 is sent as the three characters '1', '2', '3'). 

```C++
// Syntax
client.print(data);
client.print(data, BASE) ;
```

Parameters: 

- `data`: the data to print (char, byte, int, long, or string)
- `BASE`(optional): the base in which to print numbers: BIN for binary (base 2), DEC for decimal (base 10), OCT for octal (base 8), HEX for hexadecimal (base 16). 

Returns:  `byte`:  `print()` will return the number of bytes written, though reading that number is optional

### println()

Print data, followed by a carriage return and newline, to the server a client is connected to. Prints numbers as a sequence of digits, each an ASCII character (e.g. the number 123 is sent as the three characters '1', '2', '3'). 

```C++
// Syntax

client.println();
client.println(data);
client.println(data, BASE) ;
```

Parameters: 

- `data` (optional): the data to print (char, byte, int, long, or string)
- `BASE` (optional): the base in which to print numbers: BIN for binary (base 2), DEC for decimal (base 10), OCT for octal (base 8), HEX for hexadecimal (base 16). 

### available()

Returns the number of bytes available for reading (that is, the amount of data that has been written to the client by the server it is connected to). 

```C++
// SYNTAX
client.available();
```

Returns the number of bytes available. 

### read()
Read the next byte received from the server the client is connected to (after the last call to `read()`). 

```C++
// SYNTAX
client.read();
```

Returns the next byte (or character), or -1 if none is available. 

### flush()

Discard any bytes that have been written to the client but not yet read. 

```C++
// SYNTAX
client.flush();
```

### stop()

Disconnect from the server. 

```C++
// SYNTAX
client.stop();
```


UDP
-----

### UDP
### begin()
### available()
### beginPacket()
### endPacket()
### write()
### parsePacket()
### peek()
### read()
### flush()
### stop()
### remoteIP()
### remotePort()

Libraries
===

Servo
---

This library allows a Spark Core to control RC (hobby) servo motors. Servos have integrated gears and a shaft that can be precisely controlled. Standard servos allow the shaft to be positioned at various angles, usually between 0 and 180 degrees. Continuous rotation servos allow the rotation of the shaft to be set to various speeds.

```cpp
// EXAMPLE CODE

Servo myservo;  // create servo object to control a servo 
                // a maximum of eight servo objects can be created 
 
int pos = 0;    // variable to store the servo position 
 
void setup() 
{ 
  myservo.attach(A0);  // attaches the servo on the A0 pin to the servo object 
} 
 
 
void loop() 
{ 
  for(pos = 0; pos < 180; pos += 1)  // goes from 0 degrees to 180 degrees 
  {                                  // in steps of 1 degree 
    myservo.write(pos);              // tell servo to go to position in variable 'pos' 
    delay(15);                       // waits 15ms for the servo to reach the position 
  } 
  for(pos = 180; pos>=1; pos-=1)     // goes from 180 degrees to 0 degrees 
  {                                
    myservo.write(pos);              // tell servo to go to position in variable 'pos' 
    delay(15);                       // waits 15ms for the servo to reach the position 
  } 
}
```

NOTE: Unlike Arduino, you do not need to include `Servo.h`; it is included automatically.


### attach()

Set up a servo on a particular pin. Note that, on the Spark Core, Servo can only be attached to pins with a timer (A0, A1, A4, A5, A6, A7, D0, and D1).

```cpp
// SYNTAX
servo.attach(pin)
```

### write()

Writes a value to the servo, controlling the shaft accordingly. On a standard servo, this will set the angle of the shaft (in degrees), moving the shaft to that orientation. On a continuous rotation servo, this will set the speed of the servo (with 0 being full-speed in one direction, 180 being full speed in the other, and a value near 90 being no movement).

```cpp
// SYNTAX
servo.write(angle)
```

### writeMicroseconds()

Writes a value in microseconds (uS) to the servo, controlling the shaft accordingly. On a standard servo, this will set the angle of the shaft. On standard servos a parameter value of 1000 is fully counter-clockwise, 2000 is fully clockwise, and 1500 is in the middle.

```cpp
// SYNTAX
servo.writeMicroseconds(uS)
```

Note that some manufactures do not follow this standard very closely so that servos often respond to values between 700 and 2300. Feel free to increase these endpoints until the servo no longer continues to increase its range. Note however that attempting to drive a servo past its endpoints (often indicated by a growling sound) is a high-current state, and should be avoided.

Continuous-rotation servos will respond to the writeMicrosecond function in an analogous manner to the write function.


### read()

Read the current angle of the servo (the value passed to the last call to write()). Returns an integer from 0 to 180 degrees.

```cpp
// SYNTAX
servo.read()
```

### attached()

Check whether the Servo variable is attached to a pin. Returns a boolean.

```cpp
// SYNTAX
servo.attached()
```

### detach()

Detach the Servo variable from its pin.

```cpp
// SYNTAX
servo.detach()
```

Other functions
====

Time
---

### millis()

Returns the number of milliseconds since the Spark Core began running the current program. This number will overflow (go back to zero), after approximately 49 days.

`unsigned long time = millis();`

```C++
EXAMPLE USAGE
unsigned long time;

void setup()
{
  Serial.begin(9600);
}
void loop()
{
  Serial.print("Time: ");
  time = millis();
  //prints time since program started
  Serial.println(time);
  // wait a second so as not to send massive amounts of data
  delay(1000);
}
```
**Note:** 
The parameter for millis is an unsigned long, errors may be generated if a programmer tries to do math with other datatypes such as ints.

### micros()

**Not implemented yet**

Returns the number of microseconds since the Arduino board began running the current program. This number will overflow (go back to zero), after approximately 70 minutes.

`unsigned long time = micros();`

```C++
EXAMPLE USAGE
unsigned long time;

void setup()
{
  Serial.begin(9600);
}
void loop()
{
  Serial.print("Time: ");
  time = micros();
  //prints time since program started
  Serial.println(time);
  // wait a second so as not to send massive amounts of data
  delay(1000);
}
```

### delay()

Pauses the program for the amount of time (in miliseconds) specified as parameter. (There are 1000 milliseconds in a second.)

```C++
SYNTAX
delay(ms);
```

`ms` is the number of milliseconds to pause *(unsigned long)*

```C++
EXAMPLE USAGE
int ledPin = D1;                 // LED connected to digital pin D1

void setup()
{
  pinMode(ledPin, OUTPUT);      // sets the digital pin as output
}

void loop()
{
  digitalWrite(ledPin, HIGH);   // sets the LED on
  delay(1000);                  // waits for a second
  digitalWrite(ledPin, LOW);    // sets the LED off
  delay(1000);                  // waits for a second
}
```
**NOTE:** 
the parameter for millis is an unsigned long, errors may be generated if a programmer tries to do math with other datatypes such as ints.

### delayMicroseconds()

**Not implemented yet**

Pauses the program for the amount of time (in microseconds) specified as parameter. There are a thousand microseconds in a millisecond, and a million microseconds in a second.

```C++
SYNTAX
delayMicroseconds(us);
```
`us` is the number of microseconds to pause *(unsigned int)*

```C++
EXAMPLE USAGE
int outPin = D1;                 // digital pin D1

void setup()
{
  pinMode(outPin, OUTPUT);      // sets the digital pin as output
}

void loop()
{
  digitalWrite(outPin, HIGH);   // sets the pin on
  delayMicroseconds(50);        // pauses for 50 microseconds      
  digitalWrite(outPin, LOW);    // sets the pin off
  delayMicroseconds(50);        // pauses for 50 microseconds      
}
```

Interrupts
---

### attachInterrupt()

Specifies a function to call when an external interrupt occurs. Replaces any previous function that was attached to the interrupt.

The Spark Core currently supports external interrupts on the following pins:

D0, D1, D2, D3, D4  
A0, A1, A3, A4, A5, A6, A7  

`attachInterrupt(pin, function, mode);`

*Parameters:*  

- `pin`: the pin number
- `function`: the function to call when the interrupt occurs; this function must take no parameters and return nothing. This function is sometimes referred to as an *interrupt service routine* (ISR).
- `mode`: defines when the interrupt should be triggered. Four constants are predefined as valid values:
    - CHANGE to trigger the interrupt whenever the pin changes value,
    - RISING to trigger when the pin goes from low to high,
    - FALLING for when the pin goes from high to low. 

The function does not return anything.

**NOTE:**  
Inside the attached function, `delay()` won't work and the value returned by `millis()` will not increment. Serial data received while in the function may be lost. You should declare as `volatile` any variables that you modify within the attached function.

*Using Interrupts:*  
Interrupts are useful for making things happen automatically in microcontroller programs, and can help solve timing problems. Good tasks for using an interrupt may include reading a rotary encoder, or monitoring user input.

If you wanted to insure that a program always caught the pulses from a rotary encoder, so that it never misses a pulse, it would make it very tricky to write a program to do anything else, because the program would need to constantly poll the sensor lines for the encoder, in order to catch pulses when they occurred. Other sensors have a similar interface dynamic too, such as trying to read a sound sensor that is trying to catch a click, or an infrared slot sensor (photo-interrupter) trying to catch a coin drop. In all of these situations, using an interrupt can free the microcontroller to get some other work done while not missing the input. 

```C++
EXAMPLE USAGE
void blink(void);
int ledPin = D1;
volatile int state = LOW;

void setup()
{
  pinMode(ledPin, OUTPUT);
  attachInterrupt(D0, blink, CHANGE);
}

void loop()
{
  digitalWrite(ledPin, state);
}

void blink()
{
  state = !state;
}
```


### detatchInterrupt()

Turns off the given interrupt.

`detachInterrupt(pin);`

`pin` is the pin number of the interrupt to disable.

### interrupts()

Re-enables interrupts (after they've been disabled by `noInterrupts()`). Interrupts allow certain important tasks to happen in the background and are enabled by default. Some functions will not work while interrupts are disabled, and incoming communication may be ignored. Interrupts can slightly disrupt the timing of code, however, and may be disabled for particularly critical sections of code.

`interrupts()` neither accepts a parameter nor returns anything.

```C++
EXAMPLE USAGE
void setup() {}

void loop()
{
  noInterrupts();
  // critical, time-sensitive code here
  interrupts();
  // other code here
}
```

### noInterrupts()

Disables interrupts (you can re-enable them with `interrupts()`). Interrupts allow certain important tasks to happen in the background and are enabled by default. Some functions will not work while interrupts are disabled, and incoming communication may be ignored. Interrupts can slightly disrupt the timing of code, however, and may be disabled for particularly critical sections of code.

`noInterrupts()` neither accepts a parameter nor returns anything.

```C++
EXAMPLE USAGE
void setup() {}

void loop()
{
  noInterrupts();
  // critical, time-sensitive code here
  interrupts();
  // other code here
}
```

Math
---

### min()

Calculates the minimum of two numbers.

`min(x, y)`

`x` is the first number, any data type  
`y` is the second number, any data type  

The functions returns the smaller of the two numbers.

```C++
EXAMPLE USAGE
sensVal = min(sensVal, 100); // assigns sensVal to the smaller of sensVal or 100
                             // ensuring that it never gets above 100.
```

**NOTE:**
Perhaps counter-intuitively, max() is often used to constrain the lower end of a variable's range, while min() is used to constrain the upper end of the range.

**WARNING:** 
Because of the way the min() function is implemented, avoid using other functions inside the brackets, it may lead to incorrect results 

```C++
min(a++, 100);   // avoid this - yields incorrect results

a++;
min(a, 100);    // use this instead - keep other math outside the function
```

### max()

Calculates the maximum of two numbers.

`man(x, y)`

`x` is the first number, any data type  
`y` is the second number, any data type  

The functions returns the larger of the two numbers.

```C++
EXAMPLE USAGE
sensVal = max(senVal, 20); // assigns sensVal to the larger of sensVal or 20
                           // (effectively ensuring that it is at least 20)
```

**NOTE:**
Perhaps counter-intuitively, max() is often used to constrain the lower end of a variable's range, while min() is used to constrain the upper end of the range.

**WARNING:** 
Because of the way the max() function is implemented, avoid using other functions inside the brackets, it may lead to incorrect results 

```C++
max(a--, 0);   // avoid this - yields incorrect results

a--;           // use this instead -
max(a, 0);     // keep other math outside the function
```

### abs()

Computes the absolute value of a number.

`abs(x);`

where `x` is the number  

The function returns `x` if `x` is greater than or equal to `0`  
and returns `-x` if `x` is less than `0`.

**WARNING:**
Because of the way the abs() function is implemented, avoid using other functions inside the brackets, it may lead to incorrect results.

```C++
abs(a++);   // avoid this - yields incorrect results

a++;          // use this instead -
abs(a);       // keep other math outside the function
```

### constrain()

Constrains a number to be within a range.

`constrain(x, a, b);`

`x` is the number to constrain, all data types  
`a` is the lower end of the range, all data types
`b` is the upper end of the range, all data types

The function will return:  
`x`: if x is between `a` and `b`  
`a`: if `x` is less than `a`  
`b`: if `x` is greater than `b`

```C++
EXAMPLE USAGE
sensVal = constrain(sensVal, 10, 150);
// limits range of sensor values to between 10 and 150
```

### map()

Re-maps a number from one range to another. That is, a value of fromLow would get mapped to `toLow`, a `value` of `fromHigh` to `toHigh`, values in-between to values in-between, etc.

`map(value, fromLow, fromHigh, toLow, toHigh);`

Does not constrain values to within the range, because out-of-range values are sometimes intended and useful. The `constrain()` function may be used either before or after this function, if limits to the ranges are desired.

Note that the "lower bounds" of either range may be larger or smaller than the "upper bounds" so the `map()` function may be used to reverse a range of numbers, for example

`y = map(x, 1, 50, 50, 1);` 

The function also handles negative numbers well, so that this example

`y = map(x, 1, 50, 50, -100);`

is also valid and works well. 

The `map()` function uses integer math so will not generate fractions, when the math might indicate that it should do so. Fractional remainders are truncated, and are not rounded or averaged.

*Parameters:*  

- `value`: the number to map
- `fromLow`: the lower bound of the value's current range
- `fromHigh`: the upper bound of the value's current range
- `toLow`: the lower bound of the value's target range
- `toHigh`: the upper bound of the value's target range 

The function returns the mapped value

```C++
EXAMPLE USAGE
/* Map an analog value to 8 bits (0 to 255) */
void setup() {}

void loop()
{
  int val = analogRead(0);
  val = map(val, 0, 4095, 0, 255);
  analogWrite(9, val);
}
```

*Appendix:*  
For the mathematically inclined, here's the whole function

```C++
long map(long x, long in_min, long in_max, long out_min, long out_max)
{
  return (x - in_min) * (out_max - out_min) / (in_max - in_min) + out_min;
}
```

### pow()

Calculates the value of a number raised to a power. `pow()` can be used to raise a number to a fractional power. This is useful for generating exponential mapping of values or curves.

`pow(base, exponent);`

`base` is the number *(float)*  
`exponent` is the power to which the base is raised *(float)*

The function returns the result of the exponentiation *(double)*

EXAMPLE **TBD**

### sqrt()

Calculates the square root of a number.

`sqrt(x)`

`x` is the number, any data type

The function returns the number's square root *(double)*


Language Syntax
========
The following documentation is based on the Arduino reference which can be found [here.](http://arduino.cc/en/Reference/HomePage)

Structure
---
### setup()
The setup() function is called when an application starts. Use it to initialize variables, pin modes, start using libraries, etc. The setup function will only run once, after each powerup or reset of the Spark Core.

```C++
EXAMPLE USAGE
int button = D0;
int LED = D1;
//setup initializes D0 as input and D1 as output
void setup()
{
  pinMode(button, INPUT_PULLDOWN);
  pinMode(LED, OUTPUT);
}

void loop()
{
  // ...
}
```

### loop()
After creating a setup() function, which initializes and sets the initial values, the loop() function does precisely what its name suggests, and loops consecutively, allowing your program to change and respond. Use it to actively control the Spark Core.

```C++
EXAMPLE USAGE
int button = D0;
int LED = D1;
//setup initializes D0 as input and D1 as output
void setup()
{
  pinMode(button, INPUT_PULLDOWN);
  pinMode(LED, OUTPUT);
}

//loops to check if button was pressed,
//if it was, then it turns ON the LED,
//else the LED remains OFF
void loop()
{
  if (digitalRead(button) == HIGH)
    digitalWrite(LED,HIGH);
  else
    digitalWrite(LED,LOW);
}
```

Control structures
---

### if

`if`, which is used in conjunction with a comparison operator, tests whether a certain condition has been reached, such as an input being above a certain number. 

```C++
SYNTAX
if (someVariable > 50)
{
  // do something here
}
```
The program tests to see if someVariable is greater than 50. If it is, the program takes a particular action. Put another way, if the statement in parentheses is true, the statements inside the brackets are run. If not, the program skips over the code. 

The brackets may be omitted after an *if* statement. If this is done, the next line (defined by the semicolon) becomes the only conditional statement. 

```C++
if (x > 120) digitalWrite(LEDpin, HIGH); 

if (x > 120) 
digitalWrite(LEDpin, HIGH); 

if (x > 120){ digitalWrite(LEDpin, HIGH); } 

if (x > 120)
{ 
  digitalWrite(LEDpin1, HIGH);
  digitalWrite(LEDpin2, HIGH); 
}                                 // all are correct
```
The statements being evaluated inside the parentheses require the use of one or more operators:

### Comparison Operators

```C++
x == y (x is equal to y)
x != y (x is not equal to y)
x <  y (x is less than y)  
x >  y (x is greater than y) 
x <= y (x is less than or equal to y) 
x >= y (x is greater than or equal to y)
```

**WARNING:**
Beware of accidentally using the single equal sign (e.g. `if (x = 10)` ). The single equal sign is the assignment operator, and sets x to 10 (puts the value 10 into the variable x). Instead use the double equal sign (e.g. `if (x == 10)` ), which is the comparison operator, and tests whether x is equal to 10 or not. The latter statement is only true if x equals 10, but the former statement will always be true.

This is because C evaluates the statement `if (x=10)` as follows: 10 is assigned to x (remember that the single equal sign is the assignment operator), so x now contains 10. Then the 'if' conditional evaluates 10, which always evaluates to TRUE, since any non-zero number evaluates to TRUE. Consequently, `if (x = 10)` will always evaluate to TRUE, which is not the desired result when using an 'if' statement. Additionally, the variable x will be set to 10, which is also not a desired action.

`if` can also be part of a branching control structure using the `if...else`] construction. 

### if...else

*if/else* allows greater control over the flow of code than the basic *if* statement, by allowing multiple tests to be grouped together. For example, an analog input could be tested and one action taken if the input was less than 500, and another action taken if the input was 500 or greater. The code would look like this: 

```C++
SYNTAX
if (pinFiveInput < 500)
{
  // action A
}
else
{
  // action B
}
```
`else` can proceed another `if` test, so that multiple, mutually exclusive tests can be run at the same time. 

Each test will proceed to the next one until a true test is encountered. When a true test is found, its associated block of code is run, and the program then skips to the line following the entire if/else construction. If no test proves to be true, the default else block is executed, if one is present, and sets the default behavior. 

Note that an *else if* block may be used with or without a terminating *else* block and vice versa. An unlimited number of such else if branches is allowed. 

```C++
if (pinFiveInput < 500)
{
  // do Thing A
}
else if (pinFiveInput >= 1000)
{
  // do Thing B
}
else
{
  // do Thing C
}
```

Another way to express branching, mutually exclusive tests, is with the [`switch case`](http://spark.github.io/docs/#control-structures-switch-case) statement.

### for

The `for` statement is used to repeat a block of statements enclosed in curly braces. An increment counter is usually used to increment and terminate the loop. The `for` statement is useful for any repetitive operation, and is often used in combination with arrays to operate on collections of data/pins. 

There are three parts to the for loop header: 

```C++
SYNTAX
for (initialization; condition; increment) 
{
  //statement(s);
} 
```
The *initialization* happens first and exactly once. Each time through the loop, the *condition* is tested; if it's true, the statement block, and the *increment* is executed, then the condition is tested again. When the *condition* becomes false, the loop ends.

```C++
EXAMPLE USAGE
// slowy make the LED glow brighter
int ledPin = D1; // LED in series with 470 ohm resistor on pin D1

void setup()
{
  // set ledPin as an output
  pinMode(ledPin,OUTPUT);
}

void loop()
{
   for (int i=0; i <= 255; i++){
      analogWrite(ledPin, i);
      delay(10);
   } 
}
```
The C `for` loop is much more flexible than for loops found in some other computer languages, including BASIC. Any or all of the three header elements may be omitted, although the semicolons are required. Also the statements for initialization, condition, and increment can be any valid C statements with unrelated variables, and use any C datatypes including floats. These types of unusual for statements may provide solutions to some rare programming problems.

For example, using a multiplication in the increment line will generate a logarithmic progression:

```C++
for(int x = 2; x < 100; x = x * 1.5)
{
  Serial.print(x);
}
//Generates: 2,3,4,6,9,13,19,28,42,63,94 
```
Another example, fade an LED up and down with one for loop: 

```C++
// slowy make the LED glow brighter
int ledPin = D1; // LED in series with 470 ohm resistor on pin D1

void setup()
{
  // set ledPin as an output
  pinMode(ledPin,OUTPUT);
}

void loop()
{
   int x = 1;
   for (int i = 0; i > -1; i = i + x)
   {
      analogWrite(ledPin, i);
      if (i == 255) x = -1;     // switch direction at peak
      delay(10);
   } 
}
```

### switch case

Like `if` statements, `switch`...`case` controls the flow of programs by allowing programmers to specify different code that should be executed in various conditions. In particular, a switch statement compares the value of a variable to the values specified in case statements. When a case statement is found whose value matches that of the variable, the code in that case statement is run.

The `break` keyword exits the switch statement, and is typically used at the end of each case. Without a break statement, the switch statement will continue executing the following expressions ("falling-through") until a break, or the end of the switch statement is reached. 

```C++
SYNTAX
switch (var) 
{
  case label:
    // statements
    break;
  case label:
    // statements
    break;
  default: 
    // statements
}
```
`var` is the variable whose value to compare to the various cases 
`label` is a value to compare the variable to

```C++
EXAMPLE USAGE
switch (var) 
{
    case 1:
      //do something when var equals 1
      break;
    case 2:
      //do something when var equals 2
      break;
    default: 
      // if nothing else matches, do the default
      // default is optional
}
```

### while

`while` loops will loop continuously, and infinitely, until the expression inside the parenthesis, () becomes false. Something must change the tested variable, or the `while` loop will never exit. This could be in your code, such as an incremented variable, or an external condition, such as testing a sensor.

```C++
SYNTAX
while(expression)
{
  // statement(s)
}
```
`expression` is a (boolean) C statement that evaluates to true or false.

```C++
EXAMPLE USAGE
var = 0;
while(var < 200)
{
  // do something repetitive 200 times
  var++;
}
```

### do... while

The `do` loop works in the same manner as the `while` loop, with the exception that the condition is tested at the end of the loop, so the do loop will *always* run at least once. 

```C++
SYNTAX
do
{
    // statement block
} while (test condition);
```

```C++
EXAMPLE USAGE
do
{
  delay(50);          // wait for sensors to stabilize
  x = readSensors();  // check the sensors

} while (x < 100);
```

### break

`break` is used to exit from a `do`, `for`, or `while` loop, bypassing the normal loop condition. It is also used to exit from a `switch` statement.

```C++
EXAMPLE USAGE
for (int x = 0; x < 255; x ++)
{
    digitalWrite(ledPin, x);
    sens = analogRead(sensorPin);  
    if (sens > threshold)         // bail out on sensor detect
    {      
       x = 0;
       break;
    }  
    delay(50);
}
```

### continue

The continue statement skips the rest of the current iteration of a loop (`do`, `for`, or `while`). It continues by checking the conditional expression of the loop, and proceeding with any subsequent iterations.

```C++
EXAMPLE USAGE
for (x = 0; x < 255; x ++)
{
    if (x > 40 && x < 120) continue;    // create jump in values
    
    digitalWrite(PWMpin, x);
    delay(50);
}
```

### return

Terminate a function and return a value from a function to the calling function, if desired.

```C++
EXAMPLE
// A function to compare a sensor input to a threshold 
 int checkSensor()
 {       
    if (analogRead(0) > 400) return 1;
    else return 0;
}
```
The return keyword is handy to test a section of code without having to "comment out" large sections of possibly buggy code. 

```C++
void loop()
{
  // brilliant code idea to test here

  return;

  // the rest of a dysfunctional sketch here
  // this code will never be executed
}
```

### goto

Transfers program flow to a labeled point in the program

```C++
USAGE
label:

goto label; // sends program flow to the label 
```

**TIP:**
The use of `goto` is discouraged in C programming, and some authors of C programming books claim that the `goto` statement is never necessary, but used judiciously, it can simplify certain programs. The reason that many programmers frown upon the use of `goto` is that with the unrestrained use of `goto` statements, it is easy to create a program with undefined program flow, which can never be debugged.

With that said, there are instances where a `goto` statement can come in handy, and simplify coding. One of these situations is to break out of deeply nested `for` loops, or `if` logic blocks, on a certain condition. 

```C++
EXAMPLE USAGE
for(byte r = 0; r < 255; r++){
    for(byte g = 255; g > -1; g--){
        for(byte b = 0; b < 255; b++){
            if (analogRead(0) > 250){ goto bailout;}
            // more statements ... 
        }
    }
}
bailout:
```

Further syntax
---

### ; (semicolon)

Used to end a statement.

`int a = 13;`

**Tip:**
Forgetting to end a line in a semicolon will result in a compiler error. The error text may be obvious, and refer to a missing semicolon, or it may not. If an impenetrable or seemingly illogical compiler error comes up, one of the first things to check is a missing semicolon, in the immediate vicinity, preceding the line at which the compiler complained. 

### {} (curly braces)

Curly braces (also referred to as just "braces" or as "curly brackets") are a major part of the C programming language. They are used in several different constructs, outlined below, and this can sometimes be confusing for beginners. 

```C++
//The main uses of curly braces

//Functions
  void myfunction(datatype argument){
    statements(s)
  }

//Loops
  while (boolean expression)
  {
     statement(s)
  }

  do
  {
     statement(s)
  } while (boolean expression);

  for (initialisation; termination condition; incrementing expr)
  {
     statement(s)
  } 

//Conditional statements
  if (boolean expression)
  {
     statement(s)
  }

  else if (boolean expression)
  {
     statement(s)
  } 
  else
  {
     statement(s)
  }

```

An opening curly brace "{" must always be followed by a closing curly brace "}". This is a condition that is often referred to as the braces being balanced. 

Beginning programmers, and programmers coming to C from the BASIC language often find using braces confusing or daunting. After all, the same curly braces replace the RETURN statement in a subroutine (function), the ENDIF statement in a conditional and the NEXT statement in a FOR loop. 

Because the use of the curly brace is so varied, it is good programming practice to type the closing brace immediately after typing the opening brace when inserting a construct which requires curly braces. Then insert some carriage returns between your braces and begin inserting statements. Your braces, and your attitude, will never become unbalanced. 

Unbalanced braces can often lead to cryptic, impenetrable compiler errors that can sometimes be hard to track down in a large program. Because of their varied usages, braces are also incredibly important to the syntax of a program and moving a brace one or two lines will often dramatically affect the meaning of a program.


### // (single line comment)
### /\* \*/ (multi-line comment)

Comments are lines in the program that are used to inform yourself or others about the way the program works. They are ignored by the compiler, and not exported to the processor, so they don't take up any space on the Spark Core.

Comments only purpose are to help you understand (or remember) how your program works or to inform others how your program works. There are two different ways of marking a line as a comment:

```C++
EXAMPLE USAGE
 x = 5;  // This is a single line comment. Anything after the slashes is a comment 
         // to the end of the line

/* this is multiline comment - use it to comment out whole blocks of code

if (gwb == 0){   // single line comment is OK inside a multiline comment
x = 3;           /* but not another multiline comment - this is invalid */
}
// don't forget the "closing" comment - they have to be balanced!
*/
```

**TIP:**
When experimenting with code, "commenting out" parts of your program is a convenient way to remove lines that may be buggy. This leaves the lines in the code, but turns them into comments, so the compiler just ignores them. This can be especially useful when trying to locate a problem, or when a program refuses to compile and the compiler error is cryptic or unhelpful. 


### #define

`#define` is a useful C component that allows the programmer to give a name to a constant value before the program is compiled. Defined constants don't take up any program memory space on the chip. The compiler will replace references to these constants with the defined value at compile time.

`#define constantName value`  
Note that the # is necessary.

This can have some unwanted side effects though, if for example, a constant name that had been `#defined` is included in some other constant or variable name. In that case the text would be replaced by the #defined number (or text).

```C++
EXAMPLE USAGE
#define ledPin 3
// The compiler will replace any mention of ledPin with the value 3 at compile time.
```

In general, the [const]() keyword is preferred for defining constants and should be used instead of #define. 

**TIP:**
There is no semicolon after the #define statement. If you include one, the compiler will throw cryptic errors further down the page. 

`#define ledPin 3;    // this is an error`

Similarly, including an equal sign after the #define statement will also generate a cryptic compiler error further down the page. 

`#define ledPin  = 3  // this is also an error`

### #include

`#include` is used to include outside libraries in your application code. This gives the programmer access to a large group of standard C libraries (groups of pre-made functions), and also libraries written especially for Spark Core.

Note that #include, similar to #define, has no semicolon terminator, and the compiler will yield cryptic error messages if you add one.

Arithmetic operators
---

### = (assignment operator)

Stores the value to the right of the equal sign in the variable to the left of the equal sign.

The single equal sign in the C programming language is called the assignment operator. It has a different meaning than in algebra class where it indicated an equation or equality. The assignment operator tells the microcontroller to evaluate whatever value or expression is on the right side of the equal sign, and store it in the variable to the left of the equal sign. 

```C++
EXAMPLE USAGE
int sensVal;                // declare an integer variable named sensVal
senVal = analogRead(A0);    // store the (digitized) input voltage at analog pin A0 in SensVal
```
**TIP:**
The variable on the left side of the assignment operator ( = sign ) needs to be able to hold the value stored in it. If it is not large enough to hold a value, the value stored in the variable will be incorrect.

Don't confuse the assignment operator `=` (single equal sign) with the comparison operator `==` (double equal signs), which evaluates whether two expressions are equal. 

### + - / (additon subtraction division)

These operators return the sum, difference, product, or quotient (respectively) of the two operands. The operation is conducted using the data type of the operands, so, for example,`9 / 4` gives 2 since 9 and 4 are ints. This also means that the operation can overflow if the result is larger than that which can be stored in the data type (e.g. adding 1 to an int with the value 32,767 gives -32,768). If the operands are of different types, the "larger" type is used for the calculation.

If one of the numbers (operands) are of the type float or of type double, floating point math will be used for the calculation. 

```C++
EXAMPLE USAGES
y = y + 3;
x = x - 7;
i = j * 6;
r = r / 5;
```

```C++
SYNTAX
result = value1 + value2;
result = value1 - value2;
result = value1 * value2;
result = value1 / value2;
```
`value1` and `value2` can be any variable or constant.

**TIPS:** 

  - Know that integer constants default to int, so some constant calculations may overflow (e.g. 60 * 1000 will yield a negative result).
  - Choose variable sizes that are large enough to hold the largest results from your calculations  
  - Know at what point your variable will "roll over" and also what happens in the other direction e.g. (0 - 1) OR (0 - - 32768)  
  - For math that requires fractions, use float variables, but be aware of their drawbacks: large size, slow computation speeds  
  - Use the cast operator e.g. (int)myFloat to convert one variable type to another on the fly.  

### % (modulo)

Calculates the remainder when one integer is divided by another. It is useful for keeping a variable within a particular range (e.g. the size of an array).  It is defined so that `a % b == a - ((a / b) * b)`.  

`result = dividend % divisor`

`dividend` is the number to be divided and  
`divisor` is the number to divide by.

`result` is the remainder

The remainder function can have unexpected behavoir when some of the opperands are negative.  If the dividend is negative, then the result will be the smallest negative equivalency class.  In other words, when `a` is negative, `(a % b) == (a mod b) - b` where (a mod b) follows the standard mathematical definition of mod.  When the divisor is negative, the result is the same as it would be if it was positive.  

```C++
EXAMPLE USAGES
x = 9 % 5;   // x now contains 4
x = 5 % 5;   // x now contains 0
x = 4 % 5;   // x now contains 4
x = 7 % 5;   // x now contains 2
x = -7 % 5;  // x now contains -2
x = 7 % -5;  // x now contains 2
x = -7 % -5; // x now contains -2
```

```C++
EXAMPLE CODE
//update one value in an array each time through a loop

int values[10];
int i = 0;

void setup() {}

void loop()
{
  values[i] = analogRead(A0);
  i = (i + 1) % 10;   // modulo operator rolls over variable  
}
```

**TIP:**
The modulo operator does not work on floats.  For floats, an equivalent expression to `a % b` is `a - (b * ((int)(a / b)))`

Boolean operators
---

These can be used inside the condition of an if statement.

### && (and)

True only if both operands are true, e.g.

```C++
if (digitalRead(D2) == HIGH  && digitalRead(D3) == HIGH) 
{ 
  // read two switches 
  // ...
}
//is true only if both inputs are high.
```

### || (or)

True if either operand is true, e.g. 

```C++
if (x > 0 || y > 0) 
{
  // ...
}
//is true if either x or y is greater than 0.
```

### ! (not)

True if the operand is false, e.g. 

```C++
if (!x) 
{ 
  // ...
} 
//is true if x is false (i.e. if x equals 0). 
```

**WARNING:**  
Make sure you don't mistake the boolean AND operator, && (double ampersand) for the bitwise AND operator & (single ampersand). They are entirely different beasts.

Similarly, do not confuse the boolean || (double pipe) operator with the bitwise OR operator | (single pipe).

The bitwise not ~ (tilde) looks much different than the boolean not ! (exclamation point or "bang" as the programmers say) but you still have to be sure which one you want where.

`if (a >= 10 && a <= 20){}   // true if a is between 10 and 20`

Bitwise operators
---

### & (bitwise and)

The bitwise AND operator in C++ is a single ampersand, &, used between two other integer expressions. Bitwise AND operates on each bit position of the surrounding expressions independently, according to this rule: if both input bits are 1, the resulting output is 1, otherwise the output is 0. Another way of expressing this is: 

```
    0  0  1  1    operand1
    0  1  0  1    operand2
    ----------
    0  0  0  1    (operand1 & operand2) - returned result
```

```C++
EXAMPLE USAGE
int a =  92;    // in binary: 0000000001011100
int b = 101;    // in binary: 0000000001100101
int c = a & b;  // result:    0000000001000100, or 68 in decimal.
```
One of the most common uses of bitwise AND is to select a particular bit (or bits) from an integer value, often called masking. 

### | (bitwise or)

The bitwise OR operator in C++ is the vertical bar symbol, |. Like the & operator, | operates independently each bit in its two surrounding integer expressions, but what it does is different (of course). The bitwise OR of two bits is 1 if either or both of the input bits is 1, otherwise it is 0. In other words: 

```
    0  0  1  1    operand1
    0  1  0  1    operand2
    ----------
    0  1  1  1    (operand1 | operand2) - returned result
```
```C++
EXAMPLE USAGE
int a =  92;    // in binary: 0000000001011100
int b = 101;    // in binary: 0000000001100101
int c = a | b;  // result:    0000000001111101, or 125 in decimal.
```

### ^ (bitwise xor)

There is a somewhat unusual operator in C++ called bitwise EXCLUSIVE OR, also known as bitwise XOR. (In English this is usually pronounced "eks-or".) The bitwise XOR operator is written using the caret symbol ^. This operator is very similar to the bitwise OR operator |, only it evaluates to 0 for a given bit position when both of the input bits for that position are 1: 

```
    0  0  1  1    operand1
    0  1  0  1    operand2
    ----------
    0  1  1  0    (operand1 ^ operand2) - returned result
```
Another way to look at bitwise XOR is that each bit in the result is a 1 if the input bits are different, or 0 if they are the same.

```C++
EXAMPLE USAGE
int x = 12;     // binary: 1100
int y = 10;     // binary: 1010
int z = x ^ y;  // binary: 0110, or decimal 6
```

The ^ operator is often used to toggle (i.e. change from 0 to 1, or 1 to 0) some of the bits in an integer expression. In a bitwise OR operation if there is a 1 in the mask bit, that bit is inverted; if there is a 0, the bit is not inverted and stays the same.


### ~ (bitwise not)

The bitwise NOT operator in C++ is the tilde character ~. Unlike & and |, the bitwise NOT operator is applied to a single operand to its right. Bitwise NOT changes each bit to its opposite: 0 becomes 1, and 1 becomes 0. For example: 

```
    0  1    operand1
   ----------
    1  0   ~ operand1

int a = 103;    // binary:  0000000001100111
int b = ~a;     // binary:  1111111110011000 = -104
```
You might be surprised to see a negative number like -104 as the result of this operation. This is because the highest bit in an int variable is the so-called sign bit. If the highest bit is 1, the number is interpreted as negative. This encoding of positive and negative numbers is referred to as two's complement. For more information, see the Wikipedia article on [two's complement.](http://en.wikipedia.org/wiki/Twos_complement)

As an aside, it is interesting to note that for any integer x, ~x is the same as -x-1.

At times, the sign bit in a signed integer expression can cause some unwanted surprises. 

### << (bitwise left shift), >> (bitwise right shift)

There are two bit shift operators in C++: the left shift operator << and the right shift operator >>. These operators cause the bits in the left operand to be shifted left or right by the number of positions specified by the right operand.

More on bitwise math may be found [here.](http://www.arduino.cc/playground/Code/BitMath)

```
variable << number_of_bits
variable >> number_of_bits 
```

`variable` can be `byte`, `int`, `long`  
`number_of_bits` and integer <= 32  

```C++
EXAMPLE USAGE
int a = 5;        // binary: 0000000000000101
int b = a << 3;   // binary: 0000000000101000, or 40 in decimal
int c = b >> 3;   // binary: 0000000000000101, or back to 5 like we started with
```
When you shift a value x by y bits (x << y), the leftmost y bits in x are lost, literally shifted out of existence: 

```C++
int a = 5;        // binary: 0000000000000101
int b = a << 14;  // binary: 0100000000000000 - the first 1 in 101 was discarded
```
If you are certain that none of the ones in a value are being shifted into oblivion, a simple way to think of the left-shift operator is that it multiplies the left operand by 2 raised to the right operand power. For example, to generate powers of 2, the following expressions can be employed: 

```
1 <<  0  ==    1
1 <<  1  ==    2
1 <<  2  ==    4
1 <<  3  ==    8
...
1 <<  8  ==  256
1 <<  9  ==  512
1 << 10  == 1024
...
```
When you shift x right by y bits (x >> y), and the highest bit in x is a 1, the behavior depends on the exact data type of x. If x is of type int, the highest bit is the sign bit, determining whether x is negative or not, as we have discussed above. In that case, the sign bit is copied into lower bits, for esoteric historical reasons: 

```C++
int x = -16;     // binary: 1111111111110000
int y = x >> 3;  // binary: 1111111111111110
```
This behavior, called sign extension, is often not the behavior you want. Instead, you may wish zeros to be shifted in from the left. It turns out that the right shift rules are different for unsigned int expressions, so you can use a typecast to suppress ones being copied from the left: 

```C++
int x = -16;                   // binary: 1111111111110000
int y = (unsigned int)x >> 3;  // binary: 0001111111111110
```

If you are careful to avoid sign extension, you can use the right-shift operator >> as a way to divide by powers of 2. For example: 

```C++
int x = 1000;
int y = x >> 3;   // integer division of 1000 by 8, causing y = 125
```
Compound operators
---

### ++ (increment), -- (decrement)

Increment or decrement a variable

```C++
SYNTAX
x++;  // increment x by one and returns the old value of x
++x;  // increment x by one and returns the new value of x

x-- ;   // decrement x by one and returns the old value of x 
--x ;   // decrement x by one and returns the new value of x 
```

where `x` is an integer or long (possibly unsigned)

```C++
EXAMPLE USAGE
x = 2;
y = ++x;      // x now contains 3, y contains 3
y = x--;      // x contains 2 again, y still contains 3 
```

### compound arithmetic

- += (compound addition)
- -= (compound subtraction)
- *= (compound multiplication)
- /= (compound division)

Perform a mathematical operation on a variable with another constant or variable. The += (et al) operators are just a convenient shorthand for the expanded syntax.

```C++
SYNTAX
x += y;   // equivalent to the expression x = x + y;
x -= y;   // equivalent to the expression x = x - y; 
x *= y;   // equivalent to the expression x = x * y; 
x /= y;   // equivalent to the expression x = x / y; 
```

`x` can be any variable type  
`y` can be any variable type or constant

```C++
EXAMPLE USAGE
x = 2;
x += 4;      // x now contains 6
x -= 3;      // x now contains 3
x *= 10;     // x now contains 30
x /= 2;      // x now contains 15
```

### &= (compound bitwise and)

The compound bitwise AND operator (&=) is often used with a variable and a constant to force particular bits in a variable to the LOW state (to 0). This is often referred to in programming guides as "clearing" or "resetting" bits.

`x &= y;   // equivalent to x = x & y;`

`x` can be a char, int or long variable  
`y` can be an integer constant, char, int, or long  

```
   0  0  1  1    operand1
   0  1  0  1    operand2
   ----------
   0  0  0  1    (operand1 & operand2) - returned result
```
Bits that are "bitwise ANDed" with 0 are cleared to 0 so, if myByte is a byte variable,  
`myByte & B00000000 = 0;`  

Bits that are "bitwise ANDed" with 1 are unchanged so,  
`myByte & B11111111 = myByte;`

**Note:** because we are dealing with bits in a bitwise operator - it is convenient to use the binary formatter with constants. The numbers are still the same value in other representations, they are just not as easy to understand. Also, B00000000 is shown for clarity, but zero in any number format is zero (hmmm something philosophical there?)

Consequently - to clear (set to zero) bits 0 & 1 of a variable, while leaving the rest of the variable unchanged, use the compound bitwise AND operator (&=) with the constant B11111100 

```
   1  0  1  0  1  0  1  0    variable  
   1  1  1  1  1  1  0  0    mask
   ----------------------
   1  0  1  0  1  0  0  0

 variable unchanged
                     bits cleared
```
Here is the same representation with the variable's bits replaced with the symbol x

```
   x  x  x  x  x  x  x  x    variable
   1  1  1  1  1  1  0  0    mask
   ----------------------
   x  x  x  x  x  x  0  0

 variable unchanged
                     bits cleared
```

So if:  
`myByte =  10101010;`  
`myByte &= B1111100 == B10101000;`


### |= (compound bitwise or)

The compound bitwise OR operator (|=) is often used with a variable and a constant to "set" (set to 1) particular bits in a variable.

```C++
SYNTAX
x |= y;   // equivalent to x = x | y; 
```
`x` can be a char, int or long variable  
`y` can be an integer constant or char, int or long  

```
   0  0  1  1    operand1
   0  1  0  1    operand2
   ----------
   0  1  1  1    (operand1 | operand2) - returned result
```   
Bits that are "bitwise ORed" with 0 are unchanged, so if myByte is a byte variable,  
`myByte | B00000000 = myByte;`

Bits that are "bitwise ORed" with 1 are set to 1 so:  
`myByte | B11111111 = B11111111;` 

Consequently - to set bits 0 & 1 of a variable, while leaving the rest of the variable unchanged, use the compound bitwise OR operator (|=) with the constant B00000011   

```
   1  0  1  0  1  0  1  0    variable
   0  0  0  0  0  0  1  1    mask
   ----------------------
   1  0  1  0  1  0  1  1

 variable unchanged
                     bits set
```
Here is the same representation with the variables bits replaced with the symbol x
```
   x  x  x  x  x  x  x  x    variable
   0  0  0  0  0  0  1  1    mask
   ----------------------
   x  x  x  x  x  x  1  1

 variable unchanged
                     bits set
```
So if:  
`myByte =  B10101010;`  
`myByte |= B00000011 == B10101011;`  

Variables
========

Constants
----

### HIGH | LOW

When reading or writing to a digital pin there are only two possible values a pin can take/be-set-to: HIGH and LOW.

`HIGH`

The meaning of `HIGH` (in reference to a pin) is somewhat different depending on whether a pin is set to an `INPUT` or `OUTPUT`. When a pin is configured as an INPUT with pinMode, and read with digitalRead, the microcontroller will report HIGH if a voltage of 3 volts or more is present at the pin.

A pin may also be configured as an `INPUT` with `pinMode`, and subsequently made `HIGH` with `digitalWrite`, this will set the internal 40K pullup resistors, which will steer the input pin to a `HIGH` reading unless it is pulled LOW by external circuitry. This is how INPUT_PULLUP works as well

When a pin is configured to `OUTPUT` with `pinMode`, and set to `HIGH` with `digitalWrite`, the pin is at 3.3 volts. In this state it can source current, e.g. light an LED that is connected through a series resistor to ground, or to another pin configured as an output, and set to `LOW.`

`LOW`

The meaning of `LOW` also has a different meaning depending on whether a pin is set to `INPUT` or `OUTPUT`. When a pin is configured as an `INPUT` with `pinMode`, and read with `digitalRead`, the microcontroller will report `LOW` if a voltage of 1.5 volts or less is present at the pin.

When a pin is configured to `OUTPUT` with `pinMode`, and set to `LOW` with digitalWrite, the pin is at 0 volts. In this state it can sink current, e.g. light an LED that is connected through a series resistor to, +3.3 volts, or to another pin configured as an output, and set to `HIGH.` 

### INPUT, OUTPUT, INPUT_PULLUP, INPUT_PULLDOWN 

Digital pins can be used as INPUT, INPUT_PULLUP, INPUT_PULLDOWN or OUTPUT. Changing a pin with `pinMode()` changes the electrical behavior of the pin.

Pins Configured as `INPUT`

The Spark Core's pins configured as `INPUT` with `pinMode()`` are said to be in a high-impedance state. Pins configured as `INPUT` make extremely small demands on the circuit that they are sampling, equivalent to a series resistor of 100 Megohms in front of the pin. This makes them useful for reading a sensor, but not powering an LED.

If you have your pin configured as an `INPUT`, you will want the pin to have a reference to ground, often accomplished with a pull-down resistor (a resistor going to ground). 

Pins Configured as `INPUT_PULLUP` or `INPUT_PULLDOWN`

The STM32 microcontroller has internal pull-up resistors (resistors that connect to power internally) and pull-down resistors (resistors that connect to ground internally) that you can access. If you prefer to use these instead of external resistors, you can use these argument in `pinMode()`. 

Pins Configured as `OUTPUT`

Pins configured as `OUTPUT` with `pinMode()`` are said to be in a low-impedance state. This means that they can provide a substantial amount of current to other circuits. STM32 pins can source (provide positive current) or sink (provide negative current) up to 20 mA (milliamps) of current to other devices/circuits. This makes them useful for powering LED's but useless for reading sensors. Pins configured as outputs can also be damaged or destroyed if short circuited to either ground or 3.3 volt power rails. The amount of current provided by the pin is also not enough to power most relays or motors, and some interface circuitry will be required. 

### true | false

There are two constants used to represent truth and falsity in the Arduino language: true, and false.

`false`

`false` is the easier of the two to define. false is defined as 0 (zero).

`true`

`true` is often said to be defined as 1, which is correct, but true has a wider definition. Any integer which is non-zero is true, in a Boolean sense. So -1, 2 and -200 are all defined as true, too, in a Boolean sense.

Note that the true and false constants are typed in lowercase unlike `HIGH, LOW, INPUT, & OUTPUT.` 
