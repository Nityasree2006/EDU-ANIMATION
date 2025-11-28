# Text-Driven Educational Animation Video Generator

A full-stack Next.js 14 application that transforms text descriptions into synchronized animated videos with voiceover narration, subtitles, and auto-highlighted text.

## Features

- **Deterministic Generators**: Built-in support for Bernoulli's Principle and 4×4 Matrix Exponential animations
- **Fallback Animations**: Sine wave, bubble sort, vector addition, and rotating triangle visualizations
- **Text-to-Speech**: Backend TTS with Web Speech API fallback for demo purposes
- **Subtitle Generation**: Automatic .srt subtitle generation with timestamp synchronization
- **Auto-Highlighted Text**: Right panel displays script with synchronized highlighting during playback
- **Canvas Recording**: Record animations as WebM video with MediaRecorder API
- **Scene Breakdown**: View detailed scene information and animation instructions

## Tech Stack

- **Frontend**: React 19, Next.js 14 (App Router), TailwindCSS, shadcn/ui
- **Backend**: Next.js API Routes
- **Audio**: Web Speech API (fallback), with integration points for ElevenLabs/Hugging Face
- **Canvas**: HTML5 Canvas for rendering animations
- **Recording**: MediaRecorder API for video export

## Getting Started

### Prerequisites

- Node.js 18+
- npm or yarn

### Installation

