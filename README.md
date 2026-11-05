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
| `library/tasks/kinematics.xml` | Standard library: movement tasks modelled on CMO (`MoveTo`, `FollowCourse`, `HoldPosition` for every kinematic; `SetAltitude`, `Orbit`, `ReturnToBase`, `TakeOff`, `Land` for fixed-wing aircraft), see `docs/tasks/kinematics.md`. |
| `src/XCGF.Data/XCGF.Data.csproj` | NuGet packaging project (content only, no code). |
| `docs/models/` | One page per standard model (`Kinematic`, `FixedWingKinematic`, `Signature`, `AircraftSignature`, `Sensor`, `Radar`, `ESM`, `InfraredSensor`, `VisualSensor`): role, data, constraints, inheritance, behaviour. Start from `docs/models/README.md`. |
| `docs/cmo-mapping.md` | Mapping of the standard equipments to the Command: Modern Operations (CMO) database, for a future importer. |
| `examples/library/library.xml` | Example library, assembled from the standard library and the files below with XInclude. |
| `examples/library/equipments.xml` | Example equipments deriving from the standard library: fighter kinematic, signature, radar, RWR, IRST, and an air-to-air missile. |
| `examples/library/equipments-rafale.xml` | Dassault Rafale C equipments: kinematic, signature, RBE2 radar, SPECTRA ESM and missile warner, OSF IRST (open-source approximations). |
| `examples/library/equipments-f35.xml` | Lockheed Martin F-35A equipments: kinematic, signature, AN/APG-81 radar, AN/ASQ-239 ESM, AN/AAQ-37 DAS, AN/AAQ-40 EOTS (open-source approximations). |
| `examples/library/tasks.xml` | Example tasks: FireOnTarget, ChangeFormation (movement tasks come from the standard library). |
| `examples/library/entities/fighter.xml` | Fighter aircraft archetype with its signature, sensors and two missiles. |
| `examples/library/entities/rafale.xml` | Rafale C archetype with its signature and sensors. |
| `examples/library/entities/f35a.xml` | F-35A archetype with its signature and sensors. |
| `examples/library/units.xml` | Patrol of two fighters. |

## Model

- **Data types** follow the HLA OMT: `BasicData`, `SimpleData`, `EnumeratedData`, `ArrayData`, `FixedRecordData`, `VariantRecordData`. `ReferenceData` (XCGF extension) types a data item that will point to an entity, equipment or unit instance chosen in the scenario.
- **Equipment definitions** declare technological data (characteristics, e.g. a radar range) and initialization data (initial state, e.g. a radar mode). Each data item has a type, a description, an optional default value (plain text, or `Field` and `Item` children for records and arrays) and optional constraints (`Min`, `Max`, `ListOfValues`, `Pattern`, `CustomConstraint`). A `Min` or `Max` can reference a technological data item. Definitions can derive from one another through `base`. The standard library (`library/`) provides the root definitions (fixed-wing kinematic, signatures, sensors) that concrete equipments derive from; they follow the CMO database (see `docs/cmo-mapping.md`).
- **Entities** have a unique name, a dotted DIS type, a mandatory kinematic equipment (which declares at least a position and an orientation) and other equipments.
- **Units** are compositions of entity archetypes and unit archetypes, each with a count and a role.
- **Tasks** group their parameters by the 5W: `WhoTasker` (optional), `WhoTaskee` (mandatory), `WhoAffected` (optional), `What`, `When`, `Where`, `Why`. A task parameter can be required and has a direction (`In`, `Out` or `InOut`). A task has one activation trigger and one or more exit triggers. Equipments, entities and units reference the tasks they can perform with `TaskRef`. The standard kinematics carry the standard movement tasks, which their derived definitions inherit.

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

## NuGet package

The `XCGF.Data` package ships the schema and the standard library (`schemas/` and `library/`); the example library is not shipped. It contains no assembly.

```sh
dotnet add package XCGF.Data
```

- With `PackageReference`, the files are copied to the build output under `XCGF.Data/schemas/` and `XCGF.Data/library/`.
- MSBuild exposes `$(XcgfDataRoot)` (folder holding `schemas/` and `library/` in the package) and `$(XcgfDataSchema)` (path of `xcgf-data.xsd`).

Build the package locally:

```sh
dotnet pack src/XCGF.Data -c Release -o artifacts -p:Version=0.1.0
```

The `NuGet` workflow validates the standard library and packs on every push and pull request. Pushing an `X.Y.Z` tag (no `v` prefix, e.g. `0.1.0`) publishes version `X.Y.Z` to GitHub Packages, and to nuget.org when the `NUGET_API_KEY` repository secret is set.
