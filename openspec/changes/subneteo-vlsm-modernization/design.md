# Design: Subneteo-VLSM Modernization

## Technical Approach

Refactor single-file CLI into three-module src-layout package. Extract calculation functions to `core.py` (pure, no I/O), menus and printing to `cli.py`, and dict outputs to typed dataclasses in `models.py`. Fix double `list(subred.hosts())` by using address arithmetic (`network_address + 1`, `broadcast_address - 1`) instead of materializing host generators. Preserve all Spanish UI strings verbatim. No algorithm changes.

## Architecture Decisions

| Decision | Choice | Alternatives | Rationale |
|----------|--------|-------------|-----------|
| Host range computation | `str(subred.network_address + 1) - str(subred.broadcast_address - 1)` arithmetic | Iterator unpacking, `list()` capture-once | O(1) arithmetic avoids materializing up to 65,534 hosts on /16. `/31` edge case: both addresses are valid in point-to-point; `/32`: return `"N/A"`. |
| Link IPs (/30) | `hosts = list(subred.hosts())` once, index `[0]` and `[1]` | Same arithmetic as above | Only 2 hosts for /30 — negligible. But spec requires "no double-materialize." Single capture satisfies that while keeping code straightforward. |
| Module split | `core.py` (calc), `cli.py` (I/O), `models.py` (dataclasses) | Keep single file; split by feature | Separation enables independent testing of core logic without mocking stdio. Matches spec requirements precisely. |
| Type syntax | `from typing import List, Dict, Any, Union, Optional` | `X \| Y` union (3.10+), `from __future__ import annotations` + built-in generics (3.9+) | `openspec/config.yaml` specifies Python 3.7+. `typing` module covers all needs without runtime cost. |
| Test structure | `tests/conftest.py` with fixtures + `tests/test_core.py`, `tests/test_cli.py` | Single `tests/test_subneteo.py` | Per-module test files match module boundaries. `conftest.py` fixtures (valid IPs, sample subnets) avoid duplication. |
| Package layout | `src/subneteo/` with `pyproject.toml` at root | Flat layout (`subneteo/` at root) | `src/` layout prevents accidental imports of unpackaged code; standard for modern Python projects. |

## Data Flow

```
CLI (cli.py)                    Core (core.py)                  Models (models.py)
─────────────                   ────────────────                ──────────────────
menu_vlsm()
  ├─ input: ip, prefix, hosts ──→ calcular_vlsm()
  │                                 ├─ sorted(hosts, reverse)
  │                                 ├─ for each: calcular_prefijo_desde_hosts()
  │                                 ├─ ipaddress.ip_network()
  │                                 ├─ calcular_wildcard()
  │                                 └─ return list[SubnetResult]    SubnetResult
  │                                                                 (dataclass)
  └─ imprimir_resultados(resultado) ←───────────────────────────
       └─ iterates SubnetResult fields, prints Spanish labels
```

`cli.py` never touches `ipaddress` directly — all network math stays in `core.py`. `models.py` has zero imports beyond `dataclasses`.

## File Changes

| File | Action | Description |
|------|--------|-------------|
| `src/subneteo/__init__.py` | Create | Package init. Re-exports `SubnetResult`, `LinkResult` for convenience. |
| `src/subneteo/models.py` | Create | `@dataclass SubnetResult` (9 fields), `@dataclass LinkResult` (7 fields). |
| `src/subneteo/core.py` | Create | `validar_ip`, `validar_prefijo`, `calcular_prefijo_desde_hosts`, `calcular_wildcard`, `calcular_vlsm`, `calcular_enlaces_router`. All typed, no print/input. |
| `src/subneteo/cli.py` | Create | `menu_vlsm`, `menu_enlaces`, `imprimir_resultados`, `imprimir_enlaces`, `main`. Import from `.core` and `.models`. |
| `src/subneteo/__main__.py` | Create | `from subneteo.cli import main; main()` |
| `main.py` | Delete | All logic migrated; entry point is now `python -m subneteo` or console_script. |
| `pyproject.toml` | Create | `[build-system]`, `[project]`, `[project.scripts]`, `[project.optional-dependencies]` |
| `tests/conftest.py` | Create | Fixtures: `sample_ip`, `sample_prefix`, `vlsm_expected`, `links_expected` |
| `tests/test_core.py` | Create | Parametrized tests for all 6 core functions |
| `tests/test_cli.py` | Create | Smoke tests for Spanish output, menu flow (capsys) |

## Interfaces / Contracts

```python
# models.py
from dataclasses import dataclass

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

```python
# core.py — public signatures
from typing import List, Union, Any

def validar_ip(ip: str) -> bool: ...
def validar_prefijo(prefijo: int) -> bool: ...
def calcular_prefijo_desde_hosts(num_hosts: int) -> int: ...
def calcular_wildcard(mascara: Any) -> str: ...
def calcular_vlsm(ip_base: str, prefijo_principal: int, hosts_subredes: List[int]) -> Union[List[SubnetResult], str]: ...
def calcular_enlaces_router(ip_base: str, prefijo_principal: int, num_enlaces: int) -> Union[List[LinkResult], str]: ...
```

`calcular_wildcard` takes `Any` because `ipaddress.IPv4Address` accepts str or int — type narrowed internally.

## Testing Strategy

| Layer | What to Test | Approach |
|-------|-------------|----------|
| Unit — core | `calcular_vlsm` with 3 subnets (192.168.0.0/24), overflow case, single subnet | Parametrized pytest; assert SubnetResult fields + no overlap |
| Unit — core | `calcular_enlaces_router` with 5 links (172.16.0.0/16) | Assert 5 LinkResults, all /30, sequential |
| Unit — core | `validar_ip` (valid + invalid IPv4), `validar_prefijo` (1-30 ok, 0/31/33 fail) | Simple assert True/False |
| Unit — core | `calcular_prefijo_desde_hosts` (100→/25, 50→/26, 25→/27) | Assert integer results |
| Unit — core | `/31` and `/32` edge cases — no crash, correct host range | Direct `ipaddress.ip_network("/31")` and test host range computation |
| Integration — cli | Spanish output strings preserved exactly in `imprimir_resultados`, `imprimir_enlaces` | `capsys` capture, assert exact Spanish substrings |

## Migration / Rollout

No migration required. Delete `main.py`. `pip install -e .` for development; `pip install .` for production wheel. Rollback: restore `main.py` from git, reverse file deletions.

## Open Questions

- [ ] Should `pyproject.toml` set `requires-python = ">=3.7"` or `">=3.8"`? 3.7 is EOL but existing config says 3.7+. Recommend `>=3.9` (EOL Oct 2025) but keep `>=3.7` per spec.
