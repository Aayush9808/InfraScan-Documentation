# Khushi frontend guide

## Planned scope

Streamlit interface, upload workflow, result presentation, UI state, and error handling. Confirm completed work before presenting it as personal contribution.

## Key topics

- Streamlit reruns scripts to update UI state.
- File uploaders provide in-memory uploaded files that can be copied to temporary paths.
- The dashboard should distinguish empty, processing, success, and error states.
- Annotated media and telemetry should be displayed only when actually generated.
- Missing checkpoints must produce a clear error rather than a fake result.

## Questions

1. Why Streamlit? Fast Python-native prototyping.
2. What is a rerun? Script execution triggered by widget interaction.
3. Why temporary files? Video libraries operate on paths and uploads must not be overwritten.
4. How is an error shown? Through explicit status/error UI.
5. Why not embed a separate frontend? Existing architecture is Streamlit.
6. What is session state? Per-user values retained across reruns.
7. What is a file uploader? A UI control yielding uploaded file data.
8. How do you prevent stale output? Use per-run temporary paths and clear status.
9. What is a download button for? Delivering generated output to the user.
10. How do you test UI? Streamlit `AppTest` and a running health endpoint.
11. What if video decoding fails? Report a clear processing error.
12. Why not claim webcam browser support? It depends on the actual implementation.
13. What is accessibility? Clear labels, readable contrast, and non-color-only status.
14. What is a frontend limitation? Streamlit is not a full SPA framework.
15. How is model state managed? Through the app's configured model lifecycle.
16. Why show actual metrics only? Fabricated metrics mislead users.
17. What happens without weights? Inference is unavailable.
18. Why use tabs? Separate image, video, and system workflows.
19. How are outputs protected? Temporary handling and cleanup.
20. What should a UI screenshot show? Input, output, status, and test conditions.

## Short explanation

“I explain the Streamlit layer: it collects inputs, presents controls, calls the existing detection pipeline, and displays generated results. The important design choice is honest state handling—missing assets and failed processing are visible errors, not success-shaped placeholders.”
