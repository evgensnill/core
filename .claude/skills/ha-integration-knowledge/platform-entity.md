# Home Assistant Entity Platform

## Overview
Entities are the core building blocks of Home Assistant integrations. They represent devices or services and expose their state and attributes.

## Base Entity Class
All entities inherit from `homeassistant.helpers.entity.Entity` or a domain-specific base class (e.g., `SensorEntity`, `BinarySensorEntity`, `SwitchEntity`).

## Key Properties

### Required
- `unique_id`: A unique identifier for the entity within the integration. Should be stable across restarts.
- `name`: Human-readable name. Prefer using `_attr_name` or `_attr_has_entity_name = True` with `_attr_translation_key`.

### Recommended
- `device_info`: Links the entity to a device in the device registry.
- `available`: Whether the entity is currently available (default: `True`).
- `should_poll`: Whether HA should poll for updates (default: `True`). Set to `False` for push-based updates.

## State Management

### Polling
```python
class MyEntity(SensorEntity):
    def update(self) -> None:
        """Fetch new state data."""
        self._attr_native_value = self._client.get_value()
```

### Push-based (Coordinator)
```python
class MyEntity(CoordinatorEntity[MyCoordinator], SensorEntity):
    def __init__(self, coordinator: MyCoordinator, entry: ConfigEntry) -> None:
        super().__init__(coordinator)
        self._attr_unique_id = entry.entry_id

    @property
    def native_value(self) -> StateType:
        return self.coordinator.data["value"]
```

## Entity Naming
Follow the `has_entity_name` pattern for modern integrations:
```python
_attr_has_entity_name = True
_attr_translation_key = "temperature"
```
Translations go in `strings.json` under `entity.<domain>.<translation_key>.name`.

## Device Info
```python
@property
def device_info(self) -> DeviceInfo:
    return DeviceInfo(
        identifiers={(DOMAIN, self._device_id)},
        name=self._device_name,
        manufacturer="Acme Corp",
        model="Widget 2000",
        sw_version=self._firmware_version,
    )
```

## Quality Scale Considerations
- Use `_attr_*` class variables instead of properties where possible
- Implement `available` property to reflect connectivity state
- Use `DataUpdateCoordinator` for shared data fetching
- Avoid blocking I/O in entity methods; use `async_update` for async integrations
- Register entities via `async_setup_entry` and `async_add_entities`
