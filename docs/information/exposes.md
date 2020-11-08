---
---
# Exposes
Zigbee2MQTT exposes the device capabilities through the `exposes` property in [`zigbee2mqtt/bridge/devices`](./mqtt_topics_and_message_structure.md). The possible `exposes` are documented here.

There are two types of exposes:
- Generic: types like `numeric` and `binary`
- Specific: represents a specific capability of a device like a `light` or `switch`.

Both types will always have a `type` property. The generic types (except composite) will always have an `access` property. All generic types will always have a `name` property indicating the context. All generic types will always have a `property` type indicating where the value is exposed on, usually this is equal to the name but in case when the `endpoint` is defined it is `name_endpoint`. The specific and the generic composite type will always have a `features` property, this is an array containing the generic exposes types. Optionally both types can have a property `endpoint`, indicating the device exposes this capability on a specific endpoint.

### Access
The `access` property is a 3-bit bitmask.
- Bit 1: The property can be found in the published state of this device.
- Bit 2: The property can be set with a `/set` command
- Bit 3: The property can be retrieved with a `/get` command (when this bit is true, bit 1 will also be true)

Examples:
- A Xiaomi WSDCGQ01LM climate sensor exposes a numeric temperature sensor. Since the device is sleeping most of the time it cannot be retrieved with a `/get` command. Access will be `1` (binary: `0b001`).
- A Philips 7146060PH Hue Go light exposes brightness. This can be `/get`, `/set` and is also in the published state. Access will be `7` (binary: `0b111`)
- A Philips 7146060PH Hue Go light exposes effect (e.g. to trigger a flashing effect). This can only be `/set`. Access will be `2` (binary: `0b010`)
- A Xiaomi ZNCZ02LM power plug exposes a numeric power sensor. This can be `/get` and is published in the state. Access will be `5` (binary: `0b101`)

## Generic

### Binary
Indicates a device exposes a binary value. Always has `value_on` and `value_off` which indicates how to interpet the value. Optionally has a `value_toggle` which can be send to toggle the value.

Examples:
- `{"type":"binary","name":"occupancy","property":"occupancy","value_on":true,"value_off":false,"access":"r"}`
- `{"type":"binary","name":"state","property":"state","value_on":"ON","value_off":"OFF","value_toggle":"TOGGLE","access":"rw"}`

### Numeric
Indicates a device exposes a numeric value. Optionally has `value_max`, `value_min`, `value_step` and `unit`.

Examples:
- `{"type":"numeric","name":"brightness","property":"brightness","value_min":0,"value_max":254,"access":"rw"}`
- `{"type":"numeric","name":"temperature","property":"temperature","unit":"°C","access":"r"}`

### Enum
Indicates a device exposes an enum value. Always has `values` indicating all possible values.

Examples:
- `{"type":"enum","name":"identify","property":"identify","values":["blink","okay"],"access":"w"}`

### Text
Indicates a device exposes a textual value.

Examples:
- `{"type":"text","name":"inserted","property":"inserted","access":"w"}`

### Composite
Composite combines the above generic types in the `features` array. Unlike other generic types, it does not have the `access` property since the `access` is defined per feature.

Example:
```json
{
    "type":"composite",
    "name":"color_xy",
    "property":"color",
    "features": [
        {"type":"numeric","name":"x","property":"x","access":"rw"},
        {"type":"numeric","name":"y","property":"y","access":"rw"},
    ]
}
```

## Specific

### Light
Indicates a device exposes a light, possible features are `state`, `brightness`, `color_temp`, `color_xy` and `color_hs`.

Example:

