# Delta for Test Suite

## ADDED Requirements

### Requirement: Pytest Coverage Threshold

The test suite SHALL achieve ≥80% line coverage on `src/subneteo/` when run with `pytest --cov=src/subneteo --cov-report=term-missing`.

#### Scenario: Coverage report shows ≥80%

- GIVEN the pytest suite is executed with coverage
- WHEN the coverage report is generated
- THEN the displayed coverage percentage SHALL be ≥80%

#### Scenario: No uncovered critical paths

- GIVEN all functions in core.py are tested
- WHEN coverage is measured
- THEN there SHALL be no uncovered branches in `calcular_vlsm()` or `calcular_enlaces_router()`

### Requirement: VLSM Calculation Tests

The test suite SHALL include parametrized tests for `calcular_vlsm()` covering:

#### Scenario: Three-subnet VLSM calculation

- GIVEN base IP `192.168.0.0/24` with subnets requiring [100, 50, 25] hosts
- WHEN `calcular_vlsm()` is called
- THEN the result SHALL contain 3 subnets
- AND each SHALL have correct network_address, mask, and host range
- AND no subnet SHALL overlap another

#### Scenario: Subnet exceeds available space

- GIVEN base IP `192.168.0.0/24` with a subnet requiring 300 hosts
- WHEN `calcular_vlsm()` is called
- THEN the result SHALL be a string error message in Spanish

#### Scenario: Single subnet

- GIVEN base IP `10.0.0.0/24` with one subnet requiring 10 hosts
- WHEN `calcular_vlsm()` is called
- THEN the result SHALL contain exactly 1 subnet

### Requirement: Link Calculation Tests

The test suite SHALL include tests for `calcular_enlaces_router()`:

#### Scenario: 5 router links

- GIVEN base IP `172.16.0.0/16` and 5 links
- WHEN `calcular_enlaces_router()` is called
- THEN the result SHALL contain 5 links
- AND each SHALL be a /30
- AND each SHALL not overlap

### Requirement: Validator Tests

The test suite SHALL include tests for `validar_ip()` and `validar_prefijo()`:

#### Scenario: Valid IPv4 addresses

- GIVEN IPs `192.168.1.1`, `10.0.0.1`, `172.16.0.1`
- WHEN `validar_ip()` is called on each
- THEN each SHALL return `True`

#### Scenario: Invalid IPv4 addresses

- GIVEN IPs `999.999.999.999`, `abc`, `192.168.1.256`, `192.168.1`
- WHEN `validar_ip()` is called on each
- THEN each SHALL return `False`

#### Scenario: Valid prefixes

- GIVEN prefixes `1`, `15`, `30`
- WHEN `validar_prefijo()` is called on each
- THEN each SHALL return `True`

#### Scenario: Invalid prefixes

- GIVEN prefixes `0`, `31`, `33`
- WHEN `validar_prefijo()` is called on each
- THEN each SHALL return `False`

### Requirement: Spanish UI Smoke Test

The test suite SHALL verify that Spanish string output is preserved exactly.

#### Scenario: Spanish output preserved

- GIVEN `imprimir_resultados()` is called with valid VLSM output
- WHEN output is captured
- THEN the output SHALL contain `"Resultados de la Calculadora de Subneteo VLSM"`
- AND `"Dirección de red"`, `"Máscara de red"`, `"Broadcast"`, `"Rango de hosts"` exactly