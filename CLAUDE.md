# Home Assistant Configuration

---
## ⚠️ CRITICAL: GIT COMMIT WORKFLOW ⚠️

**MANDATORY: ALWAYS COMMIT AFTER ANY CONFIGURATION CHANGES!**

When Claude makes ANY changes to configuration files:
1. ✅ **ALWAYS commit immediately after changes**:
   ```bash
   ssh ha-local "cd /config && git add -A && git commit -m 'Claude: <description of changes made>' && git push"
   ```

**NO EXCEPTIONS. COMMIT EVERY TIME.**

For optional pre-change commits (recommended for complex changes):
```bash
ssh ha-local "cd /config && git add -A && git commit -m 'Pre-Claude: <description of planned changes>' && git push"
```

See [Git Version Control](#git-version-control) section for full details.

---

## Working with the Configuration

- Use `hass-cli` to connect to and interact with the Home Assistant instance
- For direct file editing, use SSH with the saved config: `ssh ha-local`
- Configuration files are located in `/config/` on the Home Assistant host
- Key configuration files:
  - `/config/automations.yaml` - Automation definitions
  - `/config/configuration.yaml` - Main configuration
  - `/config/scenes.yaml` - Scene definitions
  - `/config/scripts.yaml` - Script definitions
  - `/config/sensors.yaml` - Sensor configurations
- Changes to configuration files typically require restarting Home Assistant or reloading the specific integration

## Documentation

Use context7 MCP to get access to the latest HomeAssistant documentation.

## InfluxDB Access

Home Assistant logs sensor data to InfluxDB for historical analysis. See [INFLUXDB.md](INFLUXDB.md) for connection details and usage examples. 

## hass-cli Usage

The `hass-cli` tool can be used to interact with the Home Assistant instance. Use `hass-cli COMMAND --help` for detailed help on any command.

### Common Commands

**Service Commands:**
- `hass-cli service call <domain.service>` - Call a service (e.g., `automation.reload`, `homeassistant.reload_core_config`)

**State Commands:**
- `hass-cli state get <entity_id>` - Get specific entity state

## SSH Access

SSH access is available via `ssh ha-local` which provides direct access to the Home Assistant host for file editing and system management.

**Note**: Save temporary files to the `workspace/` subdirectory instead of `/tmp` for better organization. The `workspace/` directory is gitignored.

## Git Version Control

The `/config/` directory is a git repository backed up to GitHub at `git@github.com:gcamp/home-assistant-config.git`.

### Making Changes with Claude

**CRITICAL: MANDATORY POST-CHANGE COMMITS!**

✅ **AFTER every configuration change (MANDATORY):**
```bash
ssh ha-local "cd /config && git add -A && git commit -m 'Claude: <description of changes made>' && git push"
```

**This is REQUIRED. Claude must NEVER skip this step.**

Optional pre-change commit (recommended for complex/experimental changes):
```bash
ssh ha-local "cd /config && git add -A && git commit -m 'Pre-Claude: <description of planned changes>' && git push"
```

To review or revert changes:
1. Review changes: `ssh ha-local "cd /config && git diff"`
2. Revert if needed: `ssh ha-local "cd /config && git reset --hard HEAD~1 && git push --force"`

### Useful Git Commands

- **View recent changes**: `ssh ha-local "cd /config && git log --oneline -10"`
- **See what changed**: `ssh ha-local "cd /config && git diff HEAD~1"`
- **Revert to specific commit**: `ssh ha-local "cd /config && git reset --hard <commit-hash> && git push --force"`
- **Create experimental branch**: `ssh ha-local "cd /config && git checkout -b experiment"`
- **Return to main**: `ssh ha-local "cd /config && git checkout main"`

### .gitignore

The following files are excluded from version control:
- `secrets.yaml` (sensitive credentials)
- `.storage/` (internal HA state)
- `*.db*` (databases)
- `*.log` (logs)
- `*.bak`, `*.backup_*` (manual backups)

## Dashboard Configuration

The dashboard is configured in **YAML mode** using the sections layout.

### Dashboard Files

- `/config/ui-lovelace.yaml` - Main dashboard configuration (YAML format, version controlled)
- `/config/.storage/lovelace` - Legacy UI mode config (JSON format, gitignored - kept as backup)
- `/config/.storage/lovelace_resources` - Legacy resources (gitignored - no longer used)

### Editing the Dashboard

**Via YAML (Current Method):**
1. Edit `/config/ui-lovelace.yaml` directly via SSH
2. Reload configuration: `hass-cli service call homeassistant.reload_core_config`
3. Refresh browser to see changes
4. **⚠️ COMMIT CHANGES:** `ssh ha-local "cd /config && git add -A && git commit -m 'Claude: <description>' && git push"`

**Getting YAML from UI:**
- If you need to export current UI config to YAML, use the Raw Configuration Editor in the UI
- Dashboard → Edit → Three dots → Raw configuration editor
- Copy the YAML output

### YAML Mode Configuration

The dashboard is in YAML mode with resources explicitly defined in `configuration.yaml`:

```yaml
lovelace:
  mode: yaml
  resources:
    # All 13 HACS custom cards defined here
    - url: /hacsfiles/lovelace-mushroom/mushroom.js
      type: module
    # ... etc
```

**Key Resources:**
**Important**: Resources MUST be defined in `configuration.yaml` when using YAML mode, and important to validate they are there before adding any more requirements. 

### Dashboard Layout

The dashboard uses the **sections layout** (vs. masonry). Sections organize cards into titled groups for better organization. Compatible with YAML mode.

## Configuration Best Practices

- **Validate configuration**: Use `hass-cli service call homeassistant.check_config` to validate changes
- **Prefer reloading over restarting**: Most changes don't require a full restart - use reload services instead
- Dashboard changes (ui-lovelace.yaml) take effect immediately on browser refresh
- For automations/scripts, reload: `hass-cli service call automation.reload` or `homeassistant.reload_all`
- Always commit working states to git before making experimental changes

### Automation Best Practices

- **Add section titles**: Always add `alias` field to `choose` action sections for better readability in the UI
- **Use templates over conditions**: When toggling state and notifying, use templates in notification messages instead of multiple `choose` conditions
- **Example**:
  ```yaml
  # Good: Template in message
  - service: input_boolean.toggle
    target:
      entity_id: input_boolean.example
  - service: notify.all_apps
    data:
      message: >
        {% if is_state('input_boolean.example', 'on') %}
        Enabled
        {% else %}
        Disabled
        {% endif %}

  # Avoid: Multiple choose sections for same action
  - choose:
    - conditions:
      - condition: state
        entity_id: input_boolean.example
        state: 'on'
      sequence:
      - service: notify.all_apps
        data:
          message: Enabled
    - conditions:
      - condition: state
        entity_id: input_boolean.example
        state: 'off'
      sequence:
      - service: notify.all_apps
        data:
          message: Disabled
  ```

## Editing Automations with Claude

### Overview

Automations are stored in a single file (`/config/automations.yaml`) which can be large (1387 lines, 42 automations). To make Claude's editing more efficient, helper scripts are available to:
1. Generate a summary of all automations (without reading the full file)
2. Extract individual automations for focused editing
3. Merge edited automations back

### Workflow for Claude

**Step 1: Generate and Read Summary**

Always start by generating a summary to see all automations:
```bash
ssh ha-local "/config/automation-summary.sh"
```

This outputs a concise list of all automations with index, ID, and alias. **Much faster than reading the full 1387-line file.**

**Step 2: Edit Automations**

For most edits, directly edit the full file:
```bash
# Copy to workspace subdirectory
ssh ha-local 'cat /config/automations.yaml' | tee workspace/automations.yaml > /dev/null

# Edit using the Edit tool
# Find automation by ID or alias and make changes

# Upload back
scp workspace/automations.yaml ha-local:/config/automations.yaml
```

**Step 3: Reload and Verify**
```bash
# Validate
hass-cli service call homeassistant.check_config

# Reload automations
hass-cli service call automation.reload

# Verify specific automation
hass-cli state get automation.<entity_id>
```

**Step 4: ⚠️ COMMIT CHANGES**
```bash
ssh ha-local "cd /config && git add -A && git commit -m 'Claude: <description of automation changes>' && git push"
```

### Helper Scripts

**Automation Summary** (`/config/automation-summary.sh`)
- Generates real-time summary of all automations (outputs to stdout)
- Usage: `ssh ha-local "/config/automation-summary.sh"`

### File Transfer Best Practices

When copying files from HA to local for editing:

**ALWAYS use this pattern to avoid corrupting files:**
```bash
# Direct file copy to workspace subdirectory
ssh ha-local 'cat /config/automations.yaml' | tee workspace/automations.yaml > /dev/null
```

**Key points:**
- Use `| tee` with `> /dev/null` to copy file content cleanly
- Never redirect stderr (`2>&1`) when capturing file contents - error messages will corrupt the file
- Save to `workspace/` subdirectory (gitignored) instead of `/tmp`
- Always verify the first line after copying: `head -1 workspace/automations.yaml`
- If first line contains error text, the file is corrupted and must be re-fetched
