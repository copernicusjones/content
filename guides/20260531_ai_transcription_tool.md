+++"tags": ['ai', 'transcription', 'whisper', 'openai', 'groq', 'azure', 'video', 'automation']
---

# AI Transcription Tool: The Complete Guide to Transcribing Video with OpenAI, Groq, and Azure

## Introduction

Transcribing video content used to mean hours of manual typing or expensive third-party services. That changed with the arrival of AI-powered transcription APIs. Today, you can convert video to text in minutes using OpenAI's Whisper model — either through OpenAI's own API, Groq's ultra-fast inference, or Azure OpenAI's enterprise-grade deployment.

In this guide, you'll learn how to set up and use [Sapat](https://github.com/nkkko/sapat), an open-source Python tool that automates the entire transcription pipeline. It converts video files to audio, sends them to your choice of transcription API, and saves the results as clean text files.

### TL;DR

- **What**: Sapat is a Python CLI tool that transcribes video files using AI APIs
- **Why**: Automate transcription of podcasts, meetings, interviews, and video content
- **How**: Install Sapat, configure API credentials, run `sapat <video_file>`
- **Cost**: OpenAI Whisper $0.006/min, Groq free tier, Azure enterprise pricing

## What Is Sapat?

[Sapat](https://github.com/nkkko/sapat) is an open-source Python package that automates video transcription:

1. **Video-to-Audio** — FFmpeg extracts MP3 from video files
2. **Transcription** — Sends audio to OpenAI, Groq, or Azure OpenAI
3. **Text Output** — Saves transcription as `.txt` file
4. **Batch Processing** — Handles single files or entire directories

### Supported APIs

| API | Model | Speed | Cost | Best For |
|-----|-------|-------|------|----------|
| **OpenAI** | whisper-1 | Fast | $0.006/min | General use |
| **Groq** | whisper-large-v3-turbo | Very Fast | Free tier | High-volume |
| **Azure OpenAI** | whisper | Fast | Enterprise | Organizations |

## Prerequisites

- Python 3.6+
- FFmpeg installed
- API credentials for at least one service

### Installing FFmpeg

**macOS:** `brew install ffmpeg`
**Ubuntu:** `sudo apt install ffmpeg`
**Windows:** `choco install ffmpeg`

## Installation

```bash
git clone https://github.com/nkkko/sapat.git
cd sapat
pip install -r requirements.txt
python -m build
pip install dist/sapat-0.1.1-py3-none-any.whl
```

## Configuration

Create `.env` file with your API credentials. See [Sapat README](https://github.com/nkkko/sapat) for full configuration details.

## Usage

```bash
# Single file
sapat my_video.mp4

# Choose API
sapat my_video.mp4 --api groq

# Batch process
sapat /path/to/videos/ --api openai

# Advanced: language, quality, prompt
sapat my_video.mp4 --api groq --language en --quality H --prompt "Tech podcast"
```

## API Comparison

**OpenAI**: Simplest, pay-as-you-go, $0.006/min
**Groq**: Fastest (10-50x real-time), free tier
**Azure**: Enterprise-grade for organizations with Azure contracts

## Tips

1. Use `--quality H` for better accuracy
2. Specify `--language` for non-English audio
3. Use `--prompt` with key terms for context
4. Set `--temperature 0` for deterministic output

## Conclusion

Sapat + AI transcription APIs = automated video-to-text in minutes.

/claim #13