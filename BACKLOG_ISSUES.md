# Project Ascension – Issue Backlog (Quest 3, UE 5.3.2)

Use this file to create GitHub issues. For each issue:
- Title: use the “Title” line.
- Body: copy everything from “Body (paste into issue)” down to the end of that section.
- Apply Labels and Milestone as indicated.

Labels to create:
- M0, M1, M2, M3, M4, M5
- Blueprint, UI, VR, Android, Performance, Assets, Repo, Optional, Help wanted, Good first issue

Milestones to create:
- M0 – Repo & Project Setup
- M1 – Simulation Bay MVP
- M2 – Experiment Loop & Data Panel
- M3 – Desk Hub & Flow
- M4 – Toolkit & Visualization
- M5 – Performance & Packaging

---

Title: M0: Initialize repo for Unreal (Git LFS, .gitignore, .gitattributes)
Labels: [Repo, M0, Good first issue]
Milestone: M0 – Repo & Project Setup

Body (paste into issue):
Goal:
- Set up the repository for Unreal Engine with Git LFS so binary assets are handled correctly.

Tasks:
- Install Git LFS locally: `git lfs install`
- Add .gitignore suitable for Unreal (exclude Intermediate/, Binaries/, DerivedDataCache/, Saved/)
- Add .gitattributes to track .uasset, .umap, textures, audio, etc. with LFS
- Commit and push a clean repo state
- Document steps in README

Definition of Done:
- New clones do not download build artifacts
- LFS is active; `git lfs ls-files` shows tracked assets

---

Title: M0: UE5 Quest 3 project settings and Android toolchain
Labels: [Android, VR, M0]
Milestone: M0 – Repo & Project Setup

Body (paste into issue):
Goal:
- Configure a UE 5.3.2 project for Quest 3 deployment.

Tasks:
- Enable plugins: OpenXR, OculusXR, Enhanced Input, Niagara
- Rendering: Mobile HDR Off, MSAA 4x, Mobile Multi-View On; Lumen/Nanite Off
- Android: Vulkan On, OpenGL ES Off; ARM64 only; ASTC textures
- OculusXR: Target 90 Hz; Fixed Foveated Rendering Medium
- Verify Android SDK/NDK/JDK setup via Project Settings
- Build a blank Development APK and install to Quest 3

Definition of Done:
- You can package and run a blank map on Quest 3 at 90 Hz

---

Title: M0: Create folder structure and starter maps
Labels: [Repo, M0, Good first issue]
Milestone: M0 – Repo & Project Setup

Body (paste into issue):
Goal:
- Establish content organization and create starter levels.

Tasks:
- Content/Ascension/Core (GameInstance, Save, Data assets)
- Content/Ascension/DeskHub (Map L_DeskHub, UI)
- Content/Ascension/Sim/Motorboat (Map L_Sim_Bay_Motorboat, Blueprints, UI, Materials)
- Content/Ascension/Common (Materials, Niagara, UI)
- Create empty maps: L_DeskHub, L_Sim_Bay_Motorboat

Definition of Done:
- Maps exist and open successfully
- Folders are created and committed

---

Title: M1: Create BPI_CurrentReceiver interface
Labels: [Blueprint, M1, Good first issue]
Milestone: M1 – Simulation Bay MVP

Body (paste into issue):
Goal:
- Provide a simple way for actors to accept a current vector.

Tasks:
- Create Blueprint Interface: BPI_CurrentReceiver
- Function: ApplyCurrent(CurrentVector: Vector)

Definition of Done:
- Interface exists and compiles
- Test call from a dummy actor succeeds

---

Title: M1: Build BP_RiverCurrentVolume (flow direction and speed)
Labels: [Blueprint, VR, M1]
Milestone: M1 – Simulation Bay MVP

Body (paste into issue):
Goal:
- Volume that applies a constant current to overlapping actors.

Tasks:
- Actor with BoxCollision (root)
- Variables: FlowDirection (Vector), FlowSpeed (Float), Affected (Actor Array)
- On BeginPlay: normalize FlowDirection
- On BeginOverlap: if actor implements BPI_CurrentReceiver, add to Affected
- On EndOverlap: remove from Affected
- Tick: For each A in Affected, call ApplyCurrent(FlowDirection * FlowSpeed)
- Visual: optional Niagara/mesh to indicate flow

