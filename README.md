# ESP32 OLED Batt NAT Router

This is a firmware to use the ESP32 ssd1306 1835Batt as WiFi NAT router. It can be used as
- Simple range extender for an existing WiFi network
- Setting up an additional WiFi network with different SSID/password for guests or IOT devices

It can achieve a bandwidth of more than 15mbps with alot of tweaking !! but on average 10mbp

All tests used `IPv4` and the `TCP` protocol.

## First Boot
After first boot the ESP32 NAT Router will offer a WiFi network with an open AP and the ssid "ESP32_NAT_Router". Configuration can either be done via a simple web interface or via the serial console. 

## Web Config Interface
The web interface allows for the configuration of all parameters. Connect you PC or smartphone to the WiFi SSID "ESP32_NAT_Router" and point your browser to "http://192.168.4.1". This page should appear:

# Command Line Interface

For configuration you have to use a serial console (Putty or GtkTerm with 115200 bps).
Use the "set_sta" and the "set_ap" command to configure the WiFi settings. Changes are stored persistently in NVS and are applied after next restart. Use "show" to display the current config.

Enter the `help` command get a full list of all available commands.


2. In the project directory run `make menuconfig` (or `idf.py menuconfig` for cmake).
    1. *Component config -> LWIP > [x] Enable copy between Layer2 and Layer3 packets.
    2. *Component config -> LWIP > [x] Enable IP forwarding.
    3. *Component config -> LWIP > [x] Enable NAT (new/experimental).
    4. *SDD1306 config -> SCL > 4
    5. *SDD1306 config -> SDA > 5
3. Build the project and flash it to the ESP32.

A detailed instruction on how to build, configure and flash a ESP-IDF project can also be found the official ESP-IDF guide. 

### DNS
By Default the DNS-Server which is offerd to clients connecting to the ESP32 AP is set to 8.8.8.8.
Replace the value of the *MY_DNS_IP_ADDR* with your desired DNS-Server IP address (in hex) if you want to use a different one.

## Troubleshooting

### Line Endings

The line endings in the Console Example are configured to match particular serial monitors. Therefore, if the following log output appears, consider using a different serial monitor (e.g. Putty for Windows or GtkTerm on Linux) or modify the example's UART configuration.

```
This is an example of ESP-IDF console component.
Type 'help' to get the list of commands.
Use UP/DOWN arrows to navigate through command history.
Press TAB when typing command name to auto-complete.
Your terminal application does not support escape sequences.
Line editing and history features are disabled.
On Windows, try using Putty instead.
esp32>
```
[![Video Example](doc/gifName.gif)](https://www.youtube.com/watch?v=0Z27zeoPMCY)
