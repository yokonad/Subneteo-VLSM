# Proposal: Subneteo-VLSM Modernization

## Intent

Refactor the Subneteo-VLSM Python CLI calculator to eliminate performance inefficiencies, add type safety, establish test coverage, and modernize the project structure. The core VLSM calculation logic is correct but the code suffers from double `list()` materialization on large subnets, mixed concerns, no tests, and missing packaging. No functional behavior changes.

## Scope

### In Scope
- Fix double `list(subred.hosts())` on line 46 — materialize once, use start/end directly
- Fix double `list(subred.hosts())` on line 92 — same pattern, less severe for /30
- Add Python type hints throughout
- Create `pytest` test suite with ≥80% coverage
- Modularize: `core.py` (calculation logic), `cli.py` (I/O, menus), `models.py` (data structures)
- Create `pyproject.toml` packaging
- Preserve Spanish UI strings and output format

### Out of Scope
- Adding IPv6 support
- GUI or web interface
- Config file support
- VLSM algorithm changes (logic is correct)

## Capabilities

### New Capabilities
None — pure refactor, no new functionality.

### Modified Capabilities
None — existing spec behavior preserved, only implementation quality changes.

## Approach

1. **Extract models** — create `models.py` with typed `SubnetResult`, `LinkResult` dataclasses replacing dict outputs
2. **Extract core** — move `calcular_vlsm()`, `calcular_enlaces_router()`, `calcular_wildcard()`, `calcular_prefijo_desde_hosts()` to `core.py` with full type hints
3. **Extract cli** — move menu functions, input validation, result printing to `cli.py`
4. **Fix double materialization** — use `subred.network_address` and `subred.broadcast_address` directly; for host range, use `list(subred.hosts())[0]` as single access point, not double enumerate
5. **Add pyproject.toml** — `build-system` with setuptools, `project` metadata, `pytest` dev dependency
6. **Write tests** — parametrized tests for `calcular_vlsm()`, `calcular_enlaces_router()`, validators; mock-free unit tests

## Affected Areas

| Area | Impact | Description |
|------|--------|-------------|
| `main.py` | Modified/Removed | Becomes entry point; all logic moves out |
| `src/subneteo/models.py` | New | Dataclasses for `SubnetResult`, `LinkResult` |
| `src/subneteo/core.py` | New | All calculation functions with type hints |
| `src/subneteo/cli.py` | New | I/O, menus, printing functions |
| `src/subneteo/__init__.py` | New | Package init |
| `pyproject.toml` | New | Build and packaging config |
| `tests/` | New | pytest test suite |
| `main.py` → `src/subneteo/__main__.py` | Moved | CLI entry point refactored |

## Risks

| Risk | Likelihood | Mitigation |
|------|------------|------------|
| Spanish UI strings broken during refactor | Low | Preserve exact strings; add smoke test |
| `list(subred.hosts())` edge cases on small prefixes | Low | Test /30, /31, /32 explicitly |
| pyproject.toml misconfigured | Low | Use minimal config, verify with `python -m build` |

## Rollback Plan

1. Delete `src/subneteo/`, `tests/`, `pyproject.toml`
2. Restore original `main.py` from git
3. No migration scripts needed — all state is ephemeral (CLI tool)

## Dependencies

- Python 3.7+ (ipaddress, math are stdlib)
- pytest (dev) for testing

## Success Criteria

- [ ] `python -m pytest --cov=src/subneteo` passes with ≥80% coverage
- [ ] `python -m build` produces a valid wheel from `pyproject.toml`
- [ ] VLSM and link calculations produce identical output to original for test vectors
- [ ] Spanish strings preserved exactly (smoke test)
- [ ] No `list(subred.hosts())` double materialization in new code
- [ ] All functions have PEP 484 type hints