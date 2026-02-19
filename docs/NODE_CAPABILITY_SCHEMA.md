# Node Capability Schema

**Version:** 1.0.0  
**Date:** February 5, 2026  

---

## Overview

The Node Capability Registry is the **"truth source"** that AI agents use to understand what each node can do. This replaces the implicit knowledge that human automation freelancers have.

---

## 1. Schema Definition

### 1.1 Node Capability Entity

```typescript
// node-capability.interface.ts

export interface NodeCapability {
  id: string;
  nodeTypeId: string;
  
  // What this node can do
  capabilities: string[];
  limitations: string[];
  
  // Input/Output contracts
  requiredInputs: InputField[];
  optionalInputs: InputField[];
  outputSchema: JSONSchema;
  
  // Error handling
  commonErrors: CommonError[];
  retryPolicy: RetryPolicy;
  
  // Rate limits
  rateLimits: RateLimit;
  
  // Dependencies
  requiresAuth: boolean;
  authTypes: ('oauth2' | 'api_key' | 'basic' | 'none')[];
  externalDependencies: string[];
  
  // AI hints
  useCases: string[];
  bestPractices: string[];
  
  // Metadata
  updatedAt: Date;
}

export interface InputField {
  name: string;
  type: 'string' | 'number' | 'boolean' | 'object' | 'array' | 'date' | 'email' | 'url';
  description: string;
  example: any;
  validation?: {
    minLength?: number;
    maxLength?: number;
    pattern?: string;
    enum?: any[];
  };
}

export interface CommonError {
  code: string;
  cause: string;
  solution: string;
  retryable: boolean;
}

export interface RetryPolicy {
  maxRetries: number;
  backoff: 'linear' | 'exponential';
  initialDelay: number; // ms
  maxDelay: number; // ms
  retryableErrors: string[];
}

export interface RateLimit {
  requestsPerMinute?: number;
  requestsPerHour?: number;
  requestsPerDay?: number;
  burstLimit?: number;
}
```

---

## 2. Example Node Capabilities

### 2.1 Typeform Trigger

```typescript
const typeformTriggerCapability: NodeCapability = {
  id: 'cap_typeform_trigger',
  nodeTypeId: 'typeform_trigger',
  
  capabilities: [
    'receive_form_submission',
    'capture_form_responses',
    'trigger_workflow_on_submit',
    'access_hidden_fields',
    'access_calculated_fields',
  ],
  
  limitations: [
    'requires_typeform_pro_for_webhooks',
    'max_payload_1mb',
    'no_file_attachments_in_webhook',
    'webhook_must_respond_in_30s',
  ],
  
  requiredInputs: [
    {
      name: 'formId',
      type: 'string',
      description: 'The Typeform form ID (found in the form URL)',
      example: 'abc123XY',
      validation: { pattern: '^[a-zA-Z0-9]+$' }
    }
  ],
  
  optionalInputs: [
    {
      name: 'webhookSecret',
      type: 'string',
      description: 'Secret for validating webhook authenticity',
      example: 'your-secret-key'
    }
  ],
  
  outputSchema: {
    type: 'object',
    properties: {
      formId: { type: 'string' },
      submittedAt: { type: 'string', format: 'date-time' },
      responseId: { type: 'string' },
      answers: {
        type: 'array',
        items: {
          type: 'object',
          properties: {
            field: { type: 'object' },
            type: { type: 'string' },
            text: { type: 'string' },
            email: { type: 'string' },
            number: { type: 'number' },
            boolean: { type: 'boolean' }
          }
        }
      },
      hidden: { type: 'object' },
      calculated: { type: 'object' }
    }
  },
  
  commonErrors: [
    {
      code: 'INVALID_SIGNATURE',
      cause: 'Webhook secret mismatch',
      solution: 'Verify the webhook secret matches your Typeform settings',
      retryable: false
    },
    {
      code: 'FORM_NOT_FOUND',
      cause: 'Form ID does not exist or was deleted',
      solution: 'Check the form ID in Typeform dashboard',
      retryable: false
    }
  ],
  
  retryPolicy: {
    maxRetries: 0,  // Triggers don't retry
    backoff: 'linear',
    initialDelay: 0,
    maxDelay: 0,
    retryableErrors: []
  },
  
  rateLimits: {
    // Typeform webhooks have no rate limit on receive
  },
  
  requiresAuth: true,
  authTypes: ['oauth2'],
  externalDependencies: ['typeform_api'],
  
  useCases: [
    'lead_capture',
    'survey_response',
    'quiz_completion',
    'registration_form',
    'feedback_collection',
    'job_application'
  ],
  
  bestPractices: [
    'Use hidden fields to pass UTM parameters',
    'Enable response notifications in Typeform too as backup',
    'Test with a sample submission before going live',
    'Map field names carefully - they change if you edit the form'
  ],
  
  updatedAt: new Date('2026-02-05')
};
```

