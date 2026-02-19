# OpenRouter Integration Guide

**Version:** 1.0.0  
**Date:** February 5, 2026  

---

## Overview

a2n.io uses **OpenRouter** as the AI gateway for all agent operations. This provides:

- **Model flexibility**: Switch between 100+ models without code changes
- **Cost optimization**: Route to cheaper models for simple tasks
- **Fallback chains**: Automatic retry with alternate models
- **Unified billing**: Single API key, single invoice
- **No vendor lock-in**: Easy to switch providers

---

## 1. OpenRouter Service Implementation

### 1.1 Module Structure

```
src/openrouter/
├── openrouter.module.ts
├── openrouter.service.ts
├── openrouter.config.ts
├── dto/
│   ├── chat-completion.dto.ts
│   └── model-response.dto.ts
└── interfaces/
    ├── openrouter-request.interface.ts
    └── openrouter-response.interface.ts
```

### 1.2 OpenRouter Service

```typescript
// openrouter.service.ts

import { Injectable, Logger, HttpException } from '@nestjs/common';
import { ConfigService } from '@nestjs/config';
import { HttpService } from '@nestjs/axios';
import { firstValueFrom } from 'rxjs';

export interface ChatMessage {
  role: 'system' | 'user' | 'assistant';
  content: string;
}

export interface ChatCompletionRequest {
  model: string;
  messages: ChatMessage[];
  temperature?: number;
  max_tokens?: number;
  response_format?: {
    type: 'json_schema';
    json_schema: {
      name: string;
      schema: Record<string, any>;
    };
  };
  stream?: boolean;
}

export interface ChatCompletionResponse {
  id: string;
  model: string;
  choices: {
    index: number;
    message: {
      role: string;
      content: string;
    };
    finish_reason: string;
  }[];
  usage: {
    prompt_tokens: number;
    completion_tokens: number;
    total_tokens: number;
  };
}

@Injectable()
export class OpenRouterService {
  private readonly logger = new Logger(OpenRouterService.name);
  private readonly apiKey: string;
  private readonly baseUrl = 'https://openrouter.ai/api/v1';

  constructor(
    private readonly configService: ConfigService,
    private readonly httpService: HttpService,
  ) {
    this.apiKey = this.configService.get<string>('OPENROUTER_API_KEY');
    if (!this.apiKey) {
      this.logger.warn('OPENROUTER_API_KEY not set - AI features disabled');
    }
  }

  /**
   * Send a chat completion request to OpenRouter
   */
  async chat(request: ChatCompletionRequest): Promise<ChatCompletionResponse> {
    if (!this.apiKey) {
      throw new HttpException('OpenRouter API key not configured', 503);
    }

    const startTime = Date.now();
    
    try {
      const response = await firstValueFrom(
        this.httpService.post(
          `${this.baseUrl}/chat/completions`,
          request,
          {
            headers: {
              'Authorization': `Bearer ${this.apiKey}`,
              'Content-Type': 'application/json',
              'HTTP-Referer': 'https://a2n.io',
              'X-Title': 'a2n.io Automation Platform',
            },
          },
        ),
      );

      const duration = Date.now() - startTime;
      this.logger.log(
        `OpenRouter ${request.model}: ${response.data.usage.total_tokens} tokens, ${duration}ms`,
      );

      return response.data;
    } catch (error) {
      this.logger.error(`OpenRouter error: ${error.message}`, error.stack);
      throw new HttpException(
        `AI service error: ${error.response?.data?.error?.message || error.message}`,
        error.response?.status || 500,
      );
    }
  }

  /**
   * Chat with automatic model fallback
   */
  async chatWithFallback(
    request: ChatCompletionRequest,
    fallbackModels: string[],
  ): Promise<ChatCompletionResponse> {
    const models = [request.model, ...fallbackModels];
    
    for (const model of models) {
      try {
        return await this.chat({ ...request, model });
      } catch (error) {
        this.logger.warn(`Model ${model} failed, trying next...`);
        if (model === models[models.length - 1]) {
          throw error; // Last model, re-throw
        }
      }
    }
  }

  /**
   * Streaming chat completion
   */
  async chatStream(
    request: ChatCompletionRequest,
  ): AsyncGenerator<string, void, unknown> {
    if (!this.apiKey) {
      throw new HttpException('OpenRouter API key not configured', 503);
    }

    const response = await firstValueFrom(
      this.httpService.post(
        `${this.baseUrl}/chat/completions`,
        { ...request, stream: true },
        {
          headers: {
            'Authorization': `Bearer ${this.apiKey}`,
            'Content-Type': 'application/json',
            'HTTP-Referer': 'https://a2n.io',
            'X-Title': 'a2n.io Automation Platform',
          },
          responseType: 'stream',
        },
      ),
    );

    // Stream handling implementation
    for await (const chunk of response.data) {
      const lines = chunk.toString().split('\n');
      for (const line of lines) {
        if (line.startsWith('data: ') && line !== 'data: [DONE]') {
          const data = JSON.parse(line.slice(6));
          yield data.choices[0]?.delta?.content || '';
        }
      }
    }
  }
}
```

