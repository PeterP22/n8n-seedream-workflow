# SeedDream 4.5 Image Generator

![n8n Workflow](seedream-image-generator-workflow.png)

An n8n workflow that generates AI images using ByteDance's SeedDream 4.5 model via the Replicate API, with automatic delivery to Telegram.

## What Was Built

A complete image generation pipeline with:

- **Web Form Interface** - Users enter prompts, select image size (2K/4K), and aspect ratio
- **Replicate API Integration** - Submits generation requests to SeedDream 4.5
- **Async Polling Loop** - Waits for generation to complete with automatic retries
- **Telegram Delivery** - Generated images are automatically sent to your Telegram
- **Success/Error Handling** - Returns confirmation or displays error message

### Workflow Architecture

```
┌─────────────────┐    ┌──────────────────┐    ┌─────────────────┐
│  Form Trigger   │───▶│ Create Prediction│───▶│ Wait 5 seconds  │
│  (User Input)   │    │ (Replicate API)  │    │                 │
└─────────────────┘    └──────────────────┘    └────────┬────────┘
                                                        │
                       ┌────────────────────────────────▼────────┐
                       │          Check Prediction Status        │◀─┐
                       │     GET /v1/predictions/{id}            │  │
                       └────────────────────┬────────────────────┘  │
                                            │                       │
                       ┌────────────────────▼────────────────────┐  │
                       │        Is Generation Complete?          │  │
                       │         status == "succeeded"           │  │
                       └──────┬─────────────────────────┬────────┘  │
                              │ YES                     │ NO        │
                              ▼                         ▼           │
               ┌───────────────────────┐    ┌─────────────────────┐ │
               │    Download Image     │    │ Did Generation Fail?│ │
               │  (Fetch from URL)     │    │  status == "failed" │ │
               └───────────┬───────────┘    └──────┬─────────┬────┘ │
                           │                       │ YES     │ NO   │
                           ▼                       ▼         ▼      │
               ┌───────────────────────┐  ┌──────────┐ ┌──────────┐ │
               │   Send to Telegram    │  │ Return   │ │ Wait 5s  │─┘
               │  (Photo + Caption)    │  │ Error    │ │ & Retry  │
               └───────────┬───────────┘  └──────────┘ └──────────┘
                           │
                           ▼
               ┌───────────────────────┐
               │  Return Confirmation  │
               │   (Success Page)      │
               └───────────────────────┘
```

## Setup

### Prerequisites

- n8n running locally or self-hosted (Docker recommended)
- Replicate API account and token
- Telegram account

### Running the Server

**Start n8n:**
```bash
cd /path/to/self-hosted-ai-starter-kit
docker compose --profile cpu up -d
```

**Stop n8n:**
```bash
docker compose down
```

**Quick reference:**
- `docker compose down` - stops and removes containers
- `docker compose stop` - stops without removing (faster restart)
- `docker compose logs -f n8n` - view logs if troubleshooting

Once running, access n8n at http://localhost:5678

### 1. Replicate API Credential

