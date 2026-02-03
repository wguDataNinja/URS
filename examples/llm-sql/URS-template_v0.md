# UNIVERSAL REPO SPECIFICATION (URS)
# purpose: definitive repo context for LLM-assisted work
# method: artifact-first skeleton, then docs reconciliation

meta:
  spec_id: TBD
  repo_name: TBD
  repo_type: TBD            # pipeline|service|tool|library|mixed|unknown
  status: TBD               # alpha|beta|stable|deprecated|unknown
  generated_at: TBD         # YYYY-MM-DD
  generated_by: TBD         # human|llm|mixed

process:
  passes:
    artifact_first:
      completed: false
      evidence: []          # commands run, files inspected
      notes: ""
    docs_reconcile:
      completed: false
      evidence: []          # docs opened (paths)
      notes: ""

# ------------------------------------------------------------
# TIER 1: ARTIFACT-FIRST SKELETON (REQUIRED)
# Precondition: perform a top-down presence pass (≥2 directory levels) over the repository
# to establish which artifact classes exist (code, tests, docs, data, configs, outputs).
# Populate this tier strictly from executable artifacts. Docs last.
# ------------------------------------------------------------

skeleton:
  system_kind: TBD          # batch|event|interactive|mixed|unknown
  components:
    - name: TBD
      role: TBD
      entrypoints: []       # must be real invocations found in repo
      reads: []             # inputs consumed (files/db/api/env)
      writes: []            # outputs produced (files/db/network)
      state: []             # persistent stores used
      ev: []                # file refs (path or path:line)

  entrypoints:
    primary: []             # required: at least one or explicit []
    secondary: []
    scheduled: []           # cron/launchd/ci; if none, []

  io:
    inputs:                 # required list; if none, []
      - name: TBD
        kind: TBD           # api|file|db|env|user|queue|prompt|other
        format: TBD
        source: TBD
        ev: []
    outputs:                # required list; must find actual outputs
      - name: TBD
        kind: TBD           # file|db|network|log|artifact|ui|other
        format: TBD
        location: TBD
        mutability: TBD     # append_only|overwrite|mixed|unknown
        ev: []

  state:
    stores:                 # required: db/files/cache/none
      - name: TBD
        kind: TBD           # sqlite|postgres|files|cache|none|unknown
        location: TBD
        mutability: TBD
        schema_authority: []  # schema.sql / models / migrations / etc
        ev: []

  artifacts:
    directories: []         # e.g. artifacts/, runs/, logs/
    manifests: []           # manifest.json formats, run indices
    ev: []

  constraints:
    enforced: []            # rules enforced by code (validators, guards)
    conventions: []         # rules implied but not enforced
    unknown: []             # where enforcement could not be found
    ev: []

  tests_and_validation:
    tests_present: TBD      # true|false|unknown
    validators: []          # in-repo validators/checkers
    ev: []

  gaps:
    missing_or_stubbed: []  # features referenced but not implemented
    ambiguous: []           # unclear semantics
    ev: []

# ------------------------------------------------------------
# TIER 2: DOC INVENTORY (REQUIRED, AFTER SKELETON)
# Enumerate which docs/specs exist and what they claim.
# ------------------------------------------------------------

docs:
  important_docs:           # required: list docs used in reconcile pass
    - path: TBD
      type: TBD             # readme|spec|adr|rfc|notes|runbook|other
      claims: []            # brief list of major claims in that doc
  known_specs:
    - path: TBD
      governs: TBD          # schema|pipeline|api|safety|process|other
  missing_docs:
    - TBD                   # docs you expected but did not find

# ------------------------------------------------------------
# TIER 3: RECONCILIATION + NARRATIVE (DOC-AWARE)
# This is where explanation expands, but must preserve skeleton truth.
# ------------------------------------------------------------

reconcile:
  contradictions:           # required; [] allowed only if explicitly none
    - doc_claim: TBD
      artifact_finding: TBD
      resolution: TBD       # accept_artifact|accept_doc|mixed|unknown
      notes: TBD
  updates_after_docs: []    # what changed in understanding after docs

narrative:
  purpose_and_scope:
    statement: TBD
    non_goals: []
    success_criteria: []
  architecture_notes: []    # higher-level explanation; may be verbose
  decisions_and_rationale: []  # ADR-like content (doc-aware)
  operating_model: []       # “how to run / how to modify safely”
  confidence:
    high: []
    low: []