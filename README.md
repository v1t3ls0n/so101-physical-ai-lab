<div align="center">

# 🦾 SO-101 Control

**A robot you talk to.** It answers in its own voice, thinks with Claude, learns skills from your hands, moves only within limits it cannot negotiate, and shows you everything it did.

Two SO-101 arms · a Jetson Orin Nano · LeRobot underneath · your browser anywhere on the tailnet

![python](https://img.shields.io/badge/python-3.12-3776AB?logo=python&logoColor=white)
![fastapi](https://img.shields.io/badge/FastAPI-server-009688?logo=fastapi&logoColor=white)
![lerobot](https://img.shields.io/badge/LeRobot-0.6-FF6F00)
![torch](https://img.shields.io/badge/PyTorch-CUDA%20on%20Jetson-EE4C2C?logo=pytorch&logoColor=white)
![ros2](https://img.shields.io/badge/ROS%202-Jazzy-22314E?logo=ros&logoColor=white)
![claude](https://img.shields.io/badge/Claude-operator%20model-D97757)
![mcp](https://img.shields.io/badge/MCP-driver%20as%20tools-6f42c1)
![mujoco](https://img.shields.io/badge/MuJoCo-physics%20twin-0B7285)
![isaac](https://img.shields.io/badge/Isaac%20Sim-sim--to--real-76B900?logo=nvidia&logoColor=white)
![langfuse](https://img.shields.io/badge/Langfuse-traced-1f6feb)
![tests](https://img.shields.io/badge/tests-5600%2B%20unit%20%C2%B7%20e2e%20%C2%B7%20ROS-brightgreen)
![voice](https://img.shields.io/badge/voice-English%20%C2%B7%20%D7%A2%D7%91%D7%A8%D7%99%D7%AA-8250df)

A platform for operating, teaching and researching robot arms.<br/>
This repository is the public write-up; the code base is private, and I am happy to walk through it on request.

</div>

![Teleoperated demonstration: scene camera (left) and wrist camera (right)](assets/teleop_episode.gif)

---

## 📑 Table of contents

- [💡 The idea](#-the-idea)
- [✨ What it does today](#-what-it-does-today)
- [🏗️ How it fits together](#️-how-it-fits-together)
- [📏 Scale](#-scale)
- [🔬 The research thread: robot learning on the platform](#-the-research-thread-robot-learning-on-the-platform)
- [🔭 Where it is going](#-where-it-is-going)
- [🧰 Stack](#-stack)
- [🕰️ History](#️-history)
- [👤 Author](#-author)

---

## 💡 The idea

Most robot software is a control panel: sliders, terminals, a log. You operate the machine.

This project is built around a different loop. **You talk to the robot; the robot talks back; Claude does the work; you watch and correct.** Say *"put the red cube in the bowl"* and the arm answers aloud that it is on it, Claude looks through the cameras, plans, and drives the arm through a driver that clamps every command, pausing for your approval until it has earned trust. What it learned stays in one thread, one journal and one skills registry.

The loop is not specific to one arm. The pieces are the pieces of any embodied agent:

| Layer | What it is here | Why it is general |
|---|---|---|
| **Perception** | Wrist camera, a fixed scene camera, CSI sensors, a depth camera, a microphone | Cameras by *role* (wrist, scene, top…), not by device node |
| **Conversation** | Whisper on the Jetson in; piper, OpenAI or the browser out; a conversation model that knows what the robot is doing right now | Provider-agnostic: Claude, OpenAI or a local Ollama model |
| **Reasoning** | Claude as the *operator model*: state in, commands out, a person approving until trust is earned | Tools are a driver's primitives, so the same agent drives any body behind the same driver |
| **Skills** | Built-in moves and playbooks on day one; ACT policies from your demonstrations; SmolVLA fine-tuned from a base; public checkpoints from the Hub. All registered by name, all callable by Claude | A skill is a name, an instruction and limits, whatever runs underneath |
| **Safety** | Clamped travel, capped step and speed, a voltage floor, confirmed arrival, one operator at a time, an E-STOP under everything | The limits belong to the driver, not the model |
| **Memory** | One persisted chat thread, a run journal, the skill registry, learned rest poses, a scene graph of what is on the table | The robot remembers what happened and what it can do |
| **Observability** | Langfuse traces for every run, call, motion and word; Prometheus metrics | The same instrumentation for any provider, any device |

The SO-101 is the first body. A Universal Robots arm is the second. The direction is a platform where those layers stay and the body underneath changes.

---

## ✨ What it does today

💬 **Conversation.** Chat is the front door: one thread holding what you said (typed or spoken), what the arm answered, what Claude did while it worked, its tool calls, the frames it looked at, and every run's outcome card. The arm talks back in your language through a conversation model of your choosing. Speech in through a microphone on the Jetson, transcribed on the Jetson with Whisper in Hebrew and English, with a wake word trained in your own voice.

🤖 **Claude operates.** A driver, not a prompt: the follower is owned by a driver exposing state, move, gripper, home, relax and look, with every target clamped to the calibrated travel and every step and speed capped. Each motion pauses for Approve / Deny in the chat, on the page, by voice or as a push notification on your phone; autonomous mode is opt-in and E-STOP still rules. Claude and a VLA work together: Claude plans and inspects, a trained policy does the fine motor work as a skill Claude calls. Claude can also list datasets, search the Hub, start a training run locally or on a remote GPU, follow it and register the checkpoint as a skill. The same driver is exposed over MCP for Claude Desktop or Claude Code on any machine on the tailnet.

🧰 **Skills and learning.** A skills library ready on day one (wave, nod, look around, point, hand over, present to the cameras, mimic the leader arm), skills taught by hand in a minute (move the arm through poses, capture, save), visual servoing on a colour, pick-and-place by colour, and skills the model writes itself as small scripts that are checked, rehearsed on the twin and kept only after you approve the code. A skills wizard walks Record → Train → Deploy → Verify on the twin → Test on the arm → Use with Claude. A recording quality gate scores every episode from its data alone. A shadow run drives the twin from the live camera with nothing energised. Your corrections become the next policy: take over with the leader while a skill runs, and one click fine-tunes on the takeover. Two layers learn while the arm works: a neuromorphic adaptive layer (NEF) beside the driver that learns pose-dependent sag from every landing, and a regulator over the policy's actions trained by a three-factor rule from the guards, the E-STOP, the judge and your 👍/👎.

🦿 **Body and senses.** A live 3D twin of the real arm from the official CAD, posed by the joint stream in your browser. A MuJoCo physics twin that rehearses a pose before the arm moves, finds a way round an obstacle, checks whether a grasp would hold, ranks a skill or a checkpoint on placements the recording never had, and writes simulated episodes as datasets. Calibration that cannot save garbage: from the rest pose, one joint at a time, the encoder unwrapped across its seam, every range judged against the CAD's travel. A twin check that photographs the arm in known poses and has Claude compare photo and drawing joint by joint. Cameras by name, USB and CSI, a depth camera whose stream lands in the dataset and judges the task from geometry alone, and an optional ONNX detector that names what the cameras see.

🛑 **Operating it.** Password login over Tailscale, a viewer account that watches and drives nothing, one device holding the arm at a time with takeover, an E-STOP under everything including a hardware button on a header pin, a server that stops what needs a person 60 s after your tab closes. A journal, a dashboard, a pre-flight where every failure comes with its remedy, wear and energy per joint across restarts, servo health run by run, a scrubber through any run on one clock, search across runs and notes, webhooks to Slack, Telegram or Home Assistant, a gamepad and keyboard jog through the same clamp, several rigs on one dashboard, battery and UPS awareness. A phone page with the E-STOP first and both cameras live, and push notifications when training ends, a run finishes or the arm faults. `docker compose up` runs the whole system on any laptop with a simulated servo bus. Tagged releases with checksummed archives and container images. A service worker that survives the server going away.

🧭 **Perception and grasping in the arm's frame.** The cameras place what is on the table in millimetres from the arm's base (a depth camera through a calibration solved with a ball in the jaws, the scene camera through its solved pose); a grasp is planned across the object's narrow side and reached by inverse kinematics; a learned policy can take the last centimetres, with the geometric grasp as the fallback. An open-vocabulary detector finds things by name ("the cup") in 0.2 s on the Jetson. Claude plans with these as tools: perceive, pick, check.

🤖 **It records its own demonstrations.** A scripted demonstrator plans each grasp from the scene camera, servos the ball to the jaws in the wrist picture on the way down, and keeps a take only when a verifier says the ball left the table in the jaws. It took 9 of 10 on the real arm, and as a supervised data factory it kept 5 of 7 takes in its first clean run, dropping the still frames as it records so the descents carry no pause.

🎯 **Reinforcement learning with a human hand.** HIL-SERL with the grasp verifier as the reward and the leader arm as the intervening hand, a space-bar clutch deciding who has the arm; the learner runs on a desktop GPU over the tailnet.

🦾 **A second body.** A Universal Robots arm over its IP: power, brakes, joints and tool position, clamped URScript, and the operator model with its own tool set. The skills library idea carries over.

📈 **A hundred improvements in one round (improve-100).** Ten themed batches, one issue per item, all built and tested in software:
- **Data:** every take tagged with its camera framing, the human's hovers stripped out, positions held out instead of episodes, failures kept, a stricter gate, a coverage grid that asks for takes where the error is highest.
- **Policies:** the target's position and crops around it as inputs, relative actions, frozen DINOv2/SigLIP features, phase policies chained approach → descend → grasp → lift, sim-and-real co-training, TensorRT FP16 on the Orin, depth, point-cloud and keypoint policies.
- **RL and evaluation:** takeovers become corrective data, residual RL over ACT, offline IQL over every take, a reset curriculum, checkpoints that disagree handing the grasp to geometry, nightly train-score-promote, and A/B trials that stop when the statistics say enough.
- **Perception and grasping:** hand-eye and checkerboard calibration, AprilTags with drift alarms, tracking and memory through occlusion, touch-to-select and pointing, grasp scoring on depth, place planning, a camera-fitted correction of the kinematics, success predicted before the lift, regrasps, push, slide and pour.
- **Control:** one minimum-jerk trajectory generator, a force-limited close, gravity feedforward, a collision model with a sampling planner, backlash compensation, overloads predicted before they trip.
- **Unattended nights:** an overnight scripted data factory, a tilt sensor on the base that presses the E-STOP, a depth fence, overload recovery on the tripped servo alone, wear per servo with a replacement schedule, and a live camera picture through every recording, rollout and RL run.
- **Planning and operations:** Claude re-perceiving after every step, behaviour trees, plans rehearsed in the physics twin, one trace per task with replay, a VLA served from the desktop GPU with real-time chunking, a run registry comparing experiments, an arm plugin layer.

---

## 🏗️ How it fits together

```mermaid
flowchart TB
  B[Browser on PC or phone<br/>over Tailscale] --> S
  subgraph S[so101 serve — FastAPI + SSE, one process on the Jetson]
    direction TB
    CH[chat · talk · voice · speech<br/>one thread, a conversation model, Whisper, TTS] --> AG[agent — Claude as the operator model]
    AG -->|tools| DR[robot_driver — limits, clamps, one operator, E-STOP]
    SK[skills · playbooks · ACT / SmolVLA policies] --> DR
    AG --> SK
    SS[calibration · teleop · record · train · shadow · replay · eval] --> DR
    CAM[cameras by role — USB, CSI, depth] --> AG
    CAM --> SS
    TW[twin — CAD in the browser · MuJoCo physics] --> SS
    OB[observe — Langfuse traces · Prometheus] -.-> AG
    ROS[ROS 2 driver node] --> DR
  end
  DR --> F[Follower arm]
  L[Leader arm] --> SS
  MCP[Claude Desktop / Claude Code over MCP] --> DR
```

Three models, three jobs: Claude operates (slow, careful, expensive by design), a small talker converses and routes what you said, and a trained policy moves. One rule holds everywhere: one thing holds the arm at a time, the limits live in the driver and not in any prompt, and stopping is never the privileged action.

---

## 📏 Scale

| | |
|---|---|
| Python | ~141,000 lines across ~550 modules plus ~76,000 lines of tests, typed and checked (mypy, ruff; strict on the core) |
| Browser | ~15,700 lines of plain JavaScript, one page anatomy, dark and light, a phone page |
| API | 775 routes on one FastAPI process, 19 pages, a generated SDK and an OpenAPI description |
| Tests | 5,881 unit tests, 148 end-to-end and browser (Playwright), 70 ROS 2 node tests — with a conftest that refuses to open a real camera or cut torque |
| Docs | a handbook plus a 33-chapter guide served inside the app; a changelog of 76 versions |
| Bodies | two SO-101 arms on a Jetson Orin Nano (ROS 2 Jazzy), a Universal Robots arm over IP, a MuJoCo twin, Isaac Sim / Isaac Lab and training on a desktop RTX GPU |

---

## 🔬 The research thread: robot learning on the platform

The platform's current research use is a full imitation-learning loop on real hardware, measured at every step.

```mermaid
flowchart LR
  L[Leader arm] -->|teleop| F[Follower arm]
  C[2 cameras] --> R
  F --> R[Recorder<br/>LeRobot v3 dataset]
  R -->|rsync over Tailscale| G[RTX GPU box<br/>ACT / SmolVLA fine-tune]
  G -->|checkpoints| E[Held-out scoring<br/>per checkpoint]
  E --> P[Closed-loop rollout on the arm<br/>scorecard + recorded video]
  G --> PS[Policy server<br/>large model off the edge device]
  PS --> P
  I[Isaac Lab<br/>same task in sim] -.-> G
```

Task: "pick a ball", 26 demonstrations, ~29k frames. Held-out action error (mean absolute error in degrees over two demonstrations the policy never saw), measured for every checkpoint:

![Held-out error per checkpoint](assets/heldout_error.png)

| Run | Data | Best held-out error |
|---|---|---|
| ACT v1 | 12 episodes, batch 4, 30k steps | 13.6 |
| SmolVLA | 12 episodes, expert only, 20k steps | 12.0 |
| ACT v2 | 24 episodes, batch 8, 30k steps | 12.2 |

Doubling the demonstrations cut ACT's final error from 13.6 to 12.2 and moved its whole curve ahead of v1 by roughly 10k steps; v2 plateaus from 25k steps, where SmolVLA plateaued from 10k.

On the arm, v1 reached the ball and hovered beside it. v2 reaches and descends onto the ball, and keeps the gripper closed. Reading the demonstrations explains why: the operator hovered at the ball for 10 to 17 seconds before opening the gripper, so the policy learned the hover. The next data round records the descend-open-close phase without the pause. The rollout below is ACT v1 after 15k steps.

![ACT-15000 closed-loop rollout on the arm](assets/rollout_act15k.gif)

**September: a month of real data, and what it taught.** 41 human demonstrations and 14 scripted ones later, the picture is sharper:

| Run | Data | On the arm |
|---|---|---|
| ACT v6 | 17 takes in the current camera framing (7 human, 10 scripted) | reaches the zone, comes down 10 cm short of the ball and hovers, wherever the ball is — for 45 s or 60 s alike |
| SmolVLA v3 | the same takes | too slow on the Jetson (~1 s per look): a few seconds of actions in 45 s |
| Scripted demonstrator | geometry, no learning | 9 of 10 grasps |
| Finishing policy (ACT, 19 final-descent segments) | inverse kinematics to the hover, the policy for the last centimetres | handed the arm 20 mm above where its takes began, it closed on nothing; the geometric grasp behind it took the ball 3 of 3 — it now takes over at its takes' own start height |
| Scripted factory | the demonstrator recording its own takes, supervised | 5 of 7 kept, no faults, live camera throughout |

![ACT v6 on the arm: it comes down beside the ball and waits](assets/rollout_actv6_hover.gif)

Two lessons. **A camera that moves is a new dataset:** a knocked scene camera cost a model ~11 points of held-out error, so datasets are now kept per framing. **Whole-task policies learn the average reach, not the ball's position.** The architecture that follows splits the work by distance: perception and inverse kinematics bring the jaws over the object exactly, a policy trained only on the final descents does the contact, the verifier decides, and the geometric grasp takes over on a miss. The scripted demonstrator below is that geometry alone, recording its own verified takes (scene camera left, wrist camera right):

![A take the scripted demonstrator recorded by itself](assets/scripted_take.gif)

🎮 **Sim-to-real.** The same task in Isaac Lab on NVIDIA's Sim-to-Real SO-101 workshop scene, with the workshop's vials and rack swapped for this rig's single ping-pong ball, in one of three colours drawn at random on every reset. The environment steps headless with both cameras rendering (20 steps in 1.2 s on an RTX 2080) and ends the episode when the ball is lifted off the mat; a variant keeps the red box as the target.

![Isaac Lab: the ball task on the workshop scene](assets/isaac_pick_ball.png)

🧪 **What the first real checkpoints taught.** Since LeRobot 0.4, normalisation, camera renames, batching and tokenising live in the checkpoint's processor pipelines, not in the policy; every in-app runner had met fakes only, and wrapping the policy in its own pre/post-processors fixed inference everywhere at once. Evaluating with a stride desynchronised a chunked policy's action queue; resetting before every sampled frame halved the measured error. A servo's alarm bit kills a recording at the worst moment; the recorder now names the arm, the phase and what to check. A VLA on the Jetson takes ~1.4 s per look; real-time chunking and a remote policy server are what make it usable in a 30 s episode.

🧠 **The adaptive layer, on the real arm.** The neuromorphic layer beside the driver (a NEF population that learns each pose's sag from every landing) had only been measured on a loaded fake arm, where it landed three times closer than the lookup table alone. Its protocol on the real follower: twenty poses across the reach, three rounds each, with the layer off, then on and learning, then frozen. Mean absolute first landing, before any correction:

| Pass | Mean | Pan | Lift | Elbow | Wrist | Corrections |
|---|---:|---:|---:|---:|---:|---:|
| off | 0.54° | 0.73° | 0.83° | 0.45° | 0.16° | 41 |
| on, learning | 0.48° | 0.60° | 0.84° | 0.32° | 0.16° | 38 |
| frozen | 0.44° | 0.62° | 0.80° | 0.23° | 0.12° | 25 |

The claim holds by the protocol's own test, but the gain is modest: this follower already lands within half a degree, the elbow halved and the shoulder lift did not move. One session with the passes in a fixed order can't separate learning from servos warming up; the repeat turns the order round. And a confound found the next night: the arm's compensation table kept learning through all three passes, so part of the gain may be the table's, credited to the layer. On the simulated arm, with the table learning, the layer looks perfect (0.38° → 0.00°); with the table held, its real contribution shows: 1.04° → 0.70° learning → 0.38° frozen. The repeat on the real arm holds the table. Running it at all took three fixes the fake arm never needed: the layer's switch and the first-landing error carried over the ROS graph, the poses chosen with the arm's mirrored joint directions (a pose "80 mm up" was 25 mm under the table), and every approach started from home instead of rest (a straight line out of the folded rest pose went through the table).

👁️ **An event retina from ordinary webcams.** Neuromorphic vision sends change, not frames: a pixel fires when its light moves by a step, and is silent otherwise. The platform emulates that from its webcams (per-pixel log-brightness steps, a lin-log floor so noise in dark areas stays quiet). Two first uses: the grasp now *feels a touch*, where a free ball that moves in its box while the arm stands still was brushed, and the jaws don't close on where it was; and a camera can stream as changes only, a key frame every few seconds and between them only the tiles that fired. On a still table that is 6.4 KB/s instead of 87 KB/s. A real event sensor (a GenX320 on the Raspberry Pi 5) would add microsecond latency and a wide dynamic range on top.

---

## 🔭 Where it is going

**At the bench next:** a finishing policy trained on the factory's own descents, then head to head with the geometric grasp on the same placements; the scene camera's pose solved from AprilTags so the table map stops extrapolating; the depth camera mounted and calibrated to the arm; the first HIL-SERL session with a hand on the clutch; and the factory's first night.

**Improve-100, on the arm:** all hundred are built and tested in software; the ones that touch hardware (the depth camera's grasping and kinematic correction, the tilt sensor, the leader's haptics, nightly trials on the arm, the VLA served from the desktop GPU) now each need their first real run.

**Bodies:** any LeRobot robot behind the same driver through one hardware-abstraction interface; bimanual skills after that.

---

## 🧰 Stack

Python · FastAPI · PyTorch · LeRobot · ROS 2 Jazzy · Claude (operator and conversation model) · OpenAI / Ollama · Whisper · piper · MCP · MuJoCo · Isaac Sim / Isaac Lab · Langfuse · Prometheus · Playwright · Docker · CUDA on Jetson Orin Nano · Tailscale

---

## 🕰️ History

It began as a guided calibration tool, because the stock calibration accepted a broken calibration silently, so teleop tracked wrong. From there it grew a web interface, a 3D twin of the real CAD, recording and training with a quality gate, then Claude as an operator model with a limit-enforcing driver, skills, cameras by role, a voice in and a voice out, one chat thread, traces, a second body, and the research loop above. September 2026 was the month of real data: the first policies trained on a desktop GPU and run on the arm, the discovery that they hover beside the ball, a scripted demonstrator that took nine of ten, and the perception-first architecture that came out of both.

---

## 👤 Author

Guy Vitelson — AI research, robotics, embedded. The full repository and the datasets are available on request.

---

<div align="center">

Built on a Jetson Orin Nano Super with two SO-101 arms, for the day the body underneath is something bigger.

</div>
