# UNIVERSAL REPO SPECIFICATION (URS)
# purpose: definitive repository context for LLM-assisted work
# method: artifact-first skeleton, followed by documentation reconciliation

meta:
  urs_version: "1.0"
  spec_id: "llm-reddit-fetcher-urs-2026-02-03"
  repo_name: "llm-reddit-fetcher"
  repo_type: "pipeline"
  status: "unknown"
  generated_at: "2026-02-03"
  generated_by: "llm"

process:
  passes:
    artifact_first:
      completed: true
      evidence:
        - "rg --files"
        - "sed -n 1,200 README.md"
        - "nl -ba reddit_fetcher/cli.py"
        - "nl -ba reddit-fetch"
        - "nl -ba reddit_fetcher/fetcher.py"
        - "nl -ba reddit_fetcher/specs.py"
        - "nl -ba reddit_fetcher/planner.py"
        - "nl -ba reddit_fetcher/normalize.py"
        - "nl -ba reddit_fetcher/storage.py"
        - "nl -ba reddit_fetcher/sql_selector.py"
        - "nl -ba reddit_fetcher/query_contract.py"
        - "nl -ba reddit_fetcher/llm_translate.py"
        - "sed -n 1,200 reddit_fetcher/storage/schema.sql"
        - "sed -n 1,200 tests/test_storage.py"
      notes: "Artifact-first pass focused on CLI, fetcher, planner, specs, storage, query lane, and tests. No docs used to populate Tier 1."
    docs_reconcile:
      completed: true
      evidence:
        - "sed -n 1,200 docs/PIPELINE_SPEC.md"
        - "sed -n 1,200 docs/OPTIONS_AND_EXTENSIONS.md"
        - "sed -n 1,200 docs/reddit_fetcher_URS.md"
        - "sed -n 1,200 docs/gold_test_plan.txt"
        - "sed -n 1,200 docs/sql_gold_plan.txt"
        - "sed -n 1,200 README.md"
      notes: "Docs reviewed after artifacts; contradictions listed in reconcile section."

# ------------------------------------------------------------
# TIER 1: ARTIFACT-FIRST SKELETON (REQUIRED)
# Populate this tier strictly from executable artifacts:
# code, configs, tests, data, manifests, and observed in-repo outputs.
# Documentation must NOT be used to populate this tier.
#
# A top-down presence pass (>=2 directory levels) may be used ONLY to
# ensure coverage; it must not be used as a factual source.
# ------------------------------------------------------------

