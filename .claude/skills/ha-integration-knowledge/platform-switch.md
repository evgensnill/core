# Home Assistant Switch Platform

Switch entities represent toggleable devices (on/off state).

## Base Class

`homeassistant.components.switch.SwitchEntity`

## Key Properties & Methods

| Name | Type | Description |
|------|------|-------------|
| `is_on` | `bool \| None` | Current state of the switch |
| `turn_on(**kwargs)` | `async def` | Turn the switch on |
| `turn_off(**kwargs)` | `async def` | Turn the switch off |
| `toggle(**kwargs)` | `async def` | Toggle the switch (default impl uses turn_on/turn_off) |

## Example

```python
from homeassistant.components.switch import SwitchEntity
from homeassistant.config_entries import ConfigEntry
from homeassistant.core import HomeAssistant
from homeassistant.helpers.entity_platform import AddEntitiesCallback

from .const import DOMAIN
from .coordinator import MyCoordinator


async def async_setup_entry(
    hass: HomeAssistant,
    entry: ConfigEntry,
    async_add_entities: AddEntitiesCallback,
) -> None:
    """Set up My Integration switches from a config entry."""
    coordinator: MyCoordinator = hass.data[DOMAIN][entry.entry_id]
    async_add_entities(
        MySwitch(coordinator, device_id)
        for device_id in coordinator.data
    )


class MySwitch(CoordinatorEntity[MyCoordinator], SwitchEntity):
    """Representation of a My Integration switch."""

    _attr_has_entity_name = True

    def __init__(self, coordinator: MyCoordinator, device_id: str) -> None:
        """Initialize the switch."""
        super().__init__(coordinator)
        self._device_id = device_id
        self._attr_unique_id = f"{device_id}_switch"

    @property
    def is_on(self) -> bool | None:
        """Return true if the switch is on."""
        device = self.coordinator.data.get(self._device_id)
        if device is None:
            return None
        return device["state"] == "on"

    async def async_turn_on(self, **kwargs) -> None:
        """Turn the switch on."""
        await self.coordinator.api.set_state(self._device_id, True)
        await self.coordinator.async_request_refresh()

    async def async_turn_off(self, **kwargs) -> None:
        """Turn the switch off."""
        await self.coordinator.api.set_state(self._device_id, False)
        await self.coordinator.async_request_refresh()
```

## Notes

- Always implement both `async_turn_on` and `async_turn_off`.
- Request a coordinator refresh after state changes to keep UI in sync.
- Use `_attr_has_entity_name = True` with a `device_info` property for proper device grouping.
- For switches that are slow to respond, consider optimistic mode with `_attr_assumed_state = True`.
