# Delta for Module Structure

## ADDED Requirements

### Requirement: core.py Contains Calculation Logic

`core.py` SHALL contain only pure calculation functions with no I/O, printing, or user interaction.

Functions REQUIRED in core.py:
- `validar_ip(ip: str) -> bool`
- `validar_prefijo(prefijo: int) -> bool`
- `calcular_prefijo_desde_hosts(num_hosts: int) -> int`
- `calcular_wildcard(mascara: Any) -> str`
- `calcular_vlsm(ip_base: str, prefijo_principal: int, hosts_subredes: list[int]) -> list[SubnetResult] | str`
- `calcular_enlaces_router(ip_base: str, prefijo_principal: int, num_enlaces: int) -> list[LinkResult] | str`

#### Scenario: core.py has no print/input statements

- GIVEN core.py is analyzed for string literals
- WHEN searched for `print(` or `input(`
- THEN no matches SHALL be found

### Requirement: cli.py Contains I/O Logic

`cli.py` SHALL contain all menu functions, input validation loops, and result printing.

Functions REQUIRED in cli.py:
- `imprimir_resultados(resultado: Any) -> None`
- `imprimir_enlaces(resultado: Any) -> None`
- `menu_vlsm() -> None`
- `menu_enlaces() -> None`
- `main() -> None`

#### Scenario: cli.py imports only core.py and stdlib

- GIVEN cli.py imports
- WHEN analyzed
- THEN the only non-stdlib import SHALL be from `.core` or `..core`

### Requirement: models.py Contains Data Structures

`models.py` SHALL contain only dataclass definitions and their type-related utilities.

Content REQUIRED:
- `SubnetResult` dataclass
- `LinkResult` dataclass

#### Scenario: models.py has no functions with side effects

- GIVEN models.py is analyzed
- WHEN any function definition is searched
- THEN no results SHALL be found (only class/dataclass definitions)

### Requirement: Package Entry Point

`src/subneteo/__main__.py` SHALL be the CLI entry point callable via `python -m subneteo`.

#### Scenario: Module executable

- GIVEN the package is installed
- WHEN `python -m subneteo` is executed
- THEN `main()` from cli.py SHALL be called

### Requirement: Spanish UI Strings Unchanged

All Spanish strings SHALL appear verbatim in cli.py. The following strings MUST be preserved:

| Key | Spanish String |
|-----|----------------|
| Menu title | `"CALCULADORA DE REDES"` |
| VLSM option | `"Calculadora de Subneteo VLSM"` |
| Links option | `"Calculadora de Enlaces de Router"` |
| Invalid IP | `"La IP ingresada no es válida"` |
| Invalid prefix | `"El prefijo debe estar entre 1 y 30"` |
| Subnet result header | `"Resultados de la Calculadora de Subneteo VLSM"` |
| Links result header | `"Resultados de la Calculadora de Enlaces de Router"` |

#### Scenario: Spanish strings match exactly

- GIVEN cli.py content
- WHEN compared to the original main.py Spanish strings
- THEN each string SHALL be byte-for-byte identical