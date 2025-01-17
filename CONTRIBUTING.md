# Contribution Guide

## First steps

Clone the repository and install dependencies.

```sh
$ git clone git@github.com:/tarantool/go-tarantool
$ cd go-tarantool
$ go get .
```

## Running tests

You need to [install Tarantool](https://tarantool.io/en/download/) to run tests.
See the Installation section in the README for requirements.

To install test dependencies (such as the
[tarantool/queue](https://github.com/tarantool/queue) module), run:
```bash
make deps
```

To run tests for the main package and each subpackage:
```bash
make test
```

The tests set up all required `tarantool` processes before run and clean up
afterwards.

If you want to run the tests with specific build tags:
```bash
make test TAGS=go_tarantool_ssl_disable,go_tarantool_msgpack_v5
```

If you have Tarantool Enterprise Edition 2.10 or newer, you can run additional
SSL tests. To do this, you need to set an environment variable 'TEST_TNT_SSL':

```bash
TEST_TNT_SSL=true make test
```

If you want to run the tests for a specific package:
```bash
make test-<SUBDIR>
```
For example, for running tests in `multi`, `uuid` and `main` packages, call
```bash
make test-multi test-uuid test-main
```

To run [fuzz tests](https://go.dev/doc/tutorial/fuzz) for the main package and each subpackage:
```bash
make TAGS="go_tarantool_decimal_fuzzing" fuzzing
```

To check if the current changes will pass the linter in CI, install
golangci-lint from [sources](https://golangci-lint.run/usage/install/)
and run it with next command:
```bash
make golangci-lint
```

To format the code install [goimports](https://pkg.go.dev/golang.org/x/tools/cmd/goimports)
and run it with next command:
```bash
make format
```

## Benchmarking

### Quick start

To run all benchmark tests from the current branch run:

```bash
make bench
```

To measure performance difference between master and the current branch run:

```bash
make bench-diff
```

Note: `benchstat` should be in `PATH`. If it is not set, call:

```bash
export PATH="/home/${USER}/go/bin:${PATH}"
```

or

```bash
export PATH="${HOME}/go/bin:${PATH}"
```

### Customize benchmarking

Before running benchmark or measuring performance degradation, install benchmark dependencies:
```bash
make bench-deps BENCH_PATH=custom_path
```

Use the variable `BENCH_PATH` to specify the path of benchmark artifacts.
It is set to `bench` by default.

To run benchmark tests, call:
```bash
make bench DURATION=5s COUNT=7 BENCH_PATH=custom_path TEST_PATH=.
```

Use the variable `DURATION` to set the duration of perf tests. That variable is mapped on
testing [flag](https://pkg.go.dev/cmd/go#hdr-Testing_flags) `-benchtime` for gotest.
It may take the values in seconds (e.g, `5s`) or count of iterations (e.g, `1000x`).
It is set to `3s` by default.

Use the variable `COUNT` to control the count of benchmark runs for each test.
It is set to `5` by default. That variable is mapped on testing flag `-count`.
Use higher values if the benchmark numbers aren't stable.

Use the variable `TEST_PATH` to set the directory of test files.
It is set to `./...` by default, so it runs all the Benchmark tests in the project.

To measure performance degradation after changes in code, run:
```bash
make bench-diff BENCH_PATH=custom_path
```

Note: the variable `BENCH_PATH` is not purposed to be used with absolute paths.

## Recommendations for how to achieve stable results

Before any judgments, verify whether results are stable on given host and how large the noise. Run `make bench-diff` without changes and look on the report. Several times.

There are suggestions how to achieve best results:

* Close all background applications, especially web browser. Look at `top` (`htop`, `atop`, ...) and if something bubbles there, close it.
* Disable cron daemon.
* Disable TurboBoost and set fixed frequency.
  * If you're using `intel_pstate` frequency driver (it is usually default):

    Disable TurboBoost:

    ```shell
    $ echo 0 > /sys/devices/system/cpu/intel_pstate/no_turbo
    ```

    Set fixed frequency: not sure it is possible.

    * If you're using `acpi-cpufreq` driver:

    Ensure you actually don't use intel_pstate:

    ```shell
    $ grep -o 'intel_pstate=\w\+' /proc/cmdline
     intel_pstate=disable
     $ cpupower -c all frequency-info | grep driver:
       driver: acpi-cpufreq
       <...>
     ```

     Disable TurboBoost:

     ```shell
     $ echo 0 > /sys/devices/system/cpu/cpufreq/boost
     ```

     Set fixed frequency:

     ```shell
     $ cpupower -c all frequency-set -g userspace
     $ cpupower -c all frequency-set -f 1.80GHz # adjust for your CPU
     ```

## Code review checklist

- Public API contains functions, variables, constants that are needed from
  outside by users. All the rest should be left closed.
- Public functions, variables and constants contain at least a single-line
  comment.
- Code is DRY (see "Do not Repeat Yourself" principle).
- New features have functional and probably performance tests.
- There are no changes in files not related to the issue.
- There are no obvious flaky tests.
- Commits with bugfixes have tests based on reproducers.
- Changelog entry is present in `CHANGELOG.md`.
- Public methods contain executable examples (contains a comment with
  reference output).
- Autogenerated documentation looks good. Run `godoc -http=:6060` and point
  your web browser to address "http://127.0.0.1:6060" for evaluating.
- Commit message header may start with a prefix with a short description
  follows after colon. It is applicable to changes in a README, examples, tests
  and CI configuration files. Examples: `github-ci: add Tarantool 2.x-latest`
  and `readme: describe how to run tests`.
- Check your comments, commit title, and even variable names to be
  grammatically correct. Start sentences from a capital letter, end with a dot.
  Everywhere - in the code, in the tests, in the commit message.

See also:

- https://github.com/tarantool/tarantool/wiki/Code-review-procedure
- https://www.tarantool.io/en/doc/latest/contributing/