### 1.3 Model Configuration

```typescript
// openrouter.config.ts

export interface ModelConfig {
  default: string;
  fallback: string;
  maxTokens: number;
  temperature: number;
}

export const AGENT_MODEL_CONFIG: Record<string, ModelConfig> = {
  
  // Discovery Agent - Conversational, needs to be friendly
  discovery: {
    default: 'anthropic/claude-3-sonnet',
    fallback: 'openai/gpt-4-turbo',
    maxTokens: 2000,
    temperature: 0.7,  // Slightly creative for conversation
  },
  
  // Planning Agent - Complex reasoning
  planning: {
    default: 'anthropic/claude-3-opus',
    fallback: 'openai/gpt-4-turbo',
    maxTokens: 4000,
    temperature: 0.3,  // More deterministic
  },
  
  // Builder Agent - Precise structured output
  builder: {
    default: 'openai/gpt-4-turbo',
    fallback: 'anthropic/claude-3-sonnet',
    maxTokens: 8000,
    temperature: 0.1,  // Very deterministic for JSON
  },
  
  // Testing Agent - Analytical
  testing: {
    default: 'openai/gpt-4-turbo',
    fallback: 'anthropic/claude-3-sonnet',
    maxTokens: 4000,
    temperature: 0.2,
  },
  
  // Monitoring Agent - Quick checks
  monitoring: {
    default: 'openai/gpt-3.5-turbo',  // Cheaper for frequent checks
    fallback: 'anthropic/claude-3-haiku',
    maxTokens: 1000,
    temperature: 0.1,
  },
  
  // Maintenance Agent - Fix generation
  maintenance: {
    default: 'anthropic/claude-3-sonnet',
    fallback: 'openai/gpt-4-turbo',
    maxTokens: 4000,
    temperature: 0.2,
  },
};

// Cost per 1K tokens (approximate, for budgeting)
export const MODEL_COSTS: Record<string, { input: number; output: number }> = {
  'anthropic/claude-3-opus': { input: 0.015, output: 0.075 },
  'anthropic/claude-3-sonnet': { input: 0.003, output: 0.015 },
  'anthropic/claude-3-haiku': { input: 0.00025, output: 0.00125 },
  'openai/gpt-4-turbo': { input: 0.01, output: 0.03 },
  'openai/gpt-4': { input: 0.03, output: 0.06 },
  'openai/gpt-3.5-turbo': { input: 0.0005, output: 0.0015 },
  'google/gemini-pro': { input: 0.00025, output: 0.0005 },
  'meta-llama/llama-3-70b-instruct': { input: 0.0008, output: 0.0008 },
};
```

---

## 2. Agent Base Class with OpenRouter

