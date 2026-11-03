# Signature

Root signature: physical dimensions of the platform. Every signature derives from it.

| | |
|---|---|
| Kind | Equipment definition (`EquipmentDefinition`), `type="Signature"` |
| Defined in | [`library/equipments/signatures.xml`](../../library/equipments/signatures.xml) |
| Base | — (root definition) |
| Derivation chain | [`Signature`](signature.md) |
| Derived in the standard library | [`AircraftSignature`](aircraft-signature.md) |

## Role

`Signature` is the root of every signature model. A signature equipment tells how detectable its
platform is; sensors of other entities read it to decide whether they detect the platform. The root
only carries the physical dimensions of the platform, which every kind of platform has.

An entity archetype carries its signature as an ordinary equipment (by convention named
`Signature`), whose definition derives from `Signature`.

## Technological data

Characteristics of the model, fixed for a given equipment type.

| Data | Type | Unit | Default | Constraints | Description |
|---|---|---|---|---|---|
| `Dimensions` | `Dimensions` | — | — | — | Length, span and height of the platform. |

## Initialization data

None declared by `Signature`.

## Behaviour

- `Dimensions` is a record of `Length` (nose to tail), `Span` (wingtip to wingtip, or beam for a
  ship) and `Height`, in metres.
- The root has no detection data: each kind of platform adds its own in a derived definition
  ([`AircraftSignature`](aircraft-signature.md) for aircraft; ships and submarines will add their
  sonar signatures the same way).

## Constraints and rules

- No constraint is declared on `Dimensions`; its fields are expected to be positive.
- Kind of equipment: `type="Signature"`, inherited by every derived definition that does not set
  its own `type`.

## Deriving a concrete model

```xml
<EquipmentDefinition name="ShipSignature" base="Signature">
  <Description>Surface ship signature: radar, infrared and acoustic.</Description>
  <TechnologicalData>
    <Parameter name="RadiatedNoise" dataType="Decibels">
      <Description>Noise radiated in the water, heard by passive sonars.</Description>
    </Parameter>
  </TechnologicalData>
</EquipmentDefinition>
```

## CMO mapping

See [CMO mapping](../cmo-mapping.md#signature) for the CMO database field behind each data item.
