# giise2ebench
GIISE2EBench: a benchmark of E2E test suites with their applications

## Demonstrators

Each demonstrator is a system under test (SUT) together with its end-to-end test suite, included as a Git submodule.

### FullTeaching

FullTeaching is an educational web application that makes online classes easy for teachers and students: courses, sessions, a forum per course, file sharing and video-conference classes through OpenVidu (Angular and Spring Boot).

Repository: [giise2ebench-st-fullteaching](https://github.com/augustocristian/giise2ebench-st-fullteaching) (submodule [`giise2ebench-st-fullteaching`](giise2ebench-st-fullteaching))

<p align="center"><img src="img/hero-fullteaching.svg" alt="FullTeaching dashboard (courses and calendar) on desktop and mobile" width="900"/></p>

### DevorApp

DevorApp is a restaurant discovery and recommendation app (React and Capacitor frontend, FastAPI backend, PostgreSQL and a Keras recommendation service).

Repository: [giise2ebench-st-devorapp](https://github.com/augustocristian/giise2ebench-st-devorapp) (submodule [`giise2ebench-st-devorapp`](giise2ebench-st-devorapp))

<p align="center"><img src="img/hero-devorapp.svg" alt="DevorApp home screen on desktop and mobile" width="900"/></p>

### ClimaNuvem

ClimaNuvem is a mobile application that analyses cloud photographs and generates a short-term local weather forecast (Expo and React Native app, FastAPI backend, Firebase authentication, PostgreSQL and a multimodal model served by Ollama).

Repository: [giise2ebench-st-climanuvem](https://github.com/augustocristian/giise2ebench-st-climanuvem) (submodule [`giise2ebench-st-climanuvem`](giise2ebench-st-climanuvem))

<p align="center"><img src="img/hero-climanuvem.svg" alt="ClimaNuvem home screen (guest session) on desktop and mobile" width="900"/></p>

## Common structure of the demonstrator repositories

All the demonstrators follow the same layout, so a benchmark user (or tool) can find the same things in the same place in every one of them:

```
giise2ebench-st-<name>/
├── docs/                    # Requirements of the SUT
│   ├── userrequirements_en.txt
│   └── userrequirements_es.txt
├── sut/                     # System under test (vendored as-is; fetched by setup-sut in FullTeaching)
├── deploy.sh / deploy.ps1   # Deployment script of the SUT (Linux/macOS and Windows)
├── selenium-java/           # Test suite 1: Java + JUnit 5 + Selenium
├── cypress-js/              # Test suite 2: JavaScript + Cypress
├── playwright-csharp/       # Test suite 3: C# + NUnit + Playwright
├── puppeteer-python/        # Test suite 4: Python + pytest + Puppeteer (through a Node bridge)
├── .github/                 # CI (workflows, start-sut action, Dependabot, test-summary script)
├── img/                     # Images used by the README (hero banner, GIIS footer)
├── CLAUDE.md                # Context for agentic tools working in the repo
└── README.md                # Entry point: deployment, how to run each suite, CI
```

Every suite covers the same scenarios (the requirements in `docs/`) with its own stack, and all of them are structured in the same way: shared helpers (`Common/`, `common/`, `support/`), one Page Object per screen, and the tests split in API tests and UI (system) tests.

### Where is each thing

| What | Where | Notes |
|---|---|---|
| **Requirements** | `docs/userrequirements_en.txt`, `docs/userrequirements_es.txt` | Numbered list of user requirements (English and Spanish). They are consumed by the benchmark, not by the SUT |
| **System under test** | `sut/` | Application source, Dockerfiles and Docker Compose files. Upstream documentation in `sut/README.md` |
| **Deployment scripts** | `deploy.sh` (Linux/macOS), `deploy.ps1` (Windows), at the repository root | Wrappers over Docker Compose; see [Deployment](#deployment-scripts) |
| **Test suites** | `selenium-java/`, `cypress-js/`, `playwright-csharp/`, `puppeteer-python/` | Each one has its own `README.md` with prerequisites, configuration and run commands |
| **Test data** | Per suite: `src/test/resources/` (Selenium), `cypress/fixtures/` (Cypress), `Resources/` (Playwright), `tests/resources/` or `resources/inputs/` (Puppeteer) | Test accounts are loaded from a CSV; the `*.template.csv` is versioned, the `*.local.csv` is git-ignored |
| **CI/CD** | `.github/workflows/` | Builds the SUT images once and runs every suite in parallel, each on its own runner and its own SUT instance |

### Deployment scripts

| Repository | Script | What it does |
|---|---|---|
| FullTeaching | `setup-sut.sh` / `setup-sut.ps1`, then `deploy.sh` / `deploy.ps1` | `setup-sut` clones [`giis-uniovi/retorch-st-fullteaching`](https://github.com/giis-uniovi/retorch-st-fullteaching) and splits it into `sut/` and `selenium-java/` (both git-ignored, fetched on demand). `deploy` delegates to `selenium-java/deploy-sut.sh` (`deploy.ps1 -Down` / `deploy.sh down` stops it). The SUT answers on `https://localhost:5000`. CI and the Cypress, Playwright and Puppeteer suites run against `sut/docker-compose.yml` plus `docker-compose.sut-openvidu.yml` (an override that runs a compatible OpenVidu); see the repository's README |
| DevorApp | `deploy.sh` / `deploy.ps1` | Options `--mode docker\|native`, `--component all\|backend\|frontend`, `--dev` (dev stack with hot reload), `--skip-tests`, `--apk`, `--stop` (PowerShell: `-Dev`). The Compose files are in `sut/` (`docker-compose.yml` + `docker-compose.dev.override.yml`) |
| ClimaNuvem | `deploy.sh` / `deploy.ps1` | `backend`, `frontend`, `all` or `down [--volumes]`. The Compose files are in `sut/backend/`; the test stack (Firebase emulator, test mode) is in `sut/docker-compose.test.yml` and in `selenium-java/scripts/sut-compose.sh` |

The test suites are run against the stack deployed in **dev/test mode** (Firebase emulator, mocks of external services); production deployment needs real accounts and is described in each repository's `README.md`.

### Test suites

| Suite | Directory | Stack | Run |
|---|---|---|---|
| Selenium + Java | `selenium-java/` | JDK, Maven, JUnit 5, Selenium | `mvn test` |
| Cypress + JavaScript | `cypress-js/` | Node.js, Cypress, Mocha | `npm test` |
| Playwright + C# | `playwright-csharp/` | .NET SDK, NUnit, Microsoft.Playwright | `dotnet test` |
| Puppeteer + Python | `puppeteer-python/` | Python, pytest, Node.js bridge, Chrome | `pytest` (see its README) |

Inside each suite the tests are organised in layers:

- **API tests** (`tests/api`, `Tests/Api`, `cypress/e2e/api`, `.../tests/api`): HTTP-level contract tests against the backend, without a browser.
- **UI / system tests** (`tests/e2e`, `Tests/E2e` or `Tests/E2E`, `cypress/e2e/system` or `cypress/e2e/ui`, `.../tests/e2e`): drive the web frontend through a Page Object Model.
- **Bridge smoke tests** (`puppeteer-python/tests/bridge`): check the Python-Node bridge only, no SUT needed.

Besides the E2E suites, the SUT has its own **component tests** (unit/integration), which are independent of the benchmark suites:

| Repository | Component tests |
|---|---|
| DevorApp | `sut/backend/tests` (pytest), `sut/keras-api` (pytest), `sut/gplaces-mock` (pytest), `sut/frontend/src/tests` (Vitest) and `sut/frontend/e2e` (Playwright with the network mocked) |
| ClimaNuvem | `sut/backend/tests` (pytest) and `sut/frontend/__tests__` (Jest) |
| FullTeaching | Not vendored: `sut/` is fetched from upstream |

### Differences between repositories

| | FullTeaching | DevorApp | ClimaNuvem |
|---|---|---|---|
| `sut/` | Fetched with `setup-sut` (git-ignored) | Committed | Committed |
| `selenium-java/` | Fetched with `setup-sut` (git-ignored) | Committed | Committed |
| `cypress-js/`, `playwright-csharp/` | Committed | Committed | Committed |
| `puppeteer-python/` | Committed | Committed | Committed |
| Test layers | UI/system only (courses, forum, links, REST operations, chat and video sessions) | API + UI | API + UI (login, register, profile, image analysis) |
| CI workflow | `ci.yml` | `ci.yml` | `ci-cd.yml` (also publishes docs and the Android APK) |

---

<p align="center">
  <a href="https://giis.uniovi.es/"><img src="img/giis-footer.svg" alt="GIIS - Software Engineering Research Group, University of Oviedo" height="28"/></a>
</p>
