# Poster content

## Header

**InfraScan — AI-Powered Road Inspection and Pothole Detection System**
Group ID: **[TO BE CONFIRMED]**
Team: Aayush Kumar Shrivastav, Kajal Rawal, Khushi Shakya, Ronak Saini
Supervisor: **[TO BE CONFIRMED]**

## Abstract

InfraScan is a Python-based computer-vision workflow for pothole inspection. It connects an Ultralytics object-detection pipeline with image, video, camera-oriented, and Streamlit dashboard workflows. The system produces annotated outputs and telemetry where supported. The public release intentionally excludes the trained checkpoint and dataset because redistribution rights are not verified.

## Introduction

Pothole inspection is a visual task that can benefit from consistent computer-assisted screening. InfraScan is intended as a demonstrator and engineering prototype, not a certified infrastructure-management system.

## Objectives and motivation

- Localize potholes in road imagery.
- Support practical image and video inspection.
- Present results in an accessible dashboard.
- Preserve reproducible source documentation and honest evidence boundaries.

## Methodology

Input → validation/temporary handling → model initialization → YOLO inference → confidence filtering and box rendering → severity/telemetry processing → dashboard display and export.

## Experimentation and results

Report only executed tests. Current release evidence includes syntax checks, dependency checks, dashboard smoke testing, and missing-checkpoint handling. Model accuracy and fresh dataset evaluation are blocked until authorized assets are available.

## Conclusion

InfraScan integrates detection logic with a usable inspection interface. Future work includes licensed data access, fresh evaluation, calibration, multi-environment testing, and deployment assessment.

## References

Use the bibliography in `10-references-and-governance/bibliography.md`; cite Ultralytics and the upstream repository separately.
