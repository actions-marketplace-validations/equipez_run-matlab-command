# equipez/run-matlab-command

Run MATLAB commands in GitHub Actions using MathWorks' `run-matlab-command`.
Optionally enforce a `timeout` and treat timeout as **success** (exit code `0`).

This action is designed for long-running stress tests where “ran long enough without crashing” is considered a pass.
It is a homemade solution to https://github.com/matlab-actions/run-command/issues/84.

**Vibe-coded, use at your own risk!!!**

## Supported runners / platforms

This action supports **Linux** and **macOS** runners (GitHub-hosted or self-hosted), for example:

- `ubuntu-latest`, `ubuntu-22.04`, `ubuntu-24.04`
- `macos-latest`, `macos-13`, `macos-14` (Intel or Apple Silicon)

Windows is **not** supported.

### Requirements on the runner

- Bash
- `curl` (used to download MathWorks' `run-matlab-command` if it is not already available)
- You must install and license MATLAB first, for example, using `matlab-actions/setup-matlab`.

**This action only runs commands; it does not set up MATLAB licensing by itself.**

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

- If the MATLAB command finishes **before** the timeout:
  - exit code `0` → step succeeds
  - exit code nonzero → step fails (propagates the exit code)
- If the timeout is **reached**:
  - the MATLAB command is terminated
  - the step is treated as **success** (exit `0`)
  - output `timed_out` is set to `true`
  - output `exit_code` is typically `124` (GNU `timeout` convention)

An example:

```yaml
- name: Stress test (timeout => success)
  uses: equipez/run-matlab-command@v1
  with:
    timeout: 5h  # If the commands are still running after 5 hours, the step is terminated and treated as success.
    command: |
      disp("Starting stress test");
      my_stress_test();
```

Timeout handling is implemented by delegating to
[`equipez/run-bash-command`](https://github.com/equipez/run-bash-command),
which wraps the underlying command using GNU `timeout`:

- On **Linux**: uses `timeout`
- On **macOS**: uses `gtimeout` (from Homebrew `coreutils`)

### Note about exit code 124

GNU `timeout` returns `124` when a timeout occurs. If the wrapped command itself returns `124`,
it may be indistinguishable from a timeout under the current design of the timeout wrapper.

## Inputs

| Name | Required | Default | Description |
|---|---:|---|---|
| `command` | yes | — | MATLAB commands to execute (multi-line supported). |
| `timeout` | no | `""` | If set (e.g. `5h`, `45m`, `300s`), terminate after this duration and mark as success. |
| `working-directory` | no | `""` | Working directory to run from. |
| `install-run-matlab-command` | no | `true` | Install MathWorks' `run-matlab-command` if missing. |
| `signal` | no | `TERM` | Signal passed to timeout wrapper (only used if `timeout` is set). |
| `kill-after` | no | `30s` | Kill-after passed to timeout wrapper (only used if `timeout` is set). |
| `quiet` | no | `false` | Reduce logs (only used if `timeout` is set). |

## Outputs

| Name | Description |
|---|---|
| `timed_out` | `true` if the timeout was reached (and treated as success), else `false`. |
| `exit_code` | Exit code of the underlying command (note: `124` typically indicates timeout). |
| `run_matlab_command_path` | Path to the MathWorks' `run-matlab-command` binary. |
| `script_path` | Path to the generated `.m` script file containing your commands. |

## How it works

This action performs the following steps.

1. Ensures MathWorks' `run-matlab-command` is available (downloads it from MathWorks if needed).
2. Writes your `command` into a temporary `.m` file.
3. Executes `run-matlab-command "run('<file>');"`.
4. If `timeout` is set, wraps the execution using `equipez/run-bash-command` so that timeout becomes success.

## About MathWorks' `run-matlab-command`

This action executes MATLAB via MathWorks’ `run-matlab-command` utility, which is designed for CI usage.
In some CI configurations, invoking the MATLAB executable directly (for example `matlab -batch`) may fail due
to licensing/activation context, whereas `run-matlab-command` works reliably.

Note, however, that MathWorks' `run-matlab-command` is undocumented and not officially supported by MathWorks,
so use at your own
risk. See https://github.com/matlab-actions/run-command/issues/53 for more discussion.

If MathWorks' `run-matlab-command` stops working, consider using the officially supported
[`matlab-batch` utility](https://github.com/mathworks-ref-arch/matlab-dockerfile/blob/main/alternates/non-interactive/MATLAB-BATCH.md).
`matlab-batch` needs a [MATLAB batch licensing token](https://github.com/matlab-actions#use-matlab-batch-licensing-token)
provided with the `MLM_LICENSE_TOKEN` environment variable via a [secret](https://docs.github.com/en/actions/how-tos/write-workflows/choose-what-workflows-do/use-secrets).

## License

See `LICENSE.txt`.
