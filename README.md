# youtube-automation
AI-powered YouTube video automation workflow built with n8n, OpenAI, ElevenLabs, Kling AI, fal.ai, Google Drive, and YouTube.

🤖 YouTube Automation Workflow
📌 Project Overview

This project is an end-to-end AI-powered YouTube automation workflow built with n8n.

The workflow automates multiple stages of video production — from generating a video script and voice-over to creating AI visuals, generating video clips, merging audio and video, and finally publishing the completed video to YouTube.

The goal is to reduce repetitive manual work and demonstrate how multiple AI services and APIs can be connected together through workflow automation.


✨ Key Features
🤖 AI-powered script generation
📝 Automatic title and description generation
🎙️ AI voice generation using ElevenLabs
✂️ Automatic script segmentation
🎨 AI visual prompt generation
🖼️ AI image generation
☁️ Automatic image storage using Google Drive
🎬 AI image-to-video generation using Kling AI
🔄 Automatic video status polling
🎧 Audio and video merging using fal.ai / FFmpeg
📥 Automatic final video download
📺 Automated YouTube publishing


## 🛠️ Tech Stack
Technology	Purpose
n8n	Workflow automation and orchestration
OpenAI	Script, title, description, visual prompts and image generation
ElevenLabs	AI text-to-speech
Google Drive	Generated image storage and sharing
Kling AI	Image-to-video generation
fal.ai / FFmpeg API	Audio and video merging
YouTube	Final video publishing
HTTP APIs	Communication with external services
JSON	Workflow configuration and data exchange


## ⚡ How It Works

The automation follows a complete AI-powered video production pipeline.

┌──────────────────────────────┐
│        MANUAL TRIGGER        │
│        Start Workflow        │
└──────────────┬───────────────┘
               │
               ▼
┌──────────────────────────────┐
│       OPENAI — IDEATOR       │
│ Script + Title + Description │
└──────────────┬───────────────┘
               │
               ▼
┌──────────────────────────────┐
│            SCRIPT            │
│   Prepare Generated Content  │
└──────────────┬───────────────┘
               │
        ┌──────┴─────────────┐
        │                    │
        ▼                    ▼
┌──────────────────┐  ┌──────────────────┐
│    ELEVENLABS    │  │  SPLIT SCRIPT    │
│  Text → Speech   │  │   ~6 Sec Chunks  │
└──────────────────┘  └────────┬─────────┘
                                │
                                ▼
                    ┌────────────────────┐
                    │   OPENAI PROMPT    │
                    │  Visual Prompt Gen │
                    └─────────┬──────────┘
                              │
                              ▼
                    ┌────────────────────┐
                    │   OPENAI IMAGE     │
                    │     GENERATION     │
                    └─────────┬──────────┘
                              │
                              ▼
                    ┌────────────────────┐
                    │    GOOGLE DRIVE    │
                    │  Store + Share     │
                    └─────────┬──────────┘
                              │
                              ▼
                    ┌────────────────────┐
                    │      KLING AI      │
                    │   Image → Video    │
                    └─────────┬──────────┘
                              │
                              ▼
                    ┌────────────────────┐
                    │    WAIT + POLL     │
                    │   Check Status     │
                    └─────────┬──────────┘
                              │
                         ┌────▼─────┐
                         │  Ready?  │
                         └──┬────┬──┘
                            │    │
                         No │    │ Yes
                            │    ▼
                            │ ┌────────────────────┐
                            │ │   FAL.AI / FFMPEG  │
                            │ │ Merge Audio+Video  │
                            │ └─────────┬──────────┘
                            │           │
                            │           ▼
                            │ ┌────────────────────┐
                            │ │ Download Final     │
                            │ │ Video              │
                            │ └─────────┬──────────┘
                            │           │
                            │           ▼
                            │ ┌────────────────────┐
                            │ │  YOUTUBE UPLOAD    │
                            │ │   Publish Video    │
                            │ └────────────────────┘
                            │
                            └──────► Wait + Poll



  ## 🔄 Workflow Breakdown
1. Manual Trigger

The workflow starts using n8n's Manual Trigger node.

The workflow can be executed directly from the n8n editor.

2. AI Content Generation

The Ideator node uses OpenAI to generate structured content.

The generated content contains:

Intro
Main script
CTA
Video title
Video description

The workflow uses the o3-mini model for this stage.

3. Script Preparation

The Script node extracts the generated content and prepares:

Complete script text
Video title
Video description

These values are later used by the voice generation and YouTube publishing stages.

4. AI Voice Generation

The Audio Generator node sends the generated script to ElevenLabs.

The configured model is:

eleven_multilingual_v2

The workflow requests the generated audio as an MP3 file.

5. Script Segmentation

The generated script is divided into smaller sections.

The workflow estimates approximately:

2.5 words per second

and creates approximately:

6-second chunks

Each chunk contains:

Chunk index
Script text
Start time
End time
Total number of chunks
6. Visual Prompt Generation

Each script chunk is processed by OpenAI.

OpenAI generates a detailed cinematic visual prompt based on the content and emotion of the specific script segment.

7. AI Image Generation

The generated visual prompt is sent to OpenAI image generation.

The workflow uses:

Size: 1792 × 1024
Quality: HD
Style: Vivid

The generated image is then downloaded by the workflow.

8. Google Drive Storage

The generated images are uploaded to Google Drive.

The workflow creates a timestamped folder for the generated images.

The images are then shared and a public URL is generated for further processing.

9. Kling AI — Image to Video

The generated image URL and corresponding script chunk are sent to Kling AI.

Kling AI converts the generated image into a video clip.

The workflow stores the generated Kling task ID so that the processing status can be checked later.

10. Video Processing & Polling

