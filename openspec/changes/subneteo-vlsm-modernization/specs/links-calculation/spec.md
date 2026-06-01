# Delta for Links Calculation

## MODIFIED Requirements

### Requirement: Link Usable IPs Calculation

The system SHALL compute usable IPs for a /30 link using `list(subred.hosts())[0]` and `list(subred.hosts())[1]` as a single-pass access, not double-enumerating.

(Previously: Two separate `list(subred.hosts())` calls each materialized the entire /30 generator)

#### Scenario: Single link usable IPs

- GIVEN a /30 link subnet `192.168.0.0/30`
- WHEN usable IPs are computed
- THEN the output SHALL be `"192.168.0.1 - 192.168.0.2"`

#### Scenario: Multiple sequential links

- GIVEN base IP `10.0.0.0` and 3 links
- WHEN each link's usable IPs are computed
- THEN each link SHALL be assigned sequentially without overlap
- AND no link SHALL double-materialize hosts via `list(subred.hosts())`

## ADDED Requirements

### Requirement: Type Hints on Link Functions

The function `calcular_enlaces_router()` SHALL return `list[dict[str, Any]] | str` with full type annotations.

### Requirement: IP Validation

The function `validar_ip()` SHALL accept `(ip: str) -> bool`.

### Requirement: Prefix Validation

The function `validar_prefijo()` SHALL accept `(prefijo: int) -> bool` and validate 1 ≤ prefijo ≤ 30.