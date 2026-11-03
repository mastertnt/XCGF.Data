# FixedWingKinematic

Fixed-wing aircraft kinematic. Speeds and fuel consumption come from a performance table (altitude band x throttle setting), as in CMO; the other data bound or complete it.

| | |
|---|---|
| Kind | Kinematic definition (`KinematicDefinition`) |
| Defined in | [`library/equipments/kinematics.xml`](../../library/equipments/kinematics.xml) |
| Base | [`Kinematic`](kinematic.md) |
| Derivation chain | [`Kinematic`](kinematic.md) → [`FixedWingKinematic`](fixed-wing-kinematic.md) |
| Derived in the standard library | — |

## Role

`FixedWingKinematic` moves a fixed-wing aircraft. It follows the aircraft performance data of
Command: Modern Operations (CMO): speeds and fuel consumption come from a performance table indexed
by altitude band and throttle setting, and the other data bound it (stall speed, maximum speed,
ceiling, climb rate) or complete it (masses, engines, fuel, agility, runway).

It is a generic model: it carries no aircraft values. A concrete aircraft is described by a
definition derived from it (for instance `RafaleKinematic`) that sets the default values; the entity
archetype then references that derived definition without overriding anything.

## Technological data

Characteristics of the model, fixed for a given equipment type.

| Data | Type | Unit | Default | Constraints | Description |
|---|---|---|---|---|---|
| `EmptyMass` | `MassKilograms` | kg | — | > 0 | Mass of the aircraft without fuel nor payload. |
| `MaxTakeoffMass` | `MassKilograms` | kg | — | ≥ `EmptyMass` | Highest mass at which the aircraft can take off, fuel and payload included. |
| `EngineType` | `EngineTypeEnum` | — | `Turbofan` | — | Kind of engines propelling the aircraft. |
| `EngineCount` | `Count` | — | `1` | ≥ 1 | Number of engines. |
| `DryThrust` | `ForceNewtons` | N | — | ≥ 0 | Thrust of one engine at maximum dry power (Full throttle). |
| `AfterburnerThrust` | `ForceNewtons` | N | — | ≥ `DryThrust` | Thrust of one engine with afterburner (Flank throttle); equal to the dry thrust without afterburner. |
| `FuelType` | `FuelTypeEnum` | — | `AviationFuel` | — | Fuel burnt by the engines. |
| `FuelCapacity` | `MassKilograms` | kg | — | ≥ 0 | Internal fuel capacity. External tanks are separate equipments. |
| `PerformanceTable` | `PerformanceTable` | — | — | — | Speed and fuel consumption for each altitude band and throttle setting. The kinematic interpolates between bands. |
| `StallSpeed` | `SpeedMetersPerSecond` | m/s | — | > 0 | Lowest speed at which the aircraft keeps flying. The kinematic never flies slower. |
| `MaxSpeed` | `SpeedMetersPerSecond` | m/s | — | ≥ `StallSpeed` | Highest speed the aircraft can reach. The kinematic never flies faster. |
| `ClimbRate` | `SpeedMetersPerSecond` | m/s | — | > 0 | Highest vertical speed in climb. |
| `Ceiling` | `DistanceMeters` | m | — | > 0 | Highest altitude the aircraft can reach. Movement orders above it are capped. |
| `Agility` | `Rating` | — | `3` | ≥ 0 | Manoeuvrability rating, as in CMO: the higher, the better the aircraft turns and evades. |
| `MinRunwayLength` | `DistanceMeters` | m | — | ≥ 0 | Shortest runway the aircraft can take off from and land on. |

## Initialization data

State of the equipment when the entity is created; the scenario usually sets it.

| Data | Type | Unit | Default | Constraints | Description |
|---|---|---|---|---|---|
| `InitialSpeed` | `SpeedMetersPerSecond` | m/s | — | ≥ `StallSpeed`, ≤ `MaxSpeed` | Speed of the aircraft when it is created, bounded by the stall speed and the maximum speed. |
| `InitialThrottle` | `ThrottleEnum` | — | `Cruise` | — | Throttle setting when the aircraft is created. |
| `InitialFuel` | `MassKilograms` | kg | — | ≥ 0, ≤ `FuelCapacity` | Fuel on board when the aircraft is created, bounded by the fuel capacity. |

## Inherited data

Declared by the base definitions and available in this model. A derived definition sets their default values by repeating them with the same name.

| Data | Kind | Type | Declared in |
|---|---|---|---|
| `Position` | initialization | `GeoPosition` | [`Kinematic`](kinematic.md) |
| `Orientation` | initialization | `Orientation` | [`Kinematic`](kinematic.md) |

## Behaviour

