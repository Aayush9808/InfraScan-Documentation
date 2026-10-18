# Project overview

InfraScan is a pothole-detection application built around Python, OpenCV, Ultralytics, and Streamlit. The source repository contains a detection engine, dashboard, training/evaluation/export utilities, camera support, and dataset preparation scripts.

## Scope

The implemented scope is visual detection and result presentation. It does not establish road-condition certification, depth measurement, geolocation, automatic maintenance decisions, or safety-critical deployment.

## Data flow

```mermaid
flowchart LR
  Input[Image / video / camera] --> Validate[Validate input]
  Validate --> Model[Load compatible checkpoint]
  Model --> Infer[Ultralytics inference]
  Infer --> Render[Boxes and severity rendering]
  Render --> Output[Dashboard, saved media, telemetry]
```
