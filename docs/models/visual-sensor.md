# VisualSensor

Visual or electro-optical sensor (eye, TV camera). Its detections depend on the visual signature of the target.

| | |
|---|---|
| Kind | Equipment definition (`EquipmentDefinition`), `type="Visual"` |
| Defined in | [`library/equipments/sensors.xml`](../../library/equipments/sensors.xml) |
| Base | [`Sensor`](sensor.md) |
| Derivation chain | [`Sensor`](sensor.md) → [`VisualSensor`](visual-sensor.md) |
| Derived in the standard library | — |

## Role

`VisualSensor` is a visual or electro-optical sensor in the visible spectrum: the crew's eyes, a TV
camera, a low-light camera. It detects targets through the `Visual` signature of
[`AircraftSignature`](aircraft-signature.md) and emits nothing.

## Technological data

Characteristics of the model, fixed for a given equipment type.

| Data | Type | Unit | Default | Constraints | Description |
|---|---|---|---|---|---|
| `NightCapable` | `HLAboolean` | — | `HLAfalse` | — | Whether the sensor works at night (low-light camera, night vision). |
| `FieldOfView` | `AngleDegrees` | deg | — | > 0, ≤ 360 | Instantaneous field of view of the sensor. |

## Initialization data

None declared by `VisualSensor` (see inherited data below).

## Inherited data

Declared by the base definitions and available in this model. A derived definition sets their default values by repeating them with the same name.

| Data | Kind | Type | Declared in |
|---|---|---|---|
| `Role` | technological | `SensorRoleEnum` | [`Sensor`](sensor.md) |
| `Generation` | technological | `SensorGenerationEnum` | [`Sensor`](sensor.md) |
| `MinRange` | technological | `DistanceMeters` | [`Sensor`](sensor.md) |
| `MaxRange` | technological | `DistanceMeters` | [`Sensor`](sensor.md) |
| `MinAltitude` | technological | `DistanceMeters` | [`Sensor`](sensor.md) |
| `MaxAltitude` | technological | `DistanceMeters` | [`Sensor`](sensor.md) |
| `ScanInterval` | technological | `DurationSeconds` | [`Sensor`](sensor.md) |
| `Capabilities` | technological | `SensorCapabilities` | [`Sensor`](sensor.md) |
| `DetectionArcs` | technological | `SensorArcs` | [`Sensor`](sensor.md) |
| `MaxContacts` | technological | `Count` | [`Sensor`](sensor.md) |
| `Operational` | initialization | `HLAboolean` | [`Sensor`](sensor.md) |

## Behaviour

- **`NightCapable`** tells whether the sensor still works at night (low-light camera, night
  vision); by default it does not.
- **`FieldOfView`** is the instantaneous field of view, in degrees; the volume covered over time
  is given by the `DetectionArcs` inherited from [`Sensor`](sensor.md).
- Detection range is the target's `Visual` signature for the seen aspect (a daylight range),
  scaled by the sensor `Generation`, and limited by `MaxRange`.

## Dynamic behaviour (pseudo-code)

Reference behaviour of the model during a simulation, written in the pseudo-code described in [Model interactions](interactions.md#pseudo-code-conventions). `data.X` is the resolved value of the data item `X`; functions and constants not given by the archetype (turn rate, detection threshold, weather...) belong to the simulation.

- **Reads:** its own kinematic; the kinematic and `AircraftSignature` (`Visual`) of each target.
- **Read by:** the tasks and triggers of its entity.

```text
init(entity, data)
    Sensor.init(entity, data)

detect(target) -> bool
    sig = target.equipmentOfType(Signature)
    if sig is none: return false
    Rref = sig.visualRange(owner.kinematic.getPosition())          # daylight range for the seen aspect
    Rdet = Rref * generationFactor(data.Generation)
    if isNight(world.time, owner.position)
        Rdet = data.NightCapable ? Rdet * nightFactor : 0           # simulation constant
    Rdet = Rdet * visibilityFactor(weather)                         # fog, clouds, rain
    return rangeTo(target) <= min(Rdet, data.MaxRange)
```

## Constraints and rules

- 0 < `FieldOfView` ≤ 360.
- Kind of equipment: `type="Visual"`.

## Deriving a concrete model

No example library derives from it yet. A crew's eyes could be described as:

```xml
<EquipmentDefinition name="PilotEyes" base="VisualSensor">
  <Description>Eyes of the pilot, through the canopy.</Description>
  <TechnologicalData>
    <Parameter name="Role" dataType="SensorRoleEnum"><DefaultValue>AirSearch</DefaultValue></Parameter>
    <Parameter name="MaxRange" dataType="DistanceMeters"><DefaultValue>20000</DefaultValue></Parameter>
    <Parameter name="FieldOfView" dataType="AngleDegrees"><DefaultValue>120</DefaultValue></Parameter>
  </TechnologicalData>
</EquipmentDefinition>
```

## CMO mapping

See [CMO mapping](../cmo-mapping.md#infrared-and-visual) for the CMO database field behind each data item.
