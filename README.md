# n8n Webhook AI Agent Automation Workflow

## Overview

This workflow exposes an n8n AI Agent through a **Webhook** endpoint. External applications such as Postman can send requests to the webhook, and the AI Agent can process the request using an OpenAI Chat Model, conversation memory, and multiple connected tools.

The workflow supports tasks such as:

- Sending emails through Gmail
- Appending information to Google Sheets
- Creating calendar events in Google Calendar
- Creating and updating Google Docs
- Searching Amazon products through SerpApi

## Workflow Architecture

```text
Webhook
   |
   v
AI Agent
   |
   +-- OpenAI Chat Model
   +-- Simple Memory
   +-- Send a message in Gmail
   +-- Append row in Google Sheets
   +-- Create an event in Google Calendar
   +-- Create a document in Google Docs
   +-- Amazon search in SerpApi
   +-- Update a document in Google Docs
```

## Components

### Webhook

The Webhook node is the entry point for external requests from Postman, applications, or other systems.

The screenshot shows a `GET` method. For structured requests, `POST` with a JSON body may be more suitable.

Verify:

- HTTP method
- Webhook path
- Authentication settings
- Response mode
- Incoming query parameters or body fields

### AI Agent

The AI Agent interprets the incoming request and selects the appropriate connected tool.

It is connected to the OpenAI Chat Model, Simple Memory, Gmail, Google Sheets, Google Calendar, Google Docs, and SerpApi tools.

The agent should be instructed to:

- Understand the user's request
- Select the appropriate tool
- Ask for missing information
- Confirm completion only after a tool succeeds
- Never claim an action was completed if the tool failed

### OpenAI Chat Model

Provides the language model used by the AI Agent.

Verify the configured credentials, model, token limits, and system instructions.

### Simple Memory

Stores conversation context for the AI Agent. Use a stable and unique session ID for each user or conversation so that sessions remain separate.

Recommended checks:

- Use a stable session ID
- Separate users' conversations
- Avoid excessively large memory values
- Confirm the memory node is connected to the AI Agent

### Gmail: Send a Message

Allows the AI Agent to send email messages through Gmail.

Required information normally includes:

- Recipient email address
- Subject
- Email body

Example request:

```text
Send an email to user@example.com with the subject "KOART Update" and the body "KOART is back online."
```

Verify Gmail credentials, permissions, recipient mapping, subject mapping, and message body mapping.

### Google Sheets: Append Row

Appends data to a Google Sheets worksheet.

Verify:

- Google Sheets credentials
- Spreadsheet ID
- Sheet name
- Column mapping
- Values received from the AI Agent

### Google Calendar: Create Event

Creates a calendar event. The agent may require the event title, start time, end time, time zone, attendees, and description.

The agent should request missing scheduling details before creating an event.

### Google Docs: Create Document

Creates a new Google Docs document using the supplied title and content.

Verify the document title, content, and destination settings if applicable.

### SerpApi: Amazon Search

Searches Amazon products through SerpApi.

Potential inputs include product keywords, search terms, and optional filters. The agent should distinguish search results from confirmed availability or purchase status.

Verify SerpApi credentials, search engine configuration, query mapping, and result handling.

### Google Docs: Update Document

Updates an existing Google Docs document.

Verify the document ID and update instructions before making changes. Avoid overwriting content unintentionally.

## Postman Testing

### GET Request

If the Webhook remains configured for `GET`, a request may look like:

```text
https://YOUR_N8N_DOMAIN/webhook/YOUR_WEBHOOK_PATH?message=Send%20an%20email%20to%20user@example.com
```

The exact query parameter depends on the Webhook configuration.

### POST Request (Recommended)

For structured requests, configure Postman as follows:

**Method**

```text
POST
```

**Header**

```text
Content-Type: application/json
```

**Body**

```json
{
  "message": "Send an email to user@example.com",
  "subject": "KOART Update",
  "body": "KOART is back online."
}
```

