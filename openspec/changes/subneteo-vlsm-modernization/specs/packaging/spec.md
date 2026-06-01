# Delta for Packaging

## ADDED Requirements

### Requirement: pyproject.toml Exists

`pyproject.toml` SHALL exist at the repository root and define a build system with setuptools.

#### Scenario: pyproject.toml minimal structure

- GIVEN `pyproject.toml` exists
- WHEN parsed
- THEN it SHALL contain `[build-system]` with `requires = ["setuptools"]`
- AND `[project]` with `name`, `version`, and `requires-python`

### Requirement: Pytest as Dev Dependency

`pyproject.toml` SHALL list `pytest` as a development dependency.

#### Scenario: pytest in dependencies

- GIVEN `pyproject.toml` content
- WHEN `pytest` is searched
- THEN it SHALL appear under `[project.optional-dependencies]` with key `dev` or `test`

### Requirement: Package Builds Successfully

`python -m build` SHALL produce a valid wheel (`.whl`) file from `pyproject.toml`.

#### Scenario: Build produces wheel

- GIVEN the project root
- WHEN `python -m build` is executed
- THEN a `.whl` file SHALL be created in `dist/`
- AND the wheel SHALL be installable via `pip install`

### Requirement: Package Installs as Module

The package SHALL be installable in editable mode and callable as a module.

#### Scenario: Editable install

- GIVEN `pip install -e .` is executed
- WHEN `python -c "import subneteo"` is run
- THEN no ImportError SHALL occur

### Requirement: Source Layout

The source package SHALL be located at `src/subneteo/` to comply with modern Python packaging conventions.

#### Scenario: Source path

- GIVEN the project structure
- WHEN `src/subneteo/__init__.py` is accessed
- THEN it SHALL exist
- AND `calcular_vlsm` SHALL be importable from `subneteo.core`