Definition of Done:
- Overlapping test actors receive current
- Toggling FlowSpeed updates behavior at runtime

---

Title: M1: Build BP_Raft (kinematic drift with current)
Labels: [Blueprint, M1]
Milestone: M1 – Simulation Bay MVP

Body (paste into issue):
Goal:
- Raft that drifts passively with the river current.

Tasks:
- StaticMesh root (simple primitive)
- Implements BPI_CurrentReceiver; stores CurrentVector
- Tick: Velocity = CurrentVector; SetActorLocation(Location + Velocity * DeltaSeconds)
- Function ResetAt(Transform): reset transform and zero velocity

Definition of Done:
- When placed inside current volume, raft drifts consistently
- ResetAt returns raft to start without residual motion

---

Title: M1: Build BP_Boat (kinematic movement + throttle)
Labels: [Blueprint, M1]
Milestone: M1 – Simulation Bay MVP

Body (paste into issue):
Goal:
- Boat with forward throttle combined with current.

Tasks:
- StaticMesh root; optional Arrow for forward
- Variables: Throttle (0..1), MaxSpeed, CurrentVector, Velocity
- Implements BPI_CurrentReceiver
- Function SetThrottle(float)
- Tick: ForwardVel = ForwardVector * Throttle * MaxSpeed; Velocity = ForwardVel + CurrentVector; move by Velocity * DeltaSeconds
- Function ResetAt(Transform)

Definition of Done:
- Boat responds to throttle and current
- No jitter; speed caps correctly

---

Title: M1: Build W_Throttle + BP_ControlPod (3D widget slider)
Labels: [UI, Blueprint, VR, M1]
Milestone: M1 – Simulation Bay MVP

Body (paste into issue):
Goal:
- Stationary control pod with a world-space throttle slider for VR interaction.

Tasks:
- Widget W_Throttle with Slider (0..1)
- Actor BP_ControlPod with WidgetComponent (W_Throttle)
- VR Pawn: add WidgetInteraction component to right controller
- Bind trigger button to PressPointerKey/ReleasePointerKey (LeftMouseButton)
- Slider.OnValueChanged -> Boat.SetThrottle(Value)

Definition of Done:
- In headset, you can point and drag the slider to control the boat
- No locomotion required; player remains stationary

---

Title: M1: Energy river visual (unlit material or Niagara ribbons)
Labels: [Assets, Performance, M1]
Milestone: M1 – Simulation Bay MVP

Body (paste into issue):
Goal:
- Stylized “flow” visual that is performant on Quest.

Tasks:
- Create M_EnergyFlow (Unlit, panning emissive)
- Apply to a spline mesh or long plane along river path
- Optional: Niagara particles/ribbons moving along FlowDirection (low spawn rate)

Definition of Done:
- Flow direction is obvious
- GPU cost is minimal on-device

---

Title: M1: Place Sim Bay 1.1 layout (Start A, End C, control pod)
Labels: [Blueprint, VR, M1]
Milestone: M1 – Simulation Bay MVP

Body (paste into issue):
Goal:
- Assemble the scene with start/end points and the control pod.

Tasks:
- Place StartPoint (Empty Actor at A)
- Place EndTrigger (BoxCollision at C)
- Place BP_RiverCurrentVolume covering river area
- Place BP_Raft and BP_Boat at A
- Place BP_ControlPod within easy reach/view

Definition of Done:
- You can interact with throttle and observe raft/boat behavior
- EndTrigger overlaps raft when it reaches C

---

Title: M2: Build BP_SimController (start/reset, timer, freeze)
Labels: [Blueprint, M2]
Milestone: M2 – Experiment Loop & Data Panel

Body (paste into issue):
Goal:
- Orchestrate experiment start/stop and data capture.

Tasks:
- Variables: BoatRef, RaftRef, CurrentVolRef, StartPoint, EndTrigger
- DistanceL (known) and Tau (measured)
- StartExperiment(): ResetAt(StartPoint), start timer, set running flag
- Bind EndTrigger overlap: if raft overlaps and running, compute Tau, set running false, freeze Boat/Raft (CustomTimeDilation = 0), show data panel

