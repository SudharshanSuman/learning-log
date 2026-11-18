# Day 01 (part 2) · Isaac Lab contribution guide: coding-style deep dive

Continues [day-01.md](day-01.md). Source: `docs/source/refs/contributing.rst` in `isaac-sim/IsaacLab` (`main`, commit `841286f`). The live page is [contributing.html](https://isaac-sim.github.io/IsaacLab/main/source/refs/contributing.html).

## Page map (section → where my notes are)

| Section of the page | Notes |
|---|---|
| Contributing Code, Contributing Documentation | [day-01.md](day-01.md) |
| Agent Development | below, §9 |
| Contributing assets and environments | [day-01.md](day-01.md) §5–6 |
| Maintaining package changelogs and versions | [day-01.md](day-01.md) |
| Coding Style (everything below) | this file |
| Unit Testing, Tools | [day-01.md](day-01.md) |

---

## 1. Refactoring and API design
- Smallest change that solves the problem. Read callers, tests and docs before changing an interface.
- Preserve observable behavior: ordering, shapes, dtype, device, input mutation, failure behavior. Public API removals and renames need a **deprecation and migration path**.
- Plain functions for stateless work. Classes only when they own state, enforce a lifecycle, or implement a required interface. No classes that only group static methods or forward calls.
- Reuse the module that already owns the contract before adding a new file. Keep helpers private unless callers need them. Do not extract a one-line helper just to rename something.
- No raise-and-catch for routine capability checks. Use a predicate or an optional result.
- A missing optional dependency raises Python's own `ModuleNotFoundError`. Do not wrap it to add install hints.
- Direct attribute access. `getattr` / `setattr` only when the name is dynamic. Do not use defaults to hide a missing required attribute.
- Do not use `None` and `[]` to mean the same thing. Empty selection has one representation.
- One owner for each piece of state and validation. Pass dependencies from the composition root. Do not fetch the simulation singleton for something the caller already owns.
- Keep physics and rendering separate. Select backends at shared dispatch boundaries, not by class-name strings.
- No new dependencies or compatibility layers for hypothetical future use.

## 2. Array operations and runtime cost (matters for my Warp work)
- Review allocations, copies, syncs and recomputation in code that runs per step, reset or environment.
- Preserve slices. Materialize indices only where a consumer needs them.
- Allocate directly with the right value, dtype and device (`torch.full` / `wp.full`).
- Keep Warp-owned arrays in Warp. No Torch round trips just to mutate a buffer. Use `ProxyArray` when consumers need several array interfaces.
- Batch operations. No Python loops over environments in hot paths, and no unnecessary host/device transfers.
- Allocate expensive optional buffers lazily, but before CUDA graph capture if the allocator needs it.

## 3. Naming and documentation
- `snake_case` for functions, methods and CLI arguments. Keep related public names under consistent prefixes.
- Google-style docstrings. Types live in the signature, not the docstring. State units inline, e.g. `Particle positions [m], shape [N, 3]`. Document frames and shapes.
- Comments are brief and explain intent or non-obvious constraints, not what the code already says.
- Comments describe the **current** contract. Migration notes go in the changelog or migration docs, not in code history.

## 4. File and class structure
**File order:** imports, constants, public functions, public classes, `_private` functions, `_private` classes.

**Imports:** sorted by Ruff (`uv run isaaclab -f`). Groups: `__future__`, stdlib, third-party, Omniverse, Isaac Lab, local relative. Relative imports up to 3 dots (`from ...utils import math as math_utils`). Absolute imports for deeper targets, other packages and scripts. Prefer module-level imports. Local imports are fine for optional backends.

**Class member order:** constants, `ClassVar` class variables, dunder methods (`__init__`, `__del__`), `__repr__`/`__str__`, properties, public instance / class / static methods, then the private versions. Within each group, order by how a user calls them (`initialize`, `reset`, `update`, `close`). Resource-owning classes expose `close()` or a context manager.

**Private** means a leading underscore, by convention.

## 5. Warp rules
- Pass `ProxyArray` straight to Warp kernels. One proxy per owned array. No parallel `_ta` / `_warp` / `_torch` attributes.
- When the operation is known before launch, use dedicated kernels or static Warp branches, not a runtime mode switch. Benchmark the generated kernels against the original path.
- If one thread exclusively owns an output cell, write read-add-write explicitly. Warp `+=` may become an atomic add. Keep atomics for real concurrent reductions.
- **Shared math** lives in `isaaclab.utils.warp.math`. Quaternions are `xyzw`. Spatial vectors are `(linear, angular)`. Use Warp built-ins first. Use `wp.map(..., return_kernel=True)` for elementwise launches.
- **Shared memory ops** live in `isaaclab.utils.warp.memory`. Use native `wp.copy`, `fill_`, `zero_`. Use `wp.indexedarray` for selections. Do not write kernels for things Warp already provides. Indexed views need contiguous int32 indices (`resolve_1d_indices`). Do not compact dynamic boolean masks to indices inside captured paths.
- **Custom kernel factories:** name `make_<operation>`, cache with `functools.cache`, define with `@wp.kernel(module="unique")`, and return a concrete `wp.Kernel`, following the MuJoCo Warp factory pattern. Warm up before graph capture. Check cold compilation and warm execution separately.
- No debug output in production Warp kernels (from `AGENTS.md`). Do not define Warp kernels in `python -c`. Write a temporary file so Warp can inspect the source.

## 6. Sensors
- Shared concepts have one owner, but backends can differ in capability. Validate requested features **before** allocating resources.
- Never substitute zero measurements for unsupported features. Make absence explicit.
- Use the existing sensor factory and initialization root. Do not add another capability registry.

## 7. Config pattern (the part that is unique to Isaac Lab)
**Problem:** config objects and their implementation classes live in separate files, and configs name the class as a default (`class_type`). That creates circular imports and pulls heavy modules in too early. Config classes must be constructable **before** `SimulationApp` launches.

**Four tools to know:**
1. **Config + implementation file split:** `sensor_cfg.py` holds pure data, `sensor.py` holds the runtime code (may import `omni`, `pxr`).
2. **Resolvable strings:** `class_type: type[Sensor] | str = "{DIR}.sensor:Sensor"`. `{DIR}` expands to the package of the class that defines the field. Do not write `from .sensor import Sensor; class_type = Sensor`.
3. **`TYPE_CHECKING` guard:** import the class only for the type annotation, so IDEs can autocomplete without a runtime import.
4. **Lazy exports:** `__init__.py` contains only `lazy_export()`, and an adjacent `__init__.pyi` stub lists the public names (`__all__` plus relative imports). Follows SPEC 1 and the `lazy_loader` library. Avoid `from .x import *` in stubs, because it loads eagerly. Declare `"*" = ["*.pyi"]` under `[tool.setuptools.package-data]` so stubs ship.

**Config operations:** keep config classes declarative. Merging defaults and runtime wiring belong in launch or construction code, not config methods. Use `isaaclab.utils`: `instantiate`, `clone`, `replace`, `update_from_dict`, `validate`, `to_dict`. Older `cfg.copy()` / `cfg.replace()` / `cfg.validate()` keep working. `instantiate` passes the config as the first constructor argument and does not copy or cache.

## 8. Warnings and logging
- Caller must change code or config: `warnings.warn(msg, Category, stacklevel=...)`. Categories: `DeprecationWarning` (Python API), `FutureWarning` (config values and CLI options), `UserWarning` (misuse, ignored settings).
- Runtime event the caller cannot avoid (fallback, missing optional feature, degraded speed): `logger.warning`, with a module-level `logger = logging.getLogger(__name__)`.
- Status and progress: `logger.info`. Failure before exiting a CLI tool: `logger.error`.
- No `print` for warnings or status, and no `[WARNING]` prefixes.
- In tests: `pytest.warns` for `warnings.warn`, `caplog` for `logger.warning`.

## 9. Agent development (the section for AI coding tools)
- Read only the contribution sections that apply. Find the behavior's owner, the closest reusable code, and the smallest fix.
- Use bounded searches, not whole-file dumps.
- Use a separate git worktree (and its own `uv` environment) when the checkout has unrelated changes. Check the repo root with `uv run python -c "import sys; from isaaclab.paths import ISAACLAB_ROOT; print(sys.executable); print(ISAACLAB_ROOT)"`.
- Run the narrowest check first, then the required final checks. Reuse results until relevant code changes. Keep environment failures separate from regressions.
- Launch long checks once, keep run IDs and logs, and poll sparingly.

## 10. Checklist I will use before any PR to Isaac Lab
- [ ] Issue or Discussion opened first, and a maintainer agrees it is wanted
- [ ] Feature branch (`fix-…` or `feature-…`), never `main`
- [ ] One focused change
- [ ] Google-style docstrings with units and shapes
- [ ] Type hints (`list[str]`, `X | None`)
- [ ] No `print`, correct warning or logging mechanism
- [ ] Config uses a resolvable string and a `TYPE_CHECKING` guard, if I touched a config
- [ ] Test that fails without the fix and passes with it, marked `kitless` / `ov` as needed
- [ ] Changelog fragment in `source/<pkg>/changelog.d/`
- [ ] `uv run isaaclab --format` clean
- [ ] Docs build warning-free if docs changed
- [ ] Name added to `CONTRIBUTORS.md`
- [ ] Commit subject imperative and capitalized, no AI attribution line, then comment `run-ci`
