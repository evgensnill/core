# Config Flow Platform Knowledge

## Overview
Config flows handle the UI-based configuration of integrations in Home Assistant.
They allow users to set up integrations through the frontend without editing YAML.

## Key Classes

### `ConfigFlow`
- Inherits from `homeassistant.config_entries.ConfigFlow`
- Must define `DOMAIN` class variable matching the integration domain
- Must implement `async_step_user` as the entry point for user-initiated setup
- Use `self.async_show_form()` to display forms to the user
- Use `self.async_create_entry()` to finalize and save the config entry
- Use `self.async_abort()` to cancel setup with a reason key

### `OptionsFlow`
- Handles reconfiguration of an existing config entry
- Registered via `async_get_options_flow` class method on `ConfigFlow`
- Receives the existing `config_entry` in `__init__`

## Common Patterns

### Preventing Duplicate Entries
```python
await self.async_set_unique_id(unique_id)
self._abort_if_unique_id_configured()
```

### Validating User Input
- Catch exceptions from the integration's API client
- Return `errors` dict to `async_show_form` to display inline errors
- Common error keys: `"base"`, `"cannot_connect"`, `"invalid_auth"`, `"unknown"`

### Schema Definition
```python
import voluptuous as vol
from homeassistant.helpers.selector import selector

SCHEMA = vol.Schema({
    vol.Required("host"): str,
    vol.Optional("port", default=8080): int,
})
```

## Quality Scale Requirements
- `config-flow`: Integration must have a config flow (`config_flow.py`)
- `unique-config-entry`: Must prevent duplicate entries using unique IDs
- `reauthentication-flow`: Implement `async_step_reauth` for expired credentials
- `reconfiguration-flow`: Implement `async_step_reconfigure` to update settings

## Translation Keys
All user-facing strings must be in `strings.json` under `config.step.<step_id>`.
Errors go under `config.error.<error_key>`.
Abort reasons go under `config.abort.<reason_key>`.