skeleton:
  system_kind: "batch"
  runtime: "python"

  components:
    - name: "cli"
      role: "CLI entrypoint that orchestrates plan/run/history/show/query workflows"
      entrypoints:
        - "./reddit-fetch"
        - "reddit_fetcher.cli:main"
      reads:
        - "CLI args: command + input text"
        - "DB path flag or default"
      writes:
        - "stdout/stderr output"
        - "SQLite runs database"
      state:
        - "runs/runs.db (default)"
      ev:
        - "reddit-fetch:1"
        - "reddit_fetcher/cli.py:1"
        - "reddit_fetcher/cli.py:37"
        - "reddit_fetcher/cli.py:120"
        - "reddit_fetcher/cli.py:323"

    - name: "fetcher"
      role: "PRAW-based Reddit fetch primitives"
      entrypoints:
        - "reddit_fetcher.fetcher:fetch_user_posts"
        - "reddit_fetcher.fetcher:fetch_subreddit_posts"
        - "reddit_fetcher.fetcher:fetch_subreddit_comments"
        - "reddit_fetcher.fetcher:fetch_subreddit_about"
        - "reddit_fetcher.fetcher:fetch_submission_comments"
      reads:
        - "Reddit API via PRAW"
        - "Reddit credentials from environment"
      writes:
        - "in-memory payloads"
      state: []
      ev:
        - "reddit_fetcher/fetcher.py:7"
        - "reddit_fetcher/fetcher.py:28"
        - "reddit_fetcher/fetcher.py:58"
        - "reddit_fetcher/fetcher.py:81"
        - "reddit_fetcher/fetcher.py:100"
        - "reddit_fetcher/reddit_client.py:7"

    - name: "planner"
      role: "Deterministic offline planner that emits UsagePlan + alerts"
      entrypoints:
        - "reddit_fetcher.planner:plan"
      reads:
        - "FetchSpec"
      writes:
        - "UsagePlan (ops, expected calls, alerts, ok_to_run)"
      state: []
      ev:
        - "reddit_fetcher/planner.py:12"
        - "reddit_fetcher/planner.py:22"
        - "reddit_fetcher/planner.py:82"

    - name: "specs_and_limits"
      role: "FetchSpec data model, validation, and hard limits"
      entrypoints:
        - "reddit_fetcher.specs:spec_from_json"
        - "reddit_fetcher.specs:spec_from_json_strict"
        - "reddit_fetcher.specs:validate_fetch_spec"
      reads:
        - "input JSON dicts"
      writes:
        - "validated FetchSpec with run_slug"
      state: []
      ev:
        - "reddit_fetcher/specs.py:26"
        - "reddit_fetcher/specs.py:88"
        - "reddit_fetcher/specs.py:149"
        - "reddit_fetcher/limits.py:3"
        - "reddit_fetcher/limits.py:13"

    - name: "normalization"
      role: "Normalize fetch payloads into storage item rows"
      entrypoints:
        - "reddit_fetcher.normalize:normalize"
      reads:
        - "FetchSpec"
        - "fetcher payloads"
      writes:
        - "item rows with data_json"
      state: []
      ev:
        - "reddit_fetcher/normalize.py:1"
        - "reddit_fetcher/normalize.py:138"

    - name: "storage"
      role: "SQLite persistence for runs and items, plus read-only query execution"
      entrypoints:
        - "reddit_fetcher.storage:init_db"
        - "reddit_fetcher.storage:insert_run"
        - "reddit_fetcher.storage:insert_items_bulk"
        - "reddit_fetcher.storage:get_history"
        - "reddit_fetcher.storage:get_items"
        - "reddit_fetcher.storage:execute_readonly"
      reads:
        - "SQLite db file"
        - "schema.sql"
      writes:
        - "SQLite db file"
      state:
        - "runs/runs.db (default)"
      ev:
        - "reddit_fetcher/storage.py:16"
        - "reddit_fetcher/storage.py:22"
        - "reddit_fetcher/storage.py:36"
        - "reddit_fetcher/storage.py:81"
        - "reddit_fetcher/storage.py:115"
        - "reddit_fetcher/storage.py:185"
        - "reddit_fetcher/storage/schema.sql"

    - name: "query_lane"
      role: "Query contract + template SQL compiler + NL translators (stub or LLM)"
      entrypoints:
        - "reddit_fetcher.query_contract:QueryPlan"
        - "reddit_fetcher.query_translate_stub:translate_nl_to_plan"
        - "reddit_fetcher.llm_query_translate:translate_to_query_plan"
        - "reddit_fetcher.sql_selector:compile_plan"
      reads:
        - "NL query text"
        - "run_slug"
      writes:
        - "compiled SQL"
      state: []
      ev:
        - "reddit_fetcher/query_contract.py:1"
        - "reddit_fetcher/query_contract.py:49"
        - "reddit_fetcher/sql_selector.py:1"
        - "reddit_fetcher/query_translate_stub.py:1"
        - "reddit_fetcher/llm_query_translate.py:1"

    - name: "llm_translation"
      role: "NL to FetchSpec translation via Ollama or OpenAI, optional trace files"
      entrypoints:
        - "reddit_fetcher.llm_translate:translate_to_spec"
      reads:
        - "NL request"
        - "REDDIT_FETCH_LLM_TRACE_DIR/TAG"
        - "OPENAI_API_KEY (when provider=gpt)"
      writes:
        - "FetchSpec"
        - "trace JSON files (optional)"
      state: []
      ev:
        - "reddit_fetcher/llm_translate.py:1"
        - "reddit_fetcher/llm_translate.py:42"
        - "reddit_fetcher/llm_translate.py:117"

  entrypoints:
    primary:
      - "./reddit-fetch"
    secondary:
      - "scripts/live_gold_validate.py"
      - "scripts/sql_gold_validate.py"
      - "scripts/sql_selector_validate.py"
      - "scripts/smoke_store_run.py"
      - "scripts/ollama_generate.py"
      - "scripts/ollama_check.py"
      - "scripts/llm_smoke_test.py"
    scheduled: []

  io:
    inputs:
      - name: "CLI request"
        kind: "user"
        format: "JSON FetchSpec or natural language"
        source: "stdin/argv"
        ev:
          - "reddit_fetcher/cli.py:70"
          - "reddit_fetcher/cli.py:120"
      - name: "Reddit API credentials"
        kind: "env"
        format: "REDDIT_CLIENT_ID/SECRET/USER_AGENT"
        source: "environment variables"
        ev:
          - "reddit_fetcher/reddit_client.py:7"
      - name: "LLM provider credentials"
        kind: "env"
        format: "OPENAI_API_KEY"
        source: "environment variables"
        ev:
          - "reddit_fetcher/llm_translate.py:76"
      - name: "Query lane limits"
        kind: "env"
        format: "HARD_MAX_LIMIT"
        source: "query_contract"
        ev:
          - "reddit_fetcher/query_contract.py:16"
      - name: "Reddit API"
        kind: "api"
        format: "PRAW"
        source: "https://www.reddit.com"
        ev:
          - "reddit_fetcher/fetcher.py:7"

    outputs:
      - name: "Run database"
        kind: "db"
        format: "SQLite"
        location: "runs/runs.db (default)"
        mutability: "mixed"
        ev:
          - "reddit_fetcher/cli.py:37"
          - "reddit_fetcher/storage.py:22"
      - name: "CLI output"
        kind: "log"
        format: "text"
        location: "stdout/stderr"
        mutability: "append_only"
        ev:
          - "reddit_fetcher/cli.py:91"
          - "reddit_fetcher/cli.py:303"
      - name: "LLM trace files"
        kind: "artifact"
        format: "JSON"
        location: "$REDDIT_FETCH_LLM_TRACE_DIR (optional)"
        mutability: "append_only"
        ev:
          - "reddit_fetcher/llm_translate.py:42"

  state:
    stores:
      - "runs/runs.db (default)"
      - "custom db path via --db"

  artifacts:
    directories:
      - "runs/"
      - "golds/"
      - "llm_traces/"
    manifests:
      - "requirements.txt"
      - "reddit_fetcher/storage/schema.sql"
      - "golds/live/cases.json"
    ev:
      - "requirements.txt"
      - "reddit_fetcher/storage/schema.sql"
      - "golds/live/cases.json"

  constraints:
    enforced:
      - "plan/run must be gated by --approve for execution"
      - "hard caps on request size and comment expansion"
      - "run_slug uniqueness enforced before execution"
      - "strict FetchSpec validation for execution"
      - "query SQL must be SELECT-only and single-statement"
      - "read-only query row cap (MAX_READONLY_ROWS)"
    conventions:
      - "default db path is runs/runs.db"
      - "default LLM provider is llama3"
      - "run_slug fallback patterns based on spec"
      - "subreddit_comments warn unless allow_context_loss=true"
    unknown:
      - "API rate limiting/backoff behavior"
      - "centralized logging/metrics"
    ev:
      - "reddit_fetcher/cli.py:120"
      - "reddit_fetcher/cli.py:167"
      - "reddit_fetcher/specs.py:88"
      - "reddit_fetcher/limits.py:3"
      - "reddit_fetcher/fetcher.py:111"
      - "reddit_fetcher/sql_selector.py:51"
      - "reddit_fetcher/storage.py:185"

  tests_and_validation:
    tests_present: true
    validators:
      - "reddit_fetcher.sql_selector:validate_sql_safety"
      - "reddit_fetcher.query_contract:QuerySpec.validate"
    ev:
      - "tests/test_storage.py"
      - "tests/test_gold_alerts.py"
      - "tests/test_query_gold.py"
      - "tests/test_query_llm_gold.py"
      - "reddit_fetcher/sql_selector.py:51"
      - "reddit_fetcher/query_contract.py:60"

  gaps:
    missing_or_stubbed:
      - "No .env.example file present for credential setup"
    ambiguous:
      - "Whether scripts/ are supported user entrypoints or dev-only"
    ev:
      - "README.md:20"
      - "scripts/"

