# Automated Parcel Sorting Line
[Demo](media/gif.gif)

PLC-controlled parcel sorting simulation built with **CODESYS** and **Factory I/O**, communicating over **Modbus TCP**.

The system releases parcels one at a time from an infeed conveyor, classifies each parcel by height, and routes it to one of two chutes using pneumatic pushers.

## Features

- Single-piece parcel release from a feeder conveyor
- Automatic transfer to the main conveyor
- Parcel height classification using two diffuse sensors
- Large parcel routing to **Pusher 1**
- Small parcel routing to **Pusher 2**
- Position-aware handling of long parcels
- Pusher motion confirmed with front/back limit switches
- Start / Stop / Reset control
- Structured Text state machine
- Modbus TCP communication between CODESYS and Factory I/O

## Technology

- CODESYS V3.5 SP22 Patch 3
- Factory I/O 2.4.3
- IEC 61131-3 Structured Text (ST)
- Modbus TCP/IP
- CODESYS Control Win V3 x64


### Classification

| Low Sensor | High Sensor | Classification | Route |
|---|---|---|---|
| TRUE | TRUE | Large | Pusher 1 |
| TRUE | FALSE | Small | Pusher 2 |

The PLC latches the `HighSensor` detection during the classification window so the decision is not lost when the parcel continues moving.

## State Machine

| State | Function |
|---:|---|
| 0 | Idle |
| 5 | Feed next parcel |
| 6 | Complete transfer to main conveyor |
| 10 | Travel to size sensors |
| 15 | Classify parcel |
| 20 | Position large parcel at Pusher 1 |
| 30 | Extend Pusher 1 |
| 31 | Retract Pusher 1 |
| 40 | Small parcel travels to Sensor P2 |
| 41 | Wait for small parcel trailing edge |
| 42 | Final positioning at Pusher 2 |
| 50 | Extend Pusher 2 |
| 51 | Retract Pusher 2 |
| 60 | Cycle complete / release next parcel |

## Factory I/O / Modbus Mapping

### Factory I/O -> CODESYS

| Modbus | Factory I/O signal | CODESYS variable |
|---|---|---|
| Coil 0 | Start Button | `GVL_FIO.Start` |
| Coil 1 | Stop Button | `GVL_FIO.Stop` |
| Coil 2 | Reset input* | `GVL_FIO.Reset` |
| Coil 3 | Entry diffuse sensor | `GVL_FIO.EntrySensor` |
| Coil 4 | Low size sensor | `GVL_FIO.LowSensor` |
| Coil 5 | High size sensor | `GVL_FIO.HighSensor` |
| Coil 6 | Sensor before Pusher 2 | `GVL_FIO.SensorP2` |
| Coil 7 | Pusher 1 Back Limit | `GVL_FIO.Pusher1Back` |
| Coil 8 | Pusher 1 Front Limit | `GVL_FIO.Pusher1Front` |
| Coil 9 | Pusher 2 Front Limit | `GVL_FIO.Pusher2Front` |
| Coil 10 | Pusher 2 Back Limit | `GVL_FIO.Pusher2Back` |

### CODESYS -> Factory I/O

| Modbus | Factory I/O actuator | CODESYS variable |
|---|---|---|
| Input 0 | Main Belt Conveyor (6 m) | `GVL_FIO.MainConveyor` |
| Input 1 | Pusher 1 | `GVL_FIO.Pusher1` |
| Input 2 | Pusher 2 | `GVL_FIO.Pusher2` |
| Input 3 | Feeder Belt Conveyor (2 m) | `GVL_FIO.FeederConveyor` |

Factory I/O driver configuration:

- Digital Inputs: Offset `0`, Count `11`
- Digital Outputs: Offset `0`, Count `4`
- Register Inputs: `0`
- Register Outputs: `0`

\* In the current simulation scene the reset signal may be mapped to an Emergency Stop component for testing. For a polished demo, use a dedicated Reset button or clearly label it as simulation-only reset logic.

## CODESYS Mapping

Because the Modbus server exposes the coils as bytes:

```text
Coils[0].Bit0 -> Start
Coils[0].Bit1 -> Stop
Coils[0].Bit2 -> Reset
Coils[0].Bit3 -> EntrySensor
Coils[0].Bit4 -> LowSensor
Coils[0].Bit5 -> HighSensor
Coils[0].Bit6 -> SensorP2
Coils[0].Bit7 -> Pusher1Back

Coils[1].Bit0 -> Pusher1Front
Coils[1].Bit1 -> Pusher2Front
Coils[1].Bit2 -> Pusher2Back
```

Outputs:

```text
Discrete Inputs[0].Bit0 -> MainConveyor
Discrete Inputs[0].Bit1 -> Pusher1
Discrete Inputs[0].Bit2 -> Pusher2
Discrete Inputs[0].Bit3 -> FeederConveyor
```

## Timing Parameters

The current simulation uses:

```iecst
T_Classify := T#150MS;
T_Pos1     := T#200MS;
T_Pos2     := T#150MS;
```

`T_Pos2` starts only after the trailing edge of the small parcel clears `SensorP2`, which makes the second sorting station work reliably with long parcels.



## Running the Project

1. Start CODESYS Control Win V3 x64.
2. Open the CODESYS project.
3. Start the Modbus TCP server on `127.0.0.1:502`.
4. In Factory I/O, select the Modbus TCP/IP Client driver.
5. Configure the driver using the mapping above.
6. Login to the PLC, download the application, and switch it to RUN.
7. Connect Factory I/O and start the scene.
8. Press Start.

## Demo
A good portfolio demo should show:

1. Parcels waiting on the feeder conveyor.
2. One parcel entering the main conveyor.
3. The feeder stopping after the parcel fully transfers.
4. A large parcel being routed by Pusher 1.
5. A small parcel being routed by Pusher 2.
6. CODESYS Online/Debug showing `State`, `BoxType`, sensors and outputs.


## Notes

This project is a simulation and is intended for PLC/automation learning and portfolio demonstration. It is not a safety-rated machine control implementation.