```typescript
// agents/base-agent.ts

import { Injectable } from '@nestjs/common';
import { OpenRouterService, ChatMessage } from '../openrouter/openrouter.service';
import { AGENT_MODEL_CONFIG, ModelConfig } from '../openrouter/openrouter.config';
import { PrismaService } from '../prisma/prisma.service';

export interface AgentContext {
  intentId?: string;
  planId?: string;
  lifecycleId?: string;
  userId: string;
  workspaceId: string;
  additionalContext?: Record<string, any>;
}

export interface AgentResult {
  success: boolean;
  output: any;
  reasoning?: string[];
  tokensUsed: number;
  costUsd: number;
  model: string;
}

@Injectable()
export abstract class BaseAutomationAgent {
  protected abstract readonly agentType: string;
  
  constructor(
    protected readonly openRouterService: OpenRouterService,
    protected readonly prisma: PrismaService,
  ) {}

  /**
   * Get model config for this agent
   */
  protected getModelConfig(): ModelConfig {
    return AGENT_MODEL_CONFIG[this.agentType];
  }

  /**
   * Override in subclasses to define system prompt
   */
  abstract getSystemPrompt(context: AgentContext): string;

  /**
   * Override in subclasses to define output schema
   */
  abstract getOutputSchema(): Record<string, any>;

  /**
   * Build user prompt from context
   */
  abstract buildUserPrompt(context: AgentContext): string;

  /**
   * Execute the agent
   */
  async execute(context: AgentContext): Promise<AgentResult> {
    const config = this.getModelConfig();
    const startTime = Date.now();

    // Create agent session for logging
    const session = await this.prisma.agentSession.create({
      data: {
        lifecycleId: context.lifecycleId,
        agentType: this.agentType,
        agentModel: config.default,
        status: 'running',
        inputContext: context as any,
      },
    });

    try {
      // Build messages
      const messages: ChatMessage[] = [
        { role: 'system', content: this.getSystemPrompt(context) },
        { role: 'user', content: this.buildUserPrompt(context) },
      ];

      // Call OpenRouter with fallback
      const response = await this.openRouterService.chatWithFallback(
        {
          model: config.default,
          messages,
          max_tokens: config.maxTokens,
          temperature: config.temperature,
          response_format: {
            type: 'json_schema',
            json_schema: {
              name: `${this.agentType}_output`,
              schema: this.getOutputSchema(),
            },
          },
        },
        [config.fallback],
      );

      // Parse output
      const output = JSON.parse(response.choices[0].message.content);
      const duration = Date.now() - startTime;

      // Calculate cost
      const costUsd = this.calculateCost(
        response.model,
        response.usage.prompt_tokens,
        response.usage.completion_tokens,
      );

      // Update session
      await this.prisma.agentSession.update({
        where: { id: session.id },
        data: {
          status: 'completed',
          finishedAt: new Date(),
          outputResult: output,
          tokensUsed: response.usage.total_tokens,
          costUsd,
          durationMs: duration,
        },
      });

      return {
        success: true,
        output,
        tokensUsed: response.usage.total_tokens,
        costUsd,
        model: response.model,
      };

    } catch (error) {
      // Update session with error
      await this.prisma.agentSession.update({
        where: { id: session.id },
        data: {
          status: 'failed',
          finishedAt: new Date(),
          errorMessage: error.message,
        },
      });

      throw error;
    }
  }

  /**
   * Calculate cost based on model and tokens
   */
  private calculateCost(
    model: string,
    inputTokens: number,
    outputTokens: number,
  ): number {
    const costs = MODEL_COSTS[model] || { input: 0.01, output: 0.03 };
    return (
      (inputTokens / 1000) * costs.input +
      (outputTokens / 1000) * costs.output
    );
  }
}
```

---

## 3. Example Agent Implementation

### 3.1 Discovery Agent

```typescript
// agents/discovery-agent.ts

import { Injectable } from '@nestjs/common';
import { BaseAutomationAgent, AgentContext, AgentResult } from './base-agent';
import { MARKETING_DISCOVERY_PROMPTS } from './prompts/marketing.prompts';

@Injectable()
export class DiscoveryAgent extends BaseAutomationAgent {
  protected readonly agentType = 'discovery';

  getSystemPrompt(context: AgentContext): string {
    return `You are a friendly automation discovery assistant for a2n.io, a workflow automation platform.

Your job is to help non-technical users (mostly marketers and small business owners) describe what they want to automate.

RULES:
1. Be conversational and friendly - no jargon
2. Ask only essential questions (max 3 at a time)
3. Offer multiple choice options when possible
4. Recognize common tools: ${Object.values(MARKETING_DISCOVERY_PROMPTS.knownTools).flat().join(', ')}
5. Extract: trigger, actions, conditions, tools used, volume, frequency

OUTPUT FORMAT:
You must respond with valid JSON matching the schema provided.
- If you need more info, set isComplete: false and provide questions
- If you have enough info, set isComplete: true and provide extractedRequirements`;
  }

  getOutputSchema(): Record<string, any> {
    return {
      type: 'object',
      properties: {
        message: {
          type: 'string',
          description: 'Friendly message to the user',
        },
        isComplete: {
          type: 'boolean',
          description: 'Whether discovery is complete',
        },
        questions: {
          type: 'array',
          items: {
            type: 'object',
            properties: {
              id: { type: 'string' },
              question: { type: 'string' },
              type: { 
                type: 'string', 
                enum: ['multiple_choice', 'text', 'yes_no'] 
              },
              options: { 
                type: 'array', 
                items: { type: 'string' } 
              },
            },
          },
          description: 'Questions to ask the user',
        },
        extractedRequirements: {
          type: 'object',
          properties: {
            trigger: {
              type: 'object',
              properties: {
                type: { type: 'string' },
                tool: { type: 'string' },
                event: { type: 'string' },
              },
            },
            actions: {
              type: 'array',
              items: {
                type: 'object',
                properties: {
                  type: { type: 'string' },
                  tool: { type: 'string' },
                  operation: { type: 'string' },
                },
              },
            },
            conditions: { type: 'array', items: { type: 'string' } },
            estimatedVolume: { type: 'string' },
            frequency: { type: 'string' },
          },
          description: 'Extracted requirements (only when isComplete: true)',
        },
      },
      required: ['message', 'isComplete'],
    };
  }

  buildUserPrompt(context: AgentContext): string {
    const { additionalContext } = context;
    
    if (additionalContext?.conversationHistory) {
      // Continue conversation
      return `Previous conversation:
