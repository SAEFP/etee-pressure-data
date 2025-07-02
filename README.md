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
- [Data Recording for LSL](#-data-recording)

## SteamVR Setup

### A. Configure Input Bindings

1. Launch **SteamVR**.
2. Go to **Devices > Controller Settings > Manage Bindings**. (Do this after starting the **VR Preview** of your Unreal project!)
3. Choose the input mapping context: `IMC_hands.

Through editting the IMC_Hands, Unreal Engine will know how to interpret the input data from the controllers.

For the current project, only the trackpad is used as a button. 

<img src="./images/imc-hands-bindings.png" alt="imc-hands-bindings" width="600"/> 

### B. Assign Trackers 

(This step is a reminder. Etee VR Controllers can be used with Quest headsets with Vive Ultimate trackers. Ensure that Vive ultimate trackers are assigned to the accurate hands for accurate alibration. Another repo in this projects's GitHub explains how to use Etee VR Controllers and Vive Ultimate trackers.)

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

1. Use the `Input Action` node for each finger (e.g. `IA_Hand_IndexCurl_Right`)

Input Actions which are events that happens when you move or press something (like curling your index finger) is used.

For each finger (thumb, index, middle, etc.), we set up an input action, like:

IA_Hand_IndexCurl_Right

IA_Hand_ThumbCurl_Left

2. Connect to a `Set` node to update `RH_Pressure`

When an input action is triggered (for example, when you curl your index finger), we use a Set node to store that finger’s pressure value in a variable.

3. Store values in a pressure array with 5 elements:

0: Index
1: Middle
2: Ring
3: Pinky
4: Thumb

We collect all finger pressures in a list (called an array). 

Repeat the same for `LH_Pressure`.

<img src="./images/VRPawn_Event_graph.png" alt="VRPawn_Event_graph" width="600"/>     
<img src="./images/VRPawn_LH.png" alt="VRPawn_LH" width="600"/>        
<img src="./images/VRPawn_RH.png" alt="VRPawn_RH" width="600"/> 

## LSL Cube

We created another Blueprint actor called the  `LSL_Cube`. It runs constantly (using Event Tick, which updates every frame). It records the finger pressure values and hand position/orientation and headset position/orientation. It combines everything into one array (a long list of values).

### Components in the LSL Cube

- `LSLOutletMarkers`
- `LSLOutlet_Headset`
- `LSLOutlet_RH`
- `LSLOutlet_LH`
<img src="./images/LSL_Components.png" alt="LSL_Components" width="600"/> 

These LSL stream outlets streams all the data to LSL including position and orientation of the controllers and the headset and pressure data from each finger.

### Event Graph Logic

#### Step-by-step:

1. **Event Tick** → `Get Player Pawn`
2. Cast to `VRPawn`
3. Access `RH_Pressure` and pressure array
4. Use 5 `Get` nodes to extract individual finger values
5. Retrieve `Hand_Right` socket location and rotation
- Use `Get Socket Location` and `Get Socket Rotation`
- Break the vectors into X/Y/Z and Pitch/Yaw/Roll.

We also get the hand’s position and rotation from the VR Pawn by checking the socket on the hand. This gives us:

Position (X, Y, Z)

Rotation (Pitch, Yaw, Roll)

<img src="./images/LSL-cube.png" alt="LSL-cube" width="600"/> 

6. Create a final array using `Make Array`and identify the name of the each index for each hand:
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

<img src="./images/LSLOutlet_RHand_Details.png" alt="LSLOutlet_RHand_Details" width="600"/> 

7. Send this array using `Push Sample` to `LSLOutlet_RH`.

Repeat for left hand using `LSLOutlet_LH`.



## 📊 Data Analysis

Once streaming:

- Use tools like:
- `LabRecorder` to record `.xdf` files. 







