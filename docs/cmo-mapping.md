# Mapping from Command: Modern Operations (CMO)

The XCGF standard equipment definitions in `library/equipments/` are modelled on the database of
Command: Modern Operations (CMO, formerly CMANO; database DB3000), so that a future importer can
turn a CMO aircraft and its sensors into XCGF archetypes field by field.

> **Status.** The CMO field names below are written from knowledge of the CMO database and its
> in-game database viewer; they were not checked against a DB3000 export when this mapping was
> written. The importer work should start by checking them against the actual tables and adjusting
> this document (and the definitions, if a field is missing).

## General rules

- **One CMO record, several XCGF equipments.** A CMO aircraft record becomes an entity archetype
  whose `Kinematic` uses a definition derived from `FixedWingKinematic`, plus one `Signature`
  equipment derived from `AircraftSignature`. Each CMO sensor becomes an equipment definition
  derived from `Radar`, `ESM`, `InfraredSensor` or `VisualSensor`, used by the entity's equipments.
- **Units are SI.** The importer converts CMO units:

  | CMO unit | XCGF unit | Factor |
  |---|---|---|
  | knot (kts) | m/s | × 0.514444 |
  | foot (ft) | m | × 0.3048 |
  | feet per minute | m/s | × 0.00508 |
  | nautical mile (nm) | m | × 1852 |
  | kg/min (fuel consumption) | kg/s | ÷ 60 |
  | lbf / kgf (thrust) | N | × 4.448222 / × 9.80665 |
  | dBsm (if RCS is stored in dB) | m² | 10^(dBsm/10) |
  | µs (pulse width) | s | × 10⁻⁶ |
  | kW (peak power) | W | × 1000 |

- **Enumerations keep CMO names** where CMO has a closed list (throttle settings, generation,
  frequency bands), so the importer maps them by name.
- **Values are default values.** Every imported number is written as a `DefaultValue` of the
  derived definition; the scenario can still override it.

## FixedWingKinematic

Source: the CMO aircraft record (general data, propulsion, fuel, performance profile).

| XCGF data (`FixedWingKinematic`) | Kind | CMO field | Notes |
|---|---|---|---|
| `EmptyMass` | technological | Weight, empty | kg |
| `MaxTakeoffMass` | technological | Weight, max | kg |
| `EngineType` | technological | Propulsion type | turbofan, turbojet, turboprop, piston... |
| `EngineCount` | technological | Number of engines | |
| `DryThrust` | technological | Engine thrust (military) | per engine |
| `AfterburnerThrust` | technological | Engine thrust (afterburner) | per engine; equal to `DryThrust` without afterburner |
| `FuelType` | technological | Fuel type | |
| `FuelCapacity` | technological | Fuel capacity (internal) | kg; external tanks are separate equipments |
| `PerformanceTable` | technological | Performance profile | one `PerformancePoint` per altitude band and throttle setting (see below) |
| `StallSpeed` | technological | Lowest Loiter speed of the profile | CMO has no stall speed field; the importer takes the minimum speed |
| `MaxSpeed` | technological | Highest speed of the profile (Flank) | |
| `ClimbRate` | technological | Climb rate | ft/min → m/s |
| `Ceiling` | technological | Max altitude | |
| `Agility` | technological | Agility | CMO rating, kept as is |
| `MinRunwayLength` | technological | Runway length code | the importer converts the code to a length |
| `InitialSpeed` | initialization | (scenario) | bounded by `StallSpeed` and `MaxSpeed` |
| `InitialThrottle` | initialization | (scenario) | `ThrottleEnum` |
| `InitialFuel` | initialization | (scenario) | bounded by `FuelCapacity` |
| `Position`, `Orientation` | initialization | (scenario) | inherited from `Kinematic` |

Performance profile: CMO gives, for each altitude band, the speed (kts) and fuel consumption
(kg/min) at each throttle setting. Each (band, throttle) pair becomes one `PerformancePoint`:

| `PerformancePoint` field | CMO |
|---|---|
| `MinAltitude`, `MaxAltitude` | altitude band limits (ft → m) |
| `Throttle` | `Loiter`, `Cruise`, `Full`, `Flank` (`ThrottleEnum`, same names) |
| `Speed` | speed (kts → m/s) |
| `FuelConsumption` | consumption (kg/min → kg/s) |

