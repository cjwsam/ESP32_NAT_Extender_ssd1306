# ESP32 NAT Router / WiFi Range Extender with SSD1306 OLED Display

An ESP32-based WiFi NAT (Network Address Translation) router and range extender with real-time status output on an SSD1306 OLED display. The device connects to an existing WiFi network as a station (STA) and re-broadcasts it as its own access point (AP), effectively extending your WiFi coverage.

## Features

- **WiFi Range Extension** -- connects to an upstream WiFi network and creates a new access point that other devices can join
- **NAT Routing** -- performs Network Address Translation so all connected clients share a single IP on the upstream network
- **SSD1306 OLED Display** -- shows boot progress, connection status, and a countdown animation on a 128x64 or 128x32 OLED screen
- **Web Configuration Interface** -- configure SSID and password settings through a browser at `http://192.168.4.1`
- **Serial Console (CLI)** -- full command-line interface over UART (115200 bps) for configuration and diagnostics
- **Persistent Settings** -- WiFi credentials are stored in NVS (Non-Volatile Storage) and survive reboots
- **Up to 8 simultaneous client connections**

## Network Topology

```
                        NAT
 [Upstream Router]  <---------->  [ESP32 NAT Extender]  <---------->  [Client Devices]
   (existing WiFi)    STA side        192.168.4.1          AP side     (phones, laptops,
                                   OLED shows status                    IoT devices)
```

1. The ESP32 connects to your existing WiFi router as a **station (STA)**.
2. The ESP32 creates its own **access point (AP)** with a configurable SSID/password (default: `ESP32_NAT_Router`, open).
3. Client devices connect to the ESP32 AP and are NAT-routed through to the upstream network.
4. The default AP IP address is **192.168.4.1** and DNS is forwarded to **8.8.8.8** (Google DNS).

## Hardware Requirements

| Component | Description |
|-----------|-------------|
| **ESP32 development board** | Any ESP32 board (ESP32-DevKitC, NodeMCU-32S, etc.) |
| **SSD1306 OLED display** | 128x64 (default) or 128x32 pixel, I2C or SPI interface |
| **18650 battery + holder** (optional) | For portable/battery-powered operation |

## Wiring / Pin Connections

### I2C Interface (Default)

| SSD1306 Pin | ESP32 GPIO | Description |
|-------------|------------|-------------|
| SDA         | GPIO 21    | I2C Data    |
| SCL         | GPIO 22    | I2C Clock   |
| VCC         | 3.3V       | Power       |
| GND         | GND        | Ground      |

The I2C address is **0x3C** (default for most SSD1306 modules).

GPIO pin assignments can be changed in `menuconfig` under **SSD1306 Configuration**:
- `CONFIG_SDA_GPIO` -- SDA pin (default: 21)
- `CONFIG_SCL_GPIO` -- SCL pin (default: 22)
- `CONFIG_RESET_GPIO` -- Reset pin (default: -1, meaning no reset pin)

### SPI Interface (Optional)

| SSD1306 Pin | ESP32 GPIO | Description  |
|-------------|------------|--------------|
| MOSI        | GPIO 13    | SPI Data Out |
| SCLK        | GPIO 14    | SPI Clock    |
| CS          | GPIO 15    | Chip Select  |
| DC          | GPIO 33    | Data/Command |
| RST         | -1         | Reset (optional) |
| VCC         | 3.3V       | Power        |
| GND         | GND        | Ground       |

Select the SPI interface in `menuconfig` under **SSD1306 Configuration > Interface**.

## OLED Display Usage

The OLED display shows:
- **Boot screen** -- "BOOTING" header with "Initialising..." message and display resolution info
- **Countdown animation** -- A 9-to-0 countdown during initialization phases
- **Connection status** -- "DONE" message once WiFi is initialized

## Configuration

### First Boot

On first boot, the ESP32 creates an **open** WiFi network named **ESP32_NAT_Router**. Connect to it and configure via the web interface or serial console.

### Web Interface

1. Connect your device to the `ESP32_NAT_Router` WiFi network.
2. Open a browser and navigate to **http://192.168.4.1**.
3. Enter the **STA settings** (upstream WiFi SSID and password) and click **Connect**.
4. Optionally change the **AP settings** (the SSID/password the ESP32 broadcasts).
   - AP passwords shorter than 8 characters result in an open (no password) network.
5. The device will restart and connect to the configured upstream network.

### Serial Console (CLI)

Connect via a serial terminal (PuTTY, GtkTerm, etc.) at **115200 bps**.

| Command | Description |
|---------|-------------|
| `set_sta <ssid> <password>` | Set upstream WiFi SSID and password |
| `set_ap <ssid> <password>` | Set the ESP32 AP SSID and password |
| `show` | Display current configuration and connection status |
| `restart` | Restart the device |
| `help` | List all available commands |

### DNS Configuration

The DNS server offered to AP clients is **8.8.8.8** (Google DNS) by default. To change it, modify the `MY_DNS_IP_ADDR` value in `main/esp32_nat_router.c`:

```c
#define MY_DNS_IP_ADDR 0x08080808 // 8.8.8.8
```

The value is a 32-bit hex representation of the IPv4 address.

## Build and Flash Instructions

### Prerequisites

- [ESP-IDF](https://docs.espressif.com/projects/esp-idf/en/latest/esp32/get-started/) (Espressif IoT Development Framework) installed and configured
- An ESP-IDF version with NAPT (NAT) support in LWIP

### Configure

```bash
idf.py menuconfig
```

Set the following options:

1. **Component config > LWIP > Enable copy between Layer2 and Layer3 packets** -- Enable
2. **Component config > LWIP > Enable IP forwarding** -- Enable
3. **Component config > LWIP > Enable NAT (new/experimental)** -- Enable
4. **SSD1306 Configuration > Interface** -- Select I2C or SPI
5. **SSD1306 Configuration > SCL GPIO number** -- Set SCL pin (default: 22)
6. **SSD1306 Configuration > SDA GPIO number** -- Set SDA pin (default: 21)
7. **SSD1306 Configuration > Panel Type** -- Select 128x64 or 128x32

### Build

```bash
idf.py build
```

### Flash

```bash
idf.py -p <PORT> flash
```

Replace `<PORT>` with your serial port (e.g., `COM3` on Windows, `/dev/ttyUSB0` on Linux).

### Monitor

```bash
idf.py -p <PORT> monitor
```

### Using Legacy Make (Alternative)

```bash
make menuconfig
make
make flash
```

## Pre-built Binary

A pre-built binary (`esp32_nat_router.bin`) is included in the repository for quick flashing:

```bash
esptool.py --chip esp32 --port <PORT> write_flash 0x10000 esp32_nat_router.bin
```

## Performance

Typical throughput is around **10 Mbps** (IPv4/TCP), with up to **15 Mbps** achievable with tuning.

## Troubleshooting

### Terminal Escape Sequences

If your serial terminal shows a message about unsupported escape sequences, try using PuTTY (Windows) or GtkTerm (Linux). Line editing and history features require escape sequence support.

### WiFi Not Connecting

- Verify the upstream SSID and password are correct using the `show` command.
- Ensure the ESP32 is within range of the upstream router.
- Restart the device after changing settings.

## Demo

[![Video Example](ESP32_OLED(ssd1306)_Batt1835_NAT_Router.gif)](https://www.youtube.com/watch?v=0Z27zeoPMCY)

## License

This project is in the Public Domain (or CC0 licensed, at your option).
