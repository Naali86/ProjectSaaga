# Project Saaga - Complete Workflow Documentation

This document contains all three workflows for demonstration purposes.
**All API keys and sensitive credentials have been removed.**

---

## Workflow Overview

### WF1: Script Generation (Automated)
- **Trigger**: Schedule (daily/weekly) or Manual
- **Purpose**: Generate news scripts from trending topics
- **Output**: Saves to Google Sheets with `id`, `title`, `script`, `image_prompt`
- **Status**: `SCRIPT_READY`

### WF2: Video Generation (Manual)
- **Trigger**: Manual - operator selects News ID
- **Purpose**: Create lip-synced AI video with voiceover
- **Output**: Video saved to `Saaga_Previews` folder
- **Status**: `PREVIEW_READY`

### WF3: YouTube Publishing (Manual)
- **Trigger**: Manual - after reviewing preview
- **Purpose**: Upload approved video to YouTube
- **Output**: Published video on YouTube channel
- **Status**: `PUBLISHED`

---

## Technology Stack

### AI Services
- **OpenAI GPT-4**: News script generation
- **OpenAI TTS (Nova voice)**: Audio generation (to be replaced with ElevenLabs)
- **Fal.ai Infinitalk**: Video-to-video lip-sync animation

### Cloud Services
- **Google Sheets**: Data storage and tracking
- **Google Drive**: Intermediate file storage
- **YouTube Data API v3**: Video publishing

### Platform
- **n8n**: Workflow automation (self-hosted v1.120.4)

---

## Data Flow

```
┌──────────────────────────────────────────────────────┐
│                    WF1: Script Factory                │
│                                                       │
│  Schedule → OpenAI GPT-4 → Google Sheets             │
│                                                       │
│  Output: id, title, script, image_prompt             │
│  Status: SCRIPT_READY                                │
└──────────────────────────────────────────────────────┘
                           ↓
┌──────────────────────────────────────────────────────┐
│                  WF2: Video Generator                 │
│                                                       │
│  Manual Trigger → Input News ID                      │
│  → Get Script Data from Sheets                       │
│  → OpenAI TTS (audio)                                │
│  → Upload Audio to Drive                             │
│  → Fal.ai Infinitalk (video generation)              │
│  → Save to Saaga_Previews                            │
│  → Update Status                                     │
│                                                       │
│  Output: Video file in preview folder                │
│  Status: PREVIEW_READY                               │
└──────────────────────────────────────────────────────┘
                           ↓
                    (Manual Review)
                           ↓
┌──────────────────────────────────────────────────────┐
│                 WF3: YouTube Publisher                │
│                                                       │
│  Manual Trigger → Input News ID                      │
│  → Get Metadata from Sheets                          │
│  → Download Video from Preview                       │
│  → Upload to YouTube (with metadata)                 │
│  → Update Status                                     │
│                                                       │
│  Output: Published YouTube video                     │
│  Status: PUBLISHED                                   │
└──────────────────────────────────────────────────────┘
```

---

## Google Sheets Structure

**Document ID**: `1M1rAqnnCbAFXqTJtn_sg_iTI5ovlKb_j6Puz1ASv3bg`
**Sheet Name**: `Taulukko1`

### Columns:
- `id` (string) - Unique news ID (e.g., "t2tj9j")
- `title` (string) - Video title
- `script` (string) - News script for TTS
- `image_prompt` (string) - Prompt for video generation
- `source_title` (string) - Source name
- `source_url` (string) - Source link
- `status` (string) - Workflow status
  - "SCRIPT_READY" - After WF1
  - "PREVIEW_READY" - After WF2
  - "PUBLISHED" - After WF3
- `youtube_url` (string) - Published video URL

---

## Google Drive Folder Structure

### Temp Audio
- **Folder ID**: `1ZZa6zVWk3AyF3zv68LBrFzI_WmdySXq0`
- **Purpose**: Temporary audio files from OpenAI TTS
- **Lifecycle**: Created in WF2, can be deleted after video generation

### Saaga Previews
- **Folder ID**: `1mKzfiq5FQRcqaO_e9HZ5lgUJiyv2EQr1`
- **Purpose**: Preview videos for manual review
- **Lifecycle**: Created in WF2, used in WF3
- **Filename Format**: `{date}_{news_id}.mp4` (e.g., "20251202_t2tj9j.mp4")

### Base Video
- **File ID**: `1NLHSXHw48kdbghusj_FoQgm0Prli5B8b`
- **Purpose**: Base video for Fal.ai lip-sync
- **Direct URL**: `https://drive.google.com/uc?export=download&id=1NLHSXHw48kdbghusj_FoQgm0Prli5B8b`

---

## WF1: Script Generation

### Nodes:
1. **Schedule Trigger** - Runs automatically (daily/weekly)
2. **Generate News Script** (OpenAI) - Creates script from trending topics
3. **Save to Sheets** (Google Sheets) - Stores script data

### Required Credentials:
- OpenAI API Key
- Google Sheets OAuth2

### Configuration:
- **Model**: GPT-4
- **Language**: Finnish
- **Output**: News script + image prompt for video generation

---

## WF2: Video Generation

