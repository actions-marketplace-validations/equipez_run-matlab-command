# equipez/run-matlab-command

Run MATLAB commands in GitHub Actions using MathWorks `run-matlab-command`.
Optionally enforce a `timeout` and treat timeout as **success** (exit code `0`).

This action is designed for long-running stress tests where “ran long enough without crashing” is considered a pass.

## Supported runners

- `ubuntu-latest`
- `macos-latest`

(Windows is intentionally not supported.)

## Prerequisites

You must install and license MATLAB first (for example):

- `matlab-actions/setup-matlab@v2`
- `matlab-actions/activate-matlab@v2`

This action only runs commands; it does not set up MATLAB licensing by itself.

## Usage

### Basic (no timeout)

```yaml
- name: Run MATLAB commands
  uses: equipez/run-matlab-command@v1
  with:
    command: |
      disp(version);
      results = runtests;
      assertSuccess(results);
```

### Timeout that becomes success

If the commands are still running after `5h`, the step is terminated and treated as success.

```yaml
- name: Stress test (timeout => success)
  uses: equipez/run-matlab-command@v1
  with:
    timeout: 5h
    command: |
      disp("Starting stress test");
      my_stress_test();
```

## Inputs

| Name | Required | Default | Description |
|---|---:|---|---|
| `command` | yes | — | MATLAB commands to execute (multi-line supported). |
| `timeout` | no | `""` | If set (e.g. `5h`, `45m`, `300s`), terminate after this duration and mark as success. |
| `working-directory` | no | `""` | Working directory to run from. |
| `install-run-matlab-command` | no | `true` | Install `run-matlab-command` if missing. |
| `signal` | no | `TERM` | Signal passed to timeout wrapper (only used if `timeout` is set). |
| `kill-after` | no | `30s` | Kill-after passed to timeout wrapper (only used if `timeout` is set). |
| `quiet` | no | `false` | Reduce logs (only used if `timeout` is set). |

## Outputs

| Name | Description |
|---|---|
| `timed_out` | `true` if the timeout was reached (and treated as success), else `false`. |
| `exit_code` | Exit code of the underlying command (note: `124` typically indicates timeout). |
| `run_matlab_command_path` | Path to the `run-matlab-command` binary. |
| `script_path` | Path to the generated `.m` script file containing your commands. |

## How it works

1. Ensures `run-matlab-command` is available (downloads it from MathWorks if needed).
2. Writes your `command` into a temporary `.m` file.
3. Executes `run-matlab-command "run('<file>');"`.
4. If `timeout` is set, wraps the execution using `equipez/success-on-timeout` so that timeout becomes success.

## License

See `LICENSE.txt`.
