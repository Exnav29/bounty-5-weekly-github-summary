# Weekly GitHub Dev Summary — n8n + OpenRouter + Claude

This submission includes two implementations:

1. **Reference workflow** — a clean implementation of the weekly GitHub activity summary.
2. **Bulletproof Edition** — the same business function with production-oriented validation, controlled delivery, retries, and an optional error-notification workflow.

## Why OpenRouter

The workflow intentionally uses n8n's native **OpenRouter Chat Model** with:

`anthropic/claude-sonnet-4`

OpenRouter was chosen to keep the AI layer provider-flexible. The workflow can continue using Claude today while allowing a future operator to change models without rewriting the workflow's business logic, prompts, GitHub collection, or delivery steps.

This is a deliberate implementation choice. The original bounty wording names the Claude API directly, so reviewers should be aware that this version accesses Claude through OpenRouter rather than Anthropic billing directly.

## Files

### `01-reference-openrouter.json`
Simple reviewer-friendly implementation.

- Friday 5 PM schedule
- GitHub commits
- closed issues
- merged pull requests
- AI Agent
- OpenRouter Chat Model using Claude Sonnet 4
- EN/FR reporting
- Discord delivery
- retries on external HTTP calls

### `02-bulletproof-main.json`
Production-oriented implementation.

Adds:
- preflight configuration validation
- GitHub response contract validation
- AI output validation
- external-call retries
- switchable success-report delivery
- explicit failure rather than silently accepting malformed data

Set `enableSuccessDelivery` in the Configuration node to `false` if the report should be generated and validated without posting to Discord.

### `03-optional-error-handler.json`
Optional n8n error workflow.

When enabled and assigned as the main workflow's Error Workflow, it can:
- receive failed workflow executions
- identify workflow, execution, last node, and error
- send a Discord failure notification
- retry the notification if Discord temporarily fails

Set `enableErrorNotifications` to `false`, or simply do not assign/activate the error workflow, to disable alerts.

## Setup

1. Import the desired main workflow.
2. Select/create the GitHub credential.
3. Select/create the OpenRouter credential.
4. Replace the placeholder Discord webhook.
5. Test manually and activate.

For the Bulletproof Edition, import `03-optional-error-handler.json` and select it in the main workflow's **Settings → Error Workflow** if operational alerts are desired.

## Security

The submitted files intentionally contain no live Discord webhook URL and no usable credential IDs.

Never commit:
- GitHub tokens
- OpenRouter API keys
- Discord webhook URLs
- other reusable secrets

## Reliability philosophy

Retries solve transient service errors. They do not solve malformed data, missing configuration, invalid AI output, or silent operational failures.

The Bulletproof Edition therefore separates:
- **retryable provider failures** from
- **contract/validation failures** from
- **operator notification**

That distinction is the primary difference between a workflow that can run and one that can be operated.