### Nodes:
1. **Manual Trigger** - Operator starts workflow
2. **Set News ID** - Input specific news item to process
3. **Get Script Data** (Google Sheets) - Fetch script by ID
4. **Generate Audio** (OpenAI TTS) - Text-to-speech with Nova voice
5. **Debug Binary** (Code) - Ensure binary data named correctly
6. **Upload Audio to Drive** (Google Drive) - Save to temp folder
7. **Set API Data** (Code) - Prepare Fal.ai request parameters
8. **Generate Video** (Fal.ai HTTP Request) - Create lip-synced video
   - **video_url**: Base video from Drive
   - **audio_url**: Generated audio from Drive
   - **prompt**: From `image_prompt` column
   - **num_frames**: 145 (~6 seconds at 24fps)
   - **resolution**: 480p
   - **seed**: 42
9. **Wait 60s** - Allow processing time
10. **Check Status** (HTTP Request) - Poll Fal.ai for completion
11. **Is Completed?** (If) - Check if status = "COMPLETED"
12. **Set Request ID** (Code) - Ensure ID available for result fetch
13. **Get Result URL** (HTTP Request) - Fetch final video URL
14. **Download Video** (Google Drive) - Download completed video
15. **Save to Preview** (Google Drive) - Save to preview folder
16. **Update Sheet Status** (Google Sheets) - Set status = "PREVIEW_READY"

### Required Credentials:
- Google Sheets OAuth2
- OpenAI API Key
- Google Drive OAuth2
- Fal.ai API Key (Header Auth: `Key <YOUR_KEY>`)

### Critical Parameters:
- **Fal.ai HTTP Request**:
  - `sendBody: true`
  - `sendHeaders: true`
  - `specifyBody: "json"`
  - `headerParameters`: Content-Type: application/json

---

## WF3: YouTube Publishing

### Nodes:
1. **Manual Trigger** - Operator starts after reviewing preview
2. **Set News ID** - Input same ID used in WF2
3. **Get Metadata** (Google Sheets) - Fetch title, source info
4. **List Preview Folder** (Google Drive) - List files in preview folder
5. **Filter by News ID** (If) - Find file matching news ID
6. **Download Video** (Google Drive) - Download video file
7. **Upload to YouTube** (YouTube) - Publish with metadata
   - **Title**: From Sheet `title` column
   - **Description**: "Lähde: {source_title}\n{source_url}"
   - **Tags**: "uutiset,Suomi,news"
   - **Category**: 25 (News & Politics)
   - **Privacy**: Public
8. **Update Sheet Status** (Google Sheets) - Set status = "PUBLISHED", save YouTube URL

### Required Credentials:
- Google Sheets OAuth2
- Google Drive OAuth2
- YouTube OAuth2 (Brand Account)

### Important Notes:
- **Brand Account**: Ensure YouTube credential is authenticated with correct brand account
- **Metadata**: Title and description pulled from Google Sheets
- **Filename**: Must match format `{date}_{news_id}.mp4` for filtering to work

---

## Known Issues & TODO

### Critical Fixes
1. YouTube metadata not appearing (title, description, tags)
2. WF2 filename generation (currently random instead of structured)
3. End-to-end testing needed

### High Priority
4. Activate WF1 schedule
5. Replace OpenAI TTS with ElevenLabs custom voice
6. Implement cost tracking (API usage monitoring)
7. WF2 video length optimization (dynamic `num_frames`)
8. Improve video generation prompts

### Future Enhancements
- Auto-generate thumbnails
- Dynamic tags extraction
- YouTube playlists
- Error handling improvements
- Video archiving
- Analytics integration
- Multi-language support

---

## Security Notes

**This document does NOT contain:**
- ❌ API Keys
- ❌ OAuth Client Secrets
- ❌ Access Tokens
- ❌ Credentials

**For actual implementation, you need:**
- ✅ OpenAI API Key
- ✅ Fal.ai API Key
- ✅ Google Cloud OAuth2 Credentials (Client ID + Secret)
- ✅ YouTube OAuth2 Authentication

**Google OAuth2 Credentials** (for reference structure only):
```
Client ID: YOUR_CLIENT_ID.apps.googleusercontent.com
Client Secret: YOUR_CLIENT_SECRET
```

---

## Cost Estimates (Approximate)

### Per Video:
- **OpenAI TTS**: ~$0.015 per 1,000 characters (~$0.02 per video)
- **Fal.ai Infinitalk**: ~$0.05-0.10 per video
- **Google Drive**: Free (within quota)
- **YouTube**: Free

### ElevenLabs (Future):
- ~$0.30 per 1,000 characters (~$0.40 per video)
- Higher quality, custom voice

### Monthly Estimates (30 videos):
- Current: ~$2-3/month
- With ElevenLabs: ~$12-15/month

---

## Support & Documentation

- **n8n Documentation**: https://docs.n8n.io
- **Fal.ai Infinitalk**: https://fal.ai/models/infinitalk
- **OpenAI TTS**: https://platform.openai.com/docs/guides/text-to-speech
- **YouTube Data API**: https://developers.google.com/youtube/v3

---

**Project Saaga** - Automated AI News Video Production
Version 1.0 - December 2025
