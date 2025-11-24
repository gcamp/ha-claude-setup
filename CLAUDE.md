# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

This is a Home Assistant configuration repository running on Home Assistant OS. The configuration includes custom integrations, ESPHome devices, Zigbee2MQTT, and a highly customized Lovelace dashboard.

## Configuration Structure

### Main Configuration Files

- `configuration.yaml` - Main Home Assistant configuration with:
  - Package includes from `packages/` directory (currently empty but structure exists)
  - InfluxDB integration for metrics (host: 10.0.0.10)
  - MQTT sensors for Tesla data via TeslaMate
  - Lovelace YAML mode configuration
  - Custom panel definitions for Add-ons, Automations, and Integrations

- `automations.yaml` - All automation definitions
- `scripts.yaml` - Reusable scripts (backup creation, door locking, pool control)
- `scenes.yaml` - Scene definitions (lighting scenes)
- `sensors.yaml` - Template sensors including:
  - ERV (Energy Recovery Ventilator) monitoring
  - Indoor air quality scoring system (temperature, humidity, CO2)
  - Thermal comfort calculations

- `ui-lovelace.yaml` - Dashboard configuration using:
  - Decluttering templates (especially for Hilo Défi cards)
  - Custom HACS cards: Bubble Card, Mushroom, card-mod, mini-graph-card, Valetudo Map Card, etc.

- `modbus.yml` - Modbus RTU configuration for EG4 3000EHV inverter (currently commented out in configuration.yaml)
  - Serial connection: `/dev/serial/by-id/usb-FTDI_FT231X_USB_UART_D30JX932-if00-port0`
  - Baudrate: 9600, RTU mode

### Key Integrations

**Custom Components** (in `custom_components/`):
- `hilo` - Hilo energy management integration (Hydro-Québec)
- `frigate` - NVR/camera integration
- `tesla_custom` - Tesla vehicle integration
- `ecoflow_cloud` - EcoFlow battery/solar products
- `daikinone` - Daikin HVAC control
- `zigbee2mqtt` - Zigbee device management (via MQTT bridge)
- `scheduler` - Advanced scheduling
- `thermal_comfort` - Thermal comfort calculations
- `pikvm_ha` - Pi-KVM integration
- `tplink_deco` - TP-Link Deco mesh router integration
- `iphonedetect` - iPhone presence detection
- `nodered` - Node-RED integration
- `homekit_controller` - HomeKit device control
- `hacs` - Home Assistant Community Store

**ESPHome Devices** (in `esphome/`):
- `waveshare-relay.yaml` - ESP32-S3 based relay controller
- Configuration uses secrets from `esphome/secrets.yaml` (gitignored)

**Zigbee2MQTT** (in `zigbee2mqtt/`):
- Configuration in `configuration.yaml`
- Network keys and PAN IDs configured
- State files and coordinator backups are gitignored

**Go2RTC** (video streaming):
- Configuration in `go2rtc.yaml`
- Aqara G4 doorbell via HomeKit protocol with audio transcoding

## Development Workflow

### Validation and Reloading

Since the `ha` CLI is not available in this environment, use the Home Assistant UI or restart the container:

**For automation/script/template changes:**
- Use Developer Tools → YAML → Reload Automations/Scripts/Template Entities
- Or call service: `homeassistant.reload_all`

**For dashboard changes (`ui-lovelace.yaml`):**
- Just refresh the browser - Lovelace YAML mode auto-reloads

**For `configuration.yaml` changes:**
- Requires full Home Assistant restart
- Use Developer Tools → YAML → Check Configuration first
- Then Developer Tools → Server Controls → Restart

### SSH Access

**IMPORTANT: Always use `ssh ha-local` for all operations (fallback to `ssh ha` only if local unavailable)**

SSH access is available via two saved configs:
- `ssh ha-local` - **Preferred when on local network** (faster, more reliable)
- `ssh ha` - Fallback when remote or local connection unavailable

The SSH connection provides direct access to the Home Assistant host for file editing and system management.

**Note**: Save temporary files to current directory (`/Users/gcamp/Dev/home-assistant`) instead of `/tmp` for better organization.

### Git Workflow

**CRITICAL: Always commit before making changes!**

The `/config/` directory on the Home Assistant host is a git repository backed up to GitHub at `git@github.com:gcamp/home-assistant-config.git`.

**Before making ANY configuration changes, ALWAYS:**
1. Create pre-change commit: `ssh ha-local "cd /config && git add -A && git commit -m 'Pre-Claude: <description of planned changes>' && git push"`

This is NON-NEGOTIABLE and must be done BEFORE any edits, even during planning phase.

**After Claude makes changes:**
1. Review changes: `ssh ha-local "cd /config && git diff"`
2. Commit if satisfied: `ssh ha-local "cd /config && git add -A && git commit -m 'Claude: <description of changes made>' && git push"`
3. Revert if needed: `ssh ha-local "cd /config && git reset --hard HEAD~1 && git push --force"`

**Useful Git Commands:**
- View recent changes: `ssh ha-local "cd /config && git log --oneline -10"`
- See what changed: `ssh ha-local "cd /config && git diff HEAD~1"`
- Revert to specific commit: `ssh ha-local "cd /config && git reset --hard <commit-hash> && git push --force"`
- Create experimental branch: `ssh ha-local "cd /config && git checkout -b experiment"`
- Return to main: `ssh ha-local "cd /config && git checkout main"`

**`.gitignore`:**
The following files are excluded from version control:
- `secrets.yaml` (sensitive credentials)
- `.storage/` (internal HA state)
- `*.db*` (databases)
- `*.log` (logs)
- `*.bak`, `*.backup_*` (manual backups)

## Important Notes

### Security Considerations

- **Never commit secrets**: `secrets.yaml` files are gitignored
- The `configuration.yaml` contains an InfluxDB token that should be moved to `secrets.yaml`
- ESPHome API keys and OTA passwords should remain in the gitignored `esphome/secrets.yaml`
- Zigbee2MQTT network keys are in plain YAML and should ideally be moved to secrets

### Templating and French Localization

- Hilo integration heavily uses French language templates
- Défi Hilo cards use custom decluttering templates for challenge events
- Template sensors use Home Assistant's Jinja2 templating

### HACS Resources

All Lovelace resources are loaded from `/hacsfiles/` and defined in `configuration.yaml`:
- When adding new cards, add the resource URL to `configuration.yaml` under `lovelace.resources`
- Resources use `?hacstag=` for cache busting

### Modbus Integration

The EG4 inverter Modbus configuration is currently commented out in `configuration.yaml` (line 102).
To enable: uncomment `modbus: !include modbus.yml`

### Package Structure

The configuration uses `packages: !include_dir_merge_named packages/` but the directory doesn't currently exist. This allows splitting configuration into logical packages when needed.

## Common Entity Patterns

- **Hilo sensors**: `sensor.defi_hilo*`, `sensor.hilo_*`
- **Tesla sensors**: `sensor.tesla_*` (MQTT from TeslaMate)
- **Thermal comfort**: Uses `sensor.thermal_comfort_*` from thermal_comfort integration
- **Indoor air quality**: Template sensors with scoring system (temperature_score, humidity_score, co2_score)
- **ERV**: Energy Recovery Ventilator state determined by power consumption ranges

## Architecture Notes

This is a production Home Assistant instance with:
- Database: SQLite (`home-assistant_v2.db`) with WAL mode
- Running on Home Assistant OS (Linux kernel 6.12.47-haos-raspi)
- Platform: Raspberry Pi (based on kernel name)
- Version tracking in `.HA_VERSION` file