```json
{
    "type": "light",
    "features": [
        {"type":"binary","name":"state","property":"state","value_on":"ON","value_off":"OFF","value_toggle":"TOGGLE","access":"rw"},
        {"type":"numeric","name":"brightness","property":"brightness","value_min":0,"value_max":254,"access":"rw"},
        {"type":"numeric","name":"color_temp","property":"color_temp","access":"rw"},
        {
            "type":"composite",
            "name":"color_xy",
            "property":"color",
            "features": [
                {"type":"numeric","name":"x","property":"x","access":"rw"},
                {"type":"numeric","name":"y","property":"y","access":"rw"}
            ]
        },
        {
            "type":"composite",
            "name":"color_hs",
            "property":"color",
            "features": [
                {"type":"numeric","name":"hue","property":"hue","access":"rw"},
                {"type":"numeric","name":"saturation","propertsaturation":"saturation","access":"rw"}
            ]
        }
    ]
}
```

### Switch
Indicates a device exposes a switch.

Example:

```json
{
    "type": "switch",
    "features": [
        {"type":"binary","name":"state","property":"state","value_on":"ON","value_off":"OFF","value_toggle":"TOGGLE","access":"rw"}
    ]
}
```

### Fan
Indicates a device exposes a fan. Possible features are `state` and `mode`.

Example:

```json
{
    "type": "fan",
    "features": [
        {"type":"binary","name":"state","property":"fan_state","value_on":"ON","value_off":"OFF","access":"rw"},
        {"type":"enum","name":"mode","property":"fan_mode","values":["off", "low", "medium", "high", "on", "auto", "smart"],"access":"rw"}
    ]
}
```

### Cover
Indicates a device exposes a cover. Possible features are `state`, `position` and `tilt`.

Example:

```json
{
    "type": "cover",
    "features": [
        {"type":"binary","name":"state","property":"state","value_on":"OPEN","value_off":"CLOSE","access":"rw"},
        {"type":"numeric","name":"position","property":"position","value_min":0,"value_max":100,"access":"rw"},
        {"type":"numeric","name":"tilt","property":"tilt","value_min":0,"value_max":100,"access":"rw"}
    ]
}
```

### Lock
Indicates a device exposes a lock. Only possible feature is `state`.

Example:

```json
{
    "type": "lock",
    "features": [
        {"type":"binary","name":"state","property":"state","value_on":"LOCK","value_off":"UNLOCK","access":"rw"}
    ]
}
```

### Climate
Indicates this device exposes climate functionallity.
- Possible features are: `occupied_heating_setpoint`, `current_heating_setpoint`, `occupied_cooling_setpoint`, `local_temperature`, `system_mode`, `running_state`, `fan_mode`, `preset` and `away_mode`.
- Never has both `occupied_heating_setpoint` and `current_heating_setpoint`.
- Possible values for `system_mode` are `off`, `heat`, `cool`, `auto`, `dry` and `fan_only`.
- Possible values for `running_state` are `idle`, `heat`, `cool`.

Example:

```json
{
    "type": "climate",
    "features": [
        {"type":"numeric","name":"occupied_heating_setpoint","property":"occupied_heating_setpoint","value_min":7,"value_max":30,"value_step": 0.5,"access":"rw","unit":"°C"},
        {"type":"numeric","name":"occupied_cooling_setpoint","property":"occupied_heating_setpoint","value_min":7,"value_max":30,"value_step": 0.5,"access":"rw","unit":"°C"},
        {"type":"numeric","name":"local_temperature","property":"local_temperature","access":"r","unit":"°C"},
        {"type":"enum","name":"system_mode","property":"system_mode","values":["off", "auto", "heat", "cool"],"access":"rw"},
        {"type":"enum","name":"preset","property":"preset","values":["hold", "program"],"access":"rw"},
        {"type":"enum","name":"running_state","property":"running_state","values":["idle", "heat", "cool"],"access":"r"},
        {"type":"enum","name":"mode","property":"fan_mode","values":["off", "low", "medium", "high", "on", "auto", "smart"],"access":"rw"},
        {"type":"binary","name":"away_mode","property":"away_mode","value_on":"ON","value_off":"OFF","access":"rw"},
    ]
}
```