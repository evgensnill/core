# Sensor Platform

Sensor entities represent read-only values from devices or services.

## Base Class

`SensorEntity` from `homeassistant.components.sensor`

## Key Properties

- `native_value`: The current sensor value (required)
- `native_unit_of_measurement`: Unit string (e.g. `UnitOfTemperature.CELSIUS`)
- `device_class`: `SensorDeviceClass` enum value (e.g. `TEMPERATURE`, `HUMIDITY`)
- `state_class`: `SensorStateClass` enum value (`MEASUREMENT`, `TOTAL`, `TOTAL_INCREASING`)
- `suggested_display_precision`: Number of decimal places to show in UI

## Example

```python
from homeassistant.components.sensor import (
    SensorDeviceClass,
    SensorEntity,
    SensorEntityDescription,
    SensorStateClass,
)
from homeassistant.const import UnitOfTemperature

class MySensor(SensorEntity):
    _attr_device_class = SensorDeviceClass.TEMPERATURE
    _attr_native_unit_of_measurement = UnitOfTemperature.CELSIUS
    _attr_state_class = SensorStateClass.MEASUREMENT

    @property
    def native_value(self) -> float | None:
        return self._coordinator.data.get("temperature")
```

## Entity Descriptions

Use `SensorEntityDescription` dataclass to define sensors declaratively:

```python
SENSORS: tuple[SensorEntityDescription, ...] = (
    SensorEntityDescription(
        key="temperature",
        device_class=SensorDeviceClass.TEMPERATURE,
        native_unit_of_measurement=UnitOfTemperature.CELSIUS,
        state_class=SensorStateClass.MEASUREMENT,
    ),
)
```
