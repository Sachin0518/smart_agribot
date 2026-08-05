# Edge-AI Powered Smart Rover for Variable Rate Spraying and Crop Health Monitoring

A fully autonomous, Edge-AI-powered agricultural rover that detects crop diseases in real time and applies pesticide at a variable rate based on infection severity — instead of blanket spraying an entire field.


## Overview

Traditional "blanket spraying" wastes chemicals, degrades soil, and increases health risks for farmers. This project replaces that with a **ground-based 6WD rover** that:

- Runs a custom-trained **YOLOv8** object detection model natively on a **Raspberry Pi 4 Model B** — no cloud dependency, no latency.
- Detects healthy leaves, diseased leaves, and tomato ripeness (riped / unriped) in real time.
- Calculates infection **severity** from the size of the detected bounding box relative to the leaf.
- Dynamically adjusts a water pump's spray duration/pressure via **variable PWM** signals — heavier infection gets more spray, light infection gets less.
- Logs environmental telemetry (soil moisture, temperature/humidity via DHT11) and uploads it over Wi-Fi to **Google Sheets / Firebase** for a remote farmer dashboard.

## Hardware

| Component | Role |
|---|---|
| **Raspberry Pi 4 Model B** | Central Edge-AI compute unit — runs YOLOv8 inference, spray logic, and telemetry |
| 6WD rover chassis + high-torque DC motors | Mobility over uneven farm terrain |
| L298N motor driver | Drive control for navigation |
| USB camera | Live video feed for YOLOv8 |
| DC water pump + Relay module | Variable-rate spraying |
| Soil moisture sensor  | Analog telemetry (Pi 4B has no analog input pins) |
| DHT11 | Temperature & humidity, read directly via GPIO |
| battery | Field-rated power supply |

> **Note:** Since the Raspberry Pi 4 Model B has less onboard compute than a Pi 5, the YOLOv8 model is typically exported to a lighter format (e.g. `.tflite` or ONNX with reduced input resolution) to maintain acceptable real-time frame rates during field inference.

## Software / AI Pipeline

1. **Capture** — live frames from the onboard camera.
2. **Detect** — custom-trained YOLOv8 model classifies `healthy`, `unhealthy`, `riped_tomato`, `unriped_tomato`.
3. **Severity logic** — infection bounding-box area ÷ leaf area → severity score.
4. **Actuate** — severity score → PWM duty cycle → pump driver (e.g. 30% duty for mild infection, 100% for severe).
5. **Telemetry** — sensor + spray-event data packaged as JSON, pushed to cloud backend over Wi-Fi.

## System Block Diagram

![System Block Diagram](images/block_diagram.png)

Place the block diagram image at `images/block_diagram.png` in the repository so it renders here on GitHub.

## Dataset — Sample Images

A subset of the custom tomato-leaf dataset is shown below for reference: raw field captures alongside their annotated (bounding-box labelled) counterparts used to train the YOLOv8 model.

### Unannotated (Raw) Samples

| | | | |
|---|---|---|---|
| ![Raw 1](images/dataset/unannotated/sample_1.jpg) | ![Raw 2](images/dataset/unannotated/sample_2.jpg) | ![Raw 3](images/dataset/unannotated/sample_3.jpg) | ![Raw 4](images/dataset/unannotated/sample_4.jpg) |

### Annotated Samples

| | | | |
|---|---|---|---|
| ![Annotated 1](images/dataset/annotated/sample_1.jpg) | ![Annotated 2](images/dataset/annotated/sample_2.jpg) | ![Annotated 3](images/dataset/annotated/sample_3.jpg) | ![Annotated 4](images/dataset/annotated/sample_4.jpg) |

> Place 4 raw images in `images/dataset/unannotated/` and their labelled counterparts in `images/dataset/annotated/`, named `sample_1.jpg` – `sample_4.jpg` in each folder, so they render above.

## Results (Phase-1)

- Trained over 50 epochs; box loss and classification loss converge smoothly with no overfitting.
- Normalized confusion matrix: 88% healthy, 89% unhealthy, 91% riped tomato, 87% unriped tomato, 95% background true-negative rate.
- Real-time inference on the Raspberry Pi 4B with confidence scores ranging ~37–82% depending on occlusion/lighting.

### Training & Validation Loss Curves

![Training and Validation Loss Curves](images/results/training_loss_curves.png)

*Box loss and classification loss for both training and validation sets over 50 epochs, showing steady convergence with no signs of overfitting.*

## Future Scope

- Solar charging for extended field autonomy
- Broader disease/pest classes across multiple crop types
- Swarm robotics for large-scale field coverage
- Multi-axis nozzle arm to reach leaf undersides

## Team

- Preetham N — 1BY23EC079
- Sachin Huli — 1BY23EC088
- Shreeshail Singare — 1BY23EC100
- Susheel Kumar S — 1BY23EC110

**Guide:** Dr. Suryakanth B, Dept. of ECE, BMSIT&M
