---
---
# Exposes
Zigbee2MQTT exposes the device capabilities through the `exposes` property in [`zigbee2mqtt/bridge/devices`](./mqtt_topics_and_message_structure.md). The possible `exposes` are documented here.

There are two types of exposes:
- Generic: types like `numeric` and `binary`
- Specific: represents a specific capability of a device like a `light` or `switch`.

Both types will always have a `type` property. The generic types (except composite) will always have an `access` property, indicating if the property can be read (`r`), written (`w`) or both (`rw`). All generic types will always have a `name` property indicating the context. All generic types will always have a `property` type indicating where the value is exposed on, usually this is equal to the name but in case when the `endpoint` is defined it is `name_endpoint`. The specific and the generic composite type will always have a `features` property, this is an array containing the generic exposes types. Optionally both types can have a property `endpoint`, indicating the device exposes this capability on a specific endpoint.

## Generic

### Binary
Indicates a device exposes a binary value. Always has `value_on` and `value_off` which indicates how to interpet the value. Optionally has a `value_toggle` which can be send to toggle the value.

Examples:
- `{"type":"binary","name":"occupancy","property":"occupancy","value_on":true,"value_off":false,"access":"r"}`
- `{"type":"binary","name":"state","property":"state","value_on":"ON","value_off":"OFF","value_toggle":"TOGGLE","access":"rw"}`

### Numeric
Indicates a device exposes a numeric value. Opionally has `value_max`, `value_min` and `unit`.

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
Indicates a device exposes a lock.

Example:

```json
{
    "type": "lock",
    "features": [
        {"type":"binary","name":"state","property":"state","value_on":"LOCK","value_off":"UNLOCK","access":"rw"}
    ]
}
```