### 6. IMPLEMENTATION DETAILS

#### 6.1. Signal Extraction

**Purpose:** Extract clear signals from TaskSpecV1 để quyết định mode[^2]

```python
def extract_signals(task_spec: TaskSpecV1) -> dict:
    """
    Extract decision signals from TaskSpecV1.
    
    Signals:
    - needs_research: bool
    - needs_action: bool
    - needs_state_first: bool
    - risk_level: str
    - has_citations_required: bool
    """
    signals = {}
    
    # Signal 1: needs_research
    signals['needs_research'] = (
        task_spec.intent in ["summarize", "explain"] or
        task_spec.artifact in ["brief", "answer", "compare"] or
        task_spec.scope == "web" or
        bool(re.search(r'so sánh|tổng hợp|nghiên cứu|research|compare|analyze', 
                       task_spec.context.normalized_text.lower()))
    )
    
    # Signal 2: needs_action
    signals['needs_action'] = (
        task_spec.intent == "act" or
        task_spec.action_level in ["Act-1", "Act-2"] or
        bool(re.search(r'mở|điền|submit|gửi|đặt|book|click|type', 
                       task_spec.context.normalized_text.lower()))
    )
    
    # Signal 3: needs_state_first
    signals['needs_state_first'] = (
        bool(re.search(r'đang dùng|hiện tại|current|existing|gói nào|plan nào', 
                       task_spec.context.normalized_text.lower()))
    )
    
    # Signal 4: risk_level
    signals['risk_level'] = task_spec.risk
    
    # Signal 5: has_citations_required
    signals['has_citations_required'] = (
        task_spec.intent in ["summarize", "explain"] or
        task_spec.scope == "web"
    )
    
    return signals
```


#### 6.2. Mode Selection

**Rule-based Routing**[^2]

```python
def select_mode(
    task_spec: TaskSpecV1,
    signals: dict
) -> tuple[ExecutionMode, list[str]]:
    """
    Select execution mode based on signals.
    
    Returns:
    - mode: ExecutionMode
    - reason_codes: list[str] (for explainability)
    """
    reason_codes = []
    
    # Case 1: Neither research nor action → error
    if not signals['needs_research'] and not signals['needs_action']:
        raise ValueError("TaskSpec has neither research nor action intent")
    
    # Case 2: Research-only
    if signals['needs_research'] and not signals['needs_action']:
        mode = "A"
        reason_codes.append("MODE_A:research_only")
        reason_codes.append("NO_ACTION_VERBS")
        return mode, reason_codes
    
    # Case 3: Action-only (no research needed)
    if signals['needs_action'] and not signals['needs_research']:
        # Check if state inspection needed
        if signals['needs_state_first']:
            mode = "D"
            reason_codes.append("MODE_D:action_then_research")
            reason_codes.append("STATE_INSPECTION_REQUIRED")
        else:
            mode = "B"
            reason_codes.append("MODE_B:action_only")
            reason_codes.append("NO_RESEARCH_NEEDED")
        return mode, reason_codes
    
    # Case 4: Both research and action
    if signals['needs_research'] and signals['needs_action']:
        # Check if state inspection needed first
        if signals['needs_state_first']:
            mode = "D"
            reason_codes.append("MODE_D:action_then_research")
            reason_codes.append("STATE_INSPECTION_FIRST")
        else:
            mode = "C"
            reason_codes.append("MODE_C:research_then_action")
            reason_codes.append("HAS_RESEARCH_AND_ACTION")
        return mode, reason_codes
    
    # Default (should not reach here)
    mode = "C"
    reason_codes.append("MODE_C:default")
    return mode, reason_codes
```


#### 6.3. Plan Generation (per Mode)

**Mode A: Research-only**[^2]

```python
def build_plan_mode_A(task_spec: TaskSpecV1) -> ExecutionPlan:
    """
    Build execution plan for Mode A (Research-only).
    
    Pipeline: RETRIEVE → FETCH_DATA → COMPUTE (no ACT)
    """
    plan_id = str(uuid.uuid4())
    
    # Generate sub-queries
    sub_queries = generate_sub_queries(task_spec)
    
    # Build steps
    steps = [
        Step(
            step_id="stp_r1",
            type="RETRIEVE",
            name="Retrieve evidence",
            config={"top_k": 8}
        ),
        Step(
            step_id="stp_f1",
            type="FETCH_DATA",
            name="Parse structured data",
            depends_on=["stp_r1"],
            optional=True
        ),
        Step(
            step_id="stp_c1",
            type="COMPUTE",
            name="Score & synthesize",
            depends_on=["stp_r1", "stp_f1"],
            optional=True
        )
    ]
    
    # Budget
    budget = Budget(
        max_time_ms=30000,  # 30s
        max_tool_calls=10,
        max_sources=8
    )
    
    # Verify criteria
    verify_criteria = VerifyCriteria(
        require_citations=True,
        ui_receipt_required=False
    )
    
    return ExecutionPlan(
        plan_id=plan_id,
        mode="A",
        steps=steps,
        sub_queries=sub_queries,
        budget=budget,
        policy_gates=[],
        verify_criteria=verify_criteria,
        reason_codes=["MODE_A:research_only"]
    )
```

