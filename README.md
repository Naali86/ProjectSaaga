# Project Saaga - AI-Powered News Video Automation

## Overview

Project Saaga is an automated video production pipeline for creating and publishing AI-generated news videos to YouTube. The system uses a human-in-the-loop workflow that generates news scripts, creates lip-synced avatar videos with AI voiceover, and publishes approved content to a YouTube channel.

## What It Does

The prototype automates the complete news video production workflow:

1. **Script Generation (WF1)**: Automatically generates news scripts from trending topics using OpenAI GPT
2. **Video Production (WF2)**: Creates lip-synced videos using Fal.ai Infinitalk with OpenAI text-to-speech
3. **Manual Review**: Human operator reviews generated videos before publication
4. **YouTube Publishing (WF3)**: Uploads approved videos to YouTube with proper metadata

## Technologies Used

### Core Platform
- **n8n** - Workflow automation platform (self-hosted v1.120.4)

### AI Services
- **OpenAI GPT-4** - News script generation
- **OpenAI TTS (Nova voice)** - Audio generation
- **Fal.ai Infinitalk** - Video-to-video lip-sync animation

### Cloud Services
- **Google Sheets** - Data storage and tracking
- **Google Drive** - Intermediate file storage (audio, preview videos)
- **YouTube Data API v3** - Video publishing

### Languages & Formats
- Finnish language news content
- JSON workflow configurations
- JavaScript (n8n Code nodes)

## Architecture

```
┌─────────────┐
│    WF1      │  1. Script Generation (Scheduled/Manual)
│   Script    │     → Google Sheets
│  Factory    │
└─────────────┘

┌─────────────┐
│    WF2      │  2. Video Generation (Manual)
│   Video     │     → OpenAI TTS → Fal.ai Infinitalk
│ Generator   │     → Preview to Google Drive
└─────────────┘

        ↓ (Manual Review)

┌─────────────┐
│    WF3      │  3. Publishing (Manual)
│  Publish    │     → YouTube with metadata
│    Bot      │     → Sheet status update
└─────────────┘
```

## Current Status

✅ **Completed:**
- WF1: Automated script generation
- WF2: Manual video generation with preview
- WF3: Manual YouTube publishing with brand account support
- Preview workflow with status tracking
- Metadata management (title, source, tags)

## TODO List (Prioritized)

### Critical Fixes
1. **Fix YouTube metadata upload** - Title, description, and tags not appearing (currently empty/missing)
2. **Fix WF2 filename generation** - Currently produces random names instead of `{date}_{id}.mp4` format
3. **Test full end-to-end flow** - Run WF1 → WF2 → Review → WF3 with real content

### High Priority
4. **Activate WF1 schedule** - Set up automated script generation (daily/weekly)
5. **Replace OpenAI TTS with ElevenLabs** - Use custom voice for better quality and brand consistency
6. **Implement cost tracking** - Monitor API usage and costs (OpenAI, Fal.ai, ElevenLabs)
7. **WF2 video length optimization** - Adjust `num_frames` based on audio duration
8. **Improve video generation prompts** - Optimize `image_prompt` for better Fal.ai results (lighting, camera angles, expressions)

### Medium Priority (YouTube Enhancements)
9. **Auto-generate thumbnails** - Create custom thumbnails with branding (using AI image generation or templates)
10. **Dynamic tags generation** - Extract keywords from content for YouTube tags
11. **YouTube playlists** - Auto-add videos to topic-specific playlists
12. **YouTube cards & end screens** - Add interactive elements to videos
13. **Video chapters** - Auto-generate timestamps for longer videos

### Medium Priority (System Improvements)
14. **Add youtube_url column** - Store published video URLs in Google Sheets
15. **Error handling improvements** - Add retry logic and better error messages
16. **Video archiving** - Move published videos from preview to archive folder
17. **Webhook notifications** - Slack/Discord alerts for completed videos

### Low Priority (Future Enhancements)
18. **Content moderation** - Add AI content filtering before generation
19. **Multi-language support** - Expand beyond Finnish
20. **Analytics integration** - Track video performance metrics
21. **A/B testing** - Test different prompts/voices for engagement
22. **SEO optimization** - Auto-generate optimized descriptions with keywords

## File Structure

```
Project Saaga/
├── WF1_Saaga_Script_Factory_v1.json     # 1. Script generation workflow
├── WF2_Saaga_Video_Gen_Fixed.json       # 2. Video generation workflow
├── WF3_Saaga_Publish_Bot.json           # 3. YouTube publishing workflow
├── infiniti.json                         # Reference workflow (Fal.ai example)
└── README.md                             # This file
```

## Setup Notes

### Required Credentials
- Google Sheets OAuth2 API
- Google Drive OAuth2 API
- OpenAI API key
- Fal.ai API key (Header Auth: `Key <YOUR_KEY>`)
- YouTube OAuth2 API (Brand Account)

### Google Drive Folder IDs
- Temp Audio: `1ZZa6zVWk3AyF3zv68LBrFzI_WmdySXq0`
- Saaga Previews: `1mKzfiq5FQRcqaO_e9HZ5lgUJiyv2EQr1`
- Base Video: `1NLHSXHw48kdbghusj_FoQgm0Prli5B8b`

### Google Sheets Structure
- Document ID: `1M1rAqnnCbAFXqTJtn_sg_iTI5ovlKb_j6Puz1ASv3bg`
- Sheet: `Taulukko1`
- Columns: `id`, `title`, `script`, `image_prompt`, `source_title`, `source_url`, `status`, `youtube_url`

## Known Issues

1. **YouTube metadata missing**: Title, description, and tags not appearing in published videos (expressions may not be resolving correctly)
2. **WF2 filename**: Videos saved with random names instead of structured format
3. **Short videos**: Current `num_frames: 145` produces ~6 second videos (need dynamic calculation)
4. **Manual node configuration**: Some nodes require manual credential re-selection after import

## License

Proprietary - Internal use only
