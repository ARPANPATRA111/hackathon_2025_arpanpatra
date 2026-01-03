# InsightCast

A browser-based podcast analysis tool powered by local AI. Transcribe audio files, index them semantically, and search for concepts, not just keywords to jump to specific timestamps.

![Image](./public/image.webp)

## Features

- 🎙️ **Local Transcription** - Whisper AI runs entirely in your browser
- 🔍 **Semantic Search** - Find concepts and ideas, not just exact words
- 🔒 **Privacy First** - Your audio never leaves your device
- ⚡ **WebGPU Accelerated** - GPU-powered inference when available
- 📱 **Responsive UI** - Works on desktop, tablet, and mobile devices

## Tech Stack

- **Framework**: React 18 + Vite 5.x
- **Language**: TypeScript 5.x (Strict Mode)
- **Styling**: Tailwind CSS
- **AI Runtime**: @huggingface/transformers v3 (WebGPU support)
- **Vector Database**: @orama/orama (In-memory vector store)
- **State Management**: React Context + Custom Hooks

## Getting Started

### Prerequisites

- Node.js 18+ 
- A modern browser with WebGPU support (Chrome 113+, Edge 113+) for GPU acceleration
- Browsers without WebGPU will fall back to WASM (slower but works everywhere)

### Installation

##### Clone the repository
```bash
git clone https://github.com/Codemos-official/hackathon_2025_arpanpatra.git
cd insightcast
```

##### Install dependencies

```bash
npm install

```

##### Start development server

```bash
npm run dev

```

The app will be available at `http://localhost:5173`

## Architecture & System Design

### Main Thread Zero-Block Policy

All heavy compute tasks run inside a dedicated Web Worker:

* Model Loading (Whisper ASR + Embedding Model)
* Audio Transcription
* Embedding Generation
* Vector Search

The main thread handles only UI rendering and audio playback.

### Execution Flow

#### 1. Audio Processing Pipeline

This sequence details the "Write" path. When a file is uploaded, the UI thread offloads processing to the Web Worker to prevent freezing. The worker coordinates the Whisper ASR model for transcription and the Embedding model for vectorization before indexing data into Orama.