${additionalContext.conversationHistory.map(m => `${m.role}: ${m.content}`).join('\n')}

User's latest response: ${additionalContext.userMessage}`;
    }
    
    // Initial intent
    return `User's automation request: "${additionalContext?.intent}"

Start the discovery process. Ask clarifying questions to understand exactly what they need.`;
  }
}
```

---

## 4. Environment Configuration

```bash
# .env

# OpenRouter Configuration
OPENROUTER_API_KEY=sk-or-v1-your-key-here

# Optional: Override default models
DISCOVERY_MODEL=anthropic/claude-3-sonnet
PLANNING_MODEL=anthropic/claude-3-opus
BUILDER_MODEL=openai/gpt-4-turbo

# Cost limits
MAX_TOKENS_PER_REQUEST=8000
MAX_COST_PER_AUTOMATION_USD=1.00
```

---

## 5. Testing OpenRouter Integration

```typescript
// openrouter.service.spec.ts

describe('OpenRouterService', () => {
  let service: OpenRouterService;

  beforeEach(async () => {
    const module = await Test.createTestingModule({
      providers: [
        OpenRouterService,
        { provide: ConfigService, useValue: mockConfigService },
        { provide: HttpService, useValue: mockHttpService },
      ],
    }).compile();

    service = module.get<OpenRouterService>(OpenRouterService);
  });

  it('should call OpenRouter API', async () => {
    mockHttpService.post.mockReturnValue(of({
      data: {
        choices: [{ message: { content: '{"result": "test"}' } }],
        usage: { total_tokens: 100 },
      },
    }));

    const result = await service.chat({
      model: 'openai/gpt-3.5-turbo',
      messages: [{ role: 'user', content: 'test' }],
    });

    expect(result.choices[0].message.content).toBe('{"result": "test"}');
  });

  it('should fallback on error', async () => {
    mockHttpService.post
      .mockReturnValueOnce(throwError(() => new Error('Model unavailable')))
      .mockReturnValueOnce(of({
        data: { choices: [{ message: { content: '{}' } }], usage: {} },
      }));

    const result = await service.chatWithFallback(
      { model: 'anthropic/claude-3-opus', messages: [] },
      ['openai/gpt-4-turbo'],
    );

    expect(mockHttpService.post).toHaveBeenCalledTimes(2);
  });
});
```

---

## 6. Rate Limiting & Quotas

```typescript
// openrouter-rate-limiter.service.ts

@Injectable()
export class OpenRouterRateLimiter {
  private userTokens = new Map<string, { tokens: number; resetAt: Date }>();

  async checkQuota(userId: string, estimatedTokens: number): Promise<boolean> {
    const quota = await this.getUserQuota(userId);
    const usage = this.userTokens.get(userId) || { tokens: 0, resetAt: new Date() };

    if (usage.resetAt < new Date()) {
      // Reset daily quota
      usage.tokens = 0;
      usage.resetAt = new Date(Date.now() + 24 * 60 * 60 * 1000);
    }

    if (usage.tokens + estimatedTokens > quota.dailyTokenLimit) {
      throw new HttpException('Daily AI quota exceeded', 429);
    }

    return true;
  }

  async recordUsage(userId: string, tokens: number): Promise<void> {
    const usage = this.userTokens.get(userId) || { tokens: 0, resetAt: new Date() };
    usage.tokens += tokens;
    this.userTokens.set(userId, usage);
  }

  private async getUserQuota(userId: string): Promise<{ dailyTokenLimit: number }> {
    // Get from subscription plan
    // Free: 10K tokens/day
    // Pro: 100K tokens/day
    // Enterprise: Unlimited
    return { dailyTokenLimit: 100000 };
  }
}
```

---

**Next Steps:**
1. Set up OpenRouter account at https://openrouter.ai
2. Add `OPENROUTER_API_KEY` to environment
3. Implement the `OpenRouterModule`
4. Test with Discovery Agent first
