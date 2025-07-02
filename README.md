_RECORDING ETEE PRESSURE DATA FROM UNREAL ENGINE 5.3_

This repo explains how to record data from **Etee VR Controllers" using Unreal Engine and stream the data via Lab Streaming Layer (LSL).

## Table of Contents
- [SteamVR Setup](#-steamvr-setup)
- [Unreal Engine Setup](#-unreal-engine-setup)
  - [Input Mapping](#input-mapping)
  - [VR Pawn](#vr-pawn)
- [LSL Cube Actor](#-lsl-cube-actor)
  - [Components](#components)
  - [Event Graph Logic](#event-graph-logic)
- [Data Structure](#-data-structure)
- [Data Analysis](#-data-analysis)

## SteamVR Setup

### A. Configure Input Bindings

1. Launch **SteamVR**.
2. Go to **Devices > Controller Settings > Manage Bindings**. (Do this after starting the **VR Preview** of your Unreal project!)
3. Choose the input mapping context: `IMC_hands`.

<img src="./images/imc-hands-bindings.png" alt="imc-hands-bindings" width="600"/> 

### B. Assign Trackers (This step is a reminder. Etee VR Controllers can be used with Quest headsets with Vive Ultimate trackers. Ensure that Vive ultimate trackers are assigned to the accurate hands for accurate alibration. Another repo in this projects's GitHub explains how to use Etee VR Controllers and Vive Ultimate trackers.)
1. Open **SteamVR Settings > Manage Trackers**.
2. Assign **Vive Ultimate Trackers** to:
   - Right Hand
   - Left Hand

<img src="./images/manage-trackers-right.png" alt="manage-trackers-right" width="600"/> 
<img src="./images/manage-trackers-left.png" alt="manage-trackers-left" width="600"/> 

## Unreal Engine Setup

### 🎯 Input Mapping

#### In `IMC_Hands`:

Add Enhanced Input mappings:

- **Finger Curl (Each Hand)**:
  - `IA_Hand_ThumbCurl_Left/Right`
  - `IA_Hand_IndexCurl_Left/Right`
  - `IA_Hand_MiddleCurl_Left/Right`
  - `IA_Hand_RingCurl_Left/Right`
  - `IA_Hand_PinkyCurl_Left/Right`

<img src="./images/imc_hands_UE.png" alt="imc_hands_UE" width="600"/> 

### 🧍 VR Pawn

Inside your `VRPawn` Blueprint:

- Create float variables:
  - `RH_Pressure`
  - `LH_Pressure`

These represent total pressure from the right and left hands.

#### In Event Graph:

For each input:

1. Use the `Input Action` node (e.g. `IA_Hand_IndexCurl_Right`)
2. Connect to a `Set` node to update `RH_Pressure`
3. Store values in a pressure array with 5 elements:

0: Index
1: Middle
2: Ring
3: Pinky
4: Thumb

Repeat the same for `LH_Pressure`.

<img src="./images/VRPawn_Event_graph.png" alt="VRPawn_Event_graph" width="600"/>     
<img src="./images/VRPawn_LH.png" alt="VRPawn_LH" width="600"/>        
<img src="./images/VRPawn_RH.png" alt="VRPawn_RH" width="600"/> 

## LSL Cube

Create a new Actor Blueprint named `LSL_Cube`.

### Components

- `LSLOutletMarkers`
- `LSLOutlet_Headset`
- `LSLOutlet_RH`
- `LSLOutlet_LH`

These manage LSL stream outlets for data streaming to LSL from the controllers including position and orientation of the controllers and pressure data from each finger.
### Event Graph Logic

#### Step-by-step:

1. **Event Tick** → `Get Player Pawn`
2. Cast to `VRPawn`
3. Access `RH_Pressure` and pressure array
4. Use 5 `Get` nodes to extract individual finger values
5. Retrieve `Hand_Right` socket location and rotation
- Use `Get Socket Location` and `Get Socket Rotation`
- Break the vectors into X/Y/Z.

6. Create a final array using `Make Array`:
Index 0: Position X
Index 1: Position Y
Index 2: Position Z
Index 3: Orientation X (Pitch)
Index 4: Orientation Y (Yaw)
Index 5: Orientation Z (Roll)
Index 6: Index Pressure
Index 7: Middle Pressure
Index 8: Ring Pressure
Index 9: Pinky Pressure
Index 10: Thumb Pressure

7. Send this array using `Push Sample` to `LSLOutlet_RH`.

Repeat for left hand using `LSLOutlet_LH`.

<img src="./images/LSL-cube.png" alt="LSL-cube" width="600"/> 
<img src="./images/LSL_Components.png" alt="LSL_Components" width="600"/> 
<img src="./images/LSLOutlet_RHand_Details.png" alt="LSLOutlet_RHand_Details" width="600"/> 

## 📊 Data Analysis

Once streaming:

- Use tools like:
- `LabRecorder` to record `.xdf` files. 







