# Project FRAME — FRC 8153 Tech4Peace 2026 Robot Code

🧿 *çalışçak amin*

Robot code for **FRC Team 8153 Tech4Peace** (Turkey) for the 2026 *REBUILT* season. It builds on the codebase from our 2025 season. It's written in Java on WPILib's command-based framework, with a CTRE Phoenix 6 swerve drivetrain, Limelight AprilTag heading alignment and PathPlanner autonomous.

---

## ✨ Features

- **Swerve drive.** Four-module swerve on CTRE Phoenix 6, generated with the Tuner X Swerve Project Generator. TalonFX drive and steer motors, CANcoders, and a Pigeon 2 gyro.
- **Field-centric control.** Each alliance gets its own operator perspective, so "forward" always points away from your driver station. The perspective is re-applied if the code restarts mid-match.
- **AprilTag heading alignment.** A Limelight-based PID controller rotates the robot to face an AprilTag while the driver keeps control of translation.
- **PathPlanner autonomous.** Autos are chosen from a SmartDashboard chooser and followed with a holonomic drive controller. Paths flip automatically for the Red alliance.
- **SysId characterization.** Translation, steer and rotation routines are built in for tuning the drive gains.
- **Telemetry.** Robot pose, module states and speeds are published to NetworkTables and logged with CTRE `SignalLogger`.
- **Simulation.** Desktop simulation is supported, with a 5 ms physics loop.

---

## 🛠️ Tech Stack

| Component | Version |
|---|---|
| WPILib / GradleRIO | 2025.3.1 |
| Java | 17 |
| CTRE Phoenix 6 | 25.3.1 |
| PathPlannerLib | 2025.2.4 |
| REVLib | 2025 |
| WPILib New Commands | 2025 |
| Limelight | via `LimelightHelpers.java` |

> **Note:** The vendor libraries and GradleRIO version are still the ones pinned from 2025. To build with the 2026 WPILib, update `build.gradle` and the `vendordeps/` files, either with **WPILib: Import a WPILib 2025 Gradle Project** or by changing them by hand.

---

## 🎮 Driver Controls

The driver uses an Xbox controller on port 0.

| Input | Action |
|---|---|
| Left stick | Translate (field-centric) |
| Right stick X | Rotate |
| **Button 3 (X)** *(hold)* | AprilTag heading alignment |
| **A** *(hold)* | Brake mode (X-lock wheels) |
| **B** *(hold)* | Point wheels in the left-stick direction |
| **Left bumper** | Reset field-centric heading |
| **Back + Y** / **Back + X** | SysId dynamic forward / reverse |
| **Start + Y** / **Start + X** | SysId quasistatic forward / reverse |

The joysticks have a 10% deadband on translation and rotation. Maximum rotation speed is 0.75 rotations per second.

---

## 🎯 AprilTag Alignment

`AlignWithAprilTagCommand` runs while button 3 is held:

1. It reads the target pose in robot space from the Limelight (`maylimelight`).
2. If a tag is visible and the robot is more than **1.0 m** away, a PID controller (`kP = 0.1`, `kD = 0.01`, ±1° tolerance) turns the robot to face the tag squarely.
3. If no tag is visible, or the robot is already within 1.0 m, the driver has full manual rotation.
4. The driver controls translation the whole time.

Debug values are published under `AprilTag/*` on SmartDashboard: `HasTarget`, `Distance`, `HeadingError`, `RotationRate`, `IsAligning` and `IsCloseEnough`.

---

## 🤖 Autonomous

Autos live in `src/main/deploy/pathplanner/` and are picked up automatically by `AutoBuilder.buildAutoChooser()`. Choose one from **"Auto Chooser"** on SmartDashboard or Shuffleboard before the match.

| Auto | Description |
|---|---|
| `New Auto` | Follows the `sigma` path and resets odometry at the start |

PathPlanner holonomic controller gains are translation `P = 10` and rotation `P = 7`.

---

## 🔌 Hardware Map

All swerve devices sit on the CANivore bus **`ismail`**.

| Module | Drive ID | Steer ID | CANcoder ID |
|---|---|---|---|
| Front Left | 17 | 14 | 24 |
| Front Right | 15 | 16 | 21 |
| Back Left | 11 | 10 | 22 |
| Back Right | 12 | 18 | 23 |

| Device | ID |
|---|---|
| Pigeon 2 | 31 |
| SPARK MAX (`ceyhun`, brushless) | 1 (declared, not yet used) |

Gear ratios are 6.75 : 1 for drive, 21.43 : 1 for steer and 3.57 for coupling.

---

## 🚀 Getting Started

### Requirements
- [WPILib](https://docs.wpilib.org/en/stable/docs/zero-to-robot/step-2/wpilib-setup.html), which includes VS Code, JDK 17 and Gradle
- FRC Game Tools (Driver Station)
- CTRE Phoenix Tuner X, for configuring devices
- PathPlanner, for editing paths and autos

### Build
```bash
git clone <this-repo-url>
cd <repo-folder>
./gradlew build
```

### Deploy to the robot
Connect to the robot over USB, Ethernet or radio Wi-Fi, then run:
```bash
./gradlew deploy
```
The team number is read from `.wpilib/wpilib_preferences.json` (`8153`).

### Simulate
```bash
./gradlew simulateJava
```
You can also use **WPILib: Simulate Robot Code** in VS Code.

---

## 📂 Project Structure

```
src/main/
├── java/frc/robot/
│   ├── Main.java                         # Entry point
│   ├── Robot.java                        # TimedRobot lifecycle
│   ├── RobotContainer.java               # Controller bindings, auto chooser
│   ├── Telemetry.java                    # NetworkTables + SignalLogger telemetry
│   ├── LimelightHelpers.java             # Limelight NetworkTables helper library
│   ├── commands/
│   │   └── AlignWithAprilTagCommand.java # Vision-based heading alignment
│   ├── subsystems/
│   │   └── CommandSwerveDrivetrain.java  # Swerve subsystem, PathPlanner, SysId
│   └── generated/
│       └── TunerConstants.java           # Tuner X generated swerve constants
└── deploy/
    └── pathplanner/                      # Paths, autos, robot config, navgrid
```

---

## ⚠️ Known Issues / TODO

- [ ] **Check `TunerConstants.java` units.** Wheel radius is `Inches.of(20)` and module positions are `±106.5 in`. These look like mis-entered values, since a typical robot has a wheel radius of about 2 in and module offsets of about 10–13 in. Regenerate the file in Tuner X or correct the values by hand. `kSpeedAt12Volts` (47.31 m/s) is unrealistic for the same reason.
- [ ] **Update `pathplanner/robotconfig.json` and the PathPlanner GUI settings** with the real robot mass, MOI, dimensions and module config.
- [ ] **Fix the alignment fallback rotation.** In `AlignWithAprilTagCommand`, manual rotation is scaled by `maxSpeed` (m/s) instead of the max angular rate (rad/s).
- [ ] **Bring the SPARK MAX (`ceyhun`, ID 1) into a subsystem** or remove it.
- [ ] **Implement or remove** the empty button 8 and button 9 branches in the default drive command.
- [ ] **Tune drive and steer gains** with the SysId routines.
- [ ] **Move the project to 2026 WPILib and vendor libraries.**

---

## 👥 Team

**Tech4Peace — FRC Team 8153**
- 📍 Turkey
- 🗓️ Rookie year: 2020
- 📸 Instagram: [@acihsrobotics](https://www.instagram.com/acihsrobotics/)

---

## 📝 License

Licensed under the WPILib BSD license. See [`WPILib-License.md`](WPILib-License.md).
