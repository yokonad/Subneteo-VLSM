# Tasks: Subneteo-VLSM Modernization

## Review Workload Forecast

| Field | Value |
|-------|-------|
| Estimated changed lines | ~679 (453 additions + 226 deletions) |
| 400-line budget risk | High |
| Chained PRs recommended | Yes |
| Suggested split | PR 1 (Foundation, ~278 lines) → PR 2 (CLI + Cleanup, ~401 lines) |
| Delivery strategy | ask-on-risk |
| Chain strategy | pending |

Decision needed before apply: Yes
Chained PRs recommended: Yes
Chain strategy: pending
400-line budget risk: High

### Suggested Work Units

| Unit | Goal | Likely PR | Notes |
|------|------|-----------|-------|
| 1 | Package skeleton + models + core logic + core tests | PR 1 | base=main; ~278 additions, 0 deletions; independently testable |
| 2 | CLI layer + CLI tests + delete old main.py | PR 2 | base=PR 1 merge; ~175 additions + 226 deletions; completes migration |

## Phase 1: Foundation — Package Structure & Models (PR 1)

- [ ] 1.1 Create `pyproject.toml` at repo root with `[build-system]` (setuptools), `[project]` (name, version, requires-python>=3.7), `[project.scripts]` (subneteo = "subneteo.cli:main"), `[project.optional-dependencies]` (dev = ["pytest", "pytest-cov"])
- [ ] 1.2 Create `src/subneteo/__init__.py` — re-export `SubnetResult` and `LinkResult` from `.models`
- [ ] 1.3 Create `src/subneteo/models.py` with `@dataclass SubnetResult` (9 typed fields: subred, direccion_red, mascara_red, wildcard_mask, prefijo, rango_hosts, broadcast, hosts_necesarios: int, hosts_disponibles: int) and `@dataclass LinkResult` (7 typed fields: enlace, direccion_red, mascara_red, wildcard_mask, prefijo, ips_utilizables, broadcast)
- [ ] 1.4 Create `src/subneteo/__main__.py` with `from subneteo.cli import main; main()`

## Phase 2: Core Logic — Calculations & Validation (PR 1)

- [ ] 2.1 Create `src/subneteo/core.py` with `validar_ip(ip: str) -> bool` — wrap `IPv4Address()` in try/except, return bool
- [ ] 2.2 Add `validar_prefijo(prefijo: int) -> bool` — return `1 <= prefijo <= 30`
- [ ] 2.3 Add `calcular_prefijo_desde_hosts(num_hosts: int) -> int` — `32 - math.ceil(math.log2(num_hosts + 2))`
- [ ] 2.4 Add `calcular_wildcard(mascara: Any) -> str` — XOR with 0xFFFFFFFF, return str
- [ ] 2.5 Add `calcular_vlsm(ip_base: str, prefijo_principal: int, hosts_subredes: List[int]) -> Union[List[SubnetResult], str]` — migrate from main.py, replace double `list(subred.hosts())` with O(1) `str(subred.network_address + 1) - str(subred.broadcast_address - 1)`, handle /31 and /32 edge cases, return `List[SubnetResult]` instead of dicts
- [ ] 2.6 Add `calcular_enlaces_router(ip_base: str, prefijo_principal: int, num_enlaces: int) -> Union[List[LinkResult], str]` — migrate from main.py, single `list(subred.hosts())` capture for /30, return `List[LinkResult]`

## Phase 3: Core Tests (PR 1)

- [ ] 3.1 Create `tests/conftest.py` with fixtures: `sample_ip` ("192.168.0.0"), `sample_prefix` (24), `vlsm_hosts` ([100, 50, 25])
- [ ] 3.2 Create `tests/test_core.py` — parametrized test for `validar_ip`: valid IPs (192.168.1.1, 10.0.0.1, 172.16.0.1) return True; invalid (999.999.999.999, abc, 192.168.1.256, 192.168.1) return False
- [ ] 3.3 Add test for `validar_prefijo`: valid (1, 15, 30) → True; invalid (0, 31, 33) → False
- [ ] 3.4 Add test for `calcular_prefijo_desde_hosts`: 100→/25, 50→/26, 25→/27
- [ ] 3.5 Add test for `calcular_vlsm` with 3 subnets (192.168.0.0/24, [100,50,25]): assert 3 SubnetResults, correct addresses, no overlap
- [ ] 3.6 Add test for `calcular_vlsm` overflow: 192.168.0.0/24 with 300 hosts → error string
- [ ] 3.7 Add test for `calcular_vlsm` single subnet: 10.0.0.0/24 with [10] → 1 SubnetResult
- [ ] 3.8 Add test for `calcular_enlaces_router`: 172.16.0.0/16 with 5 links → 5 LinkResults, all /30, sequential, no overlap
- [ ] 3.9 Add edge-case tests for /31 and /32 host range computation — no crash, correct output

## Phase 4: CLI Layer (PR 2)

- [ ] 4.1 Create `src/subneteo/cli.py` with `imprimir_resultados(resultado: Union[List[SubnetResult], str]) -> None` — iterate SubnetResult fields, print with Spanish labels verbatim from original
- [ ] 4.2 Add `imprimir_enlaces(resultado: Union[List[LinkResult], str]) -> None` — iterate LinkResult fields, print with Spanish labels
- [ ] 4.3 Add `menu_vlsm() -> None` — input loops for IP, prefix, subnet count, host counts; call `calcular_vlsm()` then `imprimir_resultados()`; preserve all Spanish prompts exactly
- [ ] 4.4 Add `menu_enlaces() -> None` — input loops for IP, prefix, link count; call `calcular_enlaces_router()` then `imprimir_enlaces()`
- [ ] 4.5 Add `main() -> None` — top-level menu with "CALCULADORA DE REDES", options 1/2/3, dispatch to menu_vlsm/menu_enlaces

## Phase 5: CLI Tests & Cleanup (PR 2)

- [ ] 5.1 Create `tests/test_cli.py` — smoke test: call `imprimir_resultados()` with sample SubnetResult list, assert capsys output contains "Resultados de la Calculadora de Subneteo VLSM", "Dirección de red", "Máscara de red", "Broadcast", "Rango de hosts"
- [ ] 5.2 Add smoke test: call `imprimir_enlaces()` with sample LinkResult list, assert capsys output contains "Resultados de la Calculadora de Enlaces de Router"
- [ ] 5.3 Add test: `imprimir_resultados()` with error string prints the string directly
- [ ] 5.4 Delete `main.py` — all logic migrated to src/subneteo/ modules
- [ ] 5.5 Run `pytest --cov=src/subneteo --cov-report=term-missing` and verify ≥80% coverage
- [ ] 5.6 Run `python -m build` and verify wheel is produced in dist/
