# Binary Sensor Platform

Binary sensors represent entities that can only be in one of two states: `on` or `off`.

## Key Concepts

- Extend `BinarySensorEntity` from `homeassistant.components.binary_sensor`
- Override `is_on` property (returns `bool | None`)
- Use `BinarySensorDeviceClass` enum for semantic meaning
- State is `STATE_ON` or `STATE_OFF` (never a numeric value)

## Minimal Example

```python
from homeassistant.components.binary_sensor import (
    BinarySensorDeviceClass,
    BinarySensorEntity,
)
from homeassistant.core import HomeAssistant
from homeassistant.helpers.entity_platform import AddEntitiesCallback

from . import MyConfigEntry
from .coordinator import MyCoordinator


async def async_setup_entry(
    hass: HomeAssistant,
    entry: MyConfigEntry,
    async_add_entities: AddEntitiesCallback,
) -> None:
    """Set up binary sensors from a config entry."""
    coordinator: MyCoordinator = entry.runtime_data
    async_add_entities(
        MyBinarySensor(coordinator, description)
        for description in BINARY_SENSOR_DESCRIPTIONS
    )


class MyBinarySensor(CoordinatorEntity[MyCoordinator], BinarySensorEntity):
    """Representation of a binary sensor."""

    _attr_has_entity_name = True

    def __init__(self, coordinator: MyCoordinator, description: BinarySensorEntityDescription) -> None:
        """Initialize the binary sensor."""
        super().__init__(coordinator)
        self.entity_description = description
        self._attr_unique_id = f"{coordinator.config_entry.entry_id}_{description.key}"
        self._attr_device_info = coordinator.device_info

    @property
    def is_on(self) -> bool | None:
        """Return true if the binary sensor is on."""
        return self.coordinator.data.get(self.entity_description.key)
```

## Common Device Classes

| DeviceClass | Meaning |
|---|---|
| `BATTERY` | Low battery indicator |
| `CONNECTIVITY` | Connected / Disconnected |
| `DOOR` | Open / Closed |
| `MOTION` | Motion detected / Clear |
| `OCCUPANCY` | Occupied / Clear |
| `PLUG` | Plugged in / Unplugged |
| `PROBLEM` | Problem detected / OK |
| `SMOKE` | Smoke detected / Clear |
| `UPDATE` | Update available / Up-to-date |
| `WINDOW` | Open / Closed |

## Notes

- Always use `CoordinatorEntity` when data comes from a `DataUpdateCoordinator`
- Prefer `BinarySensorEntityDescription` dataclass for multi-sensor platforms
- `is_on` returning `None` means the state is unknown
- Do **not** override `state` directly; override `is_on` instead
