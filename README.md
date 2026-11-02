# XCGF.Data

XML schema describing XCGF archetypes, with the XCGF standard library and an example library.

The schema describes **archetypes only** (entity, equipment, unit and task types), never instances. Every value it carries is a default value that the scenario may override.

## Contents

| Path | Content |
| --- | --- |
| `schemas/xcgf-data.xsd` | The schema (namespace `urn:xcgf:data:1.0`). Each type and element is documented with `xs:documentation`. |
| `library/types/core.xml` | Standard library: core data types, modelled on the HLA OMT (IEEE 1516-2010). |
| `library/types/equipment.xml` | Standard library: data types of the standard equipments (masses, performance table, signatures, frequency bands, sensor arcs...). |
| `library/equipments/kinematics.xml` | Standard library: `Kinematic` and `FixedWingKinematic`. |
| `library/equipments/signatures.xml` | Standard library: `Signature` and `AircraftSignature`. |
| `library/equipments/sensors.xml` | Standard library: `Sensor` and its `Radar`, `ESM`, `InfraredSensor` and `VisualSensor` derivations. |
| `docs/cmo-mapping.md` | Mapping of the standard equipments to the Command: Modern Operations (CMO) database, for a future importer. |
| `examples/library/library.xml` | Example library, assembled from the standard library and the files below with XInclude. |
| `examples/library/equipments.xml` | Example equipments deriving from the standard library: fighter kinematic, signature, radar, RWR, IRST, and an air-to-air missile. |
| `examples/library/tasks.xml` | Tasks: MoveTo, FireOnTarget, FollowFlightPlan, ChangeFormation. |
| `examples/library/entities/fighter.xml` | Fighter aircraft archetype with its signature, sensors and two missiles. |
| `examples/library/units.xml` | Patrol of two fighters. |

## Model

- **Data types** follow the HLA OMT: `BasicData`, `SimpleData`, `EnumeratedData`, `ArrayData`, `FixedRecordData`, `VariantRecordData`. `ReferenceData` (XCGF extension) types a data item that will point to an entity, equipment or unit instance chosen in the scenario.
- **Equipment definitions** declare technological data (characteristics, e.g. a radar range) and initialization data (initial state, e.g. a radar mode). Each data item has a type, a description, an optional default value (plain text, or `Field` and `Item` children for records and arrays) and optional constraints (`Min`, `Max`, `ListOfValues`, `Pattern`, `CustomConstraint`). A `Min` or `Max` can reference a technological data item. Definitions can derive from one another through `base`. The standard library (`library/`) provides the root definitions (fixed-wing kinematic, signatures, sensors) that concrete equipments derive from; they follow the CMO database (see `docs/cmo-mapping.md`).
- **Entities** have a unique name, a dotted DIS type, a mandatory kinematic equipment (which declares at least a position and an orientation) and other equipments.
- **Units** are compositions of entity archetypes and unit archetypes, each with a count and a role.
- **Tasks** group their parameters by the 5W: `WhoTasker` (optional), `WhoTaskee` (mandatory), `WhoAffected` (optional), `What`, `When`, `Where`, `Why`. A task parameter can be required and has a direction (`In`, `Out` or `InOut`). A task has one activation trigger and one or more exit triggers. Equipments, entities and units reference the tasks they can perform with `TaskRef`.

Documentation is carried by `Description` elements (or `Semantics` for data types, as in the HLA OMT), not by XML comments, so an editor can display it.

## Splitting into files

`DataTypes`, `EquipmentLibrary`, `TaskLibrary`, `Entities`, `Entity` and `Units` are global elements, so each one can be the root of its own file. An `XcgfData` document assembles them with `xi:include`.

## Validation

Validate the whole library, cross-file references included:

```sh
xmllint -xinclude -noout -schema schemas/xcgf-data.xsd examples/library/library.xml
```

Each included file also validates on its own, references excepted:

```sh
xmllint -noout -schema schemas/xcgf-data.xsd examples/library/tasks.xml
```

Some rules cannot be expressed in XSD 1.0 and are left to the tools: a kinematic definition must derive from `Kinematic`, definitions and units must not form cycles, a `Min` or `Max` holds either `value` or `ref`, and required task parameters must be supplied in the scenario.
