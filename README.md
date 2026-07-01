# ci-cdash

Follow these steps to add a CDash workflow to an NCEPLIBS repository.

## Create the project on my.cdash.org

- Enter basic repo info:

![cdash1](https://github.com/user-attachments/assets/c5ea860f-b467-42f8-a82b-d6f2ab40665a)

- Enter URL/branch:

![cdash2](https://github.com/user-attachments/assets/3615e9e4-08ab-4aff-9964-f7b2805cff25)

- Configure test settings:

![cdash3](https://github.com/user-attachments/assets/585d0315-69e2-4f87-a106-6f17c64f9925)

- Set retention time/max builds:

![cdash4](https://github.com/user-attachments/assets/b23607ec-38cd-4db1-932d-c1c6fafa457c)

- Set maintainers (remove default cdash.org maintainer):

![cdash5](https://github.com/user-attachments/assets/d0fb3680-494f-41c6-8cb9-bfc02a5b771c)

- Create an authentication token if you do not already have one configured at https://my.cdash.org/user, under "My Authentication Tokens." Set permission to "Submit only."

## Add workflow to an NCEPLIBS repo:
- Create .github/workflows/cdash.yml:
```yaml
name: cdash
on:
  push:
    branches:
    - develop
# add test branch if needed

jobs:
  cdash:
    runs-on: ubuntu-latest
    env:
      FC: gfortran
      CC: gcc
      CDASH_TOKEN: ${{ secrets.CDASH_TOKEN }}

    steps:

    # Install dependencies, typically with the NOAA-EMC/ci-build-nceplibs@oneinstalldir action

    - name: CDash
      uses: NOAA-EMC/ci-cdash@develop
```

- Populate the CDASH_TOKEN under your repository's Settings>Secrets and Variables>Actions>Repository secrets>New repository secret:

![cdashGH](https://github.com/user-attachments/assets/045f383f-e1fb-48e6-a4df-b606123c0ab5)

## Optional: gprof profiling measurements

Set `profiling: true` to have every CTest test built with gprof instrumentation
and run its own hotspot analysis, whose results are attached to that test on
CDash as `<CTestMeasurement>` entries (function name / self time in seconds).
This is implemented by [AlexanderRichert-NOAA/ci-profile-tests](https://github.com/AlexanderRichert-NOAA/ci-profile-tests);
this action clones it, and the tested project must opt in to picking up its
`Profiling.cmake` module.

Add the following to your project's top-level `CMakeLists.txt`, immediately
after `enable_testing()`:

```cmake
enable_testing()

# Populated by ci-cdash's `profiling: true` input; no-op otherwise, so the
# project can also enable profiling on its own (see ci-profile-tests' README
# for the FetchContent-based, CDash-independent way of doing that locally).
if(DEFINED PROFILING_INCLUDE_FILE)
  include("${PROFILING_INCLUDE_FILE}")
endif()
```

Then enable it in the workflow:

```yaml
    - name: CDash
      uses: NOAA-EMC/ci-cdash@develop
      with:
        profiling: 'true'
        profiling-ref: 'main'   # optional: pin a ci-profile-tests tag/SHA
```

Notes:

- Profiling is gprof-only through this action; `PROFILING_TOOL`,
  `PROFILING_ANALYSIS`, and `ENABLE_CDASH` are set automatically.
- Only the top 25 hottest functions per test are reported by default. Override
  with `extra-cmake-options: '-DPROFILING_CDASH_TOP_N=<N>'` (`0` = no limit).
- If your project's CMakeLists.txt lives below the repo root, set `source-dir`
  accordingly (e.g. `source-dir: 'test'`).
- CDash+profiling and CDash-without-profiling (`profiling` omitted/`false`)
  and profiling-without-CDash (using ci-profile-tests directly, see its own
  README) all work independently; nothing about this action requires
  profiling, and nothing about profiling requires this action.

