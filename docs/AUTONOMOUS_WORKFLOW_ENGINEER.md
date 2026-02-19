# Autonomous Workflow Engineer (AWE) - Design Document

**Version:** 2.0.0  
**Status:** Draft  
**Date:** February 5, 2026  
**Author:** a2n.io Engineering  

---

## Table of Contents

1. [Executive Summary](#1-executive-summary)
2. [Two Creation Paths](#2-two-creation-paths)
3. [Architecture Overview](#3-architecture-overview)
4. [Database Schema Design](#4-database-schema-design)
5. [Backend Service Design](#5-backend-service-design)
6. [AI Assistant Multi-Stage Flow](#6-ai-assistant-multi-stage-flow)
7. [UI/UX Design](#7-uiux-design)
8. [API Contracts](#8-api-contracts)
9. [Niche Strategy](#9-niche-strategy)
10. [Implementation Phases](#10-implementation-phases)
11. [Security Considerations](#11-security-considerations)

---

## 1. Executive Summary

### Vision

Transform a2n.io from a **tool** (like n8n/Zapier) into an **automation employee** that can:
- Understand business intent
- Design automations
- Build workflows
- Test and deploy
- Monitor and self-heal

### Key Differentiator

```
n8n / Zapier: User builds workflows manually
a2n.io AWE:   AI builds, tests, deploys, and maintains workflows autonomously
```

### Business Value

- **Replace freelancers**: $50-200/hr automation consultants
- **Reduce time-to-automation**: Days → Minutes
- **Self-healing**: No maintenance contracts needed
- **Enterprise trust**: Approval gates, audit logs, rollbacks

---

## 2. Two Creation Paths

### Critical Decision: Dual Entry Points

a2n.io serves **two distinct user personas**. Each needs a tailored experience:

```
┌─────────────────────────────────────────────────────────────────────────┐
│                         WORKFLOW CREATION                                │
├─────────────────────────────────┬───────────────────────────────────────┤
│      🛠️ MANUAL PATH             │      🤖 ASSISTANT PATH                 │
│      (Tech-Savvy Users)         │      (Non-Tech Users)                 │
├─────────────────────────────────┼───────────────────────────────────────┤
│  Entry: /workflows/new          │  Entry: /automations/create           │
│  Interface: Canvas + Nodes      │  Interface: Chat-First                │
│  Control: Full                  │  Control: Guided                      │
│  Speed: Fast for experts        │  Speed: Fast for anyone               │
│  Learning: Required             │  Learning: None                       │
├─────────────────────────────────┼───────────────────────────────────────┤
│  User Profile:                  │  User Profile:                        │
│  • Developers                   │  • Marketing Managers                 │
│  • Technical PMs                │  • Sales Reps                         │
│  • Automation Engineers         │  • Small Business Owners              │
│  • Power Users                  │  • Non-technical Founders             │
└─────────────────────────────────┴───────────────────────────────────────┘
```

### Path Selection UX

```
┌─────────────────────────────────────────────────────────────────────────┐
│  Create New Automation                                                   │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│  How would you like to create your automation?                          │
│                                                                          │
│  ┌───────────────────────────────┐  ┌───────────────────────────────┐   │
│  │                               │  │                               │   │
│  │  🤖 AI Assistant              │  │  🛠️ Manual Builder            │   │
│  │                               │  │                               │   │
│  │  Describe what you want       │  │  Drag & drop nodes            │   │
│  │  in plain English.            │  │  Full control.                │   │
│  │  AI builds it for you.        │  │  For technical users.         │   │
│  │                               │  │                               │   │
│  │  ✓ No technical knowledge     │  │  ✓ Complete flexibility       │   │
│  │  ✓ Guided step-by-step        │  │  ✓ Advanced configurations    │   │
│  │  ✓ AI tests & monitors        │  │  ✓ Custom node logic          │   │
│  │                               │  │                               │   │
│  │         [ Start Chat ]        │  │       [ Open Canvas ]         │   │
│  │                               │  │                               │   │
│  └───────────────────────────────┘  └───────────────────────────────┘   │
│                                                                          │
│  💡 Not sure? Start with AI Assistant - you can always switch later.   │
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘
```

### Flow Comparison

```
MANUAL PATH (Existing):
────────────────────────────────────────────────────────────────────────────
User opens canvas → Drags nodes → Configures each → Connects → Tests → Deploy
                           ↑
                    Full control, full responsibility

ASSISTANT PATH (New AWE):
────────────────────────────────────────────────────────────────────────────
User describes intent → Discovery Chat → AI Plans → User Approves → AI Builds
                                                                        ↓
                                              AI Tests → User Reviews → Deploy
                                                                        ↓
                                                              AI Monitors → Auto-Fix
```

---

## 3. Architecture Overview

### Current Architecture (Manual Path - Keep As-Is)

```
┌─────────┐     ┌──────────────┐     ┌───────────┐
│  User   │────▶│  Workflow    │────▶│  Execute  │
│  (UI)   │     │  Builder     │     │  Engine   │
└─────────┘     └──────────────┘     └───────────┘
```

**Files:** `WorkflowEditor.tsx`, existing workflow APIs - **NO CHANGES NEEDED**

### New Architecture (Assistant Path - AWE)

```
┌─────────┐     ┌──────────────────────────────────────────────────────────┐
│  User   │────▶│              AI Assistant Orchestrator                   │
│  (Chat) │     │                                                          │
└─────────┘     │  ┌─────────────────────────────────────────────────────┐ │
                │  │                OPENROUTER GATEWAY                   │ │
                │  │  (Model-agnostic: GPT-4, Claude, Gemini, Llama...)  │ │
                │  └─────────────────────────────────────────────────────┘ │
                │                          │                               │
                │  ┌──────────┐ ┌──────────┐ ┌──────────────────┐         │
                │  │ Discovery│▶│ Planning │▶│     Builder      │         │
                │  │  Agent   │ │  Agent   │ │      Agent       │         │
                │  └──────────┘ └──────────┘ └──────────────────┘         │
                │  ┌──────────┐ ┌──────────┐ ┌──────────────────┐         │
                │  │ Testing  │◀│ Monitor  │◀│   Maintenance    │         │
                │  │  Agent   │ │  Agent   │ │      Agent       │         │
                │  └──────────┘ └──────────┘ └──────────────────┘         │
                └──────────────────────────────────────────────────────────┘
                              │
                              ▼
                ┌──────────────────────────────────────────────────────────┐
                │              Workflow Compiler                            │
                │  - Validate feasibility                                  │
                │  - Insert error handlers, retries, logging               │
                │  - Generate executable DAG                               │
                └──────────────────────────────────────────────────────────┘
                              │
                              ▼
                ┌──────────────────────────────────────────────────────────┐
                │            Execution Engine (Existing)                    │
                └──────────────────────────────────────────────────────────┘
```

### OpenRouter Integration (Key Design Decision)

Why **OpenRouter** as the AI gateway:
- **Model flexibility**: Switch between GPT-4, Claude, Gemini, Llama without code changes
- **Cost optimization**: Use cheaper models for simple tasks, powerful models for complex
- **Fallback chains**: If one model fails, auto-retry with another
- **Usage tracking**: Centralized billing and analytics

```typescript
// openrouter.config.ts
export const AGENT_MODEL_CONFIG = {
  discovery: {
    default: 'anthropic/claude-3-sonnet',      // Good at conversation
    fallback: 'openai/gpt-4-turbo',
  },
  planning: {
    default: 'anthropic/claude-3-opus',        // Best at complex reasoning
    fallback: 'openai/gpt-4-turbo',
  },
  builder: {
    default: 'openai/gpt-4-turbo',             // Best at structured output
    fallback: 'anthropic/claude-3-sonnet',
  },
  testing: {
    default: 'openai/gpt-4-turbo',
    fallback: 'anthropic/claude-3-sonnet',
  },
  maintenance: {
    default: 'anthropic/claude-3-sonnet',
    fallback: 'openai/gpt-3.5-turbo',          // Cheaper for simple fixes
  }
};
```

---

## 4. Database Schema Design

### 4.1 New Tables

#### `automation_intents`

Captures user's raw business intent.

```sql
CREATE TABLE automation_intents (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID NOT NULL REFERENCES app_auth.users(id) ON DELETE CASCADE,
    workspace_id UUID NOT NULL REFERENCES workspaces(id) ON DELETE CASCADE,
    
    -- User Input
    raw_intent TEXT NOT NULL,                    -- "When I get a Stripe payment, add to Sheets and send Slack"
    structured_intent JSONB DEFAULT '{}',        -- Parsed: { trigger, actions, conditions }
    
    -- Context gathered by Discovery Agent
    discovered_context JSONB DEFAULT '{}',       -- { apps_used, data_sources, frequency, volume }
    clarification_history JSONB DEFAULT '[]',    -- Conversation for missing info
    
    -- Status
    status VARCHAR(50) DEFAULT 'pending',        -- pending, analyzing, clarified, ready, converted
    
    -- Metadata
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_intents_user ON automation_intents(user_id);
CREATE INDEX idx_intents_workspace ON automation_intents(workspace_id);
CREATE INDEX idx_intents_status ON automation_intents(status);
```

#### `automation_plans`

AI-generated automation design (before building).

```sql
CREATE TABLE automation_plans (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    intent_id UUID NOT NULL REFERENCES automation_intents(id) ON DELETE CASCADE,
    user_id UUID NOT NULL REFERENCES app_auth.users(id) ON DELETE CASCADE,
    
    -- Plan Details
    plan_version INT DEFAULT 1,
    plan_summary TEXT NOT NULL,                  -- Human-readable summary
    plan_structure JSONB NOT NULL,               -- { steps: [], data_flow: [], error_handling: [] }
    
    -- Node Mapping (what nodes will be used)
    proposed_nodes JSONB NOT NULL,               -- [{ node_type, config_template, reason }]
    proposed_connections JSONB NOT NULL,         -- [{ from, to, data_mapping }]
    
    -- Feasibility
    feasibility_score DECIMAL(3,2) DEFAULT 0,    -- 0.00 - 1.00
    feasibility_issues JSONB DEFAULT '[]',       -- [{ issue, severity, suggestion }]
    
    -- Approval
    requires_approval BOOLEAN DEFAULT true,
    approved_by UUID REFERENCES app_auth.users(id),
    approved_at TIMESTAMP,
    rejection_reason TEXT,
    
    -- Status
    status VARCHAR(50) DEFAULT 'draft',          -- draft, pending_approval, approved, rejected, superseded
    
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_plans_intent ON automation_plans(intent_id);
CREATE INDEX idx_plans_status ON automation_plans(status);
```

#### `automation_lifecycles`

Tracks the full lifecycle of an automation.

```sql
CREATE TYPE automation_lifecycle_state AS ENUM (
    'draft',
    'discovery',
    'planning',
    'building',
    'testing',
    'pending_deployment',
    'deployed',
    'monitoring',
    'maintenance',
    'paused',
    'failed',
    'archived'
);

CREATE TABLE automation_lifecycles (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    intent_id UUID REFERENCES automation_intents(id),
    plan_id UUID REFERENCES automation_plans(id),
    workflow_id UUID REFERENCES workflows(id),
    user_id UUID NOT NULL REFERENCES app_auth.users(id) ON DELETE CASCADE,
    workspace_id UUID NOT NULL REFERENCES workspaces(id) ON DELETE CASCADE,
    
    -- Current State
    current_state automation_lifecycle_state DEFAULT 'draft',
    state_history JSONB DEFAULT '[]',            -- [{ state, entered_at, exited_at, reason }]
    
    -- Agent Assignments
    active_agents JSONB DEFAULT '[]',            -- [{ agent_type, started_at, status }]
    
    -- Health
    health_status VARCHAR(50) DEFAULT 'unknown', -- healthy, degraded, failing, unknown
    last_health_check TIMESTAMP,
    
    -- Metrics
    total_executions INT DEFAULT 0,
    successful_executions INT DEFAULT 0,
    failed_executions INT DEFAULT 0,
    auto_fixes_applied INT DEFAULT 0,
    
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_lifecycle_workflow ON automation_lifecycles(workflow_id);
CREATE INDEX idx_lifecycle_state ON automation_lifecycles(current_state);
CREATE INDEX idx_lifecycle_health ON automation_lifecycles(health_status);
```

#### `automation_test_runs`

Test results for AI-built workflows.

```sql
CREATE TABLE automation_test_runs (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    lifecycle_id UUID NOT NULL REFERENCES automation_lifecycles(id) ON DELETE CASCADE,
    workflow_id UUID NOT NULL REFERENCES workflows(id) ON DELETE CASCADE,
    
    -- Test Configuration
    test_type VARCHAR(50) NOT NULL,              -- dry_run, mock_apis, full_test, edge_cases
    test_data JSONB DEFAULT '{}',                -- Sample input data
    mock_responses JSONB DEFAULT '{}',           -- Mocked external API responses
    
    -- Results
    status VARCHAR(50) DEFAULT 'pending',        -- pending, running, passed, failed, error
    confidence_score DECIMAL(3,2),               -- 0.00 - 1.00
    
    -- Execution Details
    steps_executed INT DEFAULT 0,
    steps_passed INT DEFAULT 0,
    steps_failed INT DEFAULT 0,
    step_results JSONB DEFAULT '[]',             -- [{ node_id, status, input, output, error }]
    
    -- Issues Found
    issues JSONB DEFAULT '[]',                   -- [{ type, node_id, message, severity, fix_suggestion }]
    
    -- Timing
    started_at TIMESTAMP,
    finished_at TIMESTAMP,
    duration_ms INT,
    
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_test_runs_lifecycle ON automation_test_runs(lifecycle_id);
CREATE INDEX idx_test_runs_workflow ON automation_test_runs(workflow_id);
CREATE INDEX idx_test_runs_status ON automation_test_runs(status);
```

#### `automation_fixes`

Auto-fix history.

```sql
CREATE TABLE automation_fixes (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    lifecycle_id UUID NOT NULL REFERENCES automation_lifecycles(id) ON DELETE CASCADE,
    workflow_id UUID NOT NULL REFERENCES workflows(id) ON DELETE CASCADE,
    
    -- Issue Details
    issue_type VARCHAR(100) NOT NULL,            -- schema_change, auth_expired, field_mapping, rate_limit
    issue_description TEXT NOT NULL,
    issue_severity VARCHAR(50) DEFAULT 'medium', -- low, medium, high, critical
    detected_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    
    -- Fix Details
    fix_type VARCHAR(50) NOT NULL,               -- auto, suggested, manual
    fix_description TEXT NOT NULL,
    fix_changes JSONB NOT NULL,                  -- { before: {}, after: {}, nodes_affected: [] }
    
    -- Approval (for suggested fixes)
    requires_approval BOOLEAN DEFAULT false,
    approved_by UUID REFERENCES app_auth.users(id),
    approved_at TIMESTAMP,
    rejected_at TIMESTAMP,
    rejection_reason TEXT,
    
    -- Status
    status VARCHAR(50) DEFAULT 'pending',        -- pending, applied, rejected, failed
    applied_at TIMESTAMP,
    
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_fixes_lifecycle ON automation_fixes(lifecycle_id);
CREATE INDEX idx_fixes_status ON automation_fixes(status);
CREATE INDEX idx_fixes_type ON automation_fixes(issue_type);
```

#### `node_capabilities`

Registry of what each node can do (for AI).

```sql
CREATE TABLE node_capabilities (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    node_type_id UUID NOT NULL REFERENCES node_types(id) ON DELETE CASCADE,
    
    -- Capabilities
    capabilities JSONB NOT NULL,                 -- ["send_email", "read_data", "write_data", ...]
    limitations JSONB DEFAULT '[]',              -- ["max_1000_records", "no_binary_files", ...]
    
    -- Input/Output Contracts
    required_inputs JSONB NOT NULL,              -- [{ name, type, description, example }]
    optional_inputs JSONB DEFAULT '[]',
    output_schema JSONB NOT NULL,                -- { type: "object", properties: {...} }
    
    -- Error Handling
    common_errors JSONB DEFAULT '[]',            -- [{ code, cause, solution }]
    retry_policy JSONB DEFAULT '{}',             -- { max_retries, backoff, retryable_errors }
    
    -- Rate Limits
    rate_limits JSONB DEFAULT '{}',              -- { requests_per_minute, daily_limit, etc }
    
    -- Dependencies
    requires_auth BOOLEAN DEFAULT false,
    auth_types JSONB DEFAULT '[]',               -- ["oauth2", "api_key", "basic"]
    external_dependencies JSONB DEFAULT '[]',    -- ["stripe_api", "google_sheets_api"]
    
    -- AI Hints
    use_cases JSONB DEFAULT '[]',                -- ["payment_processing", "data_sync", ...]
    best_practices JSONB DEFAULT '[]',           -- Tips for AI to use this node well
    
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE UNIQUE INDEX idx_capabilities_node_type ON node_capabilities(node_type_id);
```

#### `automation_knowledge`

Persistent knowledge for AI agents.

```sql
CREATE TABLE automation_knowledge (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID REFERENCES app_auth.users(id) ON DELETE CASCADE,  -- NULL = global
    workspace_id UUID REFERENCES workspaces(id) ON DELETE CASCADE,
    
    -- Knowledge Type
    knowledge_type VARCHAR(100) NOT NULL,        -- business_rule, approved_pattern, fix_history, user_preference
    
    -- Content
    title VARCHAR(255) NOT NULL,
    content JSONB NOT NULL,
    
    -- Context
    related_intents TEXT[],                      -- Keywords for matching
    related_node_types UUID[],                   -- Relevant nodes
    
    -- Validation
    validated BOOLEAN DEFAULT false,
    validated_by UUID REFERENCES app_auth.users(id),
    
    -- Usage Stats
    times_used INT DEFAULT 0,
    last_used_at TIMESTAMP,
    
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_knowledge_type ON automation_knowledge(knowledge_type);
CREATE INDEX idx_knowledge_user ON automation_knowledge(user_id);
CREATE INDEX idx_knowledge_workspace ON automation_knowledge(workspace_id);
```

#### `agent_sessions`

Track AI agent execution.

```sql
CREATE TABLE agent_sessions (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    lifecycle_id UUID REFERENCES automation_lifecycles(id) ON DELETE CASCADE,
    
    -- Agent Info
    agent_type VARCHAR(100) NOT NULL,            -- discovery, planning, builder, testing, monitoring, maintenance
    agent_model VARCHAR(100),                    -- gpt-4, claude-3, etc
    
    -- Execution
    status VARCHAR(50) DEFAULT 'running',        -- running, completed, failed, cancelled
    started_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    finished_at TIMESTAMP,
    
    -- Input/Output
    input_context JSONB NOT NULL,
    output_result JSONB,
    
    -- Reasoning (for explainability)
    reasoning_steps JSONB DEFAULT '[]',          -- [{ step, thought, action, result }]
    
    -- Metrics
    tokens_used INT DEFAULT 0,
    cost_usd DECIMAL(10,6) DEFAULT 0,
    duration_ms INT,
    
    -- Errors
    error_message TEXT,
    
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_agent_sessions_lifecycle ON agent_sessions(lifecycle_id);
CREATE INDEX idx_agent_sessions_type ON agent_sessions(agent_type);
CREATE INDEX idx_agent_sessions_status ON agent_sessions(status);
```

### 3.2 Schema Diagram

```
┌─────────────────────┐
│ automation_intents  │
│   - raw_intent      │
│   - discovered_ctx  │
└─────────┬───────────┘
          │ 1:N
          ▼
┌─────────────────────┐
│  automation_plans   │
│   - plan_structure  │
│   - proposed_nodes  │
│   - feasibility     │
└─────────┬───────────┘
          │ 1:1
          ▼
┌─────────────────────┐      ┌─────────────────────┐
│automation_lifecycles│◀────▶│     workflows       │
│   - current_state   │      │   (existing)        │
│   - health_status   │      └─────────────────────┘
└─────────┬───────────┘
          │ 1:N
          ├─────────────────────────┬─────────────────────────┐
          ▼                         ▼                         ▼
┌─────────────────────┐  ┌─────────────────────┐  ┌─────────────────────┐
│automation_test_runs │  │  automation_fixes   │  │   agent_sessions    │
│   - test_results    │  │   - fix_changes     │  │   - reasoning       │
│   - confidence      │  │   - auto/manual     │  │   - tokens_used     │
└─────────────────────┘  └─────────────────────┘  └─────────────────────┘
```

---

## 5. Backend Service Design

### 5.1 New Services

#### Service: `automation-orchestrator`

**Location:** `src/automation-orchestrator/`

**Responsibilities:**
- Manage automation lifecycle state machine
- Coordinate agents
- Handle approvals
- Trigger transitions

```typescript
// automation-orchestrator.service.ts

@Injectable()
export class AutomationOrchestratorService {
  
  // State Machine
  async transitionState(
    lifecycleId: string, 
    targetState: AutomationLifecycleState,
    reason?: string
  ): Promise<AutomationLifecycle>;
  
  // Intent Processing
  async processIntent(
    userId: string,
    workspaceId: string,
    intent: CreateIntentDto
  ): Promise<AutomationIntent>;
  
  // Agent Coordination
  async runDiscoveryAgent(intentId: string): Promise<DiscoveryResult>;
  async runPlanningAgent(intentId: string): Promise<AutomationPlan>;
  async runBuilderAgent(planId: string): Promise<Workflow>;
  async runTestingAgent(lifecycleId: string): Promise<TestRunResult>;
  
  // Approvals
  async approvePlan(planId: string, userId: string): Promise<void>;
  async rejectPlan(planId: string, userId: string, reason: string): Promise<void>;
  async approveDeployment(lifecycleId: string, userId: string): Promise<void>;
  
  // Monitoring
  async checkHealth(lifecycleId: string): Promise<HealthStatus>;
  async applyAutoFix(fixId: string): Promise<void>;
}
```

#### Service: `node-registry`

**Location:** `src/node-registry/`

**Responsibilities:**
- Store node capabilities
- Provide AI-friendly node descriptions
- Track dependencies

```typescript
// node-registry.service.ts

@Injectable()
export class NodeRegistryService {
  
  // Query Capabilities
  async getNodeCapabilities(nodeTypeId: string): Promise<NodeCapabilities>;
  async findNodesForCapability(capability: string): Promise<NodeType[]>;
  async findNodesForUseCase(useCase: string): Promise<NodeType[]>;
  
  // AI Context
  async getNodeContextForAI(nodeTypeId: string): Promise<AINodeContext>;
  async getAllNodesForAI(): Promise<AINodeContext[]>;
  
  // Validation
  async validateNodeConfig(
    nodeTypeId: string, 
    config: Record<string, any>
  ): Promise<ValidationResult>;
  
  // Dependencies
  async getRequiredAuth(nodeTypeId: string): Promise<AuthRequirement[]>;
  async checkRateLimits(nodeTypeId: string, userId: string): Promise<RateLimitStatus>;
}
```

#### Service: `workflow-compiler`

**Location:** `src/workflow-compiler/`

**Responsibilities:**
- Convert plans to executable workflows
- Insert error handlers
- Validate feasibility

```typescript
// workflow-compiler.service.ts

@Injectable()
export class WorkflowCompilerService {
  
  // Compilation
  async compilePlan(plan: AutomationPlan): Promise<CompiledWorkflow>;
  
  // Validation
  async validateFeasibility(plan: AutomationPlan): Promise<FeasibilityResult>;
  async validateDataFlow(nodes: WorkflowNode[]): Promise<DataFlowResult>;
  
  // Enhancements
  async injectErrorHandlers(workflow: Workflow): Promise<Workflow>;
  async injectRetryLogic(workflow: Workflow): Promise<Workflow>;
  async injectLogging(workflow: Workflow): Promise<Workflow>;
  
  // Optimization
  async optimizeWorkflow(workflow: Workflow): Promise<Workflow>;
}
```

#### Service: `workflow-test-engine`

**Location:** `src/workflow-test-engine/`

**Responsibilities:**
- Dry-run workflows
- Mock external APIs
- Generate test reports

```typescript
// workflow-test-engine.service.ts

@Injectable()
export class WorkflowTestEngineService {
  
  // Test Execution
  async runDryTest(workflowId: string, testData: any): Promise<TestRunResult>;
  async runMockedTest(workflowId: string, mocks: MockConfig): Promise<TestRunResult>;
  async runEdgeCaseTests(workflowId: string): Promise<TestRunResult[]>;
  
  // Mock Generation
  async generateMocks(workflowId: string): Promise<MockConfig>;
  async generateTestData(workflowId: string): Promise<any>;
  
  // Analysis
  async analyzeTestResults(results: TestRunResult[]): Promise<TestAnalysis>;
  async suggestFixes(issues: TestIssue[]): Promise<FixSuggestion[]>;
}
```

#### Service: `automation-monitor`

**Location:** `src/automation-monitor/`

**Responsibilities:**
- Track execution health
- Detect issues
- Trigger auto-fixes

```typescript
// automation-monitor.service.ts

@Injectable()
export class AutomationMonitorService {
  
  // Health Checks
  async checkAutomationHealth(lifecycleId: string): Promise<HealthCheck>;
  async runScheduledHealthChecks(): Promise<void>;
  
  // Issue Detection
  async detectIssues(lifecycleId: string): Promise<DetectedIssue[]>;
  async analyzeFailurePatterns(lifecycleId: string): Promise<FailurePattern[]>;
  
  // Auto-Healing
  async proposeAutoFix(issue: DetectedIssue): Promise<AutoFix>;
  async applyAutoFix(fixId: string): Promise<FixResult>;
  
  // Alerts
  async notifyUser(lifecycleId: string, alert: Alert): Promise<void>;
}
```

### 4.2 AI Agent Architecture

#### Agent Base Class

```typescript
// agents/base-agent.ts

export abstract class BaseAutomationAgent {
  protected model: string;
  protected maxTokens: number;
  
  constructor(
    protected readonly openRouterService: OpenRouterService,
    protected readonly knowledgeService: AutomationKnowledgeService,
  ) {}
  
  abstract getSystemPrompt(): string;
  abstract getOutputSchema(): JSONSchema;
  
  async execute(context: AgentContext): Promise<AgentResult> {
    // 1. Load relevant knowledge
    const knowledge = await this.knowledgeService.getRelevant(context);
    
    // 2. Build prompt
    const prompt = this.buildPrompt(context, knowledge);
    
    // 3. Execute with structured output
    const result = await this.openRouterService.chat({
      model: this.model,
      messages: [
        { role: 'system', content: this.getSystemPrompt() },
        { role: 'user', content: prompt }
      ],
      response_format: { type: 'json_schema', json_schema: this.getOutputSchema() }
    });
    
    // 4. Log session
    await this.logSession(context, result);
    
    return result;
  }
}
```

#### Agent Implementations

```typescript
// agents/discovery-agent.ts
export class DiscoveryAgent extends BaseAutomationAgent {
  getSystemPrompt() {
    return `You are an automation discovery agent. Your job is to:
    1. Understand the user's business process
    2. Identify missing information
    3. Ask clarifying questions
    4. Extract structured requirements
    
    Be conversational but efficient. Ask only essential questions.`;
  }
}

// agents/planning-agent.ts
export class PlanningAgent extends BaseAutomationAgent {
  getSystemPrompt() {
    return `You are an automation planning agent. Your job is to:
    1. Design an automation workflow from requirements
    2. Select appropriate nodes from the registry
    3. Define data flow between nodes
    4. Identify potential failure points
    5. Propose error handling strategies
    
    Output a detailed, executable plan.`;
  }
}

// agents/builder-agent.ts
export class BuilderAgent extends BaseAutomationAgent {
  getSystemPrompt() {
    return `You are an automation builder agent. Your job is to:
    1. Convert a plan into actual workflow nodes
    2. Configure each node precisely
    3. Map data between nodes correctly
    4. Add error handlers and retries
    
    Your output must be valid workflow JSON.`;
  }
}
```

### 5.3 Module Structure

```
src/
├── automation-orchestrator/
│   ├── automation-orchestrator.module.ts
│   ├── automation-orchestrator.service.ts
│   ├── automation-orchestrator.controller.ts
│   ├── dto/
│   │   ├── create-intent.dto.ts
│   │   ├── approve-plan.dto.ts
│   │   └── transition-state.dto.ts
│   └── entities/
│       ├── automation-intent.entity.ts
│       ├── automation-plan.entity.ts
│       └── automation-lifecycle.entity.ts
│
├── agents/
│   ├── agents.module.ts
│   ├── base-agent.ts
│   ├── discovery-agent.ts
│   ├── planning-agent.ts
│   ├── builder-agent.ts
│   ├── testing-agent.ts
│   ├── monitoring-agent.ts
│   └── maintenance-agent.ts
│
├── openrouter/
│   ├── openrouter.module.ts
│   ├── openrouter.service.ts
│   ├── openrouter.config.ts
│   └── dto/
│       ├── chat-completion.dto.ts
│       └── model-config.dto.ts
│
├── node-registry/
│   ├── node-registry.module.ts
│   ├── node-registry.service.ts
│   └── entities/
│       └── node-capabilities.entity.ts
│
├── workflow-compiler/
│   ├── workflow-compiler.module.ts
│   ├── workflow-compiler.service.ts
│   └── compilers/
│       ├── feasibility-validator.ts
│       ├── error-handler-injector.ts
│       └── data-flow-validator.ts
│
├── workflow-test-engine/
│   ├── workflow-test-engine.module.ts
│   ├── workflow-test-engine.service.ts
│   ├── mock-generator.service.ts
│   └── entities/
│       └── automation-test-run.entity.ts
│
└── automation-monitor/
    ├── automation-monitor.module.ts
    ├── automation-monitor.service.ts
    ├── health-checker.service.ts
    ├── issue-detector.service.ts
    └── entities/
        └── automation-fix.entity.ts
```

---

## 6. AI Assistant Multi-Stage Flow (Core Innovation)

This is the **heart of AWE** - the multi-stage conversational flow that replaces human freelancers.

### 6.1 Stage Flow Diagram

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                        AI ASSISTANT CREATION FLOW                            │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  STAGE 1: INTENT CAPTURE                                                    │
│  ┌────────────────────────────────────────────────────────────────────┐    │
│  │  User: "When someone fills out my landing page form, I want to     │    │
│  │         add them to my email list and send a welcome sequence"     │    │
│  └────────────────────────────────────────────────────────────────────┘    │
│                                    │                                        │
│                                    ▼                                        │
│  STAGE 2: DISCOVERY (Conversational)                                       │
│  ┌────────────────────────────────────────────────────────────────────┐    │
│  │  🤖 AI: "Got it! A few quick questions:"                           │    │
│  │                                                                     │    │
│  │  1. What form tool are you using?                                  │    │
│  │     ○ Typeform  ○ Webflow  ○ Custom  ○ Other                       │    │
│  │                                                                     │    │
│  │  2. Where's your email list?                                       │    │
│  │     ○ Mailchimp  ○ ConvertKit  ○ ActiveCampaign  ○ Other          │    │
│  │                                                                     │    │
│  │  3. How many submissions per day do you expect?                    │    │
│  │     ○ 1-10  ○ 10-100  ○ 100-1000  ○ 1000+                         │    │
│  └────────────────────────────────────────────────────────────────────┘    │
│                                    │                                        │
│                                    ▼                                        │
│  STAGE 3: PLANNING (AI Shows Plan)                                         │
│  ┌────────────────────────────────────────────────────────────────────┐    │
│  │  🤖 AI: "Here's what I'll build for you:"                          │    │
│  │                                                                     │    │
│  │  ┌──────────┐    ┌──────────┐    ┌──────────┐                     │    │
│  │  │ Typeform │───▶│ Mailchimp│───▶│ Welcome  │                     │    │
│  │  │ Webhook  │    │ Add Sub  │    │ Email    │                     │    │
│  │  └──────────┘    └──────────┘    └──────────┘                     │    │
│  │                                                                     │    │
│  │  Features:                                                          │    │
│  │  ✓ Duplicate detection (won't add same email twice)               │    │
│  │  ✓ Error handling (retries if Mailchimp is slow)                  │    │
│  │  ✓ Logging (you can see every submission)                         │    │
│  │                                                                     │    │
│  │  Confidence: 96%                                                    │    │
│  │                                                                     │    │
│  │  [ Approve & Build ] [ Ask Question ] [ Change Something ]          │    │
│  └────────────────────────────────────────────────────────────────────┘    │
│                                    │                                        │
│                                    ▼                                        │
│  STAGE 4: BUILDING (Real-time Progress)                                    │
│  ┌────────────────────────────────────────────────────────────────────┐    │
│  │  🔧 Building your automation...                                     │    │
│  │                                                                     │    │
│  │  ✅ Created Typeform webhook trigger                                │    │
│  │  ✅ Added Mailchimp subscriber node                                 │    │
│  │  ✅ Configured welcome email sequence                               │    │
│  │  ✅ Added error handling                                            │    │
│  │  ⏳ Running validation tests...                                     │    │
│  └────────────────────────────────────────────────────────────────────┘    │
│                                    │                                        │
│                                    ▼                                        │
│  STAGE 5: TESTING (AI Validates)                                           │
│  ┌────────────────────────────────────────────────────────────────────┐    │
│  │  🧪 Testing your automation...                                      │    │
│  │                                                                     │    │
│  │  Test 1: Valid form submission ────────────────────────── ✅ Pass  │    │
│  │  Test 2: Duplicate email handling ─────────────────────── ✅ Pass  │    │
│  │  Test 3: Invalid email format ─────────────────────────── ✅ Pass  │    │
│  │  Test 4: Mailchimp connection error ───────────────────── ✅ Pass  │    │
│  │                                                                     │    │
│  │  All tests passed! Confidence: 98%                                  │    │
│  └────────────────────────────────────────────────────────────────────┘    │
│                                    │                                        │
│                                    ▼                                        │
│  STAGE 6: DEPLOYMENT APPROVAL                                              │
│  ┌────────────────────────────────────────────────────────────────────┐    │
│  │  🚀 Ready to deploy!                                                │    │
│  │                                                                     │    │
│  │  Your automation is ready to go live.                              │    │
│  │  It will start processing form submissions immediately.            │    │
│  │                                                                     │    │
│  │  [ Deploy Now ] [ Test One More Time ] [ Edit Manually ]           │    │
│  └────────────────────────────────────────────────────────────────────┘    │
│                                    │                                        │
│                                    ▼                                        │
│  STAGE 7: MONITORING (Ongoing)                                             │
│  ┌────────────────────────────────────────────────────────────────────┐    │
│  │  📊 Your automation is live!                                        │    │
│  │                                                                     │    │
│  │  Today: 12 runs • 100% success                                     │    │
│  │  This week: 84 runs • 99% success (1 auto-fixed)                   │    │
│  │                                                                     │    │
│  │  Recent: ✅ john@example.com added 2 min ago                       │    │
│  └────────────────────────────────────────────────────────────────────┘    │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 6.2 OpenRouter Agent Calls

Each stage uses specific OpenRouter calls:

```typescript
// Stage 2: Discovery Agent
const discoveryResponse = await openrouter.chat({
  model: 'anthropic/claude-3-sonnet',
  messages: [
    { role: 'system', content: DISCOVERY_PROMPT },
    { role: 'user', content: userIntent },
    ...conversationHistory
  ],
  response_format: {
    type: 'json_schema',
    json_schema: {
      name: 'discovery_response',
      schema: {
        type: 'object',
        properties: {
          message: { type: 'string' },
          questions: { type: 'array', items: { type: 'object' } },
          isComplete: { type: 'boolean' },
          extractedContext: { type: 'object' }
        }
      }
    }
  }
});

// Stage 3: Planning Agent
const planResponse = await openrouter.chat({
  model: 'anthropic/claude-3-opus',
  messages: [
    { role: 'system', content: PLANNING_PROMPT },
    { role: 'user', content: JSON.stringify(discoveredContext) }
  ],
  response_format: {
    type: 'json_schema',
    json_schema: {
      name: 'automation_plan',
      schema: {
        type: 'object',
        properties: {
          summary: { type: 'string' },
          nodes: { type: 'array' },
          connections: { type: 'array' },
          errorHandling: { type: 'array' },
          feasibilityScore: { type: 'number' },
          warnings: { type: 'array' }
        }
      }
    }
  }
});

// Stage 4: Builder Agent
const buildResponse = await openrouter.chat({
  model: 'openai/gpt-4-turbo',
  messages: [
    { role: 'system', content: BUILDER_PROMPT },
    { role: 'user', content: JSON.stringify(approvedPlan) }
  ],
  response_format: {
    type: 'json_schema',
    json_schema: {
      name: 'workflow_definition',
      schema: {
        type: 'object',
        properties: {
          nodes: { 
            type: 'array',
            items: {
              type: 'object',
              properties: {
                id: { type: 'string' },
                type: { type: 'string' },
                position: { type: 'object' },
                data: { type: 'object' }
              }
            }
          },
          edges: { type: 'array' }
        }
      }
    }
  }
});
```

### 6.3 Stage State Machine

```typescript
// automation-lifecycle.state.ts

export enum AssistantStage {
  INTENT_CAPTURE = 'intent_capture',
  DISCOVERY = 'discovery',
  DISCOVERY_COMPLETE = 'discovery_complete',
  PLANNING = 'planning',
  PLAN_REVIEW = 'plan_review',
  PLAN_APPROVED = 'plan_approved',
  BUILDING = 'building',
  BUILD_COMPLETE = 'build_complete',
  TESTING = 'testing',
  TEST_PASSED = 'test_passed',
  TEST_FAILED = 'test_failed',
  DEPLOYMENT_PENDING = 'deployment_pending',
  DEPLOYED = 'deployed',
  MONITORING = 'monitoring',
  MAINTENANCE = 'maintenance',
  PAUSED = 'paused',
  FAILED = 'failed'
}

// Valid transitions
export const STAGE_TRANSITIONS: Record<AssistantStage, AssistantStage[]> = {
  [AssistantStage.INTENT_CAPTURE]: [AssistantStage.DISCOVERY],
  [AssistantStage.DISCOVERY]: [AssistantStage.DISCOVERY, AssistantStage.DISCOVERY_COMPLETE],
  [AssistantStage.DISCOVERY_COMPLETE]: [AssistantStage.PLANNING],
  [AssistantStage.PLANNING]: [AssistantStage.PLAN_REVIEW],
  [AssistantStage.PLAN_REVIEW]: [AssistantStage.PLAN_APPROVED, AssistantStage.DISCOVERY],
  [AssistantStage.PLAN_APPROVED]: [AssistantStage.BUILDING],
  [AssistantStage.BUILDING]: [AssistantStage.BUILD_COMPLETE, AssistantStage.FAILED],
  [AssistantStage.BUILD_COMPLETE]: [AssistantStage.TESTING],
  [AssistantStage.TESTING]: [AssistantStage.TEST_PASSED, AssistantStage.TEST_FAILED],
  [AssistantStage.TEST_PASSED]: [AssistantStage.DEPLOYMENT_PENDING],
  [AssistantStage.TEST_FAILED]: [AssistantStage.BUILDING, AssistantStage.FAILED],
  [AssistantStage.DEPLOYMENT_PENDING]: [AssistantStage.DEPLOYED],
  [AssistantStage.DEPLOYED]: [AssistantStage.MONITORING],
  [AssistantStage.MONITORING]: [AssistantStage.MAINTENANCE, AssistantStage.PAUSED],
  [AssistantStage.MAINTENANCE]: [AssistantStage.MONITORING, AssistantStage.PAUSED],
  [AssistantStage.PAUSED]: [AssistantStage.MONITORING, AssistantStage.FAILED],
  [AssistantStage.FAILED]: [AssistantStage.DISCOVERY]  // Restart
};
```

---

## 7. UI/UX Design

### 7.1 Navigation Changes

```
Before:                          After:
─────────────────────────────    ─────────────────────────────
├── Dashboard                    ├── Dashboard
├── Workflows                    ├── Automations (NEW)
│   └── [workflow list]          │   ├── Active
├── Credentials                  │   ├── Building
├── Settings                     │   └── Failed
                                 ├── Create Automation (NEW)
                                 ├── Workflows (Advanced)
                                 ├── Credentials
                                 └── Settings
```

### 5.2 New Pages

#### Page: Create Automation (Chat-First)

**Route:** `/automations/create`

```
┌─────────────────────────────────────────────────────────────┐
│  🤖 What would you like to automate?                        │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ┌─────────────────────────────────────────────────────┐   │
│  │                                                     │   │
│  │  Describe your process in plain English...         │   │
│  │                                                     │   │
│  │  Example: "When a new order comes in on Shopify,  │   │
│  │  add the customer to my CRM, send a Slack         │   │
│  │  notification, and create a shipping label"       │   │
│  │                                                     │   │
│  └─────────────────────────────────────────────────────┘   │
│                                                             │
│  Optional: Add details                                      │
│  ┌────────────────┐ ┌────────────────┐ ┌────────────────┐  │
│  │ 📱 Apps Used   │ │ 📊 Volume      │ │ ⏰ Frequency   │  │
│  │ Shopify, Slack │ │ ~100/day       │ │ Real-time     │  │
│  └────────────────┘ └────────────────┘ └────────────────┘  │
│                                                             │
│                        [ 🚀 Analyze & Build ]               │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

#### Page: Automation Progress

**Route:** `/automations/:id/progress`

```
┌─────────────────────────────────────────────────────────────┐
│  Building: "Shopify → CRM → Slack"                         │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ✅ Understanding your process                    2s        │
│  ✅ Gathering requirements                        15s       │
│  ✅ Designing automation                          8s        │
│  ⏳ Building workflow                             ...       │
│  ○  Testing with sample data                               │
│  ○  Ready for deployment                                   │
│                                                             │
├─────────────────────────────────────────────────────────────┤
│  Current Step: Building workflow                            │
│  ┌─────────────────────────────────────────────────────┐   │
│  │  🔧 Creating Shopify Webhook Trigger...             │   │
│  │  🔧 Adding HubSpot Create Contact node...           │   │
│  │  🔧 Configuring Slack notification...               │   │
│  └─────────────────────────────────────────────────────┘   │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

#### Page: Plan Approval

**Route:** `/automations/:id/approve`

```
┌─────────────────────────────────────────────────────────────┐
│  📋 Review Automation Plan                    [Approve] [✗] │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  Summary                                                    │
│  ─────────────────────────────────────────────────────────  │
│  This automation will:                                      │
│  1. Listen for new Shopify orders (webhook)                 │
│  2. Create/update contact in HubSpot CRM                    │
│  3. Send notification to #orders Slack channel             │
│  4. Create shipping label via ShipStation                   │
│                                                             │
│  Estimated run time: ~3 seconds per order                   │
│  Confidence: 94%                                            │
│                                                             │
│  Workflow Preview                                           │
│  ─────────────────────────────────────────────────────────  │
│  ┌──────────┐    ┌──────────┐    ┌──────────┐             │
│  │ Shopify  │───▶│ HubSpot  │───▶│  Slack   │             │
│  │ Trigger  │    │  CRM     │    │ Message  │             │
│  └──────────┘    └──────────┘    └──────────┘             │
│        │                                                    │
│        └─────────────────────────────▶┌──────────┐         │
│                                       │ShipStation│         │
│                                       │  Label   │         │
│                                       └──────────┘         │
│                                                             │
│  [View Full Details] [Ask a Question]                       │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

#### Page: Automation Dashboard

**Route:** `/automations`

```
┌─────────────────────────────────────────────────────────────┐
│  Your Automations                        [ + New Automation ]│
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ┌─────────────────────────────────────────────────────┐   │
│  │ 🟢 Shopify → CRM → Slack                            │   │
│  │    Deployed • 1,234 runs • 99.8% success            │   │
│  │    Last run: 2 minutes ago                          │   │
│  └─────────────────────────────────────────────────────┘   │
│                                                             │
│  ┌─────────────────────────────────────────────────────┐   │
│  │ 🟡 Stripe → Accounting                              │   │
│  │    Deployed • 456 runs • 95.2% success              │   │
│  │    ⚠️ 1 auto-fix applied today                       │   │
│  └─────────────────────────────────────────────────────┘   │
│                                                             │
│  ┌─────────────────────────────────────────────────────┐   │
│  │ 🔴 Email Parser → Database                          │   │
│  │    Failed • Needs attention                         │   │
│  │    ❌ Schema change detected                         │   │
│  │    [ View Issue ] [ Auto-Fix ]                       │   │
│  └─────────────────────────────────────────────────────┘   │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

#### Page: Automation Detail / Monitoring

**Route:** `/automations/:id`

```
┌─────────────────────────────────────────────────────────────┐
│  Shopify → CRM → Slack                      [Pause] [Edit]  │
├────────────────────────────┬────────────────────────────────┤
│  Status: 🟢 Healthy        │  Quick Stats                   │
│  Deployed: Jan 15, 2026    │  ───────────────────────────   │
│  Last run: 2 min ago       │  Today:     45 runs            │
│                            │  This week: 312 runs           │
│  [View Logs] [Run Test]    │  Success:   99.8%              │
├────────────────────────────┴────────────────────────────────┤
│                                                             │
│  Recent Activity                                            │
│  ─────────────────────────────────────────────────────────  │
│  ✅ 2:34 PM  Order #1234 processed successfully     [View] │
│  ✅ 2:31 PM  Order #1233 processed successfully     [View] │
│  🔧 2:15 PM  Auto-fix: Updated CRM field mapping   [View] │
│  ✅ 2:12 PM  Order #1232 processed successfully     [View] │
│                                                             │
├─────────────────────────────────────────────────────────────┤
│  Workflow (Read-Only)                    [Enable Editing]   │
│  ─────────────────────────────────────────────────────────  │
│  ┌──────────┐    ┌──────────┐    ┌──────────┐             │
│  │ Shopify  │───▶│ HubSpot  │───▶│  Slack   │  [?]        │
│  │ Trigger  │    │  CRM     │    │ Message  │             │
│  └──────────┘    └──────────┘    └──────────┘             │
│                                                             │
│  Click any node to see: why it's here, data flow, errors   │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### 5.3 Component Library Additions

```typescript
// New UI Components

// Progress indicator for automation building
<AutomationProgress 
  steps={['understanding', 'designing', 'building', 'testing', 'ready']}
  currentStep="building"
  substeps={['Creating trigger...', 'Adding CRM node...']}
/>

// Approval gate component
<ApprovalGate
  title="Review Plan"
  summary={planSummary}
  confidence={0.94}
  onApprove={() => {}}
  onReject={(reason) => {}}
  onAskQuestion={(question) => {}}
/>

// Health status badge
<HealthBadge status="healthy" />  // 🟢
<HealthBadge status="degraded" /> // 🟡
<HealthBadge status="failed" />   // 🔴

// Auto-fix notification
<AutoFixNotification
  issue="CRM field mapping changed"
  fixDescription="Updated customer_name to customerName"
  appliedAt={timestamp}
  onViewDetails={() => {}}
  onUndo={() => {}}
/>

// Explainability popover (for nodes)
<NodeExplainer nodeId={nodeId}>
  <NodeCard {...node} />
</NodeExplainer>
// Shows: "Why is this here?", "What data flows?", "What if it fails?"
```

### 5.4 State Management

```typescript
// New Zustand stores

// Automation creation flow
interface AutomationCreationState {
  intent: string;
  structuredHints: { apps: string[]; volume: string; frequency: string };
  currentPhase: 'input' | 'discovery' | 'planning' | 'building' | 'testing' | 'approval';
  discoveryConversation: Message[];
  plan: AutomationPlan | null;
  testResults: TestRunResult | null;
  errors: string[];
  
  // Actions
  setIntent: (intent: string) => void;
  startDiscovery: () => Promise<void>;
  respondToDiscovery: (response: string) => Promise<void>;
  approvePlan: () => Promise<void>;
  rejectPlan: (reason: string) => Promise<void>;
  deployAutomation: () => Promise<void>;
}

// Automation monitoring
interface AutomationMonitorState {
  automations: AutomationLifecycle[];
  selectedAutomation: AutomationLifecycle | null;
  healthStatuses: Record<string, HealthStatus>;
  pendingFixes: AutomationFix[];
  
  // Actions
  fetchAutomations: () => Promise<void>;
  selectAutomation: (id: string) => void;
  applyFix: (fixId: string) => Promise<void>;
  pauseAutomation: (id: string) => Promise<void>;
}
```

---

## 6. API Contracts

### 6.1 Automation Orchestrator APIs

```typescript
// POST /api/automations/intents
// Create new automation intent
Request: {
  intent: string;
  hints?: {
    apps?: string[];
    volume?: string;
    frequency?: string;
  };
}
Response: {
  id: string;
  status: 'analyzing';
  conversationId: string;  // For discovery chat
}

// POST /api/automations/intents/:id/respond
// Respond to discovery question
Request: {
  message: string;
}
Response: {
  message: string;
  isComplete: boolean;
  nextQuestion?: string;
}

// POST /api/automations/intents/:id/plan
// Generate plan from intent
Response: {
  id: string;
  summary: string;
  proposedNodes: NodeProposal[];
  feasibilityScore: number;
  issues: FeasibilityIssue[];
}

// POST /api/automations/plans/:id/approve
// Approve plan for building
Response: {
  lifecycleId: string;
  status: 'building';
}

// POST /api/automations/plans/:id/reject
Request: {
  reason: string;
}

// GET /api/automations/lifecycles/:id
// Get automation lifecycle details
Response: {
  id: string;
  currentState: AutomationLifecycleState;
  stateHistory: StateTransition[];
  workflow?: Workflow;
  testResults?: TestRunResult[];
  health: HealthStatus;
}

// POST /api/automations/lifecycles/:id/deploy
// Deploy automation
Response: {
  status: 'deployed';
  workflowId: string;
}

// GET /api/automations/lifecycles/:id/health
// Get health status
Response: {
  status: 'healthy' | 'degraded' | 'failed';
  issues: HealthIssue[];
  suggestedFixes: AutoFix[];
}

// POST /api/automations/fixes/:id/apply
// Apply auto-fix
Response: {
  status: 'applied';
  changes: FixChanges;
}
```

---

## 8. Implementation Phases

### Phase 1: Foundation (Weeks 1-2)

**Database:**
- [ ] Create new tables
- [ ] Add migrations
- [ ] Update Prisma schema

**Backend:**
- [ ] `automation-orchestrator` module (basic)
- [ ] `node-registry` module
- [ ] `openrouter` module (model gateway)
- [ ] State machine implementation
- [ ] Basic agent integration

**UI:**
- [ ] Intent input page
- [ ] Progress indicator component

### Phase 2: Agent Layer (Weeks 3-4)

**Backend:**
- [ ] Discovery Agent
- [ ] Planning Agent  
- [ ] Builder Agent
- [ ] Agent session logging

**UI:**
- [ ] Discovery chat interface
- [ ] Plan approval page
- [ ] Workflow preview component

### Phase 3: Testing & Validation (Weeks 5-6)

**Backend:**
- [ ] `workflow-compiler` module
- [ ] `workflow-test-engine` module
- [ ] Mock generator
- [ ] Test runner

**UI:**
- [ ] Test results display
- [ ] Confidence score UI
- [ ] Issue list component

### Phase 4: Monitoring & Healing (Weeks 7-8)

**Backend:**
- [ ] `automation-monitor` module
- [ ] Health checker (cron job)
- [ ] Issue detector
- [ ] Auto-fix engine

**UI:**
- [ ] Automation dashboard
- [ ] Health status badges
- [ ] Auto-fix notifications
- [ ] Fix history view

### Phase 5: Polish & Enterprise (Weeks 9-10)

**Backend:**
- [ ] Knowledge persistence
- [ ] Audit logging
- [ ] Rollback capability
- [ ] Rate limiting

**UI:**
- [ ] Explainability layer
- [ ] Advanced editing toggle
- [ ] Rollback UI
- [ ] Usage analytics

---

## 9. Niche Strategy (Marketing First, Sales Second)

### 9.1 Why Start with Marketing Niche

**Strategic reasons:**
1. **Clear pain points**: Marketers hate repetitive tasks (lead sync, email triggers)
2. **Simple workflows**: Usually linear, 3-5 steps
3. **High volume**: Lots of leads = lots of runs = clear ROI
4. **Budget holders**: Marketing teams have tools budget
5. **Word-of-mouth**: Marketers talk to each other

**Target personas (Marketing Phase 1):**
- Marketing Managers (SMB)
- Growth Marketers
- Email Marketing Specialists
- Content Marketers
- Social Media Managers

### 9.2 Marketing Niche: Pre-Built Discovery Templates

The AI Assistant will have **marketing-specific intelligence**:

```typescript
// agents/discovery-agent.marketing.ts

export const MARKETING_DISCOVERY_PROMPTS = {
  
  // Common marketing triggers
  triggerPatterns: [
    'form submission',
    'new subscriber',
    'webinar registration',
    'lead magnet download',
    'cart abandonment',
    'new customer',
    'social mention',
  ],
  
  // Common marketing tools
  knownTools: {
    forms: ['Typeform', 'Webflow', 'HubSpot Forms', 'Gravity Forms', 'Jotform'],
    email: ['Mailchimp', 'ConvertKit', 'ActiveCampaign', 'Klaviyo', 'Brevo'],
    crm: ['HubSpot', 'Salesforce', 'Pipedrive', 'Close'],
    ads: ['Facebook Ads', 'Google Ads', 'LinkedIn Ads'],
    social: ['Buffer', 'Hootsuite', 'Later'],
    analytics: ['Google Analytics', 'Mixpanel', 'Segment'],
  },
  
  // Marketing-specific questions
  discoveryQuestions: [
    {
      id: 'trigger_type',
      question: 'What kicks off this automation?',
      options: [
        'Someone fills out a form',
        'Someone subscribes to my list',
        'Someone makes a purchase',
        'Something happens on social media',
        'Something else'
      ]
    },
    {
      id: 'primary_goal',
      question: 'What\'s the main goal?',
      options: [
        'Nurture leads with emails',
        'Sync data between tools',
        'Send notifications to my team',
        'Update a CRM or database',
        'Post to social media'
      ]
    },
    {
      id: 'volume',
      question: 'How many times per day will this run?',
      options: ['1-10', '10-50', '50-200', '200+']
    }
  ]
};

// Marketing-specific automation templates
export const MARKETING_TEMPLATES = [
  {
    id: 'lead-to-email',
    name: 'Lead Capture → Email Sequence',
    description: 'Automatically add form submissions to your email list',
    triggers: ['typeform', 'webflow_form', 'hubspot_form'],
    actions: ['mailchimp_add', 'convertkit_add', 'activecampaign_add'],
    popularity: 95  // Show most popular first
  },
  {
    id: 'webinar-registration',
    name: 'Webinar Registration Flow',
    description: 'Register for webinar, send confirmation, add to email list',
    triggers: ['typeform', 'calendly'],
    actions: ['zoom_register', 'mailchimp_add', 'slack_notify'],
    popularity: 78
  },
  {
    id: 'lead-to-crm',
    name: 'Lead → CRM + Team Notification',
    description: 'New leads go to CRM and notify sales team',
    triggers: ['typeform', 'webflow_form'],
    actions: ['hubspot_create_contact', 'slack_notify'],
    popularity: 85
  },
  {
    id: 'social-to-sheet',
    name: 'Social Mentions → Tracker',
    description: 'Track brand mentions in a Google Sheet',
    triggers: ['twitter_mention', 'instagram_mention'],
    actions: ['gsheet_add_row'],
    popularity: 45
  }
];
```

### 9.3 Marketing-Specific UI/UX

```
┌─────────────────────────────────────────────────────────────────────────┐
│  🎯 Marketing Automation Templates                       [ View All ]   │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│  Popular with marketers like you:                                       │
│                                                                          │
│  ┌───────────────────────────────────────────────────────────────────┐  │
│  │  📧 Lead Capture → Email Sequence                    [Use This]  │  │
│  │  Typeform → Mailchimp → Welcome Series                           │  │
│  │  ⭐ Used by 2,340 marketers                                       │  │
│  └───────────────────────────────────────────────────────────────────┘  │
│                                                                          │
│  ┌───────────────────────────────────────────────────────────────────┐  │
│  │  📅 Webinar Registration Flow                        [Use This]  │  │
│  │  Form → Zoom → Confirmation Email → Slack                        │  │
│  │  ⭐ Used by 1,892 marketers                                       │  │
│  └───────────────────────────────────────────────────────────────────┘  │
│                                                                          │
│  ┌───────────────────────────────────────────────────────────────────┐  │
│  │  👤 Lead → CRM + Team Alert                          [Use This]  │  │
│  │  Form → HubSpot → Slack notification                             │  │
│  │  ⭐ Used by 1,567 marketers                                       │  │
│  └───────────────────────────────────────────────────────────────────┘  │
│                                                                          │
│  ─────────────────────────────────────────────────────────────────────  │
│  Or describe what you need:                                             │
│  ┌───────────────────────────────────────────────────────────────────┐  │
│  │ "When someone downloads my lead magnet..."                        │  │
│  └───────────────────────────────────────────────────────────────────┘  │
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘
```

### 9.4 Phase 2: Sales Niche

After marketing is validated (3-6 months), expand to **Sales**:

**Target personas (Sales Phase 2):**
- Sales Development Reps (SDRs)
- Account Executives
- Sales Managers
- RevOps Teams

**Sales-specific automations:**

```typescript
export const SALES_TEMPLATES = [
  {
    id: 'lead-scoring',
    name: 'Lead Scoring & Routing',
    description: 'Score new leads and route to right rep',
    triggers: ['hubspot_new_contact', 'salesforce_new_lead'],
    actions: ['enrich_clearbit', 'score_lead', 'assign_rep', 'slack_notify'],
  },
  {
    id: 'meeting-booked',
    name: 'Meeting Booked Follow-up',
    description: 'Prepare rep with lead intel when meeting is booked',
    triggers: ['calendly_booked', 'hubspot_meeting'],
    actions: ['enrich_clearbit', 'slack_notify', 'create_prep_doc'],
  },
  {
    id: 'deal-stage-change',
    name: 'Deal Stage Notifications',
    description: 'Alert team when deals move stages',
    triggers: ['hubspot_deal_stage', 'salesforce_opportunity'],
    actions: ['slack_notify', 'update_sheet', 'send_email'],
  },
  {
    id: 'contract-sent',
    name: 'Contract Sent → Follow-up',
    description: 'Track contracts and remind if not signed',
    triggers: ['docusign_sent', 'pandadoc_sent'],
    actions: ['wait_3_days', 'check_status', 'send_reminder'],
  }
];
```

### 9.5 Niche Rollout Timeline

```
Month 1-2: Marketing MVP
├── Discovery Agent with marketing context
├── 4 core marketing templates
├── Integration with top 5 marketing tools
└── Beta with 100 marketing users

Month 3-4: Marketing Production
├── 10+ marketing templates
├── Integration with 15+ marketing tools
├── Self-serve onboarding
└── 1,000+ active marketing users

Month 5-6: Sales Beta
├── Sales-specific discovery prompts
├── 4 core sales templates
├── CRM integrations (HubSpot, Salesforce)
└── Beta with 50 sales users

Month 7-8: Sales Production
├── 10+ sales templates
├── Revenue tool integrations
└── Sales + Marketing cross-workflows
└── 2,000+ total active users
```

---

## 10. Security Considerations

### 10.1 Approval Gates

```typescript
// Destructive actions require approval
const REQUIRES_APPROVAL = [
  'delete_data',
  'send_email',
  'post_to_social',
  'payment_action',
  'production_deployment'
];

// Auto-fix limits
const AUTO_FIX_LIMITS = {
  maxAutoFixesPerHour: 5,
  requiresApprovalFor: ['schema_change', 'auth_change'],
  notifyUserFor: ['any_fix']
};
```

### 10.2 Audit Trail

Every action is logged:
- Who initiated
- What changed (before/after)
- When
- AI agent involved
- Reasoning

### 10.3 Rollback

All workflow changes are versioned:
- One-click rollback
- Compare versions
- Restore any previous version

---

## Appendix A: Agent System Prompts

See separate document: [AGENT_PROMPTS.md](./AGENT_PROMPTS.md)

## Appendix B: Node Capability Schema

See separate document: [NODE_CAPABILITY_SCHEMA.md](./NODE_CAPABILITY_SCHEMA.md)

## Appendix C: OpenRouter Integration

See separate document: [OPENROUTER_INTEGRATION.md](./OPENROUTER_INTEGRATION.md)

---

**Document Status:** Draft v2.0  
**Next Review:** February 12, 2026  
**Stakeholders:** Engineering, Product, Design