[![](https://mermaid.ink/img/pako:eNp1VF1P2zAU_StXfqGIgJo2IcUPSAXG1IduUqFDmvpi4ktqkdie7TBY1f--m4-ybnR5iOybc879OI43LDcSGWcef9Soc7xRonCiWmmgxwoXVK6s0AGWMxAeFijyAFNrYTAXSsP92qGQxx_hD8Y9o2soD_jY7w6g1srbHtYvpzMYTO8WByQ_VY8oG2i7kEoXMKfiy4_Im6sG9pX6EPAN82AchVa6A34xAcG8UKrlLOor4zCtpTKwtKUREm5L87MDL2enl5c7jDU-zNF7UeBgA-HNIoej4IT2uVOPeBSBF5Ut0UcgayeCMhq2fSPdm4anXgTl3x9It27ydCPgsKg1zPQTusYRGPSqvVKPOt0rbIGhdmQGvga4Xtf62cMJ3KsKfSCq36-gNMbCLQ2EjFzDHRYV6tB9-ruWXrqVa5UPgFojOHxGja7pqp_1uz9_KG3oQMkd44D0zRWnEXh05KbJ66ZKGPT6JzDHIKQI4vgDk6jLGYd3e6wzhSPPGnMCGUchpSW-UnVHsO3oqOX-iP6jtDPaNr6ekmZdhka1G6HfiUn8x2QWscIpyXhwNUasQleJZss2DWHFwhorXDFOSync84qt9JY4dJC_G1PtaM7UxZrxJ1F62tWWut_9q-9ROi4S3bWpdWA8jkdJq8L4hr3SPj1LsiSN02wyTifjOLuI2Bvj2cVZNhxORpM0Hmbn6TDZRuxXmzc-G47T0cX5OE6S4SjJJmnEUCoyYN5dGe3Nsf0NnEJfVA?type=png)](https://mermaid.live/edit#pako:eNp1VF1P2zAU_StXfqGIgJo2IcUPSAXG1IduUqFDmvpi4ktqkdie7TBY1f--m4-ybnR5iOybc879OI43LDcSGWcef9Soc7xRonCiWmmgxwoXVK6s0AGWMxAeFijyAFNrYTAXSsP92qGQxx_hD8Y9o2soD_jY7w6g1srbHtYvpzMYTO8WByQ_VY8oG2i7kEoXMKfiy4_Im6sG9pX6EPAN82AchVa6A34xAcG8UKrlLOor4zCtpTKwtKUREm5L87MDL2enl5c7jDU-zNF7UeBgA-HNIoej4IT2uVOPeBSBF5Ut0UcgayeCMhq2fSPdm4anXgTl3x9It27ydCPgsKg1zPQTusYRGPSqvVKPOt0rbIGhdmQGvga4Xtf62cMJ3KsKfSCq36-gNMbCLQ2EjFzDHRYV6tB9-ruWXrqVa5UPgFojOHxGja7pqp_1uz9_KG3oQMkd44D0zRWnEXh05KbJ66ZKGPT6JzDHIKQI4vgDk6jLGYd3e6wzhSPPGnMCGUchpSW-UnVHsO3oqOX-iP6jtDPaNr6ekmZdhka1G6HfiUn8x2QWscIpyXhwNUasQleJZss2DWHFwhorXDFOSync84qt9JY4dJC_G1PtaM7UxZrxJ1F62tWWut_9q-9ROi4S3bWpdWA8jkdJq8L4hr3SPj1LsiSN02wyTifjOLuI2Bvj2cVZNhxORpM0Hmbn6TDZRuxXmzc-G47T0cX5OE6S4SjJJmnEUCoyYN5dGe3Nsf0NnEJfVA)

#### 2. Semantic Search Flow

This sequence details the "Read" path. When a user queries a concept (e.g., "happiness"), the system converts the query into a vector and performs a similarity search against the indexed audio segments.

[![](https://mermaid.ink/img/pako:eNptU8tu2zAQ_JUFD02KyoYeli3zkIPtAvbBKBo3DVDoQksbiYhFMiTV1jH07yUtpXAeAiRwlzM7O1zxRApZIqHE4FOLosAVZ5VmTS7APYppywuumLBwZ1B_kN0AM3CLrLCwlI2SAoV9D7uX-hG1h97jfojeo1YLj_jm5Bn8xMJK7VK56IFef3Rzc7eh8OOoEL63qI8UcmfAta0syAeomVJcoDE5GUgbR-nlKChp7NZtsgqvT2BdEQpXBpku6qsAnnw96D73xP7rXPHfzOKrjvv1Rd2vzR7Lvh_YWc1F9Qa4WtAXP7uzHFyvj3vNSwqLbZzCl2F30F4tRhfVb9G2WsCSiZKXvpc1t-aVwHAqbyyNNJr2YI2zNqyg62klfujr4sT8WVMnVNUH91rXddW4uRr4BLta_oE1MtswRQJSOReEWt1iQBrUDfMhOflaObE1NpgTP6SS6Uc_lM5x3Kx_Sdm80LRsq5rQB3YwLmqVNzn8hf-zGkWJeilbYQmNoig7VyH0RP4SGqfjZBpmSTRL4zCM5tNJQI6EzubjOJlNwnQyz8I4i6IuIM9n3WgcJmk8nyaTWZaF2SRJA4IldxPY9pfhfCe6f5C8AEU?type=png)](https://mermaid.live/edit#pako:eNptU8tu2zAQ_JUFD02KyoYeli3zkIPtAvbBKBo3DVDoQksbiYhFMiTV1jH07yUtpXAeAiRwlzM7O1zxRApZIqHE4FOLosAVZ5VmTS7APYppywuumLBwZ1B_kN0AM3CLrLCwlI2SAoV9D7uX-hG1h97jfojeo1YLj_jm5Bn8xMJK7VK56IFef3Rzc7eh8OOoEL63qI8UcmfAta0syAeomVJcoDE5GUgbR-nlKChp7NZtsgqvT2BdEQpXBpku6qsAnnw96D73xP7rXPHfzOKrjvv1Rd2vzR7Lvh_YWc1F9Qa4WtAXP7uzHFyvj3vNSwqLbZzCl2F30F4tRhfVb9G2WsCSiZKXvpc1t-aVwHAqbyyNNJr2YI2zNqyg62klfujr4sT8WVMnVNUH91rXddW4uRr4BLta_oE1MtswRQJSOReEWt1iQBrUDfMhOflaObE1NpgTP6SS6Uc_lM5x3Kx_Sdm80LRsq5rQB3YwLmqVNzn8hf-zGkWJeilbYQmNoig7VyH0RP4SGqfjZBpmSTRL4zCM5tNJQI6EzubjOJlNwnQyz8I4i6IuIM9n3WgcJmk8nyaTWZaF2SRJA4IldxPY9pfhfCe6f5C8AEU)

### AI Models

* **ASR**: `onnx-community/whisper-base` - Speech recognition
* **Embeddings**: `mixedbread-ai/mxbai-embed-xsmall-v1` - Semantic embeddings (384 dims)

### WebGPU Fallback

The app automatically detects WebGPU availability:

1. Attempts WebGPU initialization first (faster)
2. Falls back to WASM if WebGPU unavailable
3. Notifies user of current mode via status indicator

## Usage

1. **Wait for Model Loading** - First load downloads ~150MB of AI models
2. **Upload Audio** - Drag and drop or click to select (MP3, WAV, M4A, OGG, WebM, FLAC)
3. **Wait for Transcription** - Processing time depends on audio length and device
4. **Search** - Enter concepts, topics, or questions in the search bar
5. **Navigate** - Click segments or waveform regions to jump to timestamps

## Supported Audio Formats

* MP3
* WAV
* M4A
* OGG
* WebM
* FLAC

## Browser Compatibility

| Browser | WebGPU | WASM Fallback |
| --- | --- | --- |
| Chrome 113+ | ✅ | ✅ |
| Edge 113+ | ✅ | ✅ |
| Firefox | ❌ | ✅ |
| Safari 18+ | ✅ | ✅ |

## Development

### Project Structure

```
src/
├── components/          # React components
│   ├── AudioVisualizer.tsx
│   ├── AudioPlayer.tsx
│   ├── DropZone.tsx
│   ├── SearchBar.tsx
│   └── TranscriptView.tsx
├── hooks/               # Custom React hooks
│   ├── useInsight.ts
│   ├── useAudioPlayer.ts
│   └── InsightContext.tsx
├── types.ts             # TypeScript definitions
├── worker.ts            # AI Web Worker
├── App.tsx              # Main application
└── main.tsx             # Entry point

```

### Key Configuration

* `vite.config.ts` - WASM asset copying and worker configuration
* `tsconfig.json` - TypeScript strict mode with WebWorker lib
* `tailwind.config.js` - Custom color palette and animations

## Performance Tips

* First load is slower due to model download (~150MB)
* Models are cached in browser storage after first download
* WebGPU provides 2-5x faster inference than WASM
* Shorter audio files transcribe faster
* Close other GPU-intensive applications for best performance

## Acknowledgments

* [Hugging Face Transformers.js](https://github.com/xenova/transformers.js)
* [Orama Search](https://github.com/oramasearch/orama)
* [OpenAI Whisper](https://github.com/openai/whisper)
