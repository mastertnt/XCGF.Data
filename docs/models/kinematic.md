# Kinematic

Root kinematic. Every kinematic derives from it and inherits its position and orientation.

| | |
|---|---|
| Kind | Kinematic definition (`KinematicDefinition`) |
| Defined in | [`library/equipments/kinematics.xml`](../../library/equipments/kinematics.xml) |
| Base | — (root definition) |
| Derivation chain | [`Kinematic`](kinematic.md) |
| Derived in the standard library | [`FixedWingKinematic`](fixed-wing-kinematic.md) |

## Role

`Kinematic` is the root of every kinematic model. It says where an entity is and which way it
points, and nothing else: it has no technological data and does not move the entity by itself.
Every entity archetype carries exactly one `Kinematic` element, whose `definition` must name
`Kinematic` or a definition derived from it, so every entity inherits a position and an orientation.

## Technological data

None declared by `Kinematic`.

## Initialization data

State of the equipment when the entity is created; the scenario usually sets it.

| Data | Type | Unit | Default | Constraints | Description |
|---|---|---|---|---|---|
| `Position` | `GeoPosition` | — | — | — | Position of the entity when it is created. |
| `Orientation` | `Orientation` | — | — | — | Heading, pitch and roll of the entity when it is created. |

## Behaviour

- `Position` is a WGS84 geodetic position (`Latitude`, `Longitude` in degrees, `Altitude` in metres
  above mean sea level).
- `Orientation` is the attitude in degrees: `Heading` from true north, clockwise, in [0, 360);
  `Pitch` positive nose up, in [-90, 90]; `Roll` positive right wing down, in [-180, 180].
- Neither has a default value: an archetype does not know where its instances will be. The
  scenario sets both when it creates the entity.

## Constraints and rules

- The schema (`KinematicDefinitionType`) requires the root kinematic to declare `Position` and
  `Orientation` as initialization data. XSD 1.0 cannot check that every kinematic derives from
  it; the tools do, and they also reject derivation cycles.
- A derived kinematic must not redeclare `Position` or `Orientation` with another data type.

## Deriving a concrete model

A kinematic for a new kind of platform (ground vehicle, ship, rotary wing...) derives directly
from `Kinematic` and adds its own data:

```xml
<KinematicDefinition name="GroundVehicleKinematic" base="Kinematic">
  <Description>Wheeled or tracked vehicle moving on the ground.</Description>
  <TechnologicalData>
    <Parameter name="MaxRoadSpeed" dataType="SpeedMetersPerSecond">
      <Description>Highest speed on a road.</Description>
      <Constraints><Min value="0" inclusive="false"/></Constraints>
    </Parameter>
  </TechnologicalData>
</KinematicDefinition>
```

## CMO mapping

`Position` and `Orientation` have no CMO counterpart: the scenario sets them. Derived kinematics are mapped in [CMO mapping](../cmo-mapping.md).