For a typical POST Webhook payload, the message may be available through:

```javascript
{{ $json.body.message }}
```

For a GET request, the message may be available through:

```javascript
{{ $json.query.message }}
```

Confirm the actual structure by inspecting the Webhook node output in n8n.

## Suggested AI Agent Instructions

```text
You are an AI assistant that can perform tasks using connected tools.

Understand the user's request and select the appropriate tool.

For email requests:
- Identify the recipient, subject, and body.
- Ask for missing information when necessary.
- Use the Gmail tool to send the email.
- Confirm success only after the Gmail tool completes successfully.

For Google Sheets requests:
- Identify the spreadsheet, worksheet, and row values.
- Ask for missing information before appending data.

For Google Calendar requests:
- Identify the event title, start time, end time, and time zone.
- Ask for missing scheduling details before creating an event.

For Google Docs requests:
- Confirm the document to create or update.
- Avoid overwriting content unless explicitly requested.

For Amazon searches:
- Use the SerpApi Amazon search tool.
- Distinguish search results from confirmed availability or purchase status.

Never claim that an action was completed if the connected tool fails.
```

## Execution Flow

1. An external application sends a request to the Webhook.
2. The Webhook passes the request to the AI Agent.
3. The AI Agent interprets the request.
4. The OpenAI Chat Model supports the reasoning and response.
5. Simple Memory provides conversation context.
6. The AI Agent selects and invokes the required tool.
7. The selected tool performs the requested action.
8. The AI Agent returns a result or asks for missing information.

## Testing Checklist

- [ ] Webhook receives a request from Postman.
- [ ] Incoming query parameters or JSON body are visible in the Webhook output.
- [ ] AI Agent receives the actual user message.
- [ ] OpenAI Chat Model responds successfully.
- [ ] Simple Memory uses a stable session ID.
- [ ] Gmail sends an email successfully.
- [ ] Google Sheets appends a row successfully.
- [ ] Google Calendar creates an event successfully.
- [ ] Google Docs creates a document successfully.
- [ ] SerpApi Amazon Search returns relevant results.
- [ ] Google Docs updates the intended document.
- [ ] Missing information is handled with clarification questions.
- [ ] Tool failures are reported accurately.

## Troubleshooting

### The AI Agent Says the Input Is Empty

Inspect the Webhook output and confirm that the request contains the expected message field.

For POST requests, the message may be accessed using:

```javascript
{{ $json.body.message }}
```

For GET requests, it may be accessed using:

```javascript
{{ $json.query.message }}
```

Use the expression that matches the actual Webhook payload.

### Gmail Does Not Send the Email

Check Gmail authentication, permissions, recipient address, subject and body mapping, the AI Agent tool description, and the n8n execution log.

### The AI Agent Selects the Wrong Tool

Improve the system prompt and tool descriptions. Explain when each tool should be used and what information is required.

### Memory Does Not Work Correctly

Check the session ID, ensure it does not change unexpectedly, confirm that users do not share the same session, and review memory size.

### Google Docs Updates the Wrong Document

Confirm that the correct document ID is supplied and instruct the AI Agent to verify the target document before updating it.

## Security Considerations

- Store credentials using n8n's credential manager.
- Do not expose API keys in Webhook URLs or request bodies.
- Consider adding Webhook authentication.
- Validate incoming requests before performing sensitive actions.
- Avoid sending confidential information unnecessarily.
- Restrict Google, Gmail, and SerpApi permissions to the minimum required.
- Review workflow execution logs for sensitive information.

## Maintenance

Periodically review AI model usage, memory behavior, tool permissions, Webhook security, Gmail recipients, Google Sheets and Docs access, Calendar permissions, SerpApi usage, and workflow errors.

## Notes

This README is based on the workflow structure visible in the provided screenshot. The exact Webhook URL, field expressions, system prompt, credentials, document IDs, spreadsheet IDs, calendar settings, and Gmail configuration are not visible and should be documented after reviewing the individual n8n node settings.