Definition of Done:
- Start/Stop loop works reliably
- Tau matches observed time

---

Title: M2: Build W_DataPanel (show l, τ, keypad, tolerance check)
Labels: [UI, Blueprint, M2]
Milestone: M2 – Experiment Loop & Data Panel

Body (paste into issue):
Goal:
- UI that displays data and validates the user’s answer.

Tasks:
- Inputs: DistanceL, Tau, TrueFlowSpeed, TolerancePct
- Show DistanceL and Tau
- Numeric keypad: digits 0–9, Clear, Backspace, Submit
- Submit: parse to float, compare to TrueFlowSpeed within tolerance, show Correct/Incorrect with hint (too high/low)

Definition of Done:
- Correct answers are accepted; incorrect gives feedback
- “Return to Desk” button available on correct

---

Title: M2: Freeze/unfreeze sim actors without pausing UI
Labels: [Blueprint, M2, Good first issue]
Milestone: M2 – Experiment Loop & Data Panel

Body (paste into issue):
Goal:
- Freeze boat and raft while leaving UI responsive.

Tasks:
- Implement helper on SimController: SetSimPaused(bool)
- Use CustomTimeDilation = 0 on sim actors; 1 on PlayerController/UI
- Unfreeze on reset or when leaving level

Definition of Done:
- UI remains interactive during freeze
- Simulation resumes cleanly

---

Title: M2: Level travel back to Desk Hub after success
Labels: [Blueprint, UI, M2]
Milestone: M2 – Experiment Loop & Data Panel

Body (paste into issue):
Goal:
- Return to the hub after successful answer entry.

Tasks:
- “Return to Desk” button: unfreeze sim actors, save result to GameInstance, OpenLevel(L_DeskHub)
- On Desk, show success state for the problem

Definition of Done:
- Travel is seamless; no crashes
- Result is visible in Desk UI

---

Title: M3: Build Desk Hub (L_DeskHub) with Earth window
Labels: [Assets, UI, M3]
Milestone: M3 – Desk Hub & Flow

Body (paste into issue):
Goal:
- Create a minimal, performant hub with a holographic desk UI.

Tasks:
- Earth view: unlit skybox/cubemap or sphere
- Place desk mesh (simple), add W_DeskPanel widget (world-space)
- Performance-friendly lighting (static/unlit)

Definition of Done:
- Hub loads fast on-device
- Visuals match “minimalist construct + window to Earth” vibe

---

Title: M3: W_DeskPanel (Transmission, Start Simulation)
Labels: [UI, M3]
Milestone: M3 – Desk Hub & Flow

Body (paste into issue):
Goal:
- Problem selection and start button UI.

Tasks:
- Show paraphrased “Transmission” text for Motorboat & Raft
- Button: “Begin Experiment”
- On click: OpenLevel(L_Sim_Bay_Motorboat)

Definition of Done:
- Player can start the experiment from the desk

---

Title: M3: GameInstance to hold problem data/state
Labels: [Blueprint, M3]
Milestone: M3 – Desk Hub & Flow

Body (paste into issue):
Goal:
- Persist simple data between levels.

Tasks:
- Create BP_AscensionGameInstance
- Variables: CurrentProblemID, LastResult (Passed/Failed), DistanceL, Tau
- Set as Project’s GameInstance class

Definition of Done:
- Data persists from Sim back to Desk

---

Title: M4: Velocity Probe actor (visualize velocity vector)
Labels: [Blueprint, M4]
Milestone: M4 – Toolkit & Visualization

Body (paste into issue):
Goal:
- Visual probe you can attach to boat/raft showing velocity arrow.

Tasks:
- Actor with Arrow/mesh; Update each Tick:
  - GetVelocity (from your own Velocity var)
  - Set arrow rotation to direction; scale by magnitude
- Attach/detach via simple button in UI

Definition of Done:
- Probe displays correct direction/magnitude
- Low overhead on device

---

Title: M4: Pause/Resume toggle in Sim Bay
Labels: [UI, Blueprint, M4, Good first issue]
Milestone: M4 – Toolkit & Visualization