Kling AI processing is asynchronous.

The workflow waits for approximately 60 seconds and then checks the job status.

Submit Video Job
       ↓
   Wait 60 sec
       ↓
  Check Status
       ↓
   Video Ready?
    /       \
   No       Yes
   │          │
   └──────────┘
      │
      ▼
 Wait + Poll Again

The workflow continues polling until the Kling task reaches a completed or failed state.

11. Audio + Video Merge

After the Kling video is ready, the workflow sends the video and generated audio to the fal.ai FFmpeg API.

The service is used to merge:

Generated Audio
       +
Generated Video
       ↓
Final Video
12. Final Video Download

After the media-processing job is completed, the workflow retrieves the resulting video and downloads it.

13. YouTube Publishing

The final video is uploaded to YouTube through the n8n YouTube integration.

The workflow uses the generated:

Title
Description

for the YouTube upload.

The supplied configuration specifies:

Region: BD
Privacy: Public
Category ID: 22


                                    
## Technical Architecture
                    ┌────────────────┐
                    │      n8n       │
                    │  Orchestration │
                    └───────┬────────┘
                            │
             ┌──────────────┼──────────────┐
             │              │              │
             ▼              ▼              ▼
        ┌─────────┐   ┌───────────┐  ┌───────────┐
        │ OpenAI  │   │ElevenLabs │  │Google     │
        │   AI    │   │   TTS     │  │Drive      │
        └────┬────┘   └───────────┘  └─────┬─────┘
             │                              │
             │                              ▼
             │                         ┌─────────┐
             │                         │ Kling AI│
             │                         └────┬────┘
             │                              │
             └──────────────┐               ▼
                            │         ┌──────────┐
                            └────────►│ fal.ai   │
                                     │  FFmpeg  │
                                     └────┬─────┘
                                          │
                                          ▼
                                     ┌─────────┐
                                     │ YouTube │
                                     └─────────┘

                                     



## Data Flow
Video Idea
    ↓
OpenAI
    ↓
Script + Title + Description
    ↓
┌───────────────┬────────────────┐
│               │                │
▼               ▼                │
ElevenLabs    Script Chunks      │
│               │                │
▼               ▼                │
Audio        OpenAI Prompts      │
                │                │
                ▼                │
           AI Images             │
                │                │
                ▼                │
           Google Drive          │
                │                │
                ▼                │
             Kling AI            │
                │                │
                ▼                │
          Video Processing       │
                │                │
                └──────┬─────────┘
                       ▼
                fal.ai / FFmpeg
                       │
                       ▼
                  Final Video
                       │
                       ▼
                    YouTube



## 👨‍💻 My Contribution

Based on the implemented workflow, this project demonstrates practical experience in:

Designing an end-to-end n8n automation workflow
Integrating multiple AI services
Integrating OpenAI for content generation
Integrating OpenAI image generation
Integrating ElevenLabs text-to-speech
Connecting Google Drive for automated asset storage
Integrating Kling AI for image-to-video generation
Implementing asynchronous API job polling
Integrating fal.ai / FFmpeg media processing
Preparing automated YouTube metadata
Automating final YouTube publishing



## 🔐 Security & Credentials

⚠️ Important: Never upload real API keys, passwords, tokens, or credentials to GitHub.

The original workflow export contained sensitive-looking API credentials inside HTTP request headers.

For the public repository:

API keys must be removed.
Exposed keys should be revoked or rotated.
Credentials should be stored securely inside n8n.
Workflow exports should be sanitized before uploading.
.env files containing secrets must never be committed.
Screenshots must not expose credentials or private information.


## ⚙️ Setup
Requirements

The workflow requires access to:

n8n
OpenAI
ElevenLabs
Google Drive
Kling AI
fal.ai
YouTube
Installation
Open your n8n instance.
Import the workflow JSON from:
workflow/youtube-automation.json
Configure your own credentials.
Review the HTTP Request nodes.
Configure Google Drive access.
Configure Kling AI API access.
Configure fal.ai API access.
Configure YouTube OAuth.
Test the workflow.
Verify the final generated video before publishing.



## Testing

Before using the workflow for production:

Test the content generation stage.
Verify ElevenLabs audio output.
Verify generated images.
Verify Google Drive permissions.
Test Kling video generation.
Confirm the polling loop works correctly.
Verify audio/video merging.
Test YouTube upload permissions.
Check the final video quality.


## 📸 Screenshots




## ⚠️ Current Limitations

The current workflow should be considered a portfolio/demo implementation unless further production testing is completed.

Current limitations include:

The workflow currently starts with a manual trigger.
Script timing is estimated using word count.
Kling video generation uses asynchronous polling.
The configured callback URL should be verified in the live n8n environment.
YouTube credentials must be configured in the user's own environment.


## Portfolio Value

This project demonstrates practical experience in:

AI Integration
       +
Workflow Automation
       +
API Integration
       +
Media Processing
       +
Cloud Storage
       +
YouTube Automation

The project showcases how multiple AI services can be orchestrated through n8n to automate a complete video-production pipeline from content idea to final YouTube publication.


## Project Status

Portfolio / Demonstration Project

Credentials and sensitive configuration should be added separately in the user's own environment.


## Final Result

The overall automation pipeline is:

IDEA
  ↓
AI SCRIPT
  ↓
AI VOICE
  ↓
SCRIPT CHUNKS
  ↓
AI VISUAL PROMPTS
  ↓
AI IMAGES
  ↓
GOOGLE DRIVE
  ↓
KLING AI VIDEO
  ↓
AUDIO + VIDEO MERGE
  ↓
FINAL VIDEO
  ↓
YOUTUBE

An end-to-end AI-powered YouTube automation workflow built with n8n.







