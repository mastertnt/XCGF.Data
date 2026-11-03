# Radar

Active radar: emission (power, gain, pulse), reception (noise, processing gain) and resolutions. Its detections depend on the radar cross section of the target in its band.

| | |
|---|---|
| Kind | Equipment definition (`EquipmentDefinition`), `type="Radar"` |
| Defined in | [`library/equipments/sensors.xml`](../../library/equipments/sensors.xml) |
| Base | [`Sensor`](sensor.md) |
| Derivation chain | [`Sensor`](sensor.md) → [`Radar`](radar.md) |
| Derived in the standard library | — |

## Role

`Radar` is an active radar: it emits, so it detects targets through their radar cross section and
can itself be intercepted by the [`ESM`](esm.md) of other entities. It describes the emission
(bands, power, antenna, pulses), the reception (noise, processing gain), the resolutions, the
processing features, and the fire control capacity.

## Technological data

Characteristics of the model, fixed for a given equipment type.

| Data | Type | Unit | Default | Constraints | Description |
|---|---|---|---|---|---|
| `FrequencyBands` | `NatoBands` | — | — | — | Bands the radar emits in. They select the target's radar cross section (A-D or E-M) and the receivers able to intercept it. |
| `PeakPower` | `PowerWatts` | W | — | > 0 | Peak emitted power. |
| `AntennaGain` | `Decibels` | dB | — | — | Gain of the antenna. |
| `HorizontalBeamwidth` | `AngleDegrees` | deg | — | > 0, ≤ 360 | Horizontal width of the beam. |
| `VerticalBeamwidth` | `AngleDegrees` | deg | — | > 0, ≤ 180 | Vertical width of the beam. |
| `PulseWidth` | `ShortDurationSeconds` | s | — | > 0 | Duration of one pulse. |
| `PulseRepetitionFrequency` | `FrequencyHertz` | Hz | — | > 0 | Number of pulses per second. |
| `SystemNoiseLevel` | `Decibels` | dB | — | — | Noise level of the receiver. |
| `ProcessingGain` | `Decibels` | dB | — | — | Gain (or loss, when negative) brought by signal processing. |
| `RangeResolution` | `DistanceMeters` | m | — | ≥ 0 | Smallest range difference at which two targets are told apart. |
| `AngularResolution` | `AngleDegrees` | deg | — | ≥ 0 | Smallest bearing difference at which two targets are told apart. |
| `Features` | `RadarFeatures` | — | — | — | Processing features (pulse-Doppler, look-down/shoot-down, track-while-scan...). |
| `TrackingArcs` | `SensorArcs` | — | — | — | Sectors where the radar can track or illuminate a target. The detection arcs when absent. |
| `MaxIlluminatedTargets` | `Count` | — | `0` | ≥ 0 | Highest number of targets illuminated at once for weapon guidance; 0 for a radar without fire control. |

## Initialization data

State of the equipment when the entity is created; the scenario usually sets it.

| Data | Type | Unit | Default | Constraints | Description |
|---|---|---|---|---|---|
| `Mode` | `RadarModeEnum` | — | `Off` | — | Operating mode of the radar when the entity is created. Off keeps the platform silent (emission control). |

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

- **Bands.** `FrequencyBands` (NATO letters A to M) select which radar cross section of the target
  is used (`RadarCrossSectionAToD` or `RadarCrossSectionEToM` of
  [`AircraftSignature`](aircraft-signature.md)) and which receivers can intercept the radar.
- **Emission and reception.** `PeakPower` (W), `AntennaGain` (dB), `HorizontalBeamwidth` and
  `VerticalBeamwidth` (degrees), `PulseWidth` (s, nanosecond resolution),
  `PulseRepetitionFrequency` (Hz), `SystemNoiseLevel` (dB) and `ProcessingGain` (dB, negative
  for a loss) are the radar equation inputs.
- **Resolutions.** `RangeResolution` and `AngularResolution` are the smallest differences at
  which two targets are told apart.
- **`Features`** list the processing features: pulse-Doppler, look-down/shoot-down,
  track-while-scan, moving target indicator, SAR, ISAR, AESA, PESA, frequency agility, low
  probability of intercept, non-cooperative target recognition.
- **Fire control.** `TrackingArcs` are the sectors where the radar can track or illuminate (the
  detection arcs when absent); `MaxIlluminatedTargets` is how many targets it guides weapons on at
  once (0 for a radar without fire control).
- **`Mode`** (initialization) is `Off`, `Search` or `Track` when the entity is created. The
  default `Off` keeps the platform silent (emission control).

## Dynamic behaviour (pseudo-code)

