# Light Platform

The light platform allows you to control light entities in Home Assistant.

## Key Attributes

- `is_on`: Whether the light is currently on
- `brightness`: Current brightness (0-255)
- `color_temp_kelvin`: Color temperature in Kelvin
- `rgb_color`: RGB color tuple (r, g, b)
- `hs_color`: Hue/Saturation color tuple
- `supported_color_modes`: Set of supported `ColorMode` values
- `color_mode`: Current active `ColorMode`

## Color Modes

Use `ColorMode` enum from `homeassistant.components.light`:
- `ColorMode.ONOFF` — on/off only
- `ColorMode.BRIGHTNESS` — brightness control
- `ColorMode.COLOR_TEMP` — color temperature
- `ColorMode.RGB` — RGB color
- `ColorMode.HS` — Hue/Saturation

## Example

```python
from homeassistant.components.light import (
    ATTR_BRIGHTNESS,
    ATTR_RGB_COLOR,
    ColorMode,
    LightEntity,
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
    """Set up My Integration lights from a config entry."""
    coordinator: MyCoordinator = entry.runtime_data
    async_add_entities(
        MyLight(coordinator, device_id)
        for device_id in coordinator.data["lights"]
    )


class MyLight(CoordinatorEntity[MyCoordinator], LightEntity):
    """Representation of a My Integration light."""

    _attr_has_entity_name = True
    _attr_supported_color_modes = {ColorMode.RGB, ColorMode.BRIGHTNESS}
    _attr_color_mode = ColorMode.RGB

    def __init__(self, coordinator: MyCoordinator, device_id: str) -> None:
        """Initialize the light."""
        super().__init__(coordinator)
        self._device_id = device_id
        self._attr_unique_id = f"{device_id}_light"

    @property
    def is_on(self) -> bool:
        """Return true if the light is on."""
        return self.coordinator.data["lights"][self._device_id]["state"] == "on"

    @property
    def brightness(self) -> int | None:
        """Return the current brightness."""
        return self.coordinator.data["lights"][self._device_id].get("brightness")

    @property
    def rgb_color(self) -> tuple[int, int, int] | None:
        """Return the current RGB color."""
        rgb = self.coordinator.data["lights"][self._device_id].get("rgb")
        if rgb:
            return (rgb["r"], rgb["g"], rgb["b"])
        return None

    async def async_turn_on(self, **kwargs: Any) -> None:
        """Turn the light on."""
        brightness = kwargs.get(ATTR_BRIGHTNESS)
        rgb = kwargs.get(ATTR_RGB_COLOR)
        await self.coordinator.client.turn_on_light(
            self._device_id, brightness=brightness, rgb=rgb
        )
        await self.coordinator.async_request_refresh()

    async def async_turn_off(self, **kwargs: Any) -> None:
        """Turn the light off."""
        await self.coordinator.client.turn_off_light(self._device_id)
        await self.coordinator.async_request_refresh()
```

## Notes

- Always declare `supported_color_modes` — it is required for all light entities.
- `color_mode` must reflect the **currently active** mode, not all supported modes.
- Use `CoordinatorEntity` for polling-based integrations.
- Import `Any` from `typing` when using `**kwargs: Any`.
