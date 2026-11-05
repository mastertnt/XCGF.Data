# AircraftSignature

Aircraft signature: visual, infrared and radar, per aspect.

| | |
|---|---|
| Kind | Equipment definition (`EquipmentDefinition`), `type="Signature"` (inherited) |
| Defined in | [`library/equipments/signatures.xml`](../../library/equipments/signatures.xml) |
| Base | [`Signature`](signature.md) |
| Derivation chain | [`Signature`](signature.md) → [`AircraftSignature`](aircraft-signature.md) |
| Derived in the standard library | — |

## Role

`AircraftSignature` is the signature of an aircraft, as in CMO: how far it is seen by eye and by
infrared sensors, and how large it looks to radars, each for four aspects (front, side, rear, top).
A concrete aircraft derives from it and sets the values (for instance `RafaleSignature`).

## Technological data

Characteristics of the model, fixed for a given equipment type.

| Data | Type | Unit | Default | Constraints | Description |
|---|---|---|---|---|---|
| `Visual` | `AspectDetectionRange` | — | — | ≥ 0 | Range at which the aircraft is seen by eye in daylight, per aspect. Visual sensors scale it by their generation. |
| `Infrared` | `AspectDetectionRange` | — | — | ≥ 0 | Range at which a reference infrared sensor detects the aircraft, per aspect. Infrared sensors scale it by their generation. |
| `RadarCrossSectionAToD` | `AspectRadarCrossSection` | — | — | ≥ 0 | Radar cross section seen by radars in bands A to D (below 2 GHz), per aspect. |
| `RadarCrossSectionEToM` | `AspectRadarCrossSection` | — | — | ≥ 0 | Radar cross section seen by radars in bands E to M (2 GHz and above), per aspect. |

## Initialization data

None declared by `AircraftSignature` (see inherited data below).

## Inherited data

Declared by the base definitions and available in this model. A derived definition sets their default values by repeating them with the same name.

| Data | Kind | Type | Declared in |
|---|---|---|---|
| `Dimensions` | technological | `Dimensions` | [`Signature`](signature.md) |

## Behaviour

- **Aspects.** Each signature is a record with `Front`, `Side` (either side), `Rear` and `Top`
  (seen from above or below). The aspect is chosen from the line of sight between the sensor and
  the aircraft.
- **`Visual`** is the range at which the aircraft is seen by eye in daylight, in metres. Visual
  sensors scale it by their [`Generation`](sensor.md).
- **`Infrared`** is the range at which a reference infrared sensor detects the aircraft, in
  metres; typically longest from the rear (engine exhaust). Infrared sensors scale it by their
  generation.
- **Radar cross sections** are in square metres (1 m² = 0 dBsm). There are two, chosen by the band
  of the radar that looks at the aircraft: `RadarCrossSectionAToD` for bands A to D (below 2 GHz)
  and `RadarCrossSectionEToM` for bands E to M (2 GHz and above). Stealth aircraft typically have
  a much lower cross section in E-M than in A-D.
- The detection model itself (how a sensor combines its data with these values) belongs to the
  simulation, not to the archetype.

## Dynamic behaviour (pseudo-code)

Reference behaviour of the model during a simulation, written in the pseudo-code described in [Model interactions](interactions.md#pseudo-code-conventions). `data.X` is the resolved value of the data item `X`; functions and constants not given by the archetype (turn rate, detection threshold, weather...) belong to the simulation.

- **Reads:** the kinematic of its own entity (position, orientation), to find the aspect seen by an observer.
- **Read by:** `Radar` (radar cross section), `InfraredSensor` (`Infrared`), `VisualSensor` (`Visual`) of the other entities.

```text
init(entity, data)
    Signature.init(entity, data)

# which face of the aircraft an observer sees
aspect(observerPosition) -> Front | Side | Rear | Top
    k   = owner.kinematic
    los = lineOfSight(k.getPosition(), observerPosition)     # bearing and elevation from the aircraft
    elevation = los.elevation - k.getOrientation().Pitch
    if abs(elevation) > topThreshold: return Top              # simulation constant, e.g. 60 degrees
    rel = normalize360(los.bearing - k.getOrientation().Heading)
    if rel <= 45 or rel >= 315: return Front
    if 135 <= rel <= 225:       return Rear
    return Side

radarCrossSection(observerPosition, band) -> m2
    table = band in [A, B, C, D] ? data.RadarCrossSectionAToD : data.RadarCrossSectionEToM
    return table[aspect(observerPosition)]

infraredRange(observerPosition) -> m
    return data.Infrared[aspect(observerPosition)]
    # possible extension: increase when the owner's throttle is Flank (afterburner plume)

visualRange(observerPosition) -> m
    return data.Visual[aspect(observerPosition)]
```

## Constraints and rules

- Every signature value is ≥ 0. The `Min` constraint is set on a record: it applies to each of
  its four fields.
- Kind of equipment: `type="Signature"`, inherited from [`Signature`](signature.md).

## Deriving a concrete model

Excerpt of `examples/library/equipments-rafale.xml`:

```xml
<EquipmentDefinition name="RafaleSignature" base="AircraftSignature">
  <Description>Rafale signature: reduced frontal radar cross section (shaping, materials), canard delta of 15.27 m x 10.90 m.</Description>
  <TechnologicalData>
    <Parameter name="Dimensions" dataType="Dimensions">
      <DefaultValue><Field name="Length">15.27</Field><Field name="Span">10.9</Field><Field name="Height">5.34</Field></DefaultValue>
    </Parameter>
    <Parameter name="RadarCrossSectionEToM" dataType="AspectRadarCrossSection">
      <DefaultValue><Field name="Front">0.5</Field><Field name="Side">4</Field><Field name="Rear">1.5</Field><Field name="Top">8</Field></DefaultValue>
    </Parameter>
  </TechnologicalData>
</EquipmentDefinition>
```

## Examples in this repository

- `F35ASignature` ([`examples/library/equipments-f35.xml`](../../examples/library/equipments-f35.xml)): F-35A signature: very low radar cross section in bands E to M (stealth shaping and coatings, internal weapons), higher in the low bands; 15.67 m x 10.67 m.
- `RafaleSignature` ([`examples/library/equipments-rafale.xml`](../../examples/library/equipments-rafale.xml)): Rafale signature: reduced frontal radar cross section (shaping, materials), canard delta of 15.27 m x 10.90 m.
- `FighterSignature` ([`examples/library/equipments.xml`](../../examples/library/equipments.xml)): Fighter signature: detection ranges in metres, radar cross sections in square metres.

## CMO mapping

See [CMO mapping](../cmo-mapping.md#signature) for the CMO database field behind each data item.