### 2.2 Mailchimp Add Subscriber

```typescript
const mailchimpAddSubscriberCapability: NodeCapability = {
  id: 'cap_mailchimp_add_subscriber',
  nodeTypeId: 'mailchimp_add_subscriber',
  
  capabilities: [
    'add_new_subscriber',
    'update_existing_subscriber',
    'set_subscriber_tags',
    'set_merge_fields',
    'trigger_automation_on_add',
    'set_subscriber_status',
  ],
  
  limitations: [
    'email_required',
    'max_merge_fields_80',
    'max_tags_per_request_10',
    'double_optin_may_delay_add',
    'invalid_emails_rejected',
    'role_based_emails_may_fail',
  ],
  
  requiredInputs: [
    {
      name: 'listId',
      type: 'string',
      description: 'Mailchimp audience/list ID',
      example: 'abc123def4',
      validation: { pattern: '^[a-f0-9]+$' }
    },
    {
      name: 'email',
      type: 'email',
      description: 'Subscriber email address',
      example: 'john@example.com'
    }
  ],
  
  optionalInputs: [
    {
      name: 'firstName',
      type: 'string',
      description: 'Subscriber first name (merge field FNAME)',
      example: 'John'
    },
    {
      name: 'lastName',
      type: 'string',
      description: 'Subscriber last name (merge field LNAME)',
      example: 'Doe'
    },
    {
      name: 'status',
      type: 'string',
      description: 'Subscription status',
      example: 'subscribed',
      validation: { enum: ['subscribed', 'pending', 'unsubscribed', 'cleaned'] }
    },
    {
      name: 'tags',
      type: 'array',
      description: 'Tags to apply to subscriber',
      example: ['lead-magnet', 'website-signup']
    },
    {
      name: 'mergeFields',
      type: 'object',
      description: 'Additional merge fields',
      example: { COMPANY: 'Acme Inc', PHONE: '555-1234' }
    }
  ],
  
  outputSchema: {
    type: 'object',
    properties: {
      id: { type: 'string', description: 'Mailchimp member ID' },
      email: { type: 'string' },
      status: { type: 'string' },
      listId: { type: 'string' },
      created: { type: 'boolean', description: 'True if new, false if updated' },
      webId: { type: 'number', description: 'Web ID for dashboard link' }
    }
  },
  
  commonErrors: [
    {
      code: 'MEMBER_EXISTS',
      cause: 'Email already in list',
      solution: 'Use upsert mode to update existing subscribers',
      retryable: false
    },
    {
      code: 'INVALID_EMAIL',
      cause: 'Email format invalid or undeliverable',
      solution: 'Validate email before sending to Mailchimp',
      retryable: false
    },
    {
      code: 'COMPLIANCE_ERROR',
      cause: 'Email violates Mailchimp compliance (role-based, disposable)',
      solution: 'Filter out role-based emails like info@, sales@',
      retryable: false
    },
    {
      code: 'RATE_LIMIT',
      cause: 'Too many requests',
      solution: 'Implement exponential backoff',
      retryable: true
    },
    {
      code: 'LIST_NOT_FOUND',
      cause: 'Audience ID invalid',
      solution: 'Verify list ID in Mailchimp dashboard',
      retryable: false
    }
  ],
  
  retryPolicy: {
    maxRetries: 3,
    backoff: 'exponential',
    initialDelay: 1000,
    maxDelay: 30000,
    retryableErrors: ['RATE_LIMIT', 'TIMEOUT', 'SERVER_ERROR']
  },
  
  rateLimits: {
    requestsPerMinute: 10,  // Mailchimp allows 10 concurrent
    requestsPerDay: 500000  // Based on plan
  },
  
  requiresAuth: true,
  authTypes: ['api_key', 'oauth2'],
  externalDependencies: ['mailchimp_api_v3'],
  
  useCases: [
    'newsletter_signup',
    'lead_magnet_delivery',
    'webinar_registration',
    'customer_onboarding',
    'event_registration',
    'waitlist_signup'
  ],
  
  bestPractices: [
    'Use status:pending for double opt-in compliance',
    'Always set tags for segmentation',
    'Map source UTM parameters to merge fields',
    'Handle MEMBER_EXISTS gracefully (update instead of fail)',
    'Validate email format before API call to save rate limit'
  ],
  
  updatedAt: new Date('2026-02-05')
};
```

