# InfraredSensor

Passive infrared sensor (IRST, FLIR, missile approach warner). Its detections depend on the infrared signature of the target.

| | |
|---|---|
| Kind | Equipment definition (`EquipmentDefinition`), `type="Infrared"` |
| Defined in | [`library/equipments/sensors.xml`](../../library/equipments/sensors.xml) |
| Base | [`Sensor`](sensor.md) |
| Derivation chain | [`Sensor`](sensor.md) → [`InfraredSensor`](infrared-sensor.md) |
| Derived in the standard library | — |

## Role

`InfraredSensor` is a passive infrared sensor: infrared search and track (IRST), forward-looking
infrared (FLIR), targeting pod channel or missile approach warner. It detects targets through the
`Infrared` signature of [`AircraftSignature`](aircraft-signature.md) and emits nothing, so it
cannot be intercepted.

## Technological data

Characteristics of the model, fixed for a given equipment type.

| Data | Type | Unit | Default | Constraints | Description |
|---|---|---|---|---|---|
| `Band` | `InfraredBandEnum` | — | `MidWave` | — | Infrared band of the detector. |
| `Imaging` | `HLAboolean` | — | `HLAfalse` | — | Whether the sensor forms an image, which allows classifying contacts. |
| `FieldOfView` | `AngleDegrees` | deg | — | > 0, ≤ 360 | Instantaneous field of view of the detector. |

## Initialization data

None declared by `InfraredSensor` (see inherited data below).

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

- **`Band`** is the detector band: near infrared, short-wave, mid-wave (default), long-wave, or
  dual band (mid and long wave).
- **`Imaging`** tells whether the sensor forms an image, which allows classifying contacts
  (default false: a point detector).
- **`FieldOfView`** is the instantaneous field of view of the detector, in degrees; the volume
  covered over time is given by the `DetectionArcs` inherited from [`Sensor`](sensor.md).
- Detection range is the target's `Infrared` signature for the seen aspect, scaled by the sensor
  `Generation`, and limited by `MaxRange`.

## Constraints and rules

- 0 < `FieldOfView` ≤ 360.
- Kind of equipment: `type="Infrared"`.

## Deriving a concrete model

Excerpt of `examples/library/equipments-rafale.xml`:

```xml
<EquipmentDefinition name="OSFIrst" base="InfraredSensor">
  <Description>Infrared channel of the Thales Optronique Secteur Frontal (OSF) in front of the canopy: passive search and track with imaging; covers a front sector.</Description>
  <TechnologicalData>
    <Parameter name="Role" dataType="SensorRoleEnum"><DefaultValue>InfraredSearchAndTrack</DefaultValue></Parameter>
    <Parameter name="MaxRange" dataType="DistanceMeters"><DefaultValue>90000</DefaultValue></Parameter>
    <Parameter name="Imaging" dataType="HLAboolean"><DefaultValue>HLAtrue</DefaultValue></Parameter>
  </TechnologicalData>
</EquipmentDefinition>
```

## Examples in this repository

- `AAQ37DAS` ([`examples/library/equipments-f35.xml`](../../examples/library/equipments-f35.xml)): Northrop Grumman AN/AAQ-37 Distributed Aperture System: six infrared cameras giving spherical coverage, for missile warning and situational awareness.
- `AAQ40EOTS` ([`examples/library/equipments-f35.xml`](../../examples/library/equipments-f35.xml)): Lockheed Martin AN/AAQ-40 Electro-Optical Targeting System under the nose: FLIR and infrared search and track, mainly looking forward and down.
- `SpectraMissileWarner` ([`examples/library/equipments-rafale.xml`](../../examples/library/equipments-rafale.xml)): Infrared missile approach warner of the SPECTRA suite, covering all around the aircraft.
- `OSFIrst` ([`examples/library/equipments-rafale.xml`](../../examples/library/equipments-rafale.xml)): Infrared channel of the Thales Optronique Secteur Frontal (OSF) in front of the canopy: passive search and track with imaging; covers a front sector.
- `FighterIRST` ([`examples/library/equipments.xml`](../../examples/library/equipments.xml)): Imaging infrared search and track in the nose, covering a 180 degree front sector.

## CMO mapping

See [CMO mapping](../cmo-mapping.md#infrared-and-visual) for the CMO database field behind each data item.
