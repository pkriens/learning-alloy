---
---
# Behaviour of a vehicle locking system

```alloy
module VehicleLocking

open Vehicle //includes RemoteFob, opens Door, 
open Location
open Lock
open Transit

```
##cross-module facts
```alloy
fact doorsAreOnOneVehicleOnly{
    doors in Vehicle one -> Door
}

fact noUnneededLocations{
    all l: Location |
        some v: Vehicle |
            l in v.locations
}

fact fobsAndVehiclesArePaired{
}
```

## Checks
The below claims should be true of the above model:

```alloy
pred aVehicleHasLocationsForDoors{
    all v: Vehicle |  v.locations != none
}

pred aDoorBelongsToAVehicle{
    all d: Door |
        one v: Vehicle |
            d in v.doors
}

pred aDoorIsInALocationOnAVehicle{
    all d: Door |
        one v: Vehicle |
            one l: v.locations |
                d = v.doorAt[l]
}

pred aDoorIsOnOneVehicleOnly{
    all disj u, v: Vehicle |
        no v.doors & u.doors
} 

pred aDoorIsInOneLocationOnly{
    all v: Vehicle |
        all d: v.doors |
            one v.doorAt :> d
}

pred doorsChangeStateTogether{
    all v: Vehicle | 
        no v.doors.lockState :> Locked or
        no v.doors.lockState :> Unlocked
}

pred aFobWorksOnlyOneVehicle{
    all v: Vehicle |
        all f: RemoteFob |
            v.fob = f <=> f.vehicle = v
}       

//it's a shame that we can't organize assertions this way, too. [or can we!?]
pred validStructure{
    aVehicleHasLocationsForDoors
    aDoorBelongsToAVehicle
    aDoorIsInOneLocationOnly

    aFobWorksOnlyOneVehicle
}

pred consistentBehaviour{
    doorsChangeStateTogether
}

pred vehicleModel{
    validStructure
    consistentBehaviour
}
```
Concrete example drawn from 2019 Ford Transit users handbook.

```alloy

check theModelAsChecked{vehicleModel} for exactly 2 Transit, 2 RemoteFob, 8 Door, 4 Location expect 0 //counterexamples
```
##Behaviour
```alloy

pred doorsAreLocked{
    all v: Vehicle |
        all d: v.doors |
            d.locked
}

pred begin{
    doorsAreLocked
}

pred someValidChanges{
    always {
         some f: RemoteFob |
            let v = f.vehicle |
             (f.lockCommanded   and v.allDoorsLocked   and allVehiclesUnchagedExcept[v] ) or
             (f.unlockCommanded and v.allDoorsUnlocked and allVehiclesUnchagedExcept[v]) or
             f.vehicle.unchanged
    }
}

pred targetState{
    eventually {
        all d: Door | d.unlocked
    }
}

pred trace{
    begin
    someValidChanges
    targetState
}

//As  we should see demonstrated thus:
//run singleVanBehaviour{trace} for exactly 1 Transit, 1 RemoteFob, 8 Door, 4 Location, 4 Time
run multipleVanBehaviour{trace} for exactly 2 Transit, 2 RemoteFob, 8 Door, 4 Location, 4 Time

```
## Utilities

```alloy
let allVehiclesUnchagedExcept[v] = all vv: Vehicle - v | vv.unchanged 
```