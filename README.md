# templatelists

[`osv-scanner.json`](https://google.github.io/osv-scanner/supported-languages-and-lockfiles/#custom-lockfiles) used as a [CMake FetchContent](https://cmake.org/cmake/help/latest/module/FetchContent.html) lockfile

## Features

- Runs `osv-scanner --lockfile osv-scanner:osv-scanner.json` if [`PROJECT_IS_TOP_LEVEL`](https://cmake.org/cmake/help/latest/variable/PROJECT_IS_TOP_LEVEL.html) is `TRUE`
- Populates content (shallow clone) using repo(s) and commit hash(es) from `osv-scanner.json`

## Versioning

- Tags - stable SemVer
- `main` branch - unstable