# ============================================================
# TIER 2: DOCUMENTATION INVENTORY (REQUIRED)
# Enumerate documentation after the skeleton is complete.
# This tier records claims only; it does not assert correctness.
# ============================================================

docs:
  important_docs:
    - path: "README.md"
      type: "readme"
      claims:
        - "Two-phase workflow (plan then run)"
        - "CLI commands: plan/run/history/show"
        - "Hard safety caps on API calls and comments"
        - "LLM translation optional; llama3 default; gpt optional"
        - "SQLite storage default at runs/runs.db"
        - "Setup mentions .env.example"
    - path: "docs/PIPELINE_SPEC.md"
      type: "spec"
      claims:
        - "Execution baseline v1 is frozen"
        - "Only supported CLI commands are plan/run/history/show"
        - "Two-phase execution is mandatory"
        - "Planner is deterministic and offline"
        - "Alerts are first-class and block execution on severity=block"
        - "SQLite is authoritative storage"
    - path: "docs/reddit_fetcher_URS.md"
      type: "spec"
      claims:
        - "Pipeline is portable beyond Reddit"
        - "LLM is operator, not authority"
        - "Hard caps and strict validation are invariants"
    - path: "docs/OPTIONS_AND_EXTENSIONS.md"
      type: "notes"
      claims:
        - "Options B/C are conceptual and not implemented"
        - "Option A (URS) remains the immutable baseline"
    - path: "docs/gold_test_plan.txt"
      type: "plan"
      claims:
        - "Gold suite calibrates NL to FetchSpec LLM"
        - "Gold cases live in golds/live/cases.json"
        - "scripts/live_gold_validate.py is the harness"
    - path: "docs/sql_gold_plan.txt"
      type: "plan"
      claims:
        - "Gold SQL tasks operate on golds/data/gold_runs.sqlite"
        - "Only SELECT allowed in SQL"
        - "Multi-statement SQL may be allowed in gold harness"

  known_specs:
    - path: "docs/PIPELINE_SPEC.md"
      governs: "pipeline"
    - path: "reddit_fetcher/storage/schema.sql"
      governs: "schema"

  missing_docs:
    - "LICENSE"
    - ".env.example"

