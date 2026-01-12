# Wingbits RTL Feeder

## Purpose

This project provides a **deterministic, RTL-SDR–based Wingbits feeder** designed for systems running **multiple SDR receivers**.
It allows explicit device selection using **RTL-SDR serial numbers**, avoiding device index conflicts when multiple dongles are present.

This repository is a fork and specialization of [sicXnull/wingbits-ultrafeeder](https://github.com/sicXnull/wingbits-ultrafeeder), adapted to:
- Focus exclusively on Wingbits
- Support reliable multi-SDR setups
- Simplify configuration and documentation
- Remove abandoned or unnecessary components

---

## Supported Platforms

This setup is supported on:

- Linux (native)
- Linux on ARM (Raspberry Pi, ARM64, ARMv7)
- Debian / Ubuntu–based distributions

Not supported:
- macOS (no native USB passthrough for RTL-SDR in Docker)
- Windows (Docker USB passthrough limitations)

Docker and Docker Compose are required.

---

## RTL-SDR Installation

### Install the RTL-SDR utilities using your distribution’s package manager.

Example (Debian/Ubuntu):

```bash
apt update
apt install rtl-sdr
```

## udev Rules (Recommended)

To allow non-root access to RTL-SDR devices, create udev rules:

### Create the rules file:

```bash
nano /etc/udev/rules.d/20-rtlsdr.rules
```

### Add the following rules:

```text
# RTL-SDR USB device permissions for ADS-B and AIS dongles

# Generic RTL2832U OEM / DVB-T dongles
SUBSYSTEMS=="usb", ATTRS{idVendor}=="0bda", ATTRS{idProduct}=="2832", MODE="0666"

# RTL2838 based devices (common for ADS-B and AIS dongles, including RTL-SDR Blog V4)
SUBSYSTEMS=="usb", ATTRS{idVendor}=="0bda", ATTRS{idProduct}=="2838", MODE="0666"

# AIRNAV / ADS-B receivers (some use RTL2832U/2838 variants)
SUBSYSTEMS=="usb", ATTRS{idVendor}=="0bda", ATTRS{idProduct}=="2830", MODE="0666"

# Other common RTL-SDR dongles used for ADS-B/AIS
SUBSYSTEMS=="usb", ATTRS{idVendor}=="0bda", ATTRS{idProduct}=="2836", MODE="0666"
SUBSYSTEMS=="usb", ATTRS{idVendor}=="0bda", ATTRS{idProduct}=="283c", MODE="0666"

# Optional: u-blox GPS dongle (if using GPS for location)
SUBSYSTEMS=="usb", ATTRS{idVendor}=="1546", ATTRS{idProduct}=="01a7", MODE="0666"
```

Notes:
- This covers the majority of ADS-B and AIS dongles on the market. Anything else is rare or exotic.
- Permissions 0666 allow any user or container to access the SDR without `sudo`.

### Reload rules:

```bash
udevadm control --reload-rules
udevadm trigger
```

A reboot is recommended after applying udev rules.

---

## Identify Your RTL-SDR Serial

### List connected RTL-SDR devices:

```bash
rtl_test -t
rtl_eeprom
```

Use the serial number (not the device index) in the `.env` file.

---

## Deployment

### Create a working directory

```bash
mkdir wingbits-rtl-feeder && cd wingbits-rtl-feeder
```
### Download the necessary files

```bash
curl -L -o .env.example https://raw.githubusercontent.com/c-man-the-man/wingbits-rtl-feeder/main/.env.example
curl -L -o docker-compose.yml https://raw.githubusercontent.com/Cc-man-the-man/wingbits-rtl-feeder/main/docker-compose.yml
```

### Set and configure the .env file

```bash
cp .env.example .env
nano .env
```

**Edit `.env` and set:**

- `WINGBITS_DEVICE_ID`
- `FEEDER_LAT`, `FEEDER_LONG`
- `FEEDER_TZ`
Optional
- `FEEDER_ALT_M`
- `ADSB_SDR_SERIAL`

### Run the container

```bash
docker compose up -d
```

### Check the logs

```bash
docker compose logs -f
```

---

## License

This project inherits the licenses of its upstream components.
Refer to individual upstream repositories for license details.
