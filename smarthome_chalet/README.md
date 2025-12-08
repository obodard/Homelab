# Smart Home Chalet - Zigbee2MQTT Setup

This repository contains a Docker-based smart home setup for managing Zigbee devices using Zigbee2MQTT and Mosquitto MQTT broker.

## Architecture

The setup consists of two main services:
- **Mosquitto**: MQTT broker for message handling
- **Zigbee2MQTT**: Bridge between Zigbee devices and MQTT

## Prerequisites

- Docker and Docker Compose installed
- A Zigbee USB adapter (e.g., SONOFF Zigbee 3.0 USB Dongle Plus V2)
- The Zigbee adapter connected to your system

## Installation

### 1. Clone the Repository

```bash
git clone <repository-url>
cd smarthome_chalet
```

### 2. Verify Zigbee Adapter Connection

First, identify your Zigbee adapter's device path:

```bash
ls -l /dev/serial/by-id/
```

Update the device path in `docker-compose.yml` if it differs from the current configuration:

```yaml
devices:
  - /dev/serial/by-id/usb-ITEAD_SONOFF_Zigbee_3.0_USB_Dongle_Plus_V2_20231118181614-if00:/dev/ttyACM0
```

### 3. Configure Zigbee2MQTT

The configuration file is located at `zigbee2mqtt/data/configuration.yaml`. Key settings include:

- **MQTT connection**: Points to the Mosquitto broker
- **Serial port**: `/dev/ttyACM0` (mapped from your USB adapter)
- **Frontend**: Enabled on port 28080
- **Permit join**: Set to `true` to allow pairing new devices

### 4. Start the Services

```bash
docker-compose up -d
```

### 5. Verify Services are Running

```bash
docker-compose ps
```

Both services should show as "running".

## Accessing Zigbee2MQTT

The Zigbee2MQTT web interface is available at:

```
http://localhost:28080
```

Or replace `localhost` with your server's IP address if accessing remotely.

## Adding a New Zigbee Device

### Step 1: Enable Pairing Mode

Pairing mode is controlled by the `permit_join` setting in `configuration.yaml` (currently enabled by default).

You can also enable/disable pairing through the web interface:

1. Open the Zigbee2MQTT web interface at `http://localhost:28080`
2. Click on **"Permit join (All)"** button to enable pairing mode
3. The pairing mode will remain active for a limited time

### Step 2: Put Your Zigbee Device in Pairing Mode

Each device has a different pairing procedure. Common methods include:

- **Pressing and holding** a button for 5-10 seconds
- **Power cycling** the device 5-6 times
- **Triple-clicking** a button
- **Following specific instructions** from the manufacturer

**For device-specific pairing instructions**, refer to the official Zigbee2MQTT documentation:

🔗 **[Zigbee2MQTT Supported Devices](https://www.zigbee2mqtt.io/supported-devices/)**

1. Search for your device model
2. Click on your device to see detailed information
3. Follow the "Pairing" instructions for your specific device

### Step 3: Verify Device Connection

Once paired, the device will appear in:

- The **Zigbee2MQTT web interface** under "Devices"
- The **logs**: `docker-compose logs -f zigbee2mqtt`

### Step 4: Configure and Test

1. In the web interface, click on your newly added device
2. Rename it for easier identification
3. Test the device functionality (e.g., toggle lights, read sensor data)

## Managing the Services

### View Logs

```bash
# All services
docker-compose logs -f

# Specific service
docker-compose logs -f zigbee2mqtt
docker-compose logs -f mosquitto
```

### Restart Services

```bash
docker-compose restart
```

### Stop Services

```bash
docker-compose down
```

### Update Services

```bash
docker-compose pull
docker-compose up -d
```

## Configuration Files

- `docker-compose.yml`: Service definitions and container configuration
- `mosquitto/mosquitto.conf`: Mosquitto MQTT broker configuration
- `zigbee2mqtt/data/configuration.yaml`: Zigbee2MQTT settings

## Troubleshooting

### Device Not Pairing

1. Put the Zigbee device as close as possible to the SONOFF Zigbee 3.0 USB Dongle
1. Ensure `permit_join: true` is enabled in the configuration or via the web interface
2. Check that the Zigbee adapter is properly connected
3. Verify the device is in pairing mode (refer to device-specific instructions)
4. Check logs for error messages: `docker-compose logs -f zigbee2mqtt`

### Cannot Access Web Interface

1. Verify the service is running: `docker-compose ps`
2. Check that port 28080 is not blocked by a firewall
3. Ensure you're using the correct IP address

### USB Adapter Not Found

1. Verify the adapter is connected: `ls -l /dev/serial/by-id/`
2. Update the device path in `docker-compose.yml`
3. Ensure proper permissions for the device
4. Restart the services: `docker-compose restart`

## Resources

- [Zigbee2MQTT Documentation](https://www.zigbee2mqtt.io/)
- [Zigbee2MQTT Supported Devices](https://www.zigbee2mqtt.io/supported-devices/)
- [Mosquitto Documentation](https://mosquitto.org/documentation/)
- [Docker Compose Documentation](https://docs.docker.com/compose/)

## Security Note

The current Mosquitto configuration allows anonymous connections. For production use, consider:

1. Enabling authentication in `mosquitto.conf`
2. Using strong passwords
3. Implementing TLS/SSL encryption
4. Restricting network access

## License

See [LICENSE](LICENSE) file for details.