### 2.3 Slack Send Message

```typescript
const slackSendMessageCapability: NodeCapability = {
  id: 'cap_slack_send_message',
  nodeTypeId: 'slack_send_message',
  
  capabilities: [
    'send_text_message',
    'send_formatted_blocks',
    'send_to_channel',
    'send_direct_message',
    'send_to_user',
    'attach_files',
    'include_buttons',
    'include_links',
    'mention_users',
    'mention_channels',
  ],
  
  limitations: [
    'max_message_40000_chars',
    'max_blocks_50',
    'max_attachments_20',
    'bot_must_be_in_channel',
    'cannot_dm_users_who_havent_interacted',
    'rate_limited_per_workspace',
  ],
  
  requiredInputs: [
    {
      name: 'channel',
      type: 'string',
      description: 'Channel ID or name (use ID for reliability)',
      example: 'C1234567890'
    },
    {
      name: 'text',
      type: 'string',
      description: 'Message text (used as fallback for blocks)',
      example: 'New lead: John Doe (john@example.com)'
    }
  ],
  
  optionalInputs: [
    {
      name: 'blocks',
      type: 'array',
      description: 'Slack Block Kit blocks for rich formatting',
      example: [{ type: 'section', text: { type: 'mrkdwn', text: '*New Lead*' } }]
    },
    {
      name: 'username',
      type: 'string',
      description: 'Bot username override',
      example: 'Lead Bot'
    },
    {
      name: 'iconEmoji',
      type: 'string',
      description: 'Bot icon emoji',
      example: ':robot_face:'
    },
    {
      name: 'threadTs',
      type: 'string',
      description: 'Thread timestamp to reply in thread',
      example: '1234567890.123456'
    }
  ],
  
  outputSchema: {
    type: 'object',
    properties: {
      ok: { type: 'boolean' },
      channel: { type: 'string' },
      ts: { type: 'string', description: 'Message timestamp ID' },
      messageLink: { type: 'string', description: 'Link to message' }
    }
  },
  
  commonErrors: [
    {
      code: 'channel_not_found',
      cause: 'Channel does not exist or bot is not a member',
      solution: 'Invite the bot to the channel or use channel ID',
      retryable: false
    },
    {
      code: 'not_in_channel',
      cause: 'Bot is not a member of the channel',
      solution: 'Add the bot to the channel: /invite @botname',
      retryable: false
    },
    {
      code: 'rate_limited',
      cause: 'Too many messages sent',
      solution: 'Implement backoff, consider batching',
      retryable: true
    },
    {
      code: 'invalid_blocks',
      cause: 'Block Kit JSON malformed',
      solution: 'Validate blocks with Slack Block Kit Builder',
      retryable: false
    },
    {
      code: 'msg_too_long',
      cause: 'Message exceeds 40,000 characters',
      solution: 'Truncate or split message',
      retryable: false
    }
  ],
  
  retryPolicy: {
    maxRetries: 3,
    backoff: 'exponential',
    initialDelay: 1000,
    maxDelay: 60000,
    retryableErrors: ['rate_limited', 'timeout', 'server_error']
  },
  
  rateLimits: {
    requestsPerMinute: 50,  // Tier 3 methods
    burstLimit: 100
  },
  
  requiresAuth: true,
  authTypes: ['oauth2'],
  externalDependencies: ['slack_web_api'],
  
  useCases: [
    'new_lead_notification',
    'deal_update_alert',
    'error_notification',
    'daily_summary',
    'team_notification',
    'customer_action_alert'
  ],
  
  bestPractices: [
    'Use channel ID instead of name for reliability',
    'Include text fallback even when using blocks',
    'Use Block Kit for rich, actionable messages',
    'Thread related messages to reduce channel noise',
    'Mention @channel sparingly, use specific users',
    'Test message format with Block Kit Builder first'
  ],
  
  updatedAt: new Date('2026-02-05')
};
```