**Mode C: Research → Action**[^2]

```python
def build_plan_mode_C(task_spec: TaskSpecV1) -> ExecutionPlan:
    """
    Build execution plan for Mode C (Research → Action).
    
    Pipeline: RETRIEVE → FETCH_DATA → COMPUTE → ACT
    """
    plan_id = str(uuid.uuid4())
    
    # Generate sub-queries
    sub_queries = generate_sub_queries(task_spec)
    
    # Build steps
    steps = [
        Step(
            step_id="stp_r1",
            type="RETRIEVE",
            name="Retrieve evidence",
            config={"top_k": 8}
        ),
        Step(
            step_id="stp_f1",
            type="FETCH_DATA",
            name="Parse packages/data",
            depends_on=["stp_r1"],
            optional=True
        ),
        Step(
            step_id="stp_c1",
            type="COMPUTE",
            name="Recommend & prepare payload",
            depends_on=["stp_r1", "stp_f1"],
            optional=True
        ),
        Step(
            step_id="stp_a1",
            type="ACT",
            name="Perform action (with confirmation)",
            depends_on=["stp_c1"],
            policy_gate_id="gate_action"
        )
    ]
    
    # Policy gates
    policy_gates = [
        PolicyGate(
            gate_id="gate_action",
            requires_user_confirm=True,
            reason=f"About to perform {task_spec.action_level} action",
            blocked_actions=["submit", "purchase", "delete"] if task_spec.action_level == "Act-2" else []
        )
    ]
    
    # Budget
    budget = Budget(
        max_time_ms=60000,  # 60s
        max_tool_calls=18,
        max_sources=8
    )
    
    # Verify criteria
    verify_criteria = VerifyCriteria(
        require_citations=True,
        ui_receipt_required=True
    )
    
    return ExecutionPlan(
        plan_id=plan_id,
        mode="C",
        steps=steps,
        sub_queries=sub_queries,
        budget=budget,
        policy_gates=policy_gates,
        verify_criteria=verify_criteria,
        reason_codes=["MODE_C:research_then_action"]
    )
```


#### 6.4. Sub-Query Generation

**Purpose:** Tạo sub-queries cho research step[^2]

```python
def generate_sub_queries(task_spec: TaskSpecV1) -> list[SubQuery]:
    """
    Generate sub-queries based on TaskSpec.
    
    Rules:
    - Official sources: terms, conditions, specs
    - News sources: latest updates, reviews
    - Data sources: pricing, availability
    - Background: general context
    """
    sub_queries = []
    
    text = task_spec.context.normalized_text
    detected_lang = task_spec.context.language
    
    # Extract main topic
    # Simple heuristic: first noun phrase or named entity
    topic = extract_main_topic(text)
    
    # Generate queries based on artifact type
    if task_spec.artifact == "compare":
        # Comparison queries
        sub_queries.append(SubQuery(
            q=f"{topic} comparison features pricing",
            role="data"
        ))
        sub_queries.append(SubQuery(
            q=f"{topic} official specifications",
            role="official"
        ))
        sub_queries.append(SubQuery(
            q=f"{topic} user reviews",
            role="news"
        ))
    
    elif task_spec.artifact == "brief":
        # Summary queries
        sub_queries.append(SubQuery(
            q=f"{topic} overview summary",
            role="background"
        ))
        sub_queries.append(SubQuery(
            q=f"{topic} key highlights",
            role="official"
        ))
    
    elif task_spec.artifact == "answer":
        # Q&A queries
        sub_queries.append(SubQuery(
            q=text,  # Use original query
            role="background"
        ))
        sub_queries.append(SubQuery(
            q=f"{topic} detailed explanation",
            role="official"
        ))
    
    # Add budget-specific query if present
    if task_spec.entities.get('budget'):
        budget_text = task_spec.entities['budget'].get('original_text', '')
        sub_queries.append(SubQuery(
            q=f"{topic} under {budget_text} price comparison",
            role="data"
        ))
    
    return sub_queries[:5]  # Limit to 5 sub-queries
```


***

