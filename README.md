# SeedDream 4.5 Image Generator

An n8n workflow that generates AI images using ByteDance's SeedDream 4.5 model via the Replicate API.

## What Was Built

A complete image generation pipeline with:

- **Web Form Interface** - Users enter prompts, select image size (2K/4K), and aspect ratio
- **Replicate API Integration** - Submits generation requests to SeedDream 4.5
- **Async Polling Loop** - Waits for generation to complete with automatic retries
- **Success/Error Handling** - Returns generated image URL or displays error message

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
                    ┌─────────────────┐    ┌─────────────────────┐  │
                    │ Return Image    │    │ Did Generation Fail?│  │
                    │ (Success Page)  │    │  status == "failed" │  │
                    └─────────────────┘    └──────┬─────────┬─────┘  │
                                                  │ YES     │ NO     │
                                                  ▼         ▼        │
                                        ┌──────────┐  ┌──────────┐   │
                                        │ Return   │  │ Wait 5s  │───┘
                                        │ Error    │  │ & Retry  │
                                        └──────────┘  └──────────┘
```

## Setup

### Prerequisites

- n8n running locally or self-hosted
- Replicate API account and token

### Environment Variables

Create a `.env` file:

```bash
REPLICATE_API_TOKEN=r8_your_token_here
```

### Credential Setup in n8n

1. Go to **Credentials** in n8n
2. Create new **Header Auth** credential:
   - Name: `Replicate API`
   - Header Name: `Authorization`
   - Header Value: `Bearer r8_your_token_here`

### Workflow Import

The workflow ID is `5nMg2KkpKfHFbouL` and runs at:
```
http://localhost:5678/form/seedream-form
```

## Usage

1. Navigate to the form URL
2. Enter your image prompt
3. Select size (2K or 4K resolution)
4. Choose aspect ratio (1:1, 16:9, 9:16, 4:3, 3:4)
5. Submit and wait ~20-40 seconds for generation
6. View your generated image URL

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

---

## Future Improvements

### Quick Wins

- [ ] **Display Image Inline** - Embed the generated image directly in the completion page using HTML/image tags instead of just showing the URL

- [ ] **Negative Prompts** - Add a field for negative prompts to exclude unwanted elements from generations

- [ ] **Save to Cloud Storage** - Auto-upload generated images to S3, Google Drive, or Cloudinary for permanent storage (Replicate URLs expire)

- [ ] **Email Notification** - Send the generated image via email so users don't have to wait on the page

### Enhanced Features

- [ ] **Image Gallery Database** - Store prompts and image URLs in a database (PostgreSQL, Airtable, Notion) to build a searchable gallery

- [ ] **Prompt Enhancement** - Add an OpenAI/Claude node before generation to enhance and optimize user prompts

- [ ] **Multiple Models** - Add a dropdown to choose between different models (FLUX, Stable Diffusion, DALL-E) with dynamic parameter options

- [ ] **Image-to-Image** - Add file upload support for img2img workflows and style transfer

- [ ] **Batch Generation** - Generate multiple variations of the same prompt in parallel

- [ ] **Webhook Notifications** - Instead of polling, use Replicate's webhook feature for instant completion callbacks

### Advanced Integrations

- [ ] **Discord/Slack Bot** - Trigger image generation from chat commands and post results back to channels

- [ ] **Telegram Bot** - Create a Telegram bot interface for mobile-friendly image generation

- [ ] **API Endpoint** - Expose the workflow as a REST API for integration with other applications

- [ ] **Cost Tracking** - Log API usage and costs per generation to monitor spending

- [ ] **Rate Limiting** - Add authentication and rate limiting to prevent abuse

- [ ] **Queue System** - For high-traffic scenarios, implement a job queue with status tracking

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

## Resources

- [n8n Documentation](https://docs.n8n.io/)
- [Replicate API Reference](https://replicate.com/docs/reference/http)
- [SeedDream 4.5 on Replicate](https://replicate.com/bytedance/seedream-4.5)

---

*Built with n8n + Replicate + SeedDream 4.5*
