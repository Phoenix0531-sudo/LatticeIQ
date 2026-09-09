# INSTALL — TablePilot

TablePilot is a hybrid stack: a **Python FastAPI analysis service** (`analysis_service/`, pinned deps) plus an optional **Qt 6 / C++ desktop shell** (`Statistical_Analysis/`). Pick the path that matches your goal. All commands verified against the repository as shipped.

## 1. Run the analysis service (the core)

Requirements: Python ≥ 3.11 (`analysis_service/pyproject.toml` pins `requires-python = ">=3.11"`), pip.

```bash
git clone https://github.com/Phoenix0531-sudo/TablePilot.git
cd TablePilot/analysis_service
python -m venv .venv
.venv\Scripts\activate            # Windows (bash: source .venv/bin/activate)
pip install -r requirements.txt   # fastapi==0.141.1, uvicorn==0.52.2, pandas==3.0.5, openpyxl==3.1.5, xlrd==2.0.2, python-multipart==0.0.32 — all pinned
uvicorn app.main:app --reload
```

- Interactive API docs (Swagger): <http://localhost:8000/docs>
- Health probe: `GET /health`
- Full endpoint reference with `curl` examples: [docs/API.md](docs/API.md)

### Dev extras (tests)

```bash
pip install -r requirements-dev.txt   # adds pytest==9.1.1, httpx==0.28.1
pytest analysis_service/tests/        # service test suite (4 files)
pytest tests/                         # repo-level integration checks (2 files)
```

## 2. Run the demo end-to-end

Bundled sample tables live in `demo/` (`quality_issues_demo.csv` has duplicate rows, missing cells, an outlier). With the service running:

```bash
bash scripts/demo_e2e.sh
```

Override the host with `BASE=http://localhost:9000 bash scripts/demo_e2e.sh`.

## 3. Docker

```bash
docker compose up --build
```

The compose file builds the analysis-service image; probe `GET /health` afterwards.

## 4. Desktop shell (optional, Qt 6 / C++)

Only needed if you want the native table UI instead of the HTTP surface:

- Project file: `Statistical_Analysis/Statistical_Analysis.pro` (Qt 6, qcustomplot bundled).
- Build recipe used by CI: see `.github/workflows/qt-desktop.yml` (Qt 6 + MSVC on Windows).
- Release packaging scripts: `packaging/build-windows-release.ps1`, documented in `packaging/README_RELEASE.md`.
- Prebuilt Windows binaries: see the download badge in [README.md](README.md) (Release v1.1.7).

## 5. Pinned versions & dependency locations

| Component | Manifest | Locking |
| --- | --- | --- |
| Analysis service runtime | `analysis_service/requirements.txt` (exact pins) | pins act as the lock |
| Analysis service dev | `analysis_service/requirements-dev.txt` | exact pins |
| Python version | `analysis_service/pyproject.toml` | `requires-python >= 3.11` |

There is intentionally **no root-level requirements file**: the service is the only Python component, and its pins live next to the code so `pip install analysis_service/` resolves the same versions.