### 2.4 HubSpot Create Contact

```typescript
const hubspotCreateContactCapability: NodeCapability = {
  id: 'cap_hubspot_create_contact',
  nodeTypeId: 'hubspot_create_contact',
  
  capabilities: [
    'create_new_contact',
    'update_existing_contact',
    'upsert_by_email',
    'set_standard_properties',
    'set_custom_properties',
    'associate_with_company',
    'associate_with_deal',
    'trigger_workflows',
  ],
  
  limitations: [
    'email_unique_identifier',
    'max_properties_per_request_100',
    'custom_properties_must_exist',
    'rate_limited_per_app',
    'read_only_properties_ignored',
  ],
  
  requiredInputs: [
    {
      name: 'email',
      type: 'email',
      description: 'Contact email (primary identifier)',
      example: 'john@example.com'
    }
  ],
  
  optionalInputs: [
    {
      name: 'firstname',
      type: 'string',
      description: 'Contact first name',
      example: 'John'
    },
    {
      name: 'lastname',
      type: 'string',
      description: 'Contact last name',
      example: 'Doe'
    },
    {
      name: 'phone',
      type: 'string',
      description: 'Phone number',
      example: '+1-555-123-4567'
    },
    {
      name: 'company',
      type: 'string',
      description: 'Company name',
      example: 'Acme Inc'
    },
    {
      name: 'lifecyclestage',
      type: 'string',
      description: 'Lifecycle stage',
      example: 'lead',
      validation: { 
        enum: ['subscriber', 'lead', 'marketingqualifiedlead', 'salesqualifiedlead', 'opportunity', 'customer', 'evangelist'] 
      }
    },
    {
      name: 'leadsource',
      type: 'string',
      description: 'Lead source',
      example: 'Website Form'
    },
    {
      name: 'customProperties',
      type: 'object',
      description: 'Additional custom properties',
      example: { utm_source: 'google', campaign: 'summer-2026' }
    }
  ],
  
  outputSchema: {
    type: 'object',
    properties: {
      id: { type: 'string', description: 'HubSpot contact ID' },
      email: { type: 'string' },
      created: { type: 'boolean', description: 'True if new, false if updated' },
      properties: { type: 'object' },
      createdAt: { type: 'string', format: 'date-time' },
      updatedAt: { type: 'string', format: 'date-time' }
    }
  },
  
  commonErrors: [
    {
      code: 'CONTACT_EXISTS',
      cause: 'Contact with this email already exists',
      solution: 'Use upsert mode or search before create',
      retryable: false
    },
    {
      code: 'PROPERTY_DOESNT_EXIST',
      cause: 'Custom property not defined in HubSpot',
      solution: 'Create the property in HubSpot settings first',
      retryable: false
    },
    {
      code: 'RATE_LIMIT',
      cause: 'API rate limit exceeded',
      solution: 'Implement exponential backoff',
      retryable: true
    },
    {
      code: 'INVALID_PROPERTY_VALUE',
      cause: 'Value does not match property type',
      solution: 'Check property type (date, number, enum) and format value',
      retryable: false
    },
    {
      code: 'INVALID_EMAIL',
      cause: 'Email format invalid',
      solution: 'Validate email before sending',
      retryable: false
    }
  ],
  
  retryPolicy: {
    maxRetries: 3,
    backoff: 'exponential',
    initialDelay: 1000,
    maxDelay: 60000,
    retryableErrors: ['RATE_LIMIT', 'TIMEOUT', 'SERVER_ERROR']
  },
  
  rateLimits: {
    requestsPerSecond: 10,  // 100 per 10 seconds
    requestsPerDay: 500000  // Depends on plan
  },
  
  requiresAuth: true,
  authTypes: ['oauth2', 'api_key'],
  externalDependencies: ['hubspot_crm_api_v3'],
  
  useCases: [
    'lead_capture',
    'contact_sync',
    'customer_onboarding',
    'form_to_crm',
    'import_contacts',
    'update_contact_info'
  ],
  
  bestPractices: [
    'Use upsert to handle existing contacts gracefully',
    'Set lifecycle stage to track lead progress',
    'Include lead source for attribution',
    'Map UTM parameters to custom properties',
    'Validate email format before API call',
    'Create custom properties in HubSpot before using'
  ],
  
  updatedAt: new Date('2026-02-05')
};
```

