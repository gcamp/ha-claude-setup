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
- For direct file editing, use SSH with the saved config: `ssh ha-local` (primary) or `ssh ha` (fallback)
- Configuration files are located in `/config/` on the Home Assistant host
- Key configuration files:
  - `/config/automations.yaml` - Automation definitions
  - `/config/configuration.yaml` - Main configuration
  - `/config/scenes.yaml` - Scene definitions
  - `/config/scripts.yaml` - Script definitions
  - `/config/sensors.yaml` - Sensor configurations
- Changes to configuration files typically require restarting Home Assistant or reloading the specific integration

## hass-cli Usage

The `hass-cli` tool can be used to interact with the Home Assistant instance. Use `hass-cli COMMAND --help` for detailed help on any command.

### Common Commands

**Service Commands:**
- `hass-cli service call <domain.service>` - Call a service (e.g., `automation.reload`, `homeassistant.reload_core_config`)

**State Commands:**
- `hass-cli state get <entity_id>` - Get specific entity state

## SSH Access

**IMPORTANT: Always use `ssh ha-local` as the default for all commands.**

SSH access is available via two saved configs:
- `ssh ha-local` - **PRIMARY** - Use this by default (faster, more reliable on local network)
- `ssh ha` - Fallback only when ha-local is unavailable

The SSH connection provides direct access to the Home Assistant host for file editing and system management.

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
- lovelace-valetudo-map-card
- lovelace-mushroom (heavily used)
- mini-graph-card (heavily used)
- lovelace-card-mod (for styling/animations)
- decluttering-card (for Hilo defi templates)
- stack-in-card
- bar-card
- config-template-card
- And 5 more...

**Important**: Resources MUST be defined in `configuration.yaml` when using YAML mode. Previously attempted YAML mode failed because resources were not explicitly defined.

### Converting from UI Mode to YAML Mode

If you need to convert again or help someone else:

1. **Export from UI**: Get YAML from Raw Configuration Editor
2. **Create ui-lovelace.yaml**: Save exported YAML to `/config/ui-lovelace.yaml`
3. **Define resources**: Add `lovelace:` block to `configuration.yaml` with all resources from `.storage/lovelace_resources`
4. **Reload config**: `hass-cli service call homeassistant.reload_core_config`
5. **Test thoroughly**: Verify all views, custom cards, and decluttering templates work

### Dashboard Layout

The dashboard uses the **sections layout** (vs. masonry). Sections organize cards into titled groups for better organization. Compatible with YAML mode.

## Configuration Best Practices

- **Always validate before restarting**: `ssh ha-local "ha core check"` then `ssh ha-local "ha core restart"` if valid
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
ssh ha-local "ha core check"

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

**Automation Helper** (`/config/automation-helper.sh`)
- Extract: `ssh ha-local "/config/automation-helper.sh extract '<id_or_alias>'"`
- List: `ssh ha-local "/config/automation-helper.sh list"`
- Merge: `ssh ha-local "/config/automation-helper.sh merge '<id_or_alias>'"`

**Use extract/merge when:** Completely rewriting complex automation (100+ lines) or testing YAML syntax in isolation
**Use direct edit when:** Small, surgical changes (90% of cases)

**Notes:**
- ✅ GUI editing still works! Single-file approach preserves UI editing
- **Always use `ssh ha-local` by default** (prefer over `ssh ha` to avoid connection errors)
- Extracted automations go to `/tmp/automation-extract/` (temporary)

### File Transfer Best Practices

When copying files from HA to local for editing:

**ALWAYS use this pattern to avoid corrupting files:**
```bash
# Good: Direct file copy to workspace subdirectory
ssh ha-local 'cat /config/automations.yaml' | tee workspace/automations.yaml > /dev/null

# Bad: Redirecting stderr can mix error messages into file
ssh ha 'cat /config/automations.yaml' > workspace/automations.yaml 2>&1
```

**Key points:**
- **Always use `ssh ha-local` by default**, fallback to `ssh ha` only if unavailable
- Use `| tee` with `> /dev/null` to copy file content cleanly
- Never redirect stderr (`2>&1`) when capturing file contents - error messages will corrupt the file
- Save to `workspace/` subdirectory (gitignored) instead of `/tmp`
- Always verify the first line after copying: `head -1 workspace/automations.yaml`
- If first line contains error text, the file is corrupted and must be re-fetched

## Labels

Labels are used to categorize and filter entities (including automations) in the Home Assistant UI.

### How Labels Work

- Labels are **NOT** stored in YAML files (automations.yaml, etc.)
- Labels are stored in the entity registry at `/config/.storage/core.entity_registry`
- The entity registry is a single-line JSON file
- Labels can only be managed through:
  1. The Home Assistant UI (Settings → Automations → Edit automation → Labels section)
  2. Direct editing of `/config/.storage/core.entity_registry` (**HA must be stopped**)

### Label Registry

Available labels are defined in `/config/.storage/core.label_registry`:
```json
{
  "labels": [
    {"label_id": "tesla", "name": "Tesla", "color": "red", "icon": "mdi:car"},
    {"label_id": "hvac", "name": "HVAC", "color": "green", "icon": "mdi:fan"},
    {"label_id": "notif", "name": "Notif", "color": "blue", "icon": "mdi:bell"},
    {"label_id": "helper", "name": "Helper", "color": "black", "icon": "mdi:progress-helper"},
    {"label_id": "light", "name": "light", "color": "white", "icon": "mdi:ceiling-light"}
  ]
}
```

### Adding Labels to Automations

**Via UI (Recommended):**
1. Go to Settings → Automations & Scenes
2. Click on the automation
3. Click the edit icon
4. Scroll to the Labels section at the bottom
5. Select or create labels

**Via Entity Registry (Advanced - CRITICAL STEPS):**

**IMPORTANT**: Home Assistant overwrites the entity registry on startup, so you MUST stop HA first!

```bash
# 1. STOP Home Assistant first
ssh ha-local "ha core stop"

# 2. Update the entity registry with jq
ssh ha-local 'jq ".data.entities |= map(if .platform == \"automation\" and (.unique_id | IN(\"1732300000001\", \"1732300000002\")) then .labels = [\"light\"] else . end)" /config/.storage/core.entity_registry > /tmp/updated_registry.json && cat /tmp/updated_registry.json > /config/.storage/core.entity_registry'

# 3. Verify labels were applied
ssh ha-local "cat /config/.storage/core.entity_registry" | jq -c '.data.entities[] | select(.unique_id == "1732300000001") | {entity_id, labels}'

# 4. Start Home Assistant
ssh ha-local "ha core start"

# 5. Wait ~30 seconds, then verify labels persisted
sleep 30 && ssh ha-local "cat /config/.storage/core.entity_registry" | jq -c '.data.entities[] | select(.unique_id == "1732300000001") | {entity_id, labels}'
```

Example entity registry entry:
```json
{
  "entity_id": "automation.porch_background_light_at_sunset",
  "unique_id": "1732300000001",
  "platform": "automation",
  "labels": ["light"],
  ...
}
```

**Important Notes:**
- **NEVER edit entity registry while HA is running** - changes will be overwritten
- Always stop HA first: `ssh ha-local "ha core stop"`
- The entity registry is gitignored (in `.storage/`)
- Labels must exist in `core.label_registry` before being assigned
- Use `jq` to safely edit the JSON (preserves format)
- After HA starts, hard refresh browser (Ctrl+Shift+R) to see labels in UI
- use actual random ids for automation ids, not increment of existing ones
- always validate the entities id you're adding in automations.