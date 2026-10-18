# Test strategy

Record command, environment, expected result, observed result, and evidence for every test.

| Area | Status policy |
|---|---|
| Syntax/imports | PASS only after execution |
| Dashboard startup | PASS only after startup or `AppTest` evidence |
| Missing checkpoint | PASS when clear error is observed |
| Image/video inference | BLOCKED if checkpoint is unavailable |
| Dataset evaluation | BLOCKED if dataset is unavailable |
| Camera | NOT RUN unless hardware and permissions are available |
| Security/path scan | PASS with command output retained |
