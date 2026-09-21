# Franka Panda Controlled Simulation Guide

**This guide explains how to install, test, and run the Franka Panda tabletop simulation with a PS5 DualSense controller.**

The final controlled demo supports:

- A Franka Panda arm mounted on the table.
- Controller-based Cartesian end-effector movement.
- Controller buttons for opening and closing the gripper.
- Colored cubes spawned on the tabletop.
- Rumble feedback when the hand or fingers contact a cube.
- Longer rumble feedback when both fingers contact the same cube.
- Optional overhead and wrist cameras.
- RGB, depth, and segmentation data for vision experiments.

You need:
- A PS5 DualSense controller for the controlled demo.
- The controller connected by USB or Bluetooth and charged.

The simulation can run without a controller, but the `--controller` option requires a connected DualSense.

## Table of Contents
1. [Installation](#1-installation)
     - [Check your Python version](#11-check-your-python-version)
     - [Install Python with pyenv, pyenv-win, Anaconda, or Miniconda](#12a-install-python-312--track-a-pyenv--pyenv-win--anaconda)
     - [Create and activate an environment](#13-create--activate-a-virtual-environment)
     - [Install dependencies](#14-install-the-exact-dependencies)
2. [Controller](#2-controller)
    - [Connect and configure the PS5 DualSense](#21-connect-and-configure-the-ps5-dualsense)
     - [Test the controller](#22-test-the-controller-before-the-robot)
  3. [PS5 DualSense Control Mapping](#3-ps5-dualsense-control-mapping)
4. [Run the Final Controlled Demo](#4-run-the-final-controlled-demo)
5. [How Movement and Feedback Work](#5-how-movement-and-feedback-work)
6. [Speed and Control Tuning](#6-speed-and-control-tuning)
7. [Rumble Intensity and Timing](#7-rumble-intensity-and-timing)
8. [File Overview](#8-file-overview)
9. [Troubleshooting](#9-troubleshooting)
10. [Quick Command Reference](#10-quick-command-reference)


---
## 1. Installation
> **Target Python:** 3.12.x. To avoid conflicts with system Python, this guide shows **two parallel tracks** until they converge:
> - **Track A (recommended):** use **pyenv / pyenv‑win / anaconda** to keep multiple Python versions side‑by‑side without touching your OS Python.
> - **Track B:** install **Python 3.12** directly on your system.
>
> Choose **either** Track A or Track B in 2. Once Python 3.12 is available, the steps **are basically the same**.

### 1.1) Check your Python version

- **Windows (PowerShell or CMD)**
  ```bat
  py --version     :: Windows Python Launcher (handles multiple versions)
  python --version
  ```
  Tip: Use a specific version with `py -3.12`.

- **macOS / Ubuntu**
  ```bash
  python3 --version
  ```

If you **don’t** have **3.12.x**, pick **Track A** (pyenv) or **Track B** (direct) below.

---

### 1.2A) Install Python 3.12 — Track A: pyenv / pyenv‑win / Anaconda

**Why this track?** It keeps multiple Python versions **side‑by‑side** without touching OS Python (safer on macOS/Ubuntu and flexible on Windows).

#### pyenv / pyenv‑win
##### macOS (Homebrew)
```bash
# Install pyenv
brew update
brew install pyenv

# Initialize pyenv in your shell (zsh/bash)
echo 'export PYENV_ROOT="$HOME/.pyenv"' >> ~/.zshrc
echo '[[ -d $PYENV_ROOT/bin ]] && export PATH="$PYENV_ROOT/bin:$PATH"' >> ~/.zshrc
echo 'eval "$(pyenv init - zsh)"' >> ~/.zshrc
exec "$SHELL"

# (Optional) helpful build deps for compiling Python via pyenv
brew install openssl readline sqlite3 xz tcl-tk zstd zlib pkgconfig
```

##### Ubuntu / Debian
```bash
# Build prerequisites (recommended before installing Python via pyenv)
sudo apt update
sudo apt install -y make build-essential libssl-dev zlib1g-dev \
  libbz2-dev libreadline-dev libsqlite3-dev curl git \
  libncursesw5-dev xz-utils tk-dev libxml2-dev libxmlsec1-dev \
  libffi-dev liblzma-dev

# Install pyenv (automatic installer)
curl -fsSL https://pyenv.run | bash

# Initialize pyenv in your shell
echo 'export PYENV_ROOT="$HOME/.pyenv"' >> ~/.bashrc
echo '[[ -d $PYENV_ROOT/bin ]] && export PATH="$PYENV_ROOT/bin:$PATH"' >> ~/.bashrc
echo 'eval "$(pyenv init - bash)"' >> ~/.bashrc
exec "$SHELL"
```

##### Windows (pyenv‑win)
```powershell
# PowerShell: install pyenv-win
Set-ExecutionPolicy -Scope CurrentUser RemoteSigned
Invoke-WebRequest -UseBasicParsing `
  -Uri "https://raw.githubusercontent.com/pyenv-win/pyenv-win/master/pyenv-win/install-pyenv-win.ps1" `
  -OutFile "$HOME\install-pyenv-win.ps1"
& "$HOME\install-pyenv-win.ps1"
# Restart the terminal so "pyenv" is on PATH
```

##### Install & select Python 3.12 (all OSes with pyenv/pyenv‑win)
```bash
pyenv install 3.12.11
pyenv global 3.12.11
python --version     # should now report 3.12.x
```

> **Tip:** If `pyenv install` fails due to missing libraries, re‑check the build‑deps above and consult pyenv’s “Common Build Problems.”

---

#### Anaconda or Miniconda

Choose **Anaconda** for the full data-science distribution, or **Miniconda** for a smaller installation. Download the installer from one of these official pages:

- Anaconda: <https://www.anaconda.com/download>
- Miniconda: <https://docs.conda.io/projects/miniconda/en/latest/>

##### Windows

You can install Miniconda or full Anaconda directly from PowerShell with WinGet:

```powershell
# Recommended: smaller Miniconda installation
winget install --id Anaconda.Miniconda3 -e

# Alternative: full Anaconda installation
winget install --id Anaconda.Anaconda3 -e
```

Install only one of the two options. Restart PowerShell after installation, then initialize Conda:

```powershell
conda init powershell
```
Then restart powershell to have `conda` command available.

You can also download and run the Windows installer from the official links above, then use the same Conda commands.

##### macOS

Download the macOS installer for your CPU architecture, or install Miniconda with Homebrew:

```bash
brew install --cask miniconda
conda init
```

Restart the terminal, then run:

```bash
conda create -n franka-vision python=3.12 -y
conda activate franka-vision
```

##### Ubuntu / Debian

Download and run the Linux installer, then restart the terminal:

```bash
bash Miniconda3-latest-Linux-x86_64.sh
source ~/.bashrc
conda create -n franka-vision python=3.12 -y
conda activate franka-vision
```

Use the ARM64 installer instead if the computer uses an ARM64 processor.


---

### 1.2B) Install Python 3.12 — Track B: direct install

**Windows** (choose one):
```powershell
# WinGet (official package manager)
winget install -e --id Python.Python.3.12

# Or download the official installer:
# https://www.python.org/downloads/windows/
# During install, check "Add Python to PATH".
```

**macOS** (choose one):
```bash
# Homebrew
brew install python@3.12

# Or use the official macOS installer:
# https://www.python.org/downloads/macos/
```

**Ubuntu**
```bash
# pyenv (Track A) is recommended on Ubuntu since python3.12 apt packages vary by release.
# If your release provides them:
sudo apt update && sudo apt install -y python3.12 python3.12-venv
```

> From here on, **Track A** and **Track B** converge — you now have **Python 3.12** available.

---


### 1.3) Create & activate a virtual environment

#### pyenv

Go to the folder containing the simulation files and type this in the terminal

```bash
python -m venv .venv
```

**Activate it**

- macOS / Linux (zsh/bash)
  ```bash
  source .venv/bin/activate
  ```
- macOS / Linux (fish)
  ```bash
  source .venv/bin/activate.fish
  ```
- Windows (PowerShell)
  ```powershell
  .venv\Scripts\Activate.ps1
  ```
- Windows (CMD)
  ```bat
  .venv\Scripts\activate.bat
  ```

You should see a `(.venv)` prefix in your terminal prompt.


The simulation can run without a controller, but the `--controller` option requires a connected DualSense.

### Anaconda

Create the project environment once:

```bat
conda create -n franka-vision python=3.12 -y
```

Activate it:

```bat
conda activate franka-vision
```

Confirm that the correct Python is active:

```bat
python --version
where python
```

The version should be Python 3.12.x, and the selected Python should be inside the `franka-vision` environment.

---

### 1.4) Install the exact dependencies

```bash
pip install "numpy==1.26.0" "pybullet==3.2.5" "opencv-python==4.10.0.82" "pillow==11.3.0" pygame
```

> If a platform wheel isn’t available for your OS/arch, try the nearest patch version or temporarily drop the pin to install a compatible build.


---

## 2. Controller
### 2.1 Connect and Configure the PS5 DualSense

1. Connect the DualSense by USB or pair it through Bluetooth.
2. Turn on the controller and keep it connected while running the simulation.
3. Close other applications that may already be using the controller.

If the controller is detected but rumble does not work, reconnect it by USB and repeat the diagnostics. Different Windows, Bluetooth, and SDL driver combinations can expose different controller capabilities.

> Another Soution for the rumble (for example on Windows), if you connected the controller with USB, you will find the controller in the volume options as a speaker, you should raise the volume. You also might face that you need to keep the speaker choice as the controller in some of the vibration demos.

### 2.2 Test the Controller Before the Robot

The controller test is independent from PyBullet. It is useful for confirming that the operating system, Pygame, axes, buttons, and rumble are working before starting the robot.

#### List controllers

```bat
python controller_test_ps5.py --list
```

Expected output resembles:

```text
[0] DualSense Wireless Controller | axes=6 buttons=13 hats=1
```

If no controller is listed, check the USB/Bluetooth connection, controller charge, and Windows game-controller settings.

#### Read axes, buttons, and D-pad

```bat
python controller_test_ps5.py --duration 10
```

Move both sticks, press the buttons, and use the D-pad. Example output:

```text
axis 0: +1.000
button 0: DOWN
button 0: UP
hat 0: (0, 1)
```

Stop a test that runs indefinitely with `Ctrl+C`.
When you press a button, it will appear DOWN then UP when you leave it. You will also find some noise in the joysticks readings.

#### Test startup rumble

```bat
python controller_test_ps5.py --rumble-test --duration 3
```

For a stronger and longer test:

```bat
python controller_test_ps5.py --rumble-test --rumble-strength 1.0 --rumble-duration 1000 --duration 3
```

The strength range is `0.0` to `1.0`. Duration is in milliseconds.

#### Test rumble after a button press

```bat
python controller_test_ps5.py --rumble-on-button --duration 10
```

For both startup rumble and button rumble:

```bat
python controller_test_ps5.py --rumble-test --rumble-on-button --rumble-strength 0.8 --rumble-duration 300 --duration 10
```

Some DualSense driver configurations do not respond to the initial rumble command until the controller has received an input event. If startup rumble does not work, press a button once and test again. This behavior is controller/driver dependent; it does not necessarily mean the Python code failed.

### R2 force reading

```bat
python controller_test_ps5.py --duration 10 --r2-force-scale 1.5
```

R2 is normalized to a `0.0..1.0` value. The script also prints a scaled force value for use by grip or motor-control code:

```text
R2 axis 5: 0.420 (force=0.630)
```

### R2 logical lock at 50%

```bat
python controller_test_ps5.py --duration 10 --r2-lock --r2-threshold 0.5 --rumble-strength 1.0
```

When R2 reaches 50%, the script reports `R2 LOCKED` and sends a maximum-strength vibration pulse. When the value drops below 50%, it reports `R2 UNLOCKED` and sends another pulse. This is a software state; it does not physically stop or harden the trigger.

### Example output while running

Your exact values will vary with the controller and how quickly the trigger is pressed, but a typical session looks like this:

```text
pygame 2.6.1 (SDL 2.32.56, Python 3.12.14)
Using [0] DualSense Wireless Controller
axes=6 buttons=13 hats=1
Move a control or press a button. Press Ctrl+C to stop.

R2 axis 5: 0.420 (force=0.420)
axis 5: +0.420
R2 axis 5: 0.537 (force=0.537)
R2 LOCKED
axis 5: +0.537
R2 axis 5: 1.000 (force=1.000)
axis 5: +1.000
R2 axis 5: 0.459 (force=0.459)
R2 UNLOCKED
axis 5: +0.459
R2 axis 5: 0.000 (force=0.000)
axis 5: -1.000
```

The `R2 axis 5` line is the normalized trigger value. The separate `axis 5` line is the raw SDL axis value, which commonly ranges from `-1.0` to `+1.0` on this controller. The vibration is felt at the `R2 LOCKED` and `R2 UNLOCKED` transitions, but it is not printed as a separate message.

### Important DualSense limitation

A standard PS5 DualSense cannot physically increase R2 trigger resistance or create a real mechanical hair-trigger through `pygame` or another normal software API. R2 intensity reading, force scaling, logical locking, and vibration feedback are supported. Actual trigger travel adjustment requires different hardware, such as a controller with mechanical trigger-stop features.

### Controller test options

| Option | Meaning | Default |
| --- | --- | --- |
| `--list` | List connected controllers and exit | Off |
| `--index N` | Select controller number `N` | `0` |
| `--duration S` | Stop after `S` seconds; `0` runs until `Ctrl+C` | `0` |
| `--poll-interval S` | Delay between event polls | `0.01` |
| `--deadzone V` | Ignore small axis changes | `0.05` |
| `--r2-threshold V` | R2 logical lock threshold | `0.25` (effective minimum `0.5`) |
| `--r2-force-scale V` | Scale the printed R2 force value | `1.0` |
| `--r2-lock` | Enable the logical R2 lock state | Off |
| `--rumble-test` | Vibrate once at startup | Off |
| `--rumble-on-button` | Vibrate after every button press | Off |
| `--rumble-strength V` | Rumble strength from `0.0` to `1.0` | `0.5` |
| `--rumble-duration MS` | Rumble duration in milliseconds | `150` |

---


## 3. PS5 DualSense Control Mapping
The following is the standard mapping reported by `pygame` for a Wireless DualSense in the active environment.

### Standard PS5 DualSense buttons

| Pygame input | PS5 control |
| --- | --- |
| `button 0` | Cross / X |
| `button 1` | Circle / O |
| `button 2` | Square / □ |
| `button 3` | Triangle / △ |
| `button 4` | L1 |
| `button 5` | R1 |
| `button 6` | L2 digital button |
| `button 7` | R2 digital button |
| `button 8` | Share |
| `button 9` | Options |
| `button 10` | Left stick click |
| `button 11` | Right stick click |
| `button 12` | PS button |


The mapping below matches the DualSense output observed with Pygame in the active environment. Always trust the live output from `controller_test_ps5.py` if a different driver reports different numbers.

### Robot controls

| Controller input | Robot action |
| --- | --- |
| Left stick horizontal, axis 0 | Move end effector in X |
| Left stick vertical, axis 1 | Move end effector in Y |
| Right stick vertical, axis 3 | Move end effector in Z |
| Button 0, Cross / X | Close gripper |
| Button 1, Circle / O | Open gripper |

The bridge applies a deadzone so small stick drift does not move the robot.


### D-pad

The D-pad is `hat 0`, with `(x, y)` values:

| Pygame input | PS5 control |
| --- | --- |
| `hat 0: (0, 1)` | Up |
| `hat 0: (0, -1)` | Down |
| `hat 0: (-1, 0)` | Left |
| `hat 0: (1, 0)` | Right |
| `hat 0: (0, 0)` | Released |


### Axes

Typical DualSense axis numbering is:

| Axis | PS5 control | Normal range |
| --- | --- | --- |
| `axis 0` | Left stick horizontal | `-1.0` to `+1.0` |
| `axis 1` | Left stick vertical | `-1.0` to `+1.0` |
| `axis 2` | Right stick horizontal | `-1.0` to `+1.0` |
| `axis 3` | Right stick vertical | `-1.0` to `+1.0` |
| `axis 4` | L2 trigger | `0.0` to `1.0` or `-1.0` to `+1.0` depending on driver |
| `axis 5` | R2 trigger | `0.0` to `1.0` or `-1.0` to `+1.0` depending on driver |

In practice, the script normalizes the trigger to a clean `0.0..1.0` force value before using it for lock or force logic.


```text
R2 axis 5: 0.420 (force=0.420)
```

To enable the software lock at 50% and maximum transition vibration:

```bat
python controller_test_ps5.py --duration 10 --r2-lock --r2-threshold 0.5 --rumble-strength 1.0
```

`R2 LOCKED` is printed at or above the threshold and `R2 UNLOCKED` when the value falls below it. A standard DualSense cannot physically increase R2 resistance or create a mechanical trigger stop; this lock is a software state with haptic feedback only.

---

## 4. Run the Final Controlled Demo

During the final demo:

1. Move the arm above a cube with the sticks.
2. Press Cross / X to close the gripper.
3. Use the sticks to bring the fingers around the cube.
4. Watch for the short contact rumble.
5. Continue closing or repositioning until both fingers touch the cube.
6. Watch for the longer grasp rumble.
7. Use Circle / O to open the gripper.

The simulation is a control and vision prototype. The grasp detector is based on PyBullet contact links, so it confirms simultaneous finger contact rather than proving that the cube is physically lifted and retained under all dynamics.


**Use this command for the normal final demonstration:**

```bat
python vision_bridge_controlled.py --gui --controller --control-speed 0.03 --use-overhead --physics-steps 4
```

The command opens the PyBullet GUI, mounts the robot on the table, enables the DualSense, enables the overhead camera, and stops after 3000 control cycles.


Stop the demo with `Ctrl+C` or close the simulation window.

### Recommended diagnostic run

Use this when checking controller values or investigating movement:

```bat
python vision_bridge_controlled.py --gui --controller --debug-controller --use-overhead --print-every 30 --steps 3000
```

Every 30 cycles it prints the latest axis values. When a stick is moved, the corresponding axis should become significantly different from zero.

### Run without the controller

The controlled bridge can also display the simulation without reading the DualSense:

```bat
python vision_bridge_controlled.py --gui --use-overhead --steps 1000
```

Without `--controller`, the arm is not commanded by the joystick.

### Use both cameras

```bat
python vision_bridge_controlled.py --gui --controller --use-overhead --use-wrist --steps 3000
```

### Save annotated camera frames

```bat
python vision_bridge_controlled.py --gui --controller --use-overhead --save output --steps 3000
```

Frames are saved in the `output` directory using names such as `overhead_000000.png`.

### Use the segmentation detector

```bat
python vision_bridge_controlled.py --gui --controller --use-overhead --detector seg --steps 3000
```

The segmentation detector groups rendered pixels by PyBullet body/link ID. It is a prototype detector for testing the RGB-D and segmentation pipeline, not a general object-recognition model.

### Use safe mode

For slower computers or graphics-driver problems:

```bat
python vision_bridge_controlled.py --gui --controller --safe-mode --steps 3000
```

Safe mode uses a 320x240 overhead image and TinyRenderer. It is slower visually but avoids dependence on hardware OpenGL rendering.

---

## 5. How Movement and Feedback Work

The controlled bridge performs the following loop:

1. Read the DualSense axes and button states using Pygame.
2. Convert joystick values into a small Cartesian end-effector target change.
3. Use PyBullet inverse kinematics to convert the target into Panda joint targets.
4. Apply position control to the seven arm joints.
5. Apply position control to the two gripper joints.
6. Step the physics simulation.
7. Check robot-cube contacts.
8. Trigger rumble when a new hand/finger contact or grasp is detected.

The robot is mounted on the tabletop by default at approximately `x=0.15`, while the cubes are spawned near `x=0.45` to `x=0.525`. This places the base away from the main cube workspace and gives the arm room to reach toward the cubes.

Contact feedback is event-based rather than continuous:

- A short rumble occurs when the Panda hand or fingers first contact a cube.
- A longer rumble occurs when both finger links contact the same cube.
- Holding contact does not repeatedly vibrate every simulation frame.

---

## 6. Speed and Control Tuning

The final bridge exposes two main speed-related options.

### End-effector speed

Default:

```text
--control-speed 0.012
```

This is the Cartesian target change applied per controller update, in meters. Increase it for faster stick response:

```bat
python vision_bridge_controlled.py --gui --controller --control-speed 0.018 --steps 3000
```

The tested faster setting is:

```bat
python vision_bridge_controlled.py --gui --controller --control-speed 0.03 --physics-steps 4 --steps 3000
```

A conservative range to try is approximately `0.008` to `0.03`.

If the arm becomes difficult to control, reduce it:

```bat
python vision_bridge_controlled.py --gui --controller --control-speed 0.006 --steps 3000
```

### Physics steps per controller update

Default:

```text
--physics-steps 2
```

This controls how many PyBullet physics steps run after each controller sample. Try this faster setting:

```bat
python vision_bridge_controlled.py --gui --controller --control-speed 0.018 --physics-steps 3 --steps 3000
```

Higher values advance the simulation farther per controller update. They do not necessarily make the controller more precise. If motion becomes jumpy, return to `--physics-steps 2`.

### Robot base position

The table-mounted robot X position can be changed at launch:

```bat
python vision_bridge_controlled.py --gui --controller --robot-x 0.10 --steps 3000
```

The default is `0.15`. The Y position can also be changed:

```bat
python vision_bridge_controlled.py --gui --controller --robot-x 0.15 --robot-y 0.10 --steps 3000
```

The robot can be deliberately placed on the floor for comparison:

```bat
python vision_bridge_controlled.py --gui --controller --robot-mount floor --steps 3000
```

For normal operation, use `--robot-mount table` or rely on the default.

---

## 7. Rumble Intensity and Timing

The final bridge exposes:

| Option | Meaning | Default |
| --- | --- | --- |
| `--rumble-strength` | Motor strength from `0.0` to `1.0` | `0.5` |
| `--rumble-duration` | Initial-contact rumble duration in milliseconds | `100` |

Example with stronger contact feedback:

```bat
python vision_bridge_controlled.py --gui --controller --rumble-strength 0.8 --rumble-duration 200 --steps 3000
```

The grasp rumble uses twice the configured duration. For example, with `--rumble-duration 150`, a grasp requests approximately 300 milliseconds.

Use a lower intensity if the controller is uncomfortable:

```bat
python vision_bridge_controlled.py --gui --controller --rumble-strength 0.3 --rumble-duration 120 --steps 3000
```

Use values between `0.0` and `1.0`. A driver may ignore rumble or report it as unsupported even when controller axes and buttons work.

---

## 8. File Overview

### `controller_test_ps5.py`

Standalone PS5 DualSense diagnostic program. It can list controllers, print live axis/button/D-pad events, read normalized R2 force, test the logical R2 lock, and test rumble. The final controlled bridge imports its reusable controller interface.


### `pybullet_vision_env_controlled.py`

Controlled environment. It contains the table mounting, controller-to-IK movement, gripper commands, and hand/finger contact checks used by the final demo.

### `vision_bridge_controlled.py`

Final controller and vision runner. It imports the controlled environment, reads the DualSense, commands the robot, checks contact/grasp state, triggers rumble, and optionally runs the camera pipeline.

---

## 9. Troubleshooting

### `No controllers found`

Check the following:

1. The DualSense is connected by USB or Bluetooth.
2. The controller is charged and powered on.
3. The controller is powered on.
4. Other applications are not holding the controller connection.
5. The correct Conda environment is active.
6. Pygame is installed in that environment.

Then run:

```bat
python controller_test_ps5.py --list
```

### Axes print correctly but the arm does not move

Run the controlled bridge with diagnostics:

```bat
python vision_bridge_controlled.py --gui --controller --debug-controller --print-every 30 --steps 1000
```

Move one stick at a time and check that the printed values change. Also verify that the command includes `--controller`.

If movement is too small, increase the speed:

```bat
python vision_bridge_controlled.py --gui --controller --control-speed 0.018 --steps 1000
```

### The robot is on the floor

The controlled runner defaults to table mounting. Explicitly select the table:

```bat
python vision_bridge_controlled.py --gui --controller --robot-mount table --steps 1000
```

Do not confuse the controlled runner with older commands that may default to floor mounting.

### Rumble does not work

First test the controller separately:

```bat
python controller_test_ps5.py --rumble-test --rumble-on-button --duration 10
```

Press a controller button once. Some DualSense driver combinations begin responding to rumble only after an input event. If it still does not work, reconnect by USB and check Windows game-controller drivers.

The controller may support input while not supporting rumble through the current SDL/Pygame backend.

### `maxVelocities is an invalid keyword argument`

This project intentionally does not pass `maxVelocities` to `setJointMotorControlArray`, because some installed PyBullet versions reject that keyword. Use the project dependency environment and do not add that keyword back without checking the installed PyBullet API.

### OpenGL or GUI problems

Try TinyRenderer or safe mode:

```bat
python vision_bridge_controlled.py --gui --controller --safe-mode --steps 1000
```

You can also test without the GUI:

```bat
python vision_bridge_controlled.py --controller --tiny --steps 1000
```

### Camera windows do not appear

Use the PyBullet GUI first. OpenCV windows require a desktop session and are enabled with:

```bat
python vision_bridge_controlled.py --gui --use-overhead --cv2-view --steps 1000
```

### The arm collides with cubes immediately

Move the table-mounted base farther from the cube workspace:

```bat
python vision_bridge_controlled.py --gui --controller --robot-x 0.10 --steps 1000
```

The default controlled base position is already separated from the normal cube spawn area, but the arm's reachable workspace can still overlap the cubes depending on the joint pose and joystick commands.

---

## 10. Quick Command Reference

### Controller diagnostics

```bat
python controller_test_ps5.py --list
python controller_test_ps5.py --duration 10
python controller_test_ps5.py --rumble-test --duration 3
python controller_test_ps5.py --rumble-on-button --duration 10
```

### Normal controlled demo

```bat
python vision_bridge_controlled.py --gui --controller --use-overhead --steps 3000
```

### Fast controlled demo

```bat
python vision_bridge_controlled.py --gui --controller --control-speed 0.03 --physics-steps 4 --steps 3000
```

### Stronger rumble

```bat
python vision_bridge_controlled.py --gui --controller --rumble-strength 0.8 --rumble-duration 200 --steps 3000
```

### Controlled demo with both cameras

```bat
python vision_bridge_controlled.py --gui --controller --use-overhead --use-wrist --steps 3000
```

### Safe mode

```bat
python vision_bridge_controlled.py --gui --controller --safe-mode --steps 3000
```

### Vision-only demo

```bat
python vision_bridge.py --gui --use-overhead --use-wrist --detector seg --steps 1000
```
