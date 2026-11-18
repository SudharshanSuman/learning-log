# Day 01 · Isaac Lab contribution rules

**Part 2 (coding style and config patterns): [day-01-coding-style.md](day-01-coding-style.md)**

**Source:** [Isaac Lab contribution guide](https://isaac-sim.github.io/IsaacLab/main/source/refs/contributing.html), read from `docs/source/refs/contributing.rst` in the `isaac-sim/IsaacLab` repo (`main`, commit `841286f`, version 3.0.0), plus `CONTRIBUTING.md`, `AGENTS.md`, `.github/PULL_REQUEST_TEMPLATE.md` and the issue templates. The rules below are what those files state. Anything marked **Not stated** is something I looked for and did not find.

---

## 1. What we have

| Asset | Where it helps |
|---|---|
| [GPU-Accelerated Plant Biomechanics](https://github.com/SudharshanSuman/gpu-accelerated-plant-biomechanics): NVIDIA Warp solver for deformable stems, Isaac Sim bridge, 24 tests, validation notes | Shows real Warp and Isaac Sim experience. It is also the kind of deformable-body work the guide lists as a capability Isaac Lab may lack. |
| [Understanding the Prompt to Robot](https://github.com/SudharshanSuman/Understanding-the-Prompt-to-Robot): ROS 2, MoveIt 2, Gazebo | ROS 2 and manipulation background. It is Gazebo-based, not Isaac Lab. |
| This learning-log | Evidence trail for each step. |

**Gap (honest):** I have not run Isaac Lab yet and have never opened a pull request to a large project. Both come first.

## 2. What we need to do (in order)

1. Install Isaac Lab and run one example (needs `uv` and a GPU machine).
2. Read the code that matches my skills: Warp kernels, sensors, deformable assets.
3. Find or file one small, real issue.
4. Fork, branch, fix, test, add a changelog fragment, format, open the PR.
5. Respond to review.

## 3. The rules the guide actually states

### Code contributions (the 7 steps)
1. Create an issue in the issue tracker to discuss the change. Reason given: avoid duplicate work and stay aligned with the roadmap.
2. Fork the repository.
3. Create a new branch.
4. Make changes and commit.
5. Push to your fork.
6. Open a pull request against the `main` branch.
7. Complete every item on the pull request template.

Other stated rules:
- Keep each PR focused on **one change**. Submit unrelated changes separately.
- Format and document the code, and run the relevant tests and CI checks.
- Maintainers review and give feedback after the PR is sent.

### Formatter and linter
- Tools: `pre-commit` and `ruff`.
- Run: `uv run isaaclab --format`
- While editing, restrict to files: `uv run isaaclab --format path/to/file.py`
- Run the full command on the final changes before committing. If hooks modify files, inspect the edits and rerun.

### Tests
- Framework: `pytest`.
- Run one test: `uv run python -m pytest source/isaaclab/test/utils/test_circular_buffer.py::test_reset`
- Run a file: `uv run python -m pytest <file>`
- Run all source-package tests: `uv run python tools/run_all_tests.py`
- Tooling tests: `uv run isaaclab --test`
- Add a test only for a distinct behavior, regression, boundary or failure mode that existing coverage does not already exercise.
- A regression test must fail without the fix, for the intended reason, and pass with it.
- Mark tests `kitless` (passes without Isaac Sim) or `ov` (uses `ovphysx`, `ovrtx` or `ovstage`).
- Test observable behavior. Avoid private details, assertion-free smoke tests and self-comparisons.

### Changelog
- **Do not edit** `CHANGELOG.rst` or package versions. CI compiles them nightly from fragments.
- For every package touched under `source/<pkg>/`, add a fragment in `source/<pkg>/changelog.d/`:
  - `<slug>.<type>.rst` where type is `added`, `changed`, `deprecated`, `removed` or `fixed`.
  - `<slug>.minor` (new public API) or `<slug>.major` (breaking change), as an empty file.
  - `<slug>.skip`, an empty file, for CI/docs/test-only PRs.
- Slug: any short unique name. Your branch name with `/` replaced by `-` is recommended.
- Bullets start with `*`, are written in **past tense**, and are concise. Include the reason where it applies.
- Prefix breaking changes with **Breaking:** and give migration guidance.
- Example: `source/isaaclab/changelog.d/fix-partial-reset.fixed.rst` containing `* Fixed contact sensor reset behavior when only a subset of environments was reset.`
- Check: `uv run python tools/changelog/cli.py check upstream/main --include-worktree`

### Coding style
- Follow the Google Python Style Guide, PEP-8, and PEP-484/585 for type hints.
- Google-style docstrings. Do not repeat types in docstrings; state units, shapes and frames instead (e.g. `Particle positions [m], shape [N, 3]`).
- Type hints: built-in collections (`list[str]`), `X | None`, `-> None` for no return value.
- `snake_case` for functions, methods and CLI arguments.
- Use `warnings.warn(..., <Category>, stacklevel=...)` when the caller must change their code. Use `logger.warning` for runtime events. Use `logger.info` for status. **Never `print`** for warnings or status, and no `[WARNING]` prefixes.
- Smallest change that solves the problem. Reuse existing mechanisms. No new dependencies for hypothetical future use.
- Public API removals or renames need a deprecation and migration path.
- Warp: no atomics for exclusively owned outputs, use Warp built-ins (`wp.copy`, `fill_`), and avoid per-environment Python loops and device syncs in hot paths.

### Documentation
- Sources are in `docs/`, written in reStructuredText, built with Sphinx.
- Before submitting, run one clean, warning-free build: `uv run --extra dev isaaclab --docs`
- Quick preview while editing: `uv run --extra dev --directory docs python -m sphinx -W --keep-going -j auto . _build/incremental`
- Only small `.jpg` files may be committed. Host larger images and videos externally.

### PR template checklist (from `.github/PULL_REQUEST_TEMPLATE.md`)
- [ ] Read the contribution guidelines
- [ ] Ran `uv run isaaclab --format`
- [ ] Updated documentation
- [ ] No new warnings
- [ ] Validated the changed behavior with focused tests
- [ ] Added a changelog fragment for every touched package
- [ ] Added my name to `CONTRIBUTORS.md` (alphabetical)
- Type of change: bug fix, new feature, breaking change, or documentation update. Attach before/after screenshots if relevant.
- Docker and GPU tests run **on demand**: push the commits, then comment `run-ci` on the PR.

### Commit and branch rules (from `AGENTS.md`, the repo's agent instructions)
- Work on a feature branch. Never commit directly to `main`.
- Focused, atomic commits. Subject is imperative, capitalized, with no trailing period.
- **Do not add AI co-author or attribution lines** to commits in that repo.
- Prefer follow-up commits over amending while addressing review feedback.
- New source files use the repo's SPDX license header. Do not change existing headers.

## 4. Technical stack present in the project

- **Language and build:** Python, `uv`-managed environment, `pyproject.toml`.
- **Quality:** `pre-commit`, `ruff`, `pytest`, `towncrier` (changelog fragments).
- **Docs:** Sphinx with the Book Theme, reStructuredText.
- **Simulation:** Isaac Sim, plus Newton, PhysX and `ovphysx`/`ovrtx` backends; Warp kernels (`wp.copy`, indexed arrays, ProxyArray, CUDA graph capture); Torch.
- **CI:** GitHub Actions. GPU/Docker tests start on request via `run-ci`. A labeler marks PRs from branch names (`feature…` becomes `enhancement`; `fix…` or `bug…` becomes `bug`).

## 5. Other ways to contribute (stated)

- **Bug reports:** via the issue tracker, using the bug template (needs a minimal reproduction and the Isaac Lab / Isaac Sim versions).
- **Proposals and ideas:** a proposal issue (`Proposal` / `Motivation` / `Alternatives` / `Build Info`) or a GitHub Discussion.
- **Questions:** GitHub Discussions (there is also a question issue template).
- **Documentation and tutorials:** same PR flow as code.
- **Assets and environments:** only if they demonstrate a capability no existing example covers. Otherwise the guide says to keep them as a **community-owned project**, created with `uv run isaaclab --new` (choose **External**: Cartpole, Stubbed or Blank).
- **Incubation:** a `contrib` package in the repo, which needs a named owner, an evaluation period and an exit plan.

## 6. How a submission is judged

**The guide states a few criteria and leaves the rest unstated.**

Stated:
- Is it focused (one change) and aligned with the roadmap (the reason for opening an issue first)?
- Does it follow the style, with formatting, docs, tests and changelog fragment in place?
- For assets and environments, the **capability gate**: it must be the smallest clear example of a capability current examples do not cover. It must reuse existing assets, declare the supported physics backend and renderer combinations, document the capability and name a support owner. Asset sources also need clear licenses, immutable versioned hosting and reproducible conversion steps.
- Maintainers may consolidate, relocate or remove an in-repository example if its ownership or dependencies stop being sustainable.

**Not stated:**
- A numeric acceptance bar (no coverage percentage, no response-time promise).
- A "good first issue" process in this guide.
- A required `Signed-off-by` line. `CONTRIBUTING.md` reproduces the Developer Certificate of Origin text, but the guide never says commits must be signed off. **Ask in my first issue or check CI before assuming.**

## 7. My plan: how I would do it

1. **Today (done):** read the guide, write these notes, commit.
2. **Day 2:** install Isaac Lab, run the cartpole example, and record what breaks in a note.
3. **Day 3–4:** browse open issues labeled bug or documentation and pick one I can reproduce. A docs fix or a small bug is the right first PR, not a feature.
4. **Before coding:** comment on the issue (or open one) so a maintainer confirms it is wanted.
5. **PR:** branch `fix-<short-name>`, one change, a test that fails before the fix, a changelog fragment, `uv run isaaclab --format`, my name in `CONTRIBUTORS.md`, a filled-in template, then `run-ci`.
6. **Longer term:** the plant stem solver may fit the "deformable or coupled bodies" capability the guide mentions. Propose it as a Discussion first, and do not build it before a maintainer says where it belongs (core, `contrib`, or community-owned).

## What surprised me

- The changelog is **not edited by hand**: contributors add tiny fragment files and CI builds the changelog.
- The default home for most projects is **outside** the repo: robot ports, reward variants and paper reproductions are told to stay community-owned.
- The repo has instructions for AI agents, including "no AI attribution lines" in commits.
