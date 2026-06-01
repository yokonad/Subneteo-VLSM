# Delta for VLSM Calculation

## MODIFIED Requirements

### Requirement: Host Range Calculation

The system SHALL compute host range for a subnet using `network_address` and `broadcast_address` directly, without double-enumerating hosts via `list(subred.hosts())`.

(Previously: `list(subred.hosts())[0]` and `list(subred.hosts())[-1]` materialized the full hosts generator twice)

#### Scenario: Standard subnet host range

- GIVEN a subnet with network_address `192.168.1.0` and broadcast_address `192.168.1.255`
- WHEN the host range is computed
- THEN the output SHALL be `"192.168.1.1 - 192.168.1.254"`

#### Scenario: /30 subnet host range

- GIVEN a /30 subnet with network_address `10.0.0.0` and broadcast_address `10.0.0.3`
- WHEN the host range is computed
- THEN the output SHALL be `"10.0.0.1 - 10.0.0.2"`

#### Scenario: /31 subnet (point-to-point link)

- GIVEN a /31 subnet
- WHEN the host range is computed
- THEN the system SHALL NOT raise an error; use network_address + 1 and broadcast_address - 1 directly

#### Scenario: /32 subnet (single host)

- GIVEN a /32 subnet
- WHEN the host range is computed
- THEN the system SHALL output `"0.0.0.0 - 0.0.0.0"` or equivalent indicating no usable hosts
- AND the system SHALL NOT enumerate hosts via `list(subred.hosts())`

#### Scenario: Large subnet performance

- GIVEN a /16 subnet with 65,534 possible hosts
- WHEN host range is computed
- THEN the operation SHALL NOT iterate all hosts
- AND SHALL use `network_address` and `broadcast_address` directly (O(1))

## ADDED Requirements

### Requirement: Type Hints on Calculation Functions

The function `calcular_vlsm()` SHALL accept `(ip_base: str, prefijo_principal: int, hosts_subredes: list[int]) -> list[dict[str, Any]] | str`.

### Requirement: Prefix Calculation from Host Count

The function `calcular_prefijo_desde_hosts()` SHALL accept `(num_hosts: int) -> int` and return `32 - math.ceil(math.log2(num_hosts + 2))`.

### Requirement: Wildcard Mask Calculation

The function `calcular_wildcard()` SHALL accept `(mascara: Any) -> str` and return the dotted-decimal wildcard mask string.

### Requirement: Link Calculation

The function `calcular_enlaces_router()` SHALL accept `(ip_base: str, prefijo_principal: int, num_enlaces: int) -> list[dict[str, Any]] | str`.