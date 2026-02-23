# Home Assistant on Raspberry Pi (Docker Compose) - Hilo Integration

This project runs **Home Assistant Container** on a Raspberry Pi using Docker Compose, integrates Hilo devices via HACS, and includes custom heating automation.

---

## 1. Architecture Overview

**Current environment:**

- Raspberry Pi (ARM)
- Raspberry Pi OS 32-bit
- Docker + `docker compose` (plugin v2)
- Home Assistant Container
- HACS (Home Assistant Community Store)
- Hilo custom integration
- Custom YAML automation file

> **Note:** Because this system runs 32-bit ARM, we use the architecture-specific image:
>
> ```
> ghcr.io/home-assistant/armv7-homeassistant:latest
> ```
>
> The standard `home-assistant:stable` image requires 64-bit ARM.

---

## 2. Docker Compose Setup

### `docker-compose.yaml`

```yaml
services:
  homeassistant:
    container_name: homeassistant
    image: ghcr.io/home-assistant/armv7-homeassistant:latest
    network_mode: host
    privileged: true
    environment:
      - TZ=America/Montreal
    volumes:
      - ./config:/config
      - /etc/localtime:/etc/localtime:ro
    restart: unless-stopped
```

### Start Home Assistant

```bash
docker compose pull
docker compose up -d
```

**Check logs:**

```bash
docker logs -f homeassistant
```

**Access UI:**

```
http://<raspberry-pi-ip>:8123
```

---

## 3. Initial Home Assistant Setup

On first launch:

1. Create admin account
2. Set:
   - Location
   - Unit system (Metric)
   - Timezone (`America/Montreal`)
3. Confirm device discovery
4. Skip integrations for now

**Recommended:**

- Disable automatic updates until stable
- Create full backup after first configuration

---

## 4. Install HACS (Required for Hilo)

**Enter container shell:**

```bash
docker exec -it homeassistant /bin/bash
```

**Install HACS:**

```bash
wget -O - https://get.hacs.xyz | bash -
```

**Exit and restart:**

```bash
exit
docker restart homeassistant
```

**In UI:**

1. Go to **Settings → Devices & Services → Add Integration**
2. Search for **"HACS"**
3. Complete GitHub authentication flow

---

## 5. Install Hilo Integration (Both Modules)

### Inside HACS

1. Open **HACS**
2. Go to **Integrations**
3. Search for **"Hilo"**
4. Install:
   - Hilo main integration
   - `python-hilo` dependency (if prompted)
5. Restart Home Assistant

### After restart

1. Go to **Settings → Devices & Services → Add Integration**
2. Search for **"Hilo"**
3. Authenticate with Hilo credentials

Devices should import automatically.

---

## 6. Custom Automation Setup

Automation YAML is stored separately *(recommended for version control)*.

### Example structure

```
config/
  automations/
    chauffage_entree.yaml
```

In `configuration.yaml` add:

```yaml
automation: !include_dir_merge_list automations/
```

> **Restart Home Assistant** after editing `configuration.yaml`.

### Automation Logic

- **Uses thermostat:** `climate.coup_de_pied_cuisine`
  - Attributes: `temperature` (setpoint), `current_temperature`
- **Controls:** `light.chauffage_entree`

**Behavior:**

| Condition | Action |
|---|---|
| Setpoint **< 7°C** | Turn **ON** at ≤ 4°C / Turn **OFF** at > 6°C |
| Setpoint **≥ 19°C** | Turn **ON** at ≤ 22°C / Turn **OFF** at > 25°C |

> Use **Automation Trace** to validate behavior.

---

## 7. Backups and Persistence

**Important directories:**

- `./config`

**Backup command:**

```bash
tar -czf ha-backup-$(date +%F).tgz config
```

**Recommended:**

- Weekly backup
- Store copy off-device
- Snapshot before upgrades

---

## 8. Upgrading to 64-bit Architecture *(Recommended Future Path)*

### Why Upgrade

- Official `home-assistant:stable` support
- Better performance
- Future-proof architecture
- Wider container compatibility

### Verify Current Architecture

```bash
uname -m
getconf LONG_BIT
docker info --format '{{.Architecture}}'
```

If 32-bit, proceed.

### Migration Steps

1. **Backup config:**

   ```bash
   tar -czf ha-config-backup.tgz config
   ```

2. Flash **Raspberry Pi OS 64-bit** (Lite recommended)
3. Reinstall Docker and `docker compose`
4. Restore `config` directory
5. Update `docker-compose.yaml` image to:

   ```yaml
   image: ghcr.io/home-assistant/home-assistant:stable
   ```

6. Start:

   ```bash
   docker compose up -d
   ```

7. Validate integrations

> **Downtime:** approximately 1–2 hours.

---

## 9. Security Notes

Current setup uses:

```yaml
privileged: true
network_mode: host
```

This simplifies hardware access but **increases attack surface**.

**Future hardening:**

- Remove `privileged` if not needed
- Restrict exposed ports
- Use firewall rules
- Enable Home Assistant authentication MFA

---

## 10. Operational Considerations

- **Cloud dependency:** Hilo integration depends on Hilo API availability
- If internet is down, device control may fail
- Community integrations may break after Home Assistant upgrades
- Always test upgrades in a maintenance window

**Upgrade safely:**

```bash
docker compose pull
docker compose up -d
```

> Monitor logs after upgrade.

---

## 11. Troubleshooting

**Check logs:**

```bash
docker logs homeassistant --tail 200
```

**Common issues:**

| Problem | Solution |
|---|---|
| Integration not visible | Restart Home Assistant |
| Hilo auth fails | Reconfigure integration |
| Automation not firing | Use **Automation Trace** |