Body (paste into issue):
Goal:
- Button to pause/resume simulation during experimentation.

Tasks:
- UI toggle in Sim Bay
- Calls SimController.SetSimPaused(true/false)
- Visual indicator when paused

Definition of Done:
- Simulation halts/resumes reliably
- UI always responsive

---

Title: M4: Optional – Short rewind buffer (state snapshots)
Labels: [Blueprint, Optional, M4]
Milestone: M4 – Toolkit & Visualization

Body (paste into issue):
Goal:
- Record a few seconds of state and scrub.

Tasks:
- Buffer array of structs per frame: Transform, Velocity, Throttle
- Scrub slider to restore past state
- Keep buffer small for mobile (e.g., 3–5 seconds at 20 Hz)

Definition of Done:
- Scrub works for short windows without hitches

---

Title: M5: Performance pass on Quest 3 (profiling + optimizations)
Labels: [Performance, Android, M5]
Milestone: M5 – Performance & Packaging

Body (paste into issue):
Goal:
- Hit 90 Hz comfortably on device.

Tasks:
- Use OVR Metrics Tool to monitor CPU/GPU/frame time
- Check stat unit and stat gpu on device
- Tune: MSAA 4x, FFR Medium; reduce particle counts; ensure Unlit materials
- Consider r.MobileContentScaleFactor 0.9 if needed

Definition of Done:
- Frame time < 11.1 ms in the Sim Bay scene

---

Title: M5: Shipping build to Quest 3 + QA checklist
Labels: [Android, Performance, M5]
Milestone: M5 – Performance & Packaging

Body (paste into issue):
Goal:
- Produce a Shipping build and validate.

Tasks:
- Package Shipping (arm64, Vulkan)
- Install and test 10-minute session (no hitches/overheats)
- Validate UI readability and controller interaction accuracy
- Document steps for next builds

Definition of Done:
- Stable Shipping APK installed and tested

---

Title: Assets: Import minimal Fab assets (unlit grid, neon materials)
Labels: [Assets, M1]
Milestone: M1 – Simulation Bay MVP

Body (paste into issue):
Goal:
- Bring in a few free assets for the construct look.

Tasks:
- From Fab: add an unlit grid floor and neon/hologram material pack
- Replace placeholder materials in Sim Bay
- Keep draw calls low; avoid heavy shaders

Definition of Done:
- Visuals improved; performance unaffected

---

Title: Optional: Replace slider with physical lever (grab + constraint)
Labels: [Blueprint, Optional, M4]
Milestone: M4 – Toolkit & Visualization

Body (paste into issue):
Goal:
- A tactile lever control.

Tasks:
- Lever mesh + PhysicsConstraint
- Grab interaction to rotate within limits
- Map lever angle (0..1) to Boat.SetThrottle

Definition of Done:
- Lever feels responsive and precise

---

Title: Optional: Hand tracking controls (Meta XR) for the throttle
Labels: [VR, Optional, M4]
Milestone: M4 – Toolkit & Visualization

Body (paste into issue):
Goal:
- Use hands instead of controllers for the throttle UI.

Tasks:
- Enable OculusXR Hand Tracking
- Add hand pointers or pinch interaction for 3D widget
- Test stability and precision

Definition of Done:
- Throttle usable by hand in-headset

---

Title: Optional: MR Passthrough experiment (Quest 3)
Labels: [VR, Optional, M5]
Milestone: M5 – Performance & Packaging

Body (paste into issue):
Goal:
- Try the Simulation Bay with background passthrough.

Tasks:
- Enable OculusXR MR plugin
- Request MR permission, display passthrough layer behind construct elements
- Test readability and contrast

Definition of Done:
- Passthrough mode works and is comfortable

---

Title: Data: Problem definition asset structure (DataTable/PrimaryDataAsset)
Labels: [Blueprint, M3]
Milestone: M3 – Desk Hub & Flow

Body (paste into issue):
Goal:
- Author problems as data instead of hardcoding.

Tasks:
- Create PrimaryDataAsset: ProblemDef (ID, Title, ShortText, SimMap, FlowSpeed, DistanceL, BoatMaxSpeed, TolerancePct)
- Load selected problem from Desk and pass to Sim

