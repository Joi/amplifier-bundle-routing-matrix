# Review instructions — amplifier-bundle-routing-matrix

Commit and range reviews read this file from the default branch; dirty
reviews and local runs read the working copy.

Joi's fork of microsoft/amplifier-bundle-routing-matrix. The bundle maps
semantic roles (`general`, `fast`, `coding` ...) to ranked provider/model
candidates in the YAML matrices under `routing/`. The `hooks-routing`
module in `modules/hooks-routing/` loads a matrix, applies user overrides
and resolves each agent's `model_role` to provider preferences, best
candidate first. `docs/MATRIX_CURATOR_GUIDE.md` holds the authoring rules
for matrix files. Its role list predates this fork's three extra roles
(`debugging`, `orchestration`, `evaluation`, commit a09966f), which exist
in `anthropic.yaml`, `balanced.yaml` and `quality.yaml`.

Already enforced (do not repeat): nothing. There is no CI and no git hook.
The pytest suites in `modules/hooks-routing/tests/` and `tests/` are run
by hand, and some of them are out of date with the fork (they hard-code
13 roles; `balanced.yaml` has 16). Do not assume a check passed because a
test for it exists.

Always flag:
- a matrix without `general` or `fast`, or a multi-provider matrix
  (`balanced`, `quality`, `economy`) without the 13 base roles. (Source:
  curator guide, Required Roles.)
- a `name` that does not match the filename without `.yaml`, or an
  `updated` date not moved to the day of the change.
- candidates that are not in best-first order, or a change to
  `balanced.yaml` (the reference matrix) not carried to the derived
  matrices where their philosophy allows. The resolver uses order as
  preference.
- a `provider` written as a module identifier (`provider-anthropic`) or as
  `google` instead of `gemini`; Anthropic model names with dots or GitHub
  Copilot model names with hyphens. A wrong exact name still resolves and
  then fails at the provider API, not in routing.
- a broad glob that crosses model classes (`claude-*`, `gpt-*`) anywhere
  but `model: "*"` for a user-managed provider such as Ollama, and any
  glob on a `github-copilot` candidate. (Source: curator guide, Pin Model
  Names.)
- a blacklisted model, the `base` keyword in a matrix file (it belongs in
  user overrides only), or a `config` block a candidate does not need.
- a role added, removed or redefined in only some of the places that list
  roles: the matrices, `context/routing-instructions.md`, both
  `context/role-definitions.md` and `skills/role-definitions/SKILL.md`,
  the README role table, the curator guide taxonomy and the tests'
  role lists.
- a resolver change that breaks: the version-aware glob sort (clean IDs
  above date-suffixed snapshots, `claude-opus-4-10` above
  `claude-opus-4-7`); exact names bypassing `list_models()`; exact
  matching of multi-instance provider IDs such as `ollama-cloud`; or the
  matrix lookup order (project `.amplifier/routing/`, then
  `~/.amplifier/routing/`, then the bundled `routing/`).
- a change under `modules/hooks-routing/` that assumes it will run from
  this fork: `behaviors/routing.yaml` installs the hook from
  microsoft/amplifier-bundle-routing-matrix@main, not from this repo.
- a test that reads or writes the real `~/.amplifier` or calls a live
  provider when a temp directory or a stub would do.
