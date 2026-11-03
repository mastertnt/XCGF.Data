# Standard models

One page per model (equipment definition) of the XCGF standard library (`library/equipments/`).
Each page gives the role of the model, its technological and initialization data with their types,
units, default values and constraints, the data it inherits, its behaviour, the rules the tools
check, how to derive a concrete model from it, and its mapping to the CMO database.

## Derivation tree

```text
Kinematic                     kinematic.md
└── FixedWingKinematic        fixed-wing-kinematic.md
Signature                     signature.md
└── AircraftSignature         aircraft-signature.md
Sensor                        sensor.md
├── Radar                     radar.md
├── ESM                       esm.md
├── InfraredSensor            infrared-sensor.md
└── VisualSensor              visual-sensor.md
```

| Model | Base | Kind | Page |
|---|---|---|---|
| `Kinematic` | — | Kinematic | [kinematic.md](kinematic.md) |
| `FixedWingKinematic` | `Kinematic` | Kinematic | [fixed-wing-kinematic.md](fixed-wing-kinematic.md) |
| `Signature` | — | `Signature` | [signature.md](signature.md) |
| `AircraftSignature` | `Signature` | `Signature` | [aircraft-signature.md](aircraft-signature.md) |
| `Sensor` | — | `Sensor` | [sensor.md](sensor.md) |
| `Radar` | `Sensor` | `Radar` | [radar.md](radar.md) |
| `ESM` | `Sensor` | `ESM` | [esm.md](esm.md) |
| `InfraredSensor` | `Sensor` | `Infrared` | [infrared-sensor.md](infrared-sensor.md) |
| `VisualSensor` | `Sensor` | `Visual` | [visual-sensor.md](visual-sensor.md) |

## Reading the pages

- **Technological data** are the characteristics of a model (a radar range, an empty mass).
  **Initialization data** are the state of the equipment when the entity is created (a radar mode,
  an initial speed); the scenario usually sets them.
- **Constraints** use `≥`/`≤` for inclusive bounds and `>`/`<` for exclusive ones. A bound written
  as a data name (for instance `≤ MaxSpeed`) is read from the value of that technological data item
  in the most derived definition.
- **Inheritance.** A definition with `base` inherits all the data of its base. Repeating a data item
  with the same name in the derived definition sets its default value or adds constraints; its
  description and inherited constraints are kept (merge by name, done by the tools).
- **Standard models carry no platform values.** The values of a real system go into a definition
  derived from the standard model (for instance `RafaleKinematic`, base `FixedWingKinematic`); the
  entity archetype references that derived definition and overrides nothing.
- **Units are SI** (m, m/s, kg, kg/s, N, W, Hz, s, m², degrees for angles, dB and dBm for ratios and
  powers). The CMO importer converts from CMO units, see [CMO mapping](../cmo-mapping.md).
