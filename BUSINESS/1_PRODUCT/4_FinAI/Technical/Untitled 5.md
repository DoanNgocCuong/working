comet-ai-browser/
│
├── 📁 docs/
│   ├── README.md                           # Main project documentation
│   ├── ARCHITECTURE.md                     # System architecture overview
│   ├── API.md                              # API specifications (endpoints, schemas)
│   ├── THREAT-MODEL.md                     # Security threat model & mitigations
│   ├── RUNBOOK.md                          # Operations & troubleshooting guide
│   ├── ROADMAP.md                          # Future features & milestones
│   └── GLOSSARY.md                         # Terminology & definitions
│
├── 📁 packages/
│   └── shared/
│       ├── tsconfig.json
│       ├── package.json
│       └── src/
│           ├── types/                      # Shared TypeScript types
│           │   ├── common.ts               # RawRequest, NormalizedInput
│           │   ├── task-spec.ts            # TaskSpecV1, ActionPlan
│           │   ├── evidence.ts             # EvidencePack, EvidenceItem
│           │   ├── answer.ts               # AnswerSkeleton, FinalAnswer
│           │   └── errors.ts               # Error types, exception classes
│           ├── schemas/                    # Validation schemas & serializers
│           │   ├── input.schema.ts         # RawRequestV1 validation
│           │   ├── task.schema.ts          # TaskSpecV1 validation
│           │   ├── evidence.schema.ts      # EvidencePack validation
│           │   ├── answer.schema.ts        # FinalAnswer validation
│           │   └── api.schema.ts           # API response/request schemas
│           └── utils/
│               ├── logger.ts               # Structured logging setup
│               ├── tracer.ts               # OpenTelemetry tracing
│               ├── hashing.ts              # SHA256, payload hashing
│               ├── url-parser.ts           # URL parsing & validation
│               ├── validators.ts           # Common validators
│               └── constants.ts            # Global constants, limits
│
├── 📁 services/
│   │
│   ├── 🔷 STAGE-1-unified-input-core/
│   │   ├── Dockerfile
│   │   ├── docker-compose.yml
│   │   ├── .env.example
│   │   ├── tsconfig.json
│   │   ├── package.json
│   │   ├── README.md
│   │   └── src/
│   │       ├── index.ts                    # Service entrypoint
│   │       ├── config.ts                   # Configuration & env vars
│   │       ├── constants.ts                # Local constants (MAX_INPUT_LENGTH, etc.)
│   │       ├── exceptions.ts               # Custom exception classes
│   │       ├── api/
│   │       │   ├── routes.ts               # FastAPI/Express route handlers
│   │       │   ├── handlers.ts             # Request/response handlers
│   │       │   ├── middleware.ts           # Auth, CORS, logging middleware
│   │       │   └── schemas.ts              # API request/response schemas
│   │       ├── stages/
│   │       │   └── stage1/                 # Stage 1 pipeline logic
│   │       │       ├── validator.ts        # 1.1 validateRawRequest
│   │       │       ├── env-builder.ts      # 1.2 buildEnv
│   │       │       ├── envelope.ts         # 1.3 initEnvelope
│   │       │       ├── classifier.ts       # 1.4 runInputClassifier
│   │       │       ├── normalizer.ts       # 1.5 runTextNormalizer
│   │       │       ├── context.ts          # 1.6 attachPageContext
│   │       │       ├── safety.ts           # 1.7 computeSafetyFlags
│   │       │       └── telemetry.ts        # 1.8 buildTelemetry
│   │       ├── modules/
│   │       │   ├── input-adapter.ts        # InputAdapter component
│   │       │   ├── context-collector.ts    # ContextCollector component
│   │       │   └── normalizer.ts           # Normalizer component
│   │       ├── integrations/
│   │       │   ├── redis.ts                # Redis client for idempotency
│   │       │   ├── cache.ts                # Caching layer
│   │       │   └── vault.ts                # Secrets management
│   │       └── observability/
│   │           ├── logging.ts              # Structured JSON logging
│   │           ├── metrics.ts              # Prometheus metrics
│   │           └── tracing.ts              # OpenTelemetry spans
│   │   └── tests/
│   │       ├── unit/
│   │       │   ├── normalizer.test.ts
│   │       │   ├── classifier.test.ts
│   │       │   ├── safety.test.ts
│   │       │   └── validator.test.ts
│   │       ├── integration/
│   │       │   ├── api.test.ts
│   │       │   ├── redis.test.ts
│   │       │   └── pipeline.test.ts
│   │       └── e2e/
│   │           ├── full-flow.test.ts
│   │           └── idempotency.test.ts
│   │
│   ├── 🔷 STAGE-2-query-understanding/
│   │   ├── Dockerfile
│   │   ├── docker-compose.yml
│   │   ├── .env.example
│   │   ├── tsconfig.json
│   │   ├── package.json
│   │   ├── README.md
│   │   └── src/
│   │       ├── index.ts
│   │       ├── config.ts
│   │       ├── constants.ts
│   │       ├── exceptions.ts
│   │       ├── api/
│   │       │   ├── routes.ts
│   │       │   ├── handlers.ts
│   │       │   └── schemas.ts
│   │       ├── stages/
│   │       │   └── stage2/
│   │       │       ├── orchestrator.ts     # runQueryUnderstanding entrypoint
│   │       │       ├── rule-engine.ts      # 2.1 Rule-based classification (A, B, C, D)
│   │       │       ├── intent-extractor.ts # Intent & Slot extraction
│   │       │       ├── entity-parser.ts    # Parse budget, time, travel, quantity
│   │       │       ├── policy-engine.ts    # 2.2 Policy overrides & safety checks
│   │       │       └── slm-module.ts       # 2.3 SLM call for slow path
│   │       ├── modules/
│   │       │   ├── policy-classifier.ts    # Toxicity, PII, injection detection
│   │       │   └── embedding-prep.ts       # Query embedding generation
│   │       ├── rules/
│   │       │   ├── intent-rules.ts         # Intent classification rules
│   │       │   ├── entity-rules.ts         # Entity extraction rules
│   │       │   └── policy-rules.ts         # Policy & safety rules
│   │       ├── integrations/
│   │       │   ├── model-gateway.ts        # SLM model calls
│   │       │   └── embeddings.ts           # Embedding service
│   │       └── observability/
│   │           ├── logging.ts
│   │           ├── metrics.ts
│   │           └── tracing.ts
│   │   └── tests/
│   │       ├── unit/
│   │       │   ├── rule-engine.test.ts
│   │       │   ├── entity-parser.test.ts
│   │       │   └── policy-engine.test.ts
│   │       ├── integration/
│   │       │   ├── slm-call.test.ts
│   │       │   └── end-to-end.test.ts
│   │       └── golden/
│   │           └── golden-tests.ts         # Deterministic outputs
│   │
│   ├── 🔷 STAGE-3-router-planner/
│   │   ├── Dockerfile
│   │   ├── docker-compose.yml
│   │   ├── .env.example
│   │   ├── tsconfig.json
│   │   ├── package.json
│   │   ├── README.md
│   │   └── src/
│   │       ├── index.ts
│   │       ├── config.ts
│   │       ├── constants.ts
│   │       ├── exceptions.ts
│   │       ├── api/
│   │       │   ├── routes.ts
│   │       │   ├── handlers.ts
│   │       │   └── schemas.ts
│   │       ├── stages/
│   │       │   └── stage3/
│   │       │       ├── orchestrator.ts     # Main Stage 3 orchestrator
│   │       │       ├── signal-extractor.ts # 3.1 Extract routing signals
│   │       │       ├── mode-selector.ts    # 3.2 Mode selection (A, B, C, D)
│   │       │       ├── plan-builder.ts     # 3.3 PlanBuilder (Planner LLM)
│   │       │       ├── plan-validator.ts   # 3.4 PlanValidator
│   │       │       ├── budget-manager.ts   # 3.5 BudgetManager
│   │       │       ├── state-manager.ts    # 3.6 StateManager & task FSM
│   │       │       └── step-executor.ts    # 3.7 Runtime step execution
│   │       ├── modules/
│   │       │   ├── plan-generator.ts       # ActionPlan generation
│   │       │   ├── policy-enforcer.ts      # Policy constraints enforcement
│   │       │   └── personalization.ts      # User preference adjustments
│   │       ├── integrations/
│   │       │   ├── model-gateway.ts        # LLM/Planner calls
│   │       │   ├── state-store.ts          # Redis/DB state persistence
│   │       │   └── tool-registry.ts        # Capability registry
│   │       └── observability/
│   │           ├── logging.ts
│   │           ├── metrics.ts
│   │           └── tracing.ts
│   │   └── tests/
│   │       ├── unit/
│   │       │   ├── plan-validator.test.ts
│   │       │   ├── budget-manager.test.ts
│   │       │   └── state-manager.test.ts
│   │       ├── integration/
│   │       │   ├── mode-selection.test.ts
│   │       │   └── plan-generation.test.ts
│   │       └── scenario/
│   │           ├── mode-a.scenario.ts
│   │           ├── mode-b.scenario.ts
│   │           ├── mode-c.scenario.ts
│   │           └── mode-d.scenario.ts
│   │
│   ├── 🔷 STAGE-4-unified-executor/
│   │   ├── Dockerfile
│   │   ├── docker-compose.yml
│   │   ├── .env.example
│   │   ├── tsconfig.json
│   │   ├── package.json
│   │   ├── README.md
│   │   └── src/
│   │       ├── index.ts
│   │       ├── config.ts
│   │       ├── constants.ts
│   │       ├── exceptions.ts
│   │       ├── api/
│   │       │   ├── routes.ts
│   │       │   ├── handlers.ts
│   │       │   └── schemas.ts
│   │       ├── stages/
│   │       │   └── stage4/
│   │       │       ├── orchestrator.ts     # Stage 4 main executor
│   │       │       ├── 4.1-retrieval.ts    # 4.1 RetrievalEngine
│   │       │       ├── 4.2-fetch.ts        # 4.2 WebFetch & SSRF Guard
│   │       │       ├── 4.3-extract.ts      # 4.3 DOM parsing & extraction
│   │       │       ├── 4.4-action.ts       # 4.4 Action execution
│   │       │       └── evidence-builder.ts # EvidenceBuilder & packaging
│   │       ├── modules/
│   │       │   ├── retrieval-engine.ts     # Hybrid search (BM25 + vector)
│   │       │   ├── reranker.ts             # Cross-encoder reranking
│   │       │   ├── web-fetch.ts            # HTTP client with retries
│   │       │   ├── ssrf-guard.ts           # SSRF protection
│   │       │   ├── dom-parser.ts           # HTML parsing & extraction
│   │       │   ├── table-extractor.ts      # Table structure extraction
│   │       │   ├── content-extractor.ts    # Main content extraction
│   │       │   ├── tool-sandbox.ts         # Tool execution sandbox
│   │       │   ├── action-policy.ts        # Action allowlisting
│   │       │   └── execution-trace.ts      # Trace collection
│   │       ├── integrations/
│   │       │   ├── model-gateway.ts        # VLM/OCR calls
│   │       │   ├── vector-store.ts         # Embedding search
│   │       │   ├── search-provider.ts      # External search API (optional)
│   │       │   ├── headless-browser.ts     # Playwright/Puppeteer
│   │       │   └── http-client.ts          # HTTP fetching
│   │       └── observability/
│   │           ├── logging.ts
│   │           ├── metrics.ts
│   │           └── tracing.ts
│   │   └── tests/
│   │       ├── unit/
│   │       │   ├── ssrf-guard.test.ts
│   │       │   ├── dom-parser.test.ts
│   │       │   └── retrieval.test.ts
│   │       ├── integration/
│   │       │   ├── web-fetch.test.ts
│   │       │   ├── action-execution.test.ts
│   │       │   └── end-to-end.test.ts
│   │       └── security/
│   │           ├── ssrf.test.ts
│   │           └── injection.test.ts
│   │
│   ├── 🔷 STAGE-5-reasoning/
│   │   ├── Dockerfile
│   │   ├── docker-compose.yml
│   │   ├── .env.example
│   │   ├── tsconfig.json
│   │   ├── package.json
│   │   ├── README.md
│   │   └── src/
│   │       ├── index.ts
│   │       ├── config.ts
│   │       ├── constants.ts
│   │       ├── exceptions.ts
│   │       ├── api/
│   │       │   ├── routes.ts
│   │       │   ├── handlers.ts
│   │       │   └── schemas.ts
│   │       ├── stages/
│   │       │   └── stage5/
│   │       │       ├── orchestrator.ts     # Stage 5 reasoning orchestrator
│   │       │       ├── 5.1-summarizer.ts   # 5.1 EvidenceSummarizer
│   │       │       ├── 5.2-reasoning.ts    # 5.2 ReasoningCore (Reasoning LLM)
│   │       │       ├── 5.3-citation.ts     # 5.3 CitationMapper
│   │       │       └── 5.4-safety.ts       # 5.4 OutputSafetyCheck
│   │       ├── modules/
│   │       │   ├── evidence-summarizer.ts  # Token reduction & compression
│   │       │   ├── claim-extractor.ts      # Atomic claim extraction
│   │       │   ├── citation-mapper.ts      # Claim-to-evidence mapping
│   │       │   ├── contradiction-resolver.ts # Handle conflicting sources
│   │       │   └── safety-checker.ts       # Output safety validation
│   │       ├── integrations/
│   │       │   ├── model-gateway.ts        # Reasoning LLM calls
│   │       │   └── state-store.ts          # Evidence retrieval
│   │       └── observability/
│   │           ├── logging.ts
│   │           ├── metrics.ts
│   │           └── tracing.ts
│   │   └── tests/
│   │       ├── unit/
│   │       │   ├── claim-extractor.test.ts
│   │       │   └── citation-mapper.test.ts
│   │       ├── integration/
│   │       │   ├── reasoning-flow.test.ts
│   │       │   └── safety-check.test.ts
│   │       └── quality/
│   │           └── citation-quality.test.ts
│   │
│   ├── 🔷 STAGE-6-answer-synthesis/
│   │   ├── Dockerfile
│   │   ├── docker-compose.yml
│   │   ├── .env.example
│   │   ├── tsconfig.json
│   │   ├── package.json
│   │   ├── README.md
│   │   └── src/
│   │       ├── index.ts
│   │       ├── config.ts
│   │       ├── constants.ts
│   │       ├── exceptions.ts
│   │       ├── api/
│   │       │   ├── routes.ts
│   │       │   ├── handlers.ts
│   │       │   └── schemas.ts
│   │       ├── stages/
│   │       │   └── stage6/
│   │       │       ├── orchestrator.ts     # Stage 6 main orchestrator
│   │       │       ├── 6.1-composer.ts     # 6.1 SynthesisComposer (LLM)
│   │       │       ├── 6.2-quick.ts        # 6.2 QuickPathSynth (SLM)
│   │       │       ├── 6.3-format.ts       # 6.3 PersonalizationFormatter
│   │       │       ├── 6.4-postproc.ts     # 6.4 PostProcessor
│   │       │       └── 6.5-package.ts      # 6.5 ResponsePackaging
│   │       ├── modules/
│   │       │   ├── composer.ts             # Answer composition
│   │       │   ├── formatter.ts            # Markdown/block formatting
│   │       │   ├── personalizer.ts         # User preference application
│   │       │   ├── post-processor.ts       # Cleanup & validation
│   │       │   └── response-builder.ts     # API response construction
│   │       ├── templates/
│   │       │   ├── answer-template.ts      # Answer structure templates
│   │       │   ├── citation-format.ts      # Citation rendering
│   │       │   └── error-messages.ts       # Standard error responses
│   │       ├── integrations/
│   │       │   ├── model-gateway.ts        # LLM calls (composer)
│   │       │   └── state-store.ts          # Evidence & skeleton retrieval
│   │       └── observability/
│   │           ├── logging.ts
│   │           ├── metrics.ts
│   │           └── tracing.ts
│   │   └── tests/
│   │       ├── unit/
│   │       │   ├── composer.test.ts
│   │       │   ├── formatter.test.ts
│   │       │   └── post-processor.test.ts
│   │       ├── integration/
│   │       │   ├── synthesis-flow.test.ts
│   │       │   └── response-build.test.ts
│   │       └── output-quality/
│   │           ├── citation-format.test.ts
│   │           └── markdown-validity.test.ts
│   │
│   ├── 🔷 model-gateway/
│   │   ├── Dockerfile
│   │   ├── docker-compose.yml
│   │   ├── .env.example
│   │   ├── tsconfig.json
│   │   ├── package.json
│   │   ├── README.md
│   │   └── src/
│   │       ├── index.ts
│   │       ├── config.ts
│   │       ├── adapters/
│   │       │   ├── llm-adapter.ts          # LLM (GPT-4, Claude, etc.)
│   │       │   ├── slm-adapter.ts          # Small LM (Llama, Phi, etc.)
│   │       │   ├── embeddings-adapter.ts   # Embeddings (OpenAI, local)
│   │       │   ├── reranker-adapter.ts     # Cross-encoder reranking
│   │       │   └── vlm-adapter.ts          # Vision-Language Model
│   │       ├── policies/
│   │       │   ├── retry-policy.ts         # Exponential backoff
│   │       │   ├── budget-policy.ts        # Token/cost enforcement
│   │       │   ├── timeout-policy.ts       # Request timeouts
│   │       │   └── fallback-policy.ts      # Model fallback chains
│   │       ├── cache/
│   │       │   ├── embedding-cache.ts
│   │       │   ├── model-response-cache.ts
│   │       │   └── cache-invalidation.ts
│   │       └── observability/
│   │           ├── logging.ts
│   │           ├── metrics.ts
│   │           └── cost-tracking.ts
│   │   └── tests/
│   │       ├── unit/
│   │       └── integration/
│   │
│   ├── 🔷 retrieval-service/
│   │   ├── Dockerfile
│   │   ├── docker-compose.yml
│   │   ├── .env.example
│   │   ├── tsconfig.json
│   │   ├── package.json
│   │   ├── README.md
│   │   └── src/
│   │       ├── index.ts
│   │       ├── config.ts
│   │       ├── hybrid/
│   │       │   ├── hybrid-search.ts        # BM25 + Vector hybrid
│   │       │   ├── bm25-engine.ts
│   │       │   └── vector-engine.ts
│   │       ├── rerank/
│   │       │   ├── cross-encoder.ts
│   │       │   └── diversity-ranker.ts
│   │       ├── cache/
│   │       │   ├── session-cache.ts        # Session-based caching
│   │       │   └── result-cache.ts
│   │       ├── dedup/
│   │       │   ├── url-dedup.ts
│   │       │   └── content-dedup.ts
│   │       └── observability/
│   │           ├── logging.ts
│   │           └── metrics.ts
│   │   └── tests/
│   │
│   └── 🔷 web-worker/
│       ├── Dockerfile
│       ├── docker-compose.yml
│       ├── .env.example
│       ├── tsconfig.json
│       ├── package.json
│       ├── README.md
│       └── src/
│           ├── index.ts
│           ├── config.ts
│           ├── fetch/
│           │   ├── http-client.ts          # HTTP fetching with retries
│           │   ├── ssrf-guard.ts           # SSRF protection
│           │   └── robots-checker.ts       # robots.txt compliance
│           ├── extract/
│           │   ├── dom-parser.ts           # HTML parsing
│           │   ├── content-extractor.ts    # Main content extraction
│           │   ├── table-extractor.ts      # Table structure extraction
│           │   └── metadata-extractor.ts   # Title, author, date, etc.
│           ├── sanitize/
│           │   ├── html-sanitizer.ts       # XSS protection
│           │   ├── pii-redactor.ts         # PII redaction
│           │   └── injection-filter.ts     # Prompt injection filtering
│           ├── sandbox/
│           │   ├── action-sandbox.ts       # Action execution sandbox
│           │   ├── allowlist-manager.ts    # Tool/domain allowlisting
│           │   └── approval-handler.ts     # Human approval workflow
│           ├── headless/
│           │   ├── browser-pool.ts         # Playwright/Puppeteer pool
│           │   ├── screenshot-engine.ts    # Screenshot capture
│           │   └── interaction-handler.ts  # Click, type, fill, submit
│           └── observability/
│               ├── logging.ts
│               └── metrics.ts
│       └── tests/
│
├── 📁 apps/
│   │
│   ├── extension/
│   │   ├── manifest.json
│   │   ├── package.json
│   │   ├── src/
│   │   │   ├── popup.html / popup.ts
│   │   │   ├── background.ts               # Service worker
│   │   │   ├── content-script.ts           # Page context capture
│   │   │   └── api-client.ts               # Backend API calls
│   │   └── tests/
│   │
│   └── web-ui/
│       ├── package.json
│       ├── vite.config.ts / next.config.js # Build config
│       ├── public/
│       ├── src/
│       │   ├── pages/
│       │   │   ├── home.tsx
│       │   │   ├── chat.tsx
│       │   │   └── results.tsx
│       │   ├── components/
│       │   │   ├── query-input.tsx
│       │   │   ├── answer-display.tsx
│       │   │   ├── citations.tsx
│       │   │   └── progress.tsx
│       │   ├── api-client.ts
│       │   └── styles/
│       └── tests/
│
├── 📁 infra/
│   ├── k8s/
│   │   ├── namespace.yaml
│   │   ├── stage1-deployment.yaml
│   │   ├── stage2-deployment.yaml
│   │   ├── stage3-deployment.yaml
│   │   ├── stage4-deployment.yaml
│   │   ├── stage5-deployment.yaml
│   │   ├── stage6-deployment.yaml
│   │   ├── model-gateway-deployment.yaml
│   │   ├── retrieval-deployment.yaml
│   │   ├── web-worker-deployment.yaml
│   │   ├── services/
│   │   │   └── *.yaml
│   │   ├── configmaps/
│   │   │   └── *.yaml
│   │   ├── secrets/
│   │   │   └── *.yaml
│   │   └── ingress/
│   │       └── *.yaml
│   ├── terraform/
│   │   ├── main.tf
│   │   ├── variables.tf
│   │   ├── outputs.tf
│   │   ├── redis.tf                       # Redis infrastructure
│   │   ├── postgres.tf                    # PostgreSQL for artifacts
│   │   ├── kubernetes.tf                  # K8s cluster
│   │   └── monitoring.tf                  # Prometheus, Grafana
│   ├── docker-compose.yml                 # Local development
│   ├── .env.example
│   └── ci/
│       ├── .github/
│       │   └── workflows/
│       │       ├── test.yml               # Unit & integration tests
│       │       ├── lint.yml               # Code quality checks
│       │       ├── security.yml           # Security scanning
│       │       └── deploy.yml             # CD pipeline
│       └── gitlab-ci.yml                  # Alternative CI/CD
│
├── 📄 .gitignore
├── 📄 .env.example
├── 📄 docker-compose.yml                  # Full stack local dev
├── 📄 package.json                        # Root monorepo config
├── 📄 pnpm-workspace.yaml                 # pnpm monorepo setup
├── 📄 tsconfig.json                       # Root TS config
├── 📄 README.md                           # Project overview
├── 📄 CONTRIBUTING.md                     # Development guidelines
├── 📄 LICENSE                             # License file
└── 📄 CHANGELOG.md                        # Version history
