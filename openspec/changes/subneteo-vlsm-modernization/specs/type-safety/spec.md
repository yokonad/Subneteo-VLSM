# Delta for Type Safety

## ADDED Requirements

### Requirement: All Functions Typed

Every function in `core.py`, `cli.py`, and `models.py` SHALL have PEP 484 type hints for all parameters and return values.

#### Scenario: core.py function signatures

- GIVEN `calcular_vlsm`, `calcular_enlaces_router`, `calcular_wildcard`, `calcular_prefijo_desde_hosts` in core.py
- WHEN the file is parsed by mypy with strict mode
- THEN no function SHALL have `Any` types except in genuinely untyped positions

#### Scenario: cli.py function signatures

- GIVEN `menu_vlsm`, `menu_enlaces`, `imprimir_resultados`, `imprimir_enlaces` in cli.py
- WHEN the file is parsed by mypy
- THEN all parameter and return types SHALL be annotated

#### Scenario: models.py dataclass signatures

- GIVEN `SubnetResult` and `LinkResult` dataclasses
- WHEN their fields are accessed
- THEN all fields SHALL have explicit type annotations

### Requirement: Dataclass Models

The system SHALL use dataclasses for result structures:

```python
@dataclass
class SubnetResult:
    subred: str
    direccion_red: str
    mascara_red: str
    wildcard_mask: str
    prefijo: str
    rango_hosts: str
    broadcast: str
    hosts_necesarios: int
    hosts_disponibles: int

@dataclass
class LinkResult:
    enlace: str
    direccion_red: str
    mascara_red: str
    wildcard_mask: str
    prefijo: str
    ips_utilizables: str
    broadcast: str
```

#### Scenario: SubnetResult fields

- GIVEN a SubnetResult instance
- WHEN fields are accessed
- THEN each SHALL be the correct type (str for addresses, int for counts)

#### Scenario: LinkResult fields

- GIVEN a LinkResult instance
- WHEN fields are accessed
- THEN each SHALL be the correct type