Reference behaviour of the model during a simulation, written in the pseudo-code described in [Model interactions](interactions.md#pseudo-code-conventions). `data.X` is the resolved value of the data item `X`; functions and constants not given by the archetype (turn rate, detection threshold, weather...) belong to the simulation.

- **Reads:** its own kinematic; the kinematic and `AircraftSignature` of each target (radar cross section).
- **Read by:** the `ESM` of the other entities (it is an emitter when not `Off`); the weapons and the `FireOnTarget` task (illumination).

```text
state                                          # in addition to Sensor
    mode        : RadarModeEnum
    illuminated : set<Entity>

init(entity, data)
    Sensor.init(entity, data)
    mode        = data.Mode                    # default Off (emission control)
    illuminated = {}

isActive()   -> operational and mode != Off
isEmitting() -> operational and mode != Off    # read by the ESM of the other entities

detect(target) -> bool
    sig = target.equipmentOfType(Signature)
    if sig is none: return false
    band = mainBand(data.FrequencyBands)
    rcs  = sig.radarCrossSection(owner.kinematic.getPosition(), band)
    R    = rangeTo(target)
    lambda = speedOfLight / centerFrequency(band)
    # monostatic radar equation, in decibels
    snr = dB(data.PeakPower) + 2 * data.AntennaGain + dB(rcs) + dB(lambda^2)
          - dB((4*pi)^3) - 4 * dB(R) + data.ProcessingGain
          - (thermalNoise(data.PulseWidth) + data.SystemNoiseLevel)
    if target is low and over land/sea and LookDownShootDown not in data.Features
        snr -= clutterLoss                     # simulation constant
    return snr >= detectionThreshold            # simulation constant, e.g. 13 dB

# order from a task or a weapon: track / illuminate a contact for guidance
illuminate(target) -> bool
    if mode != Track or target not in contacts: return false
    arcs = data.TrackingArcs ?? data.DetectionArcs
    if not inArcs(target, arcs): return false
    if |illuminated| >= data.MaxIlluminatedTargets: return false
    illuminated.add(target)
    return true

step(t, dt)
    Sensor.step(t, dt)
    illuminated = illuminated ∩ contacts       # lost contacts are no longer illuminated
    merge contacts closer than data.RangeResolution in range and data.AngularResolution in bearing

setMode(newMode)                               # from a task
    mode = newMode
    if mode != Track: illuminated.clear()
```

## Constraints and rules

- `PeakPower`, `PulseWidth`, `PulseRepetitionFrequency` > 0; 0 < `HorizontalBeamwidth` ≤ 360;
  0 < `VerticalBeamwidth` ≤ 180; resolutions ≥ 0; `MaxIlluminatedTargets` ≥ 0.
- Expected: a radar with `MaxIlluminatedTargets` > 0 also lists `TargetIllumination` in its
  `Capabilities`.
- Kind of equipment: `type="Radar"`.

## Deriving a concrete model

Excerpt of `examples/library/equipments-rafale.xml`:

```xml
<EquipmentDefinition name="RBE2Radar" base="Radar">
  <Description>Thales RBE2 AESA multimode radar in the nose: air and surface search, tracking and fire control, ground mapping; covers a 140 degree front sector.</Description>
  <TechnologicalData>
    <Parameter name="Role" dataType="SensorRoleEnum"><DefaultValue>FireControl</DefaultValue></Parameter>
    <Parameter name="MaxRange" dataType="DistanceMeters"><DefaultValue>200000</DefaultValue></Parameter>
    <Parameter name="FrequencyBands" dataType="NatoBands">
      <DefaultValue><Item>I</Item><Item>J</Item></DefaultValue>
    </Parameter>
    <Parameter name="MaxIlluminatedTargets" dataType="Count"><DefaultValue>8</DefaultValue></Parameter>
  </TechnologicalData>
  <InitializationData>
    <Parameter name="Mode" dataType="RadarModeEnum"><DefaultValue>Search</DefaultValue></Parameter>
  </InitializationData>
</EquipmentDefinition>
```

## Examples in this repository

- `APG81Radar` ([`examples/library/equipments-f35.xml`](../../examples/library/equipments-f35.xml)): Northrop Grumman AN/APG-81 AESA radar in the nose: air and surface search, fire control, SAR mapping, low probability of intercept; covers a 120 degree front sector.
- `RBE2Radar` ([`examples/library/equipments-rafale.xml`](../../examples/library/equipments-rafale.xml)): Thales RBE2 AESA multimode radar in the nose: air and surface search, tracking and fire control, ground mapping; covers a 140 degree front sector.
- `FighterRadar` ([`examples/library/equipments.xml`](../../examples/library/equipments.xml)): Pulse-Doppler fire control radar in the nose, covering a 120 degree front sector (from 300 to 60 degrees through the nose). It starts in search mode.

## CMO mapping

See [CMO mapping](../cmo-mapping.md#radar) for the CMO database field behind each data item.
