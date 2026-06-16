# 🎙️ SpeakAgentGradio — Human-like Voice Agent with Gradio UI

> An AI-powered voice assistant for **Latifoğlu Logistics** that accepts audio input, transcribes it, generates a human-like response via GPT-4o-mini, and speaks the reply back — all through a clean Gradio web interface. While processing, the agent plays natural-sounding filler phrases so callers never experience silence.

---

## ✨ Features

- **Audio Upload & Processing** — Accepts MP3/WAV input directly in the browser via Gradio
- **Fireworks Whisper STT** — Fast transcription using `whisper-v3-turbo` via Fireworks AI
- **GPT-4o-mini LLM** — Intelligent, human-like responses tailored to logistics queries
- **OpenAI TTS (gpt-4o-mini-tts)** — Warm, slow, emotionally present voice replies using the `shimmer` voice
- **Never-Repeating Filler Phrases** — While processing, the agent speaks polite "I'm working on it" messages aloud — never repeating the same one
- **Threaded Architecture** — Filler audio runs in a background thread; real pipeline runs in parallel for zero idle silence
- **TTS Cache** — Processing filler phrases are cached as MP3s to avoid redundant API calls
- **Trilingual Support** — Responds in **English**, **Turkish**, or **Arabic** based on caller language
- **Dual Audio Output** — Gradio UI shows both the spoken reply and the transcript

---

## 🗂️ Project Structure

```
SpeakAgentGradio/
├── app.py                      # Main Gradio application (production entry point)
├── main.py                     # Development history & experimental CLI versions
├── router.py                   # Request routing logic
├── api_test.py                 # API integration tests
├── requirements.txt            # Python dependencies
├── processor_filtervoice.mp3   # Sample/test audio file
├── uploaded_input.wav          # Temp file for processed WAV input
└── .env                        # API keys (create this yourself — do NOT commit)
```

---

## ⚙️ Prerequisites

- **Python 3.9+**
- **FFmpeg** installed and on your system PATH → [Download FFmpeg](https://ffmpeg.org/download.html)
- **OpenAI API key** → [Get one here](https://platform.openai.com/api-keys)
- **Fireworks AI API key** → [Get one here](https://fireworks.ai) (used for fast Whisper transcription)
- **pygame** compatible audio output (for server-side speaker playback)

---

## 🚀 Getting Started

### 1. Clone the Repository

```bash
git clone https://github.com/nimradev064/SpeakAgentGradio.git
cd SpeakAgentGradio
```

### 2. Install Dependencies

```bash
pip install -r requirements.txt
```

### 3. Configure Environment Variables

Create a `.env` file in the project root:

```env
OPENAI_API_KEY=sk-your-openai-key-here
FIREWORKS_API_KEY=your-fireworks-key-here
```

> ⚠️ **Never commit your `.env` file.** Add it to `.gitignore` immediately.

### 4. Run the App

```bash
python app.py
```

Open the Gradio UI in your browser at: **`http://localhost:7860`**

---

## 🖥️ How to Use

1. **Upload an audio file** (MP3 or WAV) using the Gradio upload widget
2. Click **"Process Audio & Talk"**
3. While the agent processes your request, you'll hear natural filler statements spoken aloud from the server (e.g. *"Let me listen to your message and get that information for you"*)
4. When the response is ready, the filler stops and the **assistant's spoken reply** plays automatically
5. The Gradio UI displays: spoken audio, original audio duration, transcript, and text response

---

## 🔄 How It Works

```
User uploads audio (MP3/WAV)
         │
         ▼
┌─────────────────────────────────────┐
│  Thread 1 (Background - Immediate)  │
│  Play never-repeating filler TTS    │
│  "I'm reviewing your request..."    │
└─────────────────────────────────────┘
         │  (runs in parallel)
┌─────────────────────────────────────┐
│  Thread 2 (Main Pipeline)           │
│  1. FFmpeg converts to PCM WAV      │
│  2. Fireworks Whisper transcribes   │
│  3. GPT-4o-mini generates reply     │
│  4. OpenAI TTS synthesizes speech   │
└─────────────────────────────────────┘
         │
         ▼
  Pipeline done → signal filler to stop
         │
         ▼
  Play final assistant reply aloud
         │
         ▼
  Return to Gradio UI:
  [Audio] [Duration] [Transcript] [Reply Text]
```

---

## 🌐 Supported Languages

The assistant automatically detects and replies in the caller's language:

| Language | Greeting Example |
|---|---|
| 🇬🇧 English | *"Hello, this is Latifoğlu Logistics. How can I assist you today?"* |
| 🇹🇷 Turkish | *"Selamünaleyküm, Latifoğlu Lojistik'tesiniz. Nasıl yardımcı olayım?"* |
| 🇸🇦 Arabic | *"مرحبًا بك في شركة لوجستيات لطيف أوغلو. كيف يمكنني مساعدتك؟"* |

> The agent only answers logistics, transportation, and dispatch-related queries. Off-topic questions are politely redirected.

---

## 🔧 Configuration

Key settings at the top of `app.py`:

| Variable | Default | Description |
|---|---|---|
| `VOICE_NAME` | `shimmer` | OpenAI TTS voice |
| `VOICE_SPEED` | `0.85` | Speaking speed (slower = more human-like) |
| `TTS_INSTRUCTIONS` | Warm, slow, caring tone | Personality instructions for TTS |
| `WHISPER_MODEL` | `whisper-v3-turbo` | Fireworks Whisper model |
| `CHARS_PER_CHUNK` | `1200` | Audio chunk size |
| `CACHE_DIR` | `processing_tts_cache/` | Folder for cached filler audio |

---

## 📦 Dependencies

| Package | Purpose |
|---|---|
| `openai` | GPT-4o-mini LLM + TTS API |
| `gradio` | Web UI |
| `pygame` | Server-side audio playback |
| `requests` | Fireworks Whisper API calls |
| `python-dotenv` | Load API keys from `.env` |
| `ffmpeg` *(system)* | Audio format conversion (MP3 → PCM WAV) |

Install all Python packages:

```bash
pip install -r requirements.txt
```

---

## ⚠️ Important Notes

- **Server-side audio playback** — The filler and reply audio plays from the machine running the server (not the user's browser speakers). For browser-based playback, the Gradio audio widget handles the final response.
- **Filler caching** — TTS filler phrases are cached in `processing_tts_cache/` to save API tokens. Delete the folder to regenerate them.
- **FFmpeg required** — The app will fail to convert audio without FFmpeg on PATH.
- **API costs** — Each request uses Fireworks (transcription) + OpenAI (LLM + TTS). Monitor usage at your respective dashboards.
- **`main.py`** — This file contains the full development history and previous CLI versions of the agent. The active production code is in `app.py`.

---

## 🛡️ Security Reminder

Never commit your `.env` file. Add to `.gitignore`:

```
.env
*.env
processing_tts_cache/
__pycache__/
*.pyc
uploaded_input.wav
assistant_response.mp3
```

---

## 🙋 Author

**nimradev064** — [GitHub Profile](https://github.com/nimradev064)
