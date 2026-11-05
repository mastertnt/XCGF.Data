# ESM

Passive electronic support measures, radar warning receivers included: detects and identifies emitters, without emitting.

| | |
|---|---|
| Kind | Equipment definition (`EquipmentDefinition`), `type="ESM"` |
| Defined in | [`library/equipments/sensors.xml`](../../library/equipments/sensors.xml) |
| Base | [`Sensor`](sensor.md) |
| Derivation chain | [`Sensor`](sensor.md) → [`ESM`](esm.md) |
| Derived in the standard library | — |

## Role

`ESM` (electronic support measures) is a passive receiver that detects, locates in bearing and
identifies emitters, without emitting itself. It covers radar warning receivers as well as full
electronic support suites; the `Role` inherited from [`Sensor`](sensor.md) tells them apart
(`RadarWarningReceiver` or `ElectronicSupport`).

## Technological data

Characteristics of the model, fixed for a given equipment type.

| Data | Type | Unit | Default | Constraints | Description |
|---|---|---|---|---|---|
| `FrequencyBands` | `NatoBands` | — | — | — | Bands the receiver listens to. Emitters outside them are not intercepted. |
| `Sensitivity` | `PowerDecibelMilliwatts` | dBm | — | — | Weakest received power the receiver detects. |
| `BearingAccuracy` | `AngleDegrees` | deg | — | ≥ 0 | Accuracy of the bearing measured on an emitter. |
| `EmitterIdentification` | `HLAboolean` | — | `HLAtrue` | — | Whether the receiver identifies the type of an intercepted emitter. |

## Initialization data

None declared by `ESM` (see inherited data below).

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

- **Interception.** An emitter (for instance a [`Radar`](radar.md) not in `Off` mode) is
  intercepted when one of its bands is in the receiver's `FrequencyBands`, the received power is
  above `Sensitivity` (dBm, the more negative the more sensitive), and the emitter lies inside the
  sensor coverage inherited from `Sensor`.
- **`BearingAccuracy`** is the error on the measured bearing, in degrees. An ESM alone gives a
  bearing, not a range.
- **`EmitterIdentification`** tells whether the receiver identifies the type of the intercepted
  emitter (default true).

## Dynamic behaviour (pseudo-code)

Reference behaviour of the model during a simulation, written in the pseudo-code described in [Model interactions](interactions.md#pseudo-code-conventions). `data.X` is the resolved value of the data item `X`; functions and constants not given by the archetype (turn rate, detection threshold, weather...) belong to the simulation.

- **Reads:** its own kinematic; the kinematic of each target and the radars it carries (`isEmitting`, bands, power, gain).
- **Read by:** the tasks and triggers of its entity (threat warning, emitter bearing).

```text
init(entity, data)
    Sensor.init(entity, data)

isActive() -> operational                      # passive: always listening, never emitting

detect(target) -> bool
    found = false
    for emitter in target.equipmentsOfType(Radar) where emitter.isEmitting()
        common = emitter.data.FrequencyBands ∩ data.FrequencyBands
        if common is empty: continue
        R      = rangeTo(target)
        lambda = speedOfLight / centerFrequency(mainBand(common))
        inBeam = emitter.inArcs(owner, emitter.data.DetectionArcs ?? all around)
        gainTx = inBeam ? emitter.data.AntennaGain : sidelobeGain     # sidelobeGain: simulation constant
        # one-way propagation: the ESM hears a radar much farther than the radar sees it
        received_dBm = dBm(emitter.data.PeakPower) + gainTx - dB((4*pi*R / lambda)^2)
        if LowProbabilityOfIntercept in emitter.data.Features: received_dBm -= lpiLoss
        if received_dBm >= data.Sensitivity
            remember(emitter)
            found = true
    return found

report(target, t) -> Contact
    c = Sensor.report(target, t)
    c.bearing = trueBearing(target) + gaussian(0, data.BearingAccuracy)
    c.range   = unknown                         # bearing only, whatever the capabilities
    c.emitters = remembered emitters of target,
                 identified by definition name if data.EmitterIdentification else by band only
    return c
```

## Constraints and rules

- `BearingAccuracy` ≥ 0.
- Kind of equipment: `type="ESM"`.

## Deriving a concrete model

Excerpt of `examples/library/equipments-rafale.xml`:

```xml
<EquipmentDefinition name="SpectraESM" base="ESM">
  <Description>Detection part of the Thales/MBDA SPECTRA self-protection suite: radar warning and electronic support all around the aircraft, with emitter identification and accurate bearing.</Description>
  <TechnologicalData>
    <Parameter name="Role" dataType="SensorRoleEnum"><DefaultValue>ElectronicSupport</DefaultValue></Parameter>
    <Parameter name="FrequencyBands" dataType="NatoBands">
      <DefaultValue><Item>C</Item><Item>D</Item><Item>E</Item><Item>F</Item><Item>G</Item><Item>H</Item><Item>I</Item><Item>J</Item><Item>K</Item></DefaultValue>
    </Parameter>
    <Parameter name="Sensitivity" dataType="PowerDecibelMilliwatts"><DefaultValue>-70</DefaultValue></Parameter>
    <Parameter name="BearingAccuracy" dataType="AngleDegrees"><DefaultValue>1</DefaultValue></Parameter>
  </TechnologicalData>
</EquipmentDefinition>
```

## Examples in this repository

- `ASQ239ESM` ([`examples/library/equipments-f35.xml`](../../examples/library/equipments-f35.xml)): BAE Systems AN/ASQ-239 electronic warfare suite (detection part): radar warning, emitter location and identification all around the aircraft.
- `SpectraESM` ([`examples/library/equipments-rafale.xml`](../../examples/library/equipments-rafale.xml)): Detection part of the Thales/MBDA SPECTRA self-protection suite: radar warning and electronic support all around the aircraft, with emitter identification and accurate bearing.
- `RadarWarningReceiver` ([`examples/library/equipments.xml`](../../examples/library/equipments.xml)): Radar warning receiver covering all around the aircraft, from band E to band J.

## CMO mapping

See [CMO mapping](../cmo-mapping.md#esm) for the CMO database field behind each data item.