1. Get your API token from [Replicate](https://replicate.com/account/api-tokens)
2. In n8n, go to **Credentials** → **Add Credential**
3. Select **Header Auth** and configure:
   - Name: `Replicate API`
   - Header Name: `Authorization`
   - Header Value: `Bearer r8_your_token_here`

### 2. Telegram Bot Setup

1. Open Telegram and message **@BotFather**
2. Send `/newbot` and follow the prompts
3. Copy the **API Token** you receive

4. Message your new bot (send "hi")
5. Get your Chat ID by visiting:
   ```
   https://api.telegram.org/bot<YOUR_BOT_TOKEN>/getUpdates
   ```
   Look for `"chat":{"id":` - that number is your Chat ID

6. In n8n, go to **Credentials** → **Add Credential**
7. Select **Telegram** and enter your bot's Access Token

### 3. Import Workflow

Import the workflow from `workflows/seedream-image-generator.json` and update:
- Link the Replicate API credential to HTTP Request nodes
- Link the Telegram credential to the "Send to Telegram" node
- Update the Chat ID in the Telegram node parameters

### Workflow URL

```
http://localhost:5678/form/seedream-form
```

## Usage

1. Navigate to the form URL
2. Enter your image prompt
3. Select size (2K or 4K resolution)
4. Choose aspect ratio (1:1, 16:9, 9:16, 4:3, 3:4)
5. Submit and wait ~20-40 seconds
6. **Receive the image directly in Telegram!**

## Technical Details

### SeedDream 4.5 Model

- **Model**: `bytedance/seedream-4.5`
- **Version**: `8356ab00a2acd0f79338ecf1ffa0e32493c6f7cdfc7178b5cfbdb1461202fdc2`
- **Resolutions**: 2K (2048x2048), 4K (4096x4096)
- **API Docs**: https://replicate.com/bytedance/seedream-4.5/api

### Key Implementation Notes

1. **Async Polling Pattern**: Replicate returns a prediction ID immediately. The workflow polls `/v1/predictions/{id}` until status is `succeeded` or `failed`.

2. **Wait Node Context**: After n8n Wait nodes, use `$json.id` to access data (not `$('NodeName').json.id`) because the execution context changes.

3. **Form Completion**: Use `n8n-nodes-base.form` with `operation: "completion"` instead of "Respond to Webhook" nodes when using Form Triggers.

4. **Binary Data for Telegram**: Download the image as binary (`responseFormat: "file"`) before sending to Telegram with `binaryData: true`.

---

## Future Improvements

### Quick Wins

- [ ] **Display Image Inline** - Embed the generated image directly in the completion page using HTML/image tags

- [ ] **Negative Prompts** - Add a field for negative prompts to exclude unwanted elements from generations

- [ ] **Email Notification** - Send the generated image via email as an alternative to Telegram

### Enhanced Features

- [ ] **Image Gallery Database** - Store prompts and image URLs in a database (PostgreSQL, Airtable, Notion) to build a searchable gallery

- [ ] **Prompt Enhancement** - Add an OpenAI/Claude node before generation to enhance and optimize user prompts

- [ ] **Multiple Models** - Add a dropdown to choose between different models (FLUX, Stable Diffusion, DALL-E) with dynamic parameter options

- [ ] **Image-to-Image** - Add file upload support for img2img workflows and style transfer

- [ ] **Batch Generation** - Generate multiple variations of the same prompt in parallel

- [ ] **Webhook Notifications** - Instead of polling, use Replicate's webhook feature for instant completion callbacks

### Advanced Integrations

- [x] **Telegram Bot** - ~~Create a Telegram bot interface for mobile-friendly image generation~~ ✅ Implemented!

- [ ] **Discord/Slack Bot** - Trigger image generation from chat commands and post results back to channels

- [ ] **API Endpoint** - Expose the workflow as a REST API for integration with other applications

- [ ] **Cost Tracking** - Log API usage and costs per generation to monitor spending

### UI/UX Improvements

- [ ] **Progress Indicator** - Show real-time generation progress instead of static "processing" message

- [ ] **History View** - Let users see their previous generations in the form interface

- [ ] **Prompt Templates** - Offer pre-built prompt templates for common use cases (portraits, landscapes, logos)

- [ ] **Style Presets** - Quick-select artistic styles (photorealistic, anime, oil painting, etc.)

---

## Lessons Learned

1. **n8n Form Triggers** require Form nodes (not Respond to Webhook) for responses
2. **Replicate API** uses version IDs, not model names, in the request body
3. **Wait nodes** break the execution context - reference data with `$json` not `$('NodeName')`
4. **Polling patterns** work well for async APIs - 5 second intervals balance responsiveness and API usage
5. **Docker file access** requires `N8N_RESTRICT_FILE_ACCESS_TO` env var for local file operations
6. **Telegram delivery** is more reliable than local file saving in containerized environments

## Resources

- [n8n Documentation](https://docs.n8n.io/)
- [Replicate API Reference](https://replicate.com/docs/reference/http)
- [SeedDream 4.5 on Replicate](https://replicate.com/bytedance/seedream-4.5)
- [Telegram Bot API](https://core.telegram.org/bots/api)

---

*Built with n8n + Replicate + SeedDream 4.5 + Telegram*
