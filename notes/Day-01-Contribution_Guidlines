# Isaac Lab Contribution Guidelines

**Page studied:** <https://isaac-sim.github.io/IsaacLab/main/source/refs/contributing.html>
**Read from:** the page's source file `docs/source/refs/contributing.rst` in the [`isaac-sim/IsaacLab`](https://github.com/isaac-sim/IsaacLab) repo (`main`, commit `841286f`, version 3.0.0). I also read `CONTRIBUTING.md`, `AGENTS.md`, `.github/PULL_REQUEST_TEMPLATE.md`, the issue templates and `.github/labeler.yml`.
**Rule of this note:** everything below is stated in those files. Things I looked for and did not find are listed under *Not stated*.

**What the project is:** Isaac Lab is a *community-maintained* robotics simulation framework. Its contribution page opens by saying contributions are welcome in the form of "bug reports, feature requests, design proposals and more".

---

## Contents
1. [What we have](#1-what-we-have)
2. [What we need to do](#2-what-we-need-to-do)
3. [How we do it](#3-how-we-do-it)
4. [The technical pieces involved](#4-the-technical-pieces-involved)
5. [Other ways to contribute](#5-other-ways-to-contribute)
6. [How to submit](#6-how-to-submit)
7. [On what basis contributions are accepted](#7-on-what-basis-contributions-are-accepted)
8. [More: style, testing, changelog, commits](#8-more-style-testing-changelog-commits)
9. [Not stated in the guide](#9-not-stated-in-the-guide)
10. [One-page checklist](#10-one-page-checklist)

---

## 1. What we have
What the project already provides to anyone who wants to contribute:

| Resource | What it is for |
|---|---|
| **GitHub repo** (`isaac-sim/IsaacLab`) | Code hosting. Contributions go in as pull requests. |
| **Issue tracker** | Bug reports and proposals. Three templates: `bug.md`, `proposal.md`, `question.md`. |
| **GitHub Discussions** | Questions, ideas and feature proposals. |
| **PR template** | A checklist every pull request must complete (see §6). |
| **`CONTRIBUTING.md`** | Short welcome text plus the Developer Certificate of Origin (DCO 1.1) text. |
| **Written guideline page** | The page studied here. It is also the "source of truth" for the repo's coding-style, PR and changelog agent skills. |
| **`uv`-based tooling** | `uv run isaaclab --format`, `--docs`, `--test`, `--new`, plus `tools/run_all_tests.py` and `tools/changelog/cli.py`. |
| **CI (GitHub Actions)** | Runs checks on PRs. Docker and GPU tests run **on demand** when someone comments `run-ci`. |
| **Auto-labeler** | Labels PRs from the branch name: a name containing `feature` gives `enhancement`, a name containing `fix` or `bug` gives `bug`. |
| **`CODEOWNERS`** | Maps file paths to the people who get asked to review changes there. |
| **Agent skills** (`skills/developer/`) | Repo-owned instructions for AI coding tools: coding style, PR workflow, changelog fragments, test audit. |
| **Docs system** | Sphinx with the Book Theme, written in reStructuredText, in `docs/`. |
| **Template generator** | `uv run isaaclab --new` creates a standalone (external) project. |

## 2. What we need to do
The minimum a contributor must do, per the guide:

1. **Discuss first.** Open an issue (or Discussion) before writing code. Reason given: avoids duplicate work and keeps changes aligned with the project roadmap.
2. **Work on a fork and a branch.** Never commit to `main`.
3. **Follow the style guide.** Google Python Style Guide, with the deviations listed in the guide's *Coding Style* section.
4. **Keep the PR to one change.**
5. **Test.** Add or extend tests only where they protect a distinct behavior; run the relevant ones.
6. **Format.** Run `uv run isaaclab --format` on the final changes.
7. **Document.** Update docstrings and docs when behavior changes. Run one clean, warning-free docs build if docs changed.
8. **Add a changelog fragment** for every touched package under `source/<pkg>/`. Do not edit `CHANGELOG.rst` or versions.
9. **Add your name** to `CONTRIBUTORS.md` (alphabetical), unless it is already there.
10. **Complete the PR template** and respond to review feedback.

## 3. How we do it
### 3.1 Code contribution, step by step
```text
1. Create an issue in the issue tracker (discuss the change)
2. Fork the repository
3. Create a new branch for the change
4. Make the change and commit it
5. Push the branch to your fork
6. Open a pull request to the `main` branch
7. Tick every item in the PR template checklist
8. Maintainers review and give feedback; you respond
```

### 3.2 Documentation contribution
- Sources live in `IsaacLab/docs`, written in reStructuredText.
- The flow is the same as code: issue, fork, branch, PR.
- Preview while editing (incremental, can hide old warnings):
  ```bash
  uv run --extra dev --directory docs python -m sphinx -W --keep-going -j auto . _build/incremental
  ```
- Final clean build before submitting (must be warning-free). Output goes to `docs/_build/current`:
  ```bash
  uv run --extra dev isaaclab --docs
  ```
- Skip Sphinx when rendered docs are unaffected (e.g. only `AGENTS.md` or `skills/` changed). Use a **clean** build for API signature or docstring changes, Sphinx config or extension changes, and theme changes.
- Media: only small `.jpg` files may be committed. Host larger images, videos and animations externally and link to them.

### 3.3 Asset or environment contribution
1. Start with a Discussion or a *proposal* issue (not a PR).
2. Include: the missing capability and which current examples you checked, the proposed location, supported physics backends and renderers, asset sources and licenses, evidence it works, who is responsible for it, and (for `contrib`) an exit plan.
3. Pick the location (see §5).

### 3.4 Changelog fragment, step by step
1. Find each package you changed under `source/<pkg>/`.
2. Create `source/<pkg>/changelog.d/<slug>.<type>.rst`. `<type>` is `added`, `changed`, `deprecated`, `removed` or `fixed`. The default slug is the branch name with `/` replaced by `-`.
3. Write bullets with `*`, in the **past tense**, short, with the reason where it applies. Prefix breaking changes with **Breaking:** and give migration guidance.
4. Add an empty `<slug>.minor` (new public API) or `<slug>.major` (breaking change) file to raise the version bump. Add an empty `<slug>.skip` for CI, docs or test-only PRs.
5. Check:
   ```bash
   uv run python tools/changelog/cli.py check upstream/main --include-worktree
   ```

Example, `source/isaaclab/changelog.d/fix-partial-reset.fixed.rst`:
```rst
* Fixed contact sensor reset behavior when only a subset of environments was reset.
```

## 4. The technical pieces involved

| Area | What the project uses |
|---|---|
| Language and environment | Python, `uv`-managed virtual environment, `pyproject.toml` |
| Style | Google Python Style Guide, PEP 8, PEP 484 and PEP 585 (type hints), Google-style docstrings |
| Formatting and linting | `pre-commit` and `ruff`, run through `uv run isaaclab --format` |
| Tests | `pytest`; markers `kitless` and `ov`; `tools/run_all_tests.py`; `uv run isaaclab --test` for tooling tests |
| Changelog | `towncrier` fragments; CI compiles `CHANGELOG.rst` nightly and bumps versions by Semantic Versioning |
| Docs | Sphinx, Book Theme, reStructuredText, an embedded demo gallery hosted separately |
| Simulation stack | Isaac Sim, Newton and PhysX backends, `ovphysx`, `ovrtx`, `ovstage`, Warp, Torch |
| CI | GitHub Actions; Docker and GPU tests via the `run-ci` comment |
| Config system | `configclass`, resolvable strings, lazy exports, `isaaclab.utils` helpers |
| Licensing | BSD-3-Clause; SPDX header on new source files; DCO text in `CONTRIBUTING.md` |

## 5. Other ways to contribute
The guide's opening list: bug reports, feature requests, design proposals, documentation, tutorials.

| Way | How |
|---|---|
| **Bug report** | Issue tracker, `bug.md` template. Include a minimal reproduction, error output, and Isaac Lab and Isaac Sim versions. |
| **Proposal / idea** | `proposal.md` issue (Proposal, Motivation, Alternatives, Build Info) or a Discussion. |
| **Question** | GitHub Discussions, or the `question.md` issue template. |
| **Documentation / tutorial** | PR to `docs/`, same flow as code. |
| **Code fix or feature** | PR, after an issue. |
| **Assets and environments** | Allowed only when they pass the capability gate (see §7), in one of three locations below. |

**Where an asset or environment can live:**

| Location | Purpose | Fits |
|---|---|---|
| **Isaac Lab core** | Stable reference implementations that are part of the framework contract. | Reusable APIs and the smallest canonical asset or environment that demonstrates them. |
| **In-repo `contrib` package** | Incubation for a new capability that may later graduate to core. | Capability-driven work with a named owner, an evaluation period, and a promotion-or-removal plan. |
| **Community-owned project** | The **default** place for downstream work. | Robot ports, task or reward variants, research reproductions, vendor examples, application workflows, trained policies, datasets, combinations of capabilities already demonstrated. |

Create a community-owned project with `uv run isaaclab --new`, then choose **External**: **Cartpole** (runnable baseline), **Stubbed** (task skeleton) or **Blank** (add Isaac Lab support to an existing project). That project owns its code, assets, docs, releases, support and versioned dependencies, and uses its own issue tracker. If it reveals a missing reusable primitive, that primitive can be proposed to Isaac Lab separately.

## 6. How to submit
1. Push your branch to your fork.
2. Open a pull request against `main` (`github.com/isaac-sim/IsaacLab/compare/main...`).
3. Fill in the PR template:
   - Description, motivation and context, dependencies, `Fixes #<issue>`.
   - **Type of change:** bug fix, new feature, breaking change, or documentation update.
   - Before and after screenshots, if relevant.
   - **Checklist:**
     - [ ] I have read the contribution guidelines
     - [ ] I ran the `pre-commit` checks with `uv run isaaclab --format`
     - [ ] I made corresponding changes to the documentation
     - [ ] My changes generate no new warnings
     - [ ] I validated the changed behavior with focused coverage and applied the test-audit criteria to any test changes
     - [ ] I added a changelog fragment under `source/<pkg>/changelog.d/` for every touched package (do **not** edit `CHANGELOG.rst` or bump versions)
     - [ ] I added my name to `CONTRIBUTORS.md` (or it is already there)
4. To run the Docker and GPU tests, push the commits you want tested, then comment **`run-ci`** on the PR.
5. Keep the PR small and focused. The template says large PRs are harder to review and merge.
6. After review comments, prefer follow-up commits over amending.

## 7. On what basis contributions are accepted
The guide does not publish a scoring rubric. These are the criteria it does state:

**For every pull request**
- Focused on **one change**; unrelated changes submitted separately.
- Aligned with the project roadmap (the stated reason for opening an issue first).
- Formatted, documented, and passing the required CI checks.
- Tests cover a distinct behavior; a regression test fails without the fix and passes with it.
- Changelog fragment present. Docs updated when behavior changes.
- Follows the coding-style rules in §8.

**For assets and environments (the "capability gate")**
- It must be the **smallest clear example** of a framework capability that current Isaac Lab examples do not cover.
- It must exercise a reusable framework path, not only a robot-specific reward, scene, policy or benchmark.
- It must reuse existing assets and task terms wherever possible.
- It must declare and validate the supported physics-backend and renderer combinations. Unsupported combinations are omitted or fail with a clear message.
- It must document the capability and name a support owner.
- Assets need clear provenance, redistribution-compatible licensing, immutable versioned hosting, reproducible conversion or authoring instructions, and an upstream owner. Large binaries stay externally hosted.
- Differences in brand, appearance, dimensions, degree-of-freedom count or commercial popularity alone are **not** enough. Adapting an existing reach, lift, locomotion or manipulation task to another robot is "usually a better fit for a community-owned project".
- Maintainers may **consolidate, relocate or remove** an in-repo example when its capability is covered elsewhere or its ownership and dependencies are no longer sustainable.

## 8. More: style, testing, changelog, commits

### 8.1 Coding style (highlights)
- Follow the Google style guide. Public symbols have Google-style docstrings with units, shapes and frames; types stay in the signature, not the docstring.
- Type hints: `list[str]`, `X | None`, `-> None` when nothing is returned.
- `snake_case` for functions, methods and CLI arguments. Private things start with `_`.
- File order: imports, constants, public functions, public classes, private functions, private classes.
- Class member order: constants, `ClassVar`s, dunder methods, `__repr__`/`__str__`, properties, public methods, then private methods, each ordered by how a user would call them.
- Make the smallest change that solves the problem. Reuse existing mechanisms. No new dependencies for hypothetical future use. Public API removals need a deprecation and migration path.
- **Warnings and logging:** `warnings.warn(..., Category, stacklevel=...)` when the caller must change their code; `logger.warning` for runtime events; `logger.info` for status; **never `print`** for warnings or status, and no `[WARNING]`-style prefixes.
- **Config pattern:** keep config classes declarative. Store implementation classes as resolvable strings (`"{DIR}.sensor:Sensor"`), import for typing only under `TYPE_CHECKING`, keep config and implementation in separate files, and use `lazy_export()` with a `.pyi` stub in `__init__.py`.
- **Performance (Warp / Torch):** avoid allocations, copies, device syncs and Python loops over environments in per-step code. Use native `wp.copy`, `fill_`, `zero_`. Pass `ProxyArray` straight to kernels. Avoid atomics where a thread owns its output cell.

### 8.2 Testing
```bash
uv run python -m pytest source/isaaclab/test/utils/test_circular_buffer.py::test_reset   # one test
uv run python -m pytest source/isaaclab/test/utils/test_circular_buffer.py               # one file
uv run python tools/run_all_tests.py                                                      # source packages
uv run isaaclab --test                                                                    # tooling tests
```
- Add a test only for a distinct behavior, regression, boundary or failure mode that existing coverage misses.
- Mark tests `kitless` (passes without Isaac Sim) or `ov` (runs or imports `ovphysx`, `ovrtx`, `ovstage`).
- Test observable behavior. No assertion-free smoke tests, no self-comparisons.
- If an optional dependency is missing: `uv run --extra <extra> python -m pytest ...`.

### 8.3 Commits and branches (from `AGENTS.md`)
- Feature branch, never `main`. Focused, atomic commits.
- Subject is imperative and capitalized, with no trailing period.
- Inspect staged changes before committing. Do not commit scratch files or generated plans.
- **Do not add AI co-author or attribution lines.**
- New source files use the repo's SPDX license header. Do not change existing headers.

### 8.4 Working with AI coding tools (the guide's *Agent Development* section)
Read only the sections that apply, find the behavior's owner and the smallest fix, use bounded searches, use a separate git worktree and `uv` environment when the checkout has unrelated changes, run narrow checks first and reuse results until relevant code changes, and keep long-running CI run IDs and logs.

## 9. Not stated in the guide
These are things I searched for and did not find:
- A numeric acceptance bar (no coverage percentage, no review-time promise).
- A "good first issue" process.
- A required `Signed-off-by` line. The DCO text is in `CONTRIBUTING.md`, but the guide does not say commits must be signed off.
- Branch-name format beyond the labeler's `feature` / `fix` / `bug` keywords.

When in doubt about any of these, ask in the issue before starting.

## 10. One-page checklist
- [ ] Issue or Discussion opened and agreed first
- [ ] Fork, then a feature branch (`fix-…` or `feature-…`), never `main`
- [ ] One focused change
- [ ] Google style, type hints, docstrings with units and shapes
- [ ] No `print`; correct warning or logging mechanism
- [ ] Test that fails without the fix and passes with it
- [ ] Changelog fragment in `source/<pkg>/changelog.d/`
- [ ] `uv run isaaclab --format` clean
- [ ] Clean docs build if docs changed
- [ ] Name in `CONTRIBUTORS.md`
- [ ] PR template fully filled in, then comment `run-ci`
- [ ] Respond to review with follow-up commits