Definition of Done:
- Changing data updates sim parameters without code changes

---

Title: Repo hygiene: Protect main branch and require PRs
Labels: [Repo, M0]
Milestone: M0 – Repo & Project Setup

Body (paste into issue):
Goal:
- Prevent accidental commits to main.

Tasks:
- Settings > Branches > Add rule: protect main (require PR, dismiss stale reviews)
- Enable linear history (optional)
- Document branch naming: feature/<short-name>

Definition of Done:
- Pushing to main is restricted; PR flow in use

---

Title: CI: Optional – GitHub Actions for LFS integrity check
Labels: [Repo, Optional, M0]
Milestone: M0 – Repo & Project Setup

Body (paste into issue):
Goal:
- Ensure .uasset/.umap are LFS-tracked in PRs.

Tasks:
- Add lightweight workflow to fail PR if binary extensions are not LFS
- Document in README

Definition of Done:
- PRs fail if assets bypass LFS

---

Title: Controls: VR Pawn with WidgetInteraction bindings
Labels: [Blueprint, VR, M1]
Milestone: M1 – Simulation Bay MVP

Body (paste into issue):
Goal:
- Basic VR pawn with controller ray to interact with 3D widgets.

Tasks:
- Add WidgetInteraction component to right-hand controller
- Map trigger to PressPointerKey/ReleasePointerKey (LeftMouseButton)
- Visible laser pointer (optional: simple line/beam)

Definition of Done:
- You can precisely interact with W_Throttle and W_DataPanel

---

Title: Sim Bay: EndTrigger placement and validation
Labels: [Blueprint, M2, Good first issue]
Milestone: M2 – Experiment Loop & Data Panel

Body (paste into issue):
Goal:
- Ensure end-of-experiment detection is robust.

Tasks:
- Box collision sized to always catch raft
- Overlap filters set (Object types)
- Debug print or UI when triggered

Definition of Done:
- End reliably fires with raft and not the boat (unless intended)

---

Title: Desk: Revelation summary card after success
Labels: [UI, M3]
Milestone: M3 – Desk Hub & Flow

Body (paste into issue):
Goal:
- Show key takeaways.

Tasks:
- Panel listing: relative velocity, frames of reference, relationship of v_flow, l, τ
- “Next” button (placeholder for future problems)

Definition of Done:
- Clear, concise summary visible after success

---

Title: Optional: Water/Chaos exploration spike (time-boxed)
Labels: [Optional, Performance, M5]
Milestone: M5 – Performance & Packaging

Body (paste into issue):
Goal:
- Investigate feasibility of Water Body River or physics forces for the boat.

Tasks:
- Build small test map using Water (unlit simplification) OR physics-based force model
- Measure on-device perf (OVR Metrics)
- Decide go/no-go for future

Definition of Done:
- Documented findings with frame times and pros/cons

---

Title: QA: On-device comfort & readability checklist
Labels: [Performance, VR, M5]
Milestone: M5 – Performance & Packaging

Body (paste into issue):
Goal:
- Ensure user comfort and clear UI.

Tasks:
- Check no unintended camera motion
- UI text legible at normal distances
- High-contrast materials; avoid excessive bloom
- Verify FPS stability in hottest scene

Definition of Done:
- Passes all checks; notes recorded

---

Title: Assets: Add minimal UI SFX for interactions
Labels: [Assets, UI, M2]
Milestone: M2 – Experiment Loop & Data Panel

Body (paste into issue):
Goal:
- Audio feedback for button presses and correct/incorrect answers.

Tasks:
- Import small SFX pack (click, confirm, error)
- Play on keypad button press and submit feedback

Definition of Done:
- Clear, non-intrusive audio cues on-device

---

Title: DX Notes: README section for build/deploy steps
Labels: [Repo, M0, Good first issue]
Milestone: M0 – Repo & Project Setup

Body (paste into issue):
Goal:
- Document how to package, install, and profile.

Tasks:
- Update README with: package settings, adb install, OVR Metrics, performance tips

Definition of Done:
- New contributors can follow steps end-to-end