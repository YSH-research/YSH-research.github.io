# How to E2E Evaluation in Carla Simulation with Bench2Drive & Roadrunner

**Date:** 2026.02.23~25 /
**Speaker:** Prof. Kichun Jo /
**Organization:** Hanyang Univ. /
**Category:** E2E, Carla

---

## Introduction

Learned about the latest trends in E2E autonomous driving, and how to create scenarios using Roadrunner, Scenariorunner, and OpenDrive for Bench2Drive. Also covered running scenarios in CARLA simulation and adding a new "speed violation" evaluation metric to Bench2Drive.

![](assets/2026-02-23/Class.jpg){ width="600" }

---

## Main Content

### Carla Sensor Configuration via Python API
Connected to Carla via Python API and configured sensors (GNSS, IMU, Camera, LiDAR).

=== "RGB Camera"
    ![](assets/2026-02-23/Class1-Carla-API/01_RGB_Cam01.png){ width="600" }

=== "LiDAR"
    ![](assets/2026-02-23/Class1-Carla-API/04_Lidar.png){ width="600" }

=== "LiDAR + Noise"
    ![](assets/2026-02-23/Class1-Carla-API/04_Lidar_With_Noise.png){ width="600" }

=== "Radar"
    ![](assets/2026-02-23/Class1-Carla-API/05___Radar.png){ width="600" }

=== "GNSS"
    ![](assets/2026-02-23/Class1-Carla-API/06_GNSS.png){ width="600" }

=== "IMU"
    ![](assets/2026-02-23/Class1-Carla-API/07_imu.png){ width="600" }

=== "RGB + IMU"
    ![](assets/2026-02-23/Class1-Carla-API/11___RGB+imu.png){ width="600" }

### Create Carla Scenario Using Scenario Runner
Created scenarios using Roadrunner and imported them into CARLA based on OpenScenario 1.0.

=== "Step 1"
    ![](assets/2026-02-23/Class2-RoadRunner/1.png){ width="600" }

=== "Step 2"
    ![](assets/2026-02-23/Class2-RoadRunner/2.png){ width="600" }

=== "Step 3"
    ![](assets/2026-02-23/Class2-RoadRunner/3.png){ width="600" }

=== "Step 4"
    ![](assets/2026-02-23/Class2-RoadRunner/4.png){ width="600" }

=== "Step 5"
    ![](assets/2026-02-23/Class2-RoadRunner/5.png){ width="600" }

### Running & Evaluating E2E(VAD) on Bench2Drive
Ran inference on VAD using Bench2Drive and implemented a new evaluation metric (speed violation) through code modification.

=== "Step 1"
    ![](assets/2026-02-23/Class3-Bench2Drive/1.png){ width="600" }

=== "Step 2"
    ![](assets/2026-02-23/Class3-Bench2Drive/2.png){ width="600" }

=== "Step 3"
    ![](assets/2026-02-23/Class3-Bench2Drive/3.png){ width="600" }

---