Not mapped yet: crew, OODA values, endurance (derived from fuel and performance), physical size
code (see `Dimensions`), weapon mounts and loadouts (future weapon equipments).

## Signature

Source: the signature table of the CMO aircraft record. CMO gives each signature for four
aspects (front, side, rear, top), which become the four fields of `AspectDetectionRange` or
`AspectRadarCrossSection`.

| XCGF data (`AircraftSignature`) | CMO signature | Notes |
|---|---|---|
| `Visual` | Visual detection range | nm → m |
| `Infrared` | Infrared detection range | nm → m |
| `RadarCrossSectionAToD` | Radar cross section, A–D band | m² (convert if stored in dBsm) |
| `RadarCrossSectionEToM` | Radar cross section, E–M band | m² (convert if stored in dBsm) |
| `Dimensions` (from `Signature`) | Length, span, height | general data of the aircraft record |

Ships and submarines will add their sonar signatures (passive and active) in new definitions
deriving from `Signature`.

## Sensors

Source: the CMO sensor record, plus the platform-to-sensor link (arcs) of the aircraft record.

### Common data (`Sensor`)

| XCGF data | CMO field | Notes |
|---|---|---|
| `type` attribute | Sensor type | Radar → `Radar`, ESM → `ESM`, Infrared → `InfraredSensor`, Visual → `VisualSensor` (base definition) |
| `Role` | Role | mapped onto `SensorRoleEnum`; `Other` for unmapped roles |
| `Generation` | Generation | `Early1950s`... `Early2020s` |
| `MinRange`, `MaxRange` | Range min, Range max | nm → m |
| `MinAltitude`, `MaxAltitude` | Altitude min, Altitude max | ft → m |
| `ScanInterval` | Scan interval | s |
| `Capabilities` | Capabilities | air / surface / land / periscope search, range, heading, speed, altitude information, classification, identification, illumination |
| `DetectionArcs` | Sensor arcs (detection) | each CMO 30° arc of the mount becomes a `SensorArc`; the importer merges contiguous arcs |
| `MaxContacts` | Max contacts | |
| `Operational` (init.) | (scenario) | damaged or switched off at start |

### Radar

| XCGF data | CMO field | Notes |
|---|---|---|
| `FrequencyBands` | Frequency band(s) | NATO letters A–M (`NatoBandEnum`) |
| `PeakPower` | Peak power | W |
| `AntennaGain` | Antenna gain | dB |
| `HorizontalBeamwidth`, `VerticalBeamwidth` | Horizontal / vertical beamwidth | degrees |
| `PulseWidth` | Pulse width | µs → s |
| `PulseRepetitionFrequency` | PRF | Hz (or derived from blind time) |
| `SystemNoiseLevel` | System noise level | dB |
| `ProcessingGain` | Processing gain/loss | dB |
| `RangeResolution`, `AngularResolution` | Resolution (range, angle) | |
| `Features` | Sensor codes | pulse-Doppler, look-down/shoot-down, track-while-scan, MTI, SAR, ISAR, AESA, PESA, frequency agility, LPI, NCTR |
| `TrackingArcs` | Sensor arcs (tracking / illumination) | |
| `MaxIlluminatedTargets` | Max targets illuminated / guided | fire control radars |
| `Mode` (init.) | (scenario, EMCON) | `Off`, `Search`, `Track` |

### ESM

| XCGF data | CMO field | Notes |
|---|---|---|
| `FrequencyBands` | Frequency band(s) | |
| `Sensitivity` | Sensitivity | dBm |
| `BearingAccuracy` | Precision / bearing accuracy | degrees |
| `EmitterIdentification` | Capability: identification | |

### Infrared and visual

| XCGF data | CMO field | Notes |
|---|---|---|
| `Band` (`InfraredSensor`) | IR band, when given | otherwise left to the default |
| `Imaging` (`InfraredSensor`) | Imaging capability (IIR / FLIR) | |
| `NightCapable` (`VisualSensor`) | Night vision / low-light capability | |
| `FieldOfView` | Field of view, when given | |

In CMO, infrared and visual detection depend on the target's `Infrared` and `Visual`
signatures, scaled by the sensor's `Generation`; XCGF keeps the same data, the detection model
belongs to the simulation.
