# Kinematic tasks

Movement tasks of the XCGF standard library (`library/tasks/kinematics.xml`), modelled on the
movement orders of Command: Modern Operations (CMO). The kinematic definitions reference them with
`TaskRef` (`library/equipments/kinematics.xml`), so every entity gets the tasks of its kinematic,
and a kinematic derived from `FixedWingKinematic` (for instance `RafaleKinematic`) inherits all of
them.

| Task | Carried by | CMO counterpart | Ends with `Success` when |
|---|---|---|---|
| `MoveTo` | `Kinematic` | course with a single waypoint | the destination is reached (500 m) |
| `FollowCourse` | `Kinematic` | plotted course | the last waypoint is reached (never when `Loop`) |
| `HoldPosition` | `Kinematic` | all stop / orbit in place | `Duration` has elapsed |
| `SetAltitude` | `FixedWingKinematic` | altitude setting, terrain following | the altitude is reached (30 m) |
| `Orbit` | `FixedWingKinematic` | station keeping, patrol racetrack | `Duration` on station has elapsed |
| `ReturnToBase` | `FixedWingKinematic` | RTB, bingo fuel | the aircraft is overhead the base (2 km) |
| `TakeOff` | `FixedWingKinematic` | launch from an airfield | the climb altitude is reached |
| `Land` | `FixedWingKinematic` | landing | the aircraft has landed |

Every task starts immediately (`Immediate`) and ends with `Aborted` on the `OrderCancelled`
event. `WhoTaskee` holds the moving entity, which defaults to the carrier of the task. None of
these tasks has a `WhoTasker` or a `WhoAffected`: they only move the entity that performs them.

## Parameters

Required parameters are in bold. `Out` parameters are set by the task.

| Task | What | When | Where | Why |
|---|---|---|---|---|
| `MoveTo` | `Throttle` (Cruise), `Speed` | `TravelTime` (Out) | **`Destination`** | |
| `FollowCourse` | `Throttle` (Cruise), `Loop` (false) | | **`Waypoints`**, `CurrentWaypoint` (InOut, 0) | |
| `HoldPosition` | | `Duration` | `HoldPoint` (Out) | |
| `SetAltitude` | **`Altitude`**, `TerrainFollowing` (false), `VerticalSpeed` | | | |
| `Orbit` | `Throttle` (Loiter), `Speed` | `Duration` | **`Center`**, `Radius` (5000 m), `LegLength` (0 m), `LegHeading` (0°) | |
| `ReturnToBase` | `Throttle` (Cruise) | | **`Base`** | `BingoFuel` |
| `TakeOff` | `ClimbAltitude` (300 m) | | **`Runway`**, **`RunwayHeading`**, `RunwayLength` | |
| `Land` | | | **`Runway`**, **`RunwayHeading`**, `RunwayLength` | |

- **Speed and throttle.** As in CMO, a movement is ordered by a throttle setting (`ThrottleEnum`:
  Loiter, Cruise, Full, Flank); an explicit `Speed` takes precedence over it. The kinematic turns
  either into a target speed (see `setOrder` in [FixedWingKinematic](../models/fixed-wing-kinematic.md)).
- **Altitudes** come from the `GeoPosition` of the destination, of the waypoints, of the orbit
  centre and of the runway (field elevation). `SetAltitude` changes it without changing the route.
- **Orbit.** `LegLength` = 0 flies a circle of radius `Radius`; otherwise the aircraft flies a
  racetrack whose inbound leg has heading `LegHeading`.
- **ReturnToBase** ends overhead the base; chain a `Land` task to land. When `BingoFuel` is set,
  the tool starts the task on its own as soon as the fuel falls to it, as CMO does.

## Constraints added by the kinematics

The generic tasks only bound their parameters by literal values. The `TaskRef` of
`FixedWingKinematic` adds bounds read from its technological data, so a scenario order outside the
flight envelope is rejected:

| Task | Parameter | Constraint |
|---|---|---|
| `MoveTo` | `Speed` | ≥ `StallSpeed`, ≤ `MaxSpeed` |
| `Orbit` | `Speed` | ≥ `StallSpeed`, ≤ `MaxSpeed` |
| `SetAltitude` | `Altitude` | ≤ `Ceiling` |
| `SetAltitude` | `VerticalSpeed` | ≤ `ClimbRate` |
| `ReturnToBase` | `BingoFuel` | ≤ `FuelCapacity` |
| `TakeOff`, `Land` | `RunwayLength` | ≥ `MinRunwayLength` |

The speeds of the `FollowCourse` waypoints are fields of a record (`FlightPlanWaypoint`), which a
constraint cannot reach: the kinematic clamps them when it flies the course.

## Triggers interpreted by the tools

Besides the standard triggers, these tasks use `CustomTrigger` kinds and events the tools must
implement:

| Kind or event | Meaning |
|---|---|
| `CourseCompleted` (`parameter`, `tolerance`) | last waypoint of the course reached, within `tolerance` metres |
| `DurationElapsed` (`parameter`, `from`) | the duration held in `parameter` has elapsed since the task started, or since the `from` event (`StationReached` for `Orbit`); never fires when the parameter is absent |
| `AltitudeReached` (`parameter`, `tolerance`, `reference`) | altitude in `parameter` reached within `tolerance` metres; above the `reference` position when given |
| `OrderCancelled` | the order was cancelled or replaced by another movement order |
| `Landed` | the aircraft touched down and stopped on the runway |
