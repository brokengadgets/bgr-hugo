---
date: '2025-01-18T23:48:03-05:00'
draft: false
title: 'ESP UART using Arduino IDE'
omit_header_text: true
featured_image: '/images/pinned.png'
type: page

---
Many projects have me finding myself in a position where I cannot find all of the information in one place to solve what should be a simple problem. 

![esp uart pinout](/images/espuart-pinout.png)

I'm on Manjaro Linux trying to use the Arduino IDE to create a usb UART interface from an esp32 devkit I have in the drawer.
Before getting started, we needed to find a board support package for the ESP32 to shove into the Arduino IDE.

1. With Arduino IDE open, press `Ctrl`+`,`
2. Under `Additional Board Manager URLs` add `https://espressif.github.io/arduino-esp32/package_esp32_dev_index.json`
3. Open Tools > Board > Board manager
4. Search `Esp32` and install `Arduino ESP32 Boards`
5. Select an Arduino esp32 option from the boards dropdown before compiling or flashing

Upon compilation it was determined that there was no serial module available for flashig phase.\
{{< highlight bash "linesos=false" >}}
import serial
ImportError: No module named serial
{{< / highlight >}}

As it turns out, I did not have pip, either. Open a terminal and input:\
{{< highlight bash "linesos=false" >}}
sudo pacman -S python-pip
pip install pyserial
{{< / highlight >}}
Come to compile again and everything checks out, going to flash it to the board now to find another error:\
{{< highlight bash "linenos=false" >}}
Serial port /dev/ttyUSB0
File "/home/YOURUSERHERE/.local/lib/python3.10/site-packages/serial/serialposix.py", line 322, in open self.fd = os.open(self.portstr, os.O_RDWR | os.O_NOCTTY | os.O_NONBLOCK)
PermissionError: [Errno 13] Permission denied: '/dev/ttyUSB0'
{{< / highlight >}}


Permissions can be checked in your terminal using:\
{{< highlight bash "linenos=false" >}}
ls -lash /dev/tty*
{{< / highlight >}}

in this case, I needed to modify my user permissions in the group uucp to access serial devices. \
{{< highlight bash "linenos=false" >}}
sudo usermod -aG YOURUSERHERE uucp
{{< / highlight >}}

To remove users from a group if you wish to do so after flashing:\
{{< highlight bash "linenos=false" >}}
sudo gpasswd -d YOURUSERHERE uucp
{{< / highlight >}}

Below pins 2 and 4 are being used. Pin 2 for receive and pin 4 for transmit.\
A substantial amount of help came from [this source](https://techoverflow.net/2021/11/19/how-to-use-esp32-as-usb-to-uart-converter-in-platformio/) 

Using any GPIO pins as Rx / Tx:

{{< highlight cpp "linenos=table,linenostart=1" >}}
#define UART_RX_PIN 2 // GPIO2
#define UART_TX_PIN 4 // GPIO4
void setup() {
//Serial connects to the computer
    Serial.begin(115200);
//Serial2 is the hardware UART port that connects to the external circuit.
//115200 is the baudrate expected for UART.
    Serial2.begin(115200, SERIAL_8N1,
    UART_RX_PIN,
    UART_TX_PIN);
    }

    void loop() {
        //Forward bytes incoming via PC serial
        while (Serial.available() > 0) {
        Serial2.write(Serial.read());
    }
    //Forward bytes incoming via UART serial
    while (Serial2.available() > 0) {
        Serial.write(Serial2.read());
    }
}

{{< / highlight >}}

Screen command for setting baudrate on your TTY port:\
{{< highlight bash "linenos=false" >}}
screen /dev/ttyUSB0 115200
{{< / highlight >}}