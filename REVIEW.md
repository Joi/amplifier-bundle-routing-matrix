# Review instructions — amplifier-bundle-routing-matrix

Commit and range reviews read this file from the default branch; dirty
reviews and local runs read the working copy.

Joi's fork of microsoft/amplifier-bundle-routing-matrix. The bundle maps
semantic roles (`general`, `fast`, `coding` ...) to ranked provider/model
candidates in the YAML matrices under `routing/`. The `hooks-routing`
module in `modules/hooks-routing/` loads a matrix, applies user overrides
and resolves each agent's `model_role` to the first candidate an installed
provider can serve. `docs/MATRIX_CURATOR_GUIDE.md` is the authoring rule
book for matrix files.

Already enforced (do not repeat): there is no CI and no git hook. The
pytest suites are run by hand: `modules/hooks-routing/tests/` (matrix
structure, the 13 roles in `balanced.yaml`, required roles, no duplicate
candidates, the model blacklist, an ISO `updated` date, and the content
of the curator guide, role definitions and routing instructions) and
`tests/test_task4_tiered_context.py` (role definitions served as a skill).
Do not repeat what those tests assert; do flag a change that would make
them fail.

Always flag:
- a matrix without `general` or `fast`, or a multi-provider matrix
  (`balanced`, `quality`, `economy`) without all 13 roles. (Source: curator
  guide, Required Roles.)
- a `name` that does not match the filename without `.yaml`.
- a `provider` written as a module identifier (`provider-anthropic`) or as
  `google` instead of `gemini`. Also Anthropic model names with dots or
  GitHub Copilot model names with hyphens; both fail resolution silently.
- a broad glob that crosses model classes (`claude-*`, `gpt-*`) anywhere
  but `model: "*"` for a user-managed provider such as Ollama, and any
  glob on a `github-copilot` candidate. (Source: curator guide, Pin Model
  Names.)
- a blacklisted model, or the `base` keyword, in a matrix file. `base` has
  meaning only in user overrides in `settings.yaml`.
- a matrix edited without moving its `updated` date to the day of the
  change, or a `config` block on a candidate that does not need one.
- a new role not added to `context/routing-instructions.md`, or a role
  change made in only one of `context/role-definitions.md` and
  `skills/role-definitions/SKILL.md`, which carry the same text.
- a resolver change that breaks the version-aware glob sort: clean
  version IDs must rank above date-suffixed snapshots, and
  `claude-opus-4-10` above `claude-opus-4-7`. Also one that makes an
  exact model name call `list_models()`, or that changes the matrix
  lookup order (project `.amplifier/routing/`, then
  `~/.amplifier/routing/`, then the bundled `routing/`).
- a change under `modules/hooks-routing/` that assumes it will run from
  this fork: `behaviors/routing.yaml` installs the hook from
  microsoft/amplifier-bundle-routing-matrix@main, not from this repo.
- a test that reads or writes the real `~/.amplifier` or calls a live
  provider when a temp directory or a stub would do.