---

## 3. Node Registry Service

```typescript
// node-registry.service.ts

@Injectable()
export class NodeRegistryService {
  constructor(private readonly prisma: PrismaService) {}

  /**
   * Get all node capabilities (for AI context)
   */
  async getAllCapabilities(): Promise<NodeCapability[]> {
    return this.prisma.nodeCapability.findMany({
      include: { nodeType: true }
    });
  }

  /**
   * Find nodes that can perform a specific action
   */
  async findNodesForCapability(capability: string): Promise<NodeCapability[]> {
    return this.prisma.nodeCapability.findMany({
      where: {
        capabilities: { has: capability }
      }
    });
  }

  /**
   * Find nodes for a use case
   */
  async findNodesForUseCase(useCase: string): Promise<NodeCapability[]> {
    return this.prisma.nodeCapability.findMany({
      where: {
        useCases: { has: useCase }
      }
    });
  }

  /**
   * Get condensed capability summary for AI prompts
   */
  async getCapabilitySummaryForAI(): Promise<string> {
    const capabilities = await this.getAllCapabilities();
    
    return capabilities.map(cap => `
## ${cap.nodeType.displayName}
- Capabilities: ${cap.capabilities.join(', ')}
- Use cases: ${cap.useCases.join(', ')}
- Requires: ${cap.requiresAuth ? cap.authTypes.join('/') : 'No auth'}
- Limitations: ${cap.limitations.slice(0, 3).join(', ')}
`).join('\n');
  }

  /**
   * Validate node configuration against capability schema
   */
  async validateNodeConfig(
    nodeTypeId: string,
    config: Record<string, any>
  ): Promise<{ valid: boolean; errors: string[] }> {
    const capability = await this.prisma.nodeCapability.findUnique({
      where: { nodeTypeId }
    });

    if (!capability) {
      return { valid: false, errors: ['Node type not found'] };
    }

    const errors: string[] = [];

    // Check required inputs
    for (const input of capability.requiredInputs) {
      if (!(input.name in config) || config[input.name] === undefined) {
        errors.push(`Missing required field: ${input.name}`);
      }
    }

    // Validate types and constraints
    for (const input of [...capability.requiredInputs, ...capability.optionalInputs]) {
      if (config[input.name] !== undefined) {
        const validation = this.validateField(config[input.name], input);
        if (!validation.valid) {
          errors.push(`${input.name}: ${validation.error}`);
        }
      }
    }

    return { valid: errors.length === 0, errors };
  }

  private validateField(value: any, input: InputField): { valid: boolean; error?: string } {
    // Type checking
    if (input.type === 'email' && !this.isValidEmail(value)) {
      return { valid: false, error: 'Invalid email format' };
    }
    if (input.type === 'url' && !this.isValidUrl(value)) {
      return { valid: false, error: 'Invalid URL format' };
    }
    
    // Validation rules
    if (input.validation) {
      if (input.validation.enum && !input.validation.enum.includes(value)) {
        return { valid: false, error: `Must be one of: ${input.validation.enum.join(', ')}` };
      }
      if (input.validation.minLength && value.length < input.validation.minLength) {
        return { valid: false, error: `Minimum length: ${input.validation.minLength}` };
      }
      if (input.validation.pattern && !new RegExp(input.validation.pattern).test(value)) {
        return { valid: false, error: `Invalid format` };
      }
    }

    return { valid: true };
  }

  private isValidEmail(email: string): boolean {
    return /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(email);
  }

  private isValidUrl(url: string): boolean {
    try {
      new URL(url);
      return true;
    } catch {
      return false;
    }
  }
}
```