- **Performance table.** Each `PerformancePoint` gives, for one altitude band
  (`MinAltitude`..`MaxAltitude`) and one throttle setting (`Loiter`, `Cruise`, `Full`, `Flank`),
  the speed reached and the fuel burnt. A complete table has one point per band and setting; the
  kinematic interpolates between bands. The table holds 1 to 64 points.
- **Throttle settings** (`ThrottleEnum`, CMO names): `Loiter` is the lowest consumption per hour
  (endurance), `Cruise` the lowest consumption per distance (range), `Full` the maximum dry
  (military) power, `Flank` the maximum power with afterburner or emergency power.
- **Speed envelope.** The kinematic never flies slower than `StallSpeed` nor faster than
  `MaxSpeed`; movement orders above `Ceiling` are capped; `ClimbRate` limits the vertical speed in
  climb.
- **Thrust** is given per engine; total thrust is `EngineCount` × `DryThrust` (or
  `AfterburnerThrust` at `Flank`). Without afterburner, `AfterburnerThrust` equals `DryThrust`.
- **Fuel.** `FuelCapacity` is internal fuel only: external tanks will be separate equipments.
  `InitialFuel` is what is on board when the entity is created.
- **Agility** is the CMO manoeuvrability rating, kept as is (default 3): the higher, the better
  the aircraft turns and evades.
- All values are SI: masses in kg, speeds in m/s, thrust in N, fuel flow in kg/s, distances in m.

## Constraints and rules

- Chain of bounds between data items: `EmptyMass` ≤ `MaxTakeoffMass`; `DryThrust` ≤
  `AfterburnerThrust`; `StallSpeed` ≤ `MaxSpeed`; `StallSpeed` ≤ `InitialSpeed` ≤ `MaxSpeed`;
  0 ≤ `InitialFuel` ≤ `FuelCapacity`. A bound given with `ref` is read from the value of that data
  item in the most derived definition, so a derived aircraft automatically gets its own envelope.
- Expected but not checked yet by the schema: the speeds of the performance table lie between
  `StallSpeed` and `MaxSpeed`, and its altitude bands stay below `Ceiling`.

## Deriving a concrete model

A concrete aircraft derives from `FixedWingKinematic` and only sets default values; descriptions
and constraints are inherited (excerpt of `examples/library/equipments-rafale.xml`):

```xml
<KinematicDefinition name="RafaleKinematic" base="FixedWingKinematic">
  <Description>Rafale C kinematic: two Safran M88-2 turbofans with afterburner, 4,700 kg of internal fuel, Mach 1.8 at altitude.</Description>
  <TechnologicalData>
    <Parameter name="EmptyMass" dataType="MassKilograms"><DefaultValue>9850</DefaultValue></Parameter>
    <Parameter name="EngineCount" dataType="Count"><DefaultValue>2</DefaultValue></Parameter>
    <Parameter name="PerformanceTable" dataType="PerformanceTable">
      <DefaultValue>
        <Item><Field name="MinAltitude">0</Field><Field name="MaxAltitude">2000</Field><Field name="Throttle">Cruise</Field><Field name="Speed">240</Field><Field name="FuelConsumption">0.55</Field></Item>
      </DefaultValue>
    </Parameter>
    <Parameter name="StallSpeed" dataType="SpeedMetersPerSecond"><DefaultValue>55</DefaultValue></Parameter>
    <Parameter name="MaxSpeed" dataType="SpeedMetersPerSecond"><DefaultValue>531</DefaultValue></Parameter>
  </TechnologicalData>
  <InitializationData>
    <Parameter name="InitialSpeed" dataType="SpeedMetersPerSecond"><DefaultValue>220</DefaultValue></Parameter>
  </InitializationData>
</KinematicDefinition>
```

The real file has one `Item` per altitude band and throttle setting (eight here). The entity then
uses the derived definition as is: `<Kinematic definition="RafaleKinematic"/>`.

## Examples in this repository

- `F35AKinematic` ([`examples/library/equipments-f35.xml`](../../examples/library/equipments-f35.xml)): F-35A kinematic: one Pratt & Whitney F135 turbofan with afterburner, 8,278 kg of internal fuel, Mach 1.6 at altitude.
- `RafaleKinematic` ([`examples/library/equipments-rafale.xml`](../../examples/library/equipments-rafale.xml)): Rafale C kinematic: two Safran M88-2 turbofans with afterburner, 4,700 kg of internal fuel, Mach 1.8 at altitude.
- `FighterKinematic` ([`examples/library/equipments.xml`](../../examples/library/equipments.xml)): Single-engine fighter with afterburner. The performance table has two altitude bands (0-2000 m and 2000-12000 m) for the four throttle settings.

## CMO mapping

See [CMO mapping](../cmo-mapping.md#fixedwingkinematic) for the CMO database field behind each data item.