1. Clone the repository (or download the v0 ZIP)
2. Install dependencies:
\`\`\`bash
npm install
\`\`\`

3. Run the development server:
\`\`\`bash
npm run dev
\`\`\`

4. Open [http://localhost:3000](http://localhost:3000) in your browser

## Usage

### Basic Workflow

1. **Input Panel (Left)**
   - Enter a description of the animation you want to generate
   - Select animation mode (simple/advanced), style, and duration
   - Configure voice options (provider, type, language)
   - Click "Generate Animation"

2. **Canvas Panel (Center)**
   - View the animated visualization
   - Use Play/Pause/Stop controls
   - Record the animation as WebM video
   - Download audio and subtitles

3. **Output Panel (Right)**
   - View scene breakdown with details
   - See animation instructions in JSON format
   - Read the full script with auto-highlighting during playback
   - Download generated subtitle file

### Quick Examples

Try these keywords to see specific animations:

- **Bernoulli's Principle**: Type "Bernoulli" in the input to generate a fluid flow visualization
- **Matrix Exponential**: Type "4x4 matrix exponential" to see mathematical transformation
- **Sine Wave**: Type "sine wave" for a basic waveform animation
- **Bubble Sort**: Type "bubble sort" to visualize sorting algorithm
- **Vector Addition**: Type "vector addition" to show vector mathematics
- **Rotating Triangle**: Type "rotate" to see a rotating shape

## Configuration

### Environment Variables

To enable external TTS providers, set these in your `.env.local`:

\`\`\`env
# ElevenLabs (optional)
ELEVENLABS_API_KEY=sk_xxx

# Hugging Face (optional)
HUGGINGFACE_API_KEY=hf_xxx
\`\`\`

If no API keys are provided, the app defaults to Web Speech API fallback.

### TTS Provider Setup

The application uses Web Speech API by default (browser-based, no API key needed). For production:

1. **ElevenLabs**:
   - Sign up at [elevenlabs.io](https://elevenlabs.io)
   - Get your API key from settings
   - Add to `.env.local`

2. **Hugging Face**:
   - Create account at [huggingface.co](https://huggingface.co)
   - Generate API token
   - Add to `.env.local`

3. **Google Cloud TTS** / **AWS Polly**:
   - Uncomment and modify `lib/tts-generator.ts`
   - Add service credentials

## API Reference

### POST /api/generate

Generates animation data, voice script, and subtitles.

**Request Body**:
\`\`\`json
{
  "text": "Explain Bernoulli's Principle",
  "mode": "simple",
  "style": "minimal",
  "includeInstructions": true,
  "voiceEnabled": true,
  "voiceProvider": "web-speech",
  "voiceType": "narrator",
  "language": "en",
  "durationPreference": "medium"
}
\`\`\`

**Response**:
\`\`\`json
{
  "success": true,
  "scenes": [
    {
      "id": "bernoulli-intro",
      "name": "Bernoulli Principle Introduction",
      "description": "...",
      "duration": 20,
      "animationType": "bernoulli",
      "parameters": {}
    }
  ],
  "instructions": [...],
  "voiceScript": "long detailed explanation...",
  "voiceScriptDuration": 180,
  "audioUrl": null,
  "subtitleSrt": "1\n00:00:00,000 --> 00:00:30,000\nFirst sentence\n\n2\n..."
}
\`\`\`

## Project Structure

\`\`\`
├── app/
│   ├── api/
│   │   └── generate/          # Main API route
│   │       └── route.ts
│   ├── page.tsx               # Main page layout
│   ├── layout.tsx             # Root layout
│   └── globals.css            # Global styles
├── components/
│   ├── control-panel.tsx      # Left panel: input controls
│   ├── canvas-panel.tsx       # Center panel: canvas & playback
│   ├── output-panel.tsx       # Right panel: outputs
│   └── ui/                    # shadcn/ui components
├── lib/
│   ├── types.ts               # TypeScript interfaces
│   ├── deterministic-generators.ts  # Animation generators
│   ├── animation-renderer.ts  # Canvas rendering functions
│   ├── subtitle-generator.ts  # SRT generation
│   └── tts-generator.ts       # TTS integration
└── public/                    # Static assets
\`\`\`

## Key Implementation Details

### Deterministic Generators

All animations are deterministic based on keyword detection:

- **Bernoulli**: `bernoulliScenePlan()`, `bernoulliInstructions()`, `bernoulliVoiceScript()`
- **Matrix Exponential**: `matrixExpmScenePlan()`, `matrixExpmInstructions()`, `matrixExpmVoiceScript()`
- **Fallbacks**: Keyword matching determines animation type

### Animation Synchronization

The canvas animation syncs with audio playback:

1. Canvas renders via `requestAnimationFrame`
2. Uses `audio.currentTime` as the source of truth when playing
3. Falls back to `performance.now()` scaled to duration if no audio

### Subtitle Generation

Sentences are split and distributed across the total duration:

- Split text by punctuation (., !, ?)
- Distribute duration equally across sentences
- Generate .srt with proper timestamp formatting

### Recording

Uses the MediaRecorder API to capture canvas stream:

1. `canvas.captureStream(60)` for 60 FPS video
2. Optional audio track added from HTML5 audio element
3. Exports as WebM video

## Extending the Application

### Adding Custom Animations

1. Create generator functions in `lib/deterministic-generators.ts`:
\`\`\`typescript
export function myCustomScenePlan(): SceneItem[] { ... }
export function myCustomInstructions(): InstructionSet[] { ... }
export function myCustomVoiceScript(): string { ... }
\`\`\`

2. Add render function in `lib/animation-renderer.ts`:
\`\`\`typescript
export function renderMyCustomAnimation(context: AnimationContext) { ... }
\`\`\`

3. Add to keyword detection in `/api/generate` route

### Adding External TTS Providers

1. Implement in `lib/tts-generator.ts`:
\`\`\`typescript
if (provider === 'my-provider' && process.env.MY_PROVIDER_KEY) {
  // Call API and return audioUrl + duration
}
\`\`\`

2. Add provider option to frontend controls

## Troubleshooting

### Audio not playing
- Check browser console for audio errors
- Ensure `crossOrigin="anonymous"` on audio element
- Verify TTS provider credentials if using external service

### Canvas not rendering
- Confirm canvas context is obtained: `ctx = canvas.getContext('2d')`
- Check animation type matches available renderers
- Verify scene data from API response

### Recording fails
- Ensure browser supports MediaRecorder API (all modern browsers)
- Check for CORS issues with audio
- Try downloading video without audio first

### Subtitles misaligned
- Verify TTS duration matches actual audio duration
- Check sentence splitting logic handles your text format
- Compare subtitle timestamps with audio length

## Performance Optimization

- **Canvas FPS**: Set to 60 for smooth playback (adjustable in `captureStream()`)
- **Animation Duration**: Scales with voiceScript words (150 wpm estimate)
- **Maximum Duration**: Configure cap to prevent excessive computation
- **Sentence Splitting**: Caches parsed sentences to avoid recalculation

## Browser Support

- Chrome/Edge: Full support (MediaRecorder, Web Audio API)
- Firefox: Full support
- Safari: Full support (some Audio API limitations)
- Mobile: Limited (no recording), but playback works

## License

MIT

## Support

For issues or questions, open an issue in the repository or check the inline code comments for implementation details.
