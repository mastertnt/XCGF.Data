# Sensor

Root sensor: role, generation, coverage (ranges, altitudes, arcs), scan rate and capabilities. Every sensor derives from it.

| | |
|---|---|
| Kind | Equipment definition (`EquipmentDefinition`), `type="Sensor"` |
| Defined in | [`library/equipments/sensors.xml`](../../library/equipments/sensors.xml) |
| Base | — (root definition) |
| Derivation chain | [`Sensor`](sensor.md) |
| Derived in the standard library | [`Radar`](radar.md), [`ESM`](esm.md), [`InfraredSensor`](infrared-sensor.md), [`VisualSensor`](visual-sensor.md) |

## Role

`Sensor` is the root of every sensor model. It carries what all sensors share: their role, their
technology generation, the volume they cover (ranges, target altitudes, angular sectors), how often
they look, what they detect and report, and how many contacts they follow. It is not used directly:
concrete sensors derive from one of its four specialisations, [`Radar`](radar.md), [`ESM`](esm.md),
[`InfraredSensor`](infrared-sensor.md) and [`VisualSensor`](visual-sensor.md).

## Technological data

Characteristics of the model, fixed for a given equipment type.

| Data | Type | Unit | Default | Constraints | Description |
|---|---|---|---|---|---|
| `Role` | `SensorRoleEnum` | — | `Other` | — | Main role of the sensor. |
| `Generation` | `SensorGenerationEnum` | — | — | — | Technology generation, which scales detection performance. |
| `MinRange` | `DistanceMeters` | m | `0` | ≥ 0, ≤ `MaxRange` | Shortest distance at which the sensor detects a target. |
| `MaxRange` | `DistanceMeters` | m | — | > 0 | Farthest distance at which the sensor detects a target. Targets beyond it are never detected. |
| `MinAltitude` | `DistanceMeters` | m | — | ≤ `MaxAltitude` | Lowest target altitude the sensor detects. |
| `MaxAltitude` | `DistanceMeters` | m | — | — | Highest target altitude the sensor detects. |
| `ScanInterval` | `DurationSeconds` | s | — | ≥ 0 | Time between two detection attempts on the same target. |
| `Capabilities` | `SensorCapabilities` | — | — | — | Kinds of targets detected and information reported about a contact. |
| `DetectionArcs` | `SensorArcs` | — | — | — | Sectors where the sensor detects. All around the platform when absent. |
| `MaxContacts` | `Count` | — | — | ≥ 0 | Highest number of contacts the sensor follows at once. |

## Initialization data

State of the equipment when the entity is created; the scenario usually sets it.

| Data | Type | Unit | Default | Constraints | Description |
|---|---|---|---|---|---|
| `Operational` | `HLAboolean` | — | `HLAtrue` | — | Whether the sensor works when the entity is created (false: damaged or out of service). |

## Behaviour

- **Coverage.** A target is detectable only if its distance is between `MinRange` and `MaxRange`,
  its altitude between `MinAltitude` and `MaxAltitude`, and its bearing and elevation inside one
  of the `DetectionArcs`. Without `DetectionArcs`, the sensor covers all around the platform.
- **Arcs.** A `SensorArc` is relative to the platform heading and horizontal plane: the azimuth
  sector goes clockwise from `AzimuthStart` to `AzimuthEnd` (both in [0, 360), so 300 → 60 is a
  120° front sector through the nose), and `ElevationMin`..`ElevationMax` bound the elevation.
- **`ScanInterval`** is the time between two detection attempts on the same target.
- **`Capabilities`** list what the sensor detects (air, surface, land, periscope search) and what
  it reports on a contact (range, heading, speed, altitude, classification, identification), plus
  `TargetIllumination` for weapon guidance.
- **`Generation`** (`Early1950s` to `Early2020s`) scales detection performance, notably for
  infrared and visual sensors.
- **`Operational`** (initialization) is false when the sensor is damaged or out of service when
  the entity is created.

## Constraints and rules

- 0 ≤ `MinRange` ≤ `MaxRange`, `MaxRange` > 0; `MinAltitude` ≤ `MaxAltitude`; `ScanInterval` ≥ 0;
  `MaxContacts` ≥ 0.
- Kind of equipment: `type="Sensor"`; each specialisation sets its own (`Radar`, `ESM`,
  `Infrared`, `Visual`).

## Deriving a concrete model

Derive a concrete sensor from one of the specialisations, not from `Sensor` itself. A new family of
sensors (sonar, for instance) would derive from `Sensor`:

```xml
<EquipmentDefinition name="Sonar" base="Sensor" type="Sonar">
  <Description>Underwater acoustic sensor.</Description>
  <TechnologicalData>
    <Parameter name="Active" dataType="HLAboolean">
      <Description>Whether the sonar emits pings.</Description>
      <DefaultValue>HLAfalse</DefaultValue>
    </Parameter>
  </TechnologicalData>
</EquipmentDefinition>
```

## CMO mapping

See [CMO mapping](../cmo-mapping.md#common-data-sensor) for the CMO database field behind each data item.