---

## 4. Seed Data for Node Capabilities

```typescript
// prisma/seed-capabilities.ts

const nodeCapabilities = [
  // Triggers
  { nodeType: 'webhook', capabilities: ['receive_http', 'validate_payload'], ... },
  { nodeType: 'schedule', capabilities: ['run_on_schedule', 'cron_expression'], ... },
  { nodeType: 'typeform_trigger', ... },
  { nodeType: 'webflow_form_trigger', ... },
  { nodeType: 'hubspot_form_trigger', ... },
  
  // Actions - Email
  { nodeType: 'mailchimp_add_subscriber', ... },
  { nodeType: 'convertkit_add_subscriber', ... },
  { nodeType: 'activecampaign_add_contact', ... },
  { nodeType: 'sendgrid_send_email', ... },
  
  // Actions - CRM
  { nodeType: 'hubspot_create_contact', ... },
  { nodeType: 'hubspot_update_contact', ... },
  { nodeType: 'salesforce_create_lead', ... },
  { nodeType: 'pipedrive_create_person', ... },
  
  // Actions - Notifications
  { nodeType: 'slack_send_message', ... },
  { nodeType: 'discord_send_message', ... },
  { nodeType: 'teams_send_message', ... },
  { nodeType: 'email_send', ... },
  
  // Actions - Data
  { nodeType: 'google_sheets_append', ... },
  { nodeType: 'airtable_create_record', ... },
  { nodeType: 'notion_create_page', ... },
  
  // Logic
  { nodeType: 'if_condition', capabilities: ['branch', 'compare', 'filter'], ... },
  { nodeType: 'switch', capabilities: ['multi_branch', 'case_match'], ... },
  { nodeType: 'loop', capabilities: ['iterate', 'batch_process'], ... },
  { nodeType: 'transform', capabilities: ['map_data', 'filter', 'aggregate'], ... },
  
  // Utilities
  { nodeType: 'http_request', capabilities: ['call_api', 'any_rest_api'], ... },
  { nodeType: 'wait', capabilities: ['delay', 'pause'], ... },
  { nodeType: 'code', capabilities: ['custom_javascript', 'transform'], ... },
];
```