# ============================================================
# TIER 3: RECONCILIATION + NARRATIVE (DOC-AWARE)
# Reconcile documentation claims against artifact truth.
# Artifacts remain authoritative unless explicitly overridden.
# ============================================================

reconcile:
  contradictions:
    - doc_claim: "CLI provides only plan/run/history/show commands"
      artifact_finding: "CLI defines an additional query command"
      resolution: "accept_artifact"
      notes: "Update README and PIPELINE_SPEC to include query lane or mark as experimental."
    - doc_claim: "Setup uses .env.example for Reddit credentials"
      artifact_finding: "No .env.example file present in repo"
      resolution: "accept_artifact"
      notes: "Either add .env.example or update README to match current setup."
    - doc_claim: "No other supported entrypoints beyond the unified CLI"
      artifact_finding: "scripts/ directory contains multiple runnable tools"
      resolution: "mixed"
      notes: "Likely dev-only scripts; clarify support status."
    - doc_claim: "Gold SQL harness allows multi-statement SQL"
      artifact_finding: "Query lane SQL safety forbids multi-statement execution"
      resolution: "mixed"
      notes: "Gold SQL plan may target a different harness; clarify scope."

  updates_after_docs: []

narrative:
  purpose_and_scope:
    statement: "Provide a safe, deterministic, two-phase Reddit data collection pipeline with optional NL translation and a read-only query lane over stored runs."
    non_goals:
      - "Autonomous background fetching"
      - "Bypassing planner or safety caps"
      - "Arbitrary SQL execution"
    success_criteria:
      - "Plan produces alerts and expected call estimates without network access"
      - "Run requires explicit approval, fetches via PRAW, normalizes, and stores in SQLite"
      - "History/show/query can inspect stored results safely"

  architecture_notes:
    - "CLI orchestrates plan -> run -> normalize -> storage; query lane compiles QueryPlan to parameterized SQL."
    - "Hard caps are enforced in both validation and fetcher boundaries."

  decisions_and_rationale:
    - "Two-phase gating reduces risk and makes behavior deterministic before execution."
    - "Template-based SQL and read-only execution limit query surface area."

  operating_model:
    - "User supplies JSON or NL; planner evaluates; run executes only with --approve; results persist in SQLite; query lane reads stored runs."

  confidence:
    high:
      - "CLI orchestration and storage pipeline"
      - "Planner alert generation and hard caps"
    low:
      - "LLM provider behavior without local environment setup"
      - "Gold harness behavior not executed in this pass"

___
