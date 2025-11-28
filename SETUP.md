# Setup and Configuration Guide

## Quick Start

### 1. Installation

\`\`\`bash
# Clone or download the project
git clone <repository-url>
cd text-driven-animation-generator

# Install dependencies
npm install

# Run development server
npm run dev
\`\`\`

Open [http://localhost:3000](http://localhost:3000) to see the application.

## Configuration

### Environment Variables

Create a `.env.local` file in the project root to configure optional external services:

\`\`\`env
# ElevenLabs TTS (optional)
# Get API key from: https://elevenlabs.io/api
ELEVENLABS_API_KEY=sk_xxx

# Hugging Face TTS (optional)
# Get token from: https://huggingface.co/settings/tokens
HUGGINGFACE_API_KEY=hf_xxx

# Google Cloud TTS (optional)
# Setup: https://cloud.google.com/docs/authentication/getting-started
GOOGLE_APPLICATION_CREDENTIALS=/path/to/credentials.json

# AWS Polly (optional)
# Configure AWS credentials
AWS_ACCESS_KEY_ID=xxx
AWS_SECRET_ACCESS_KEY=xxx
AWS_REGION=us-east-1
\`\`\`

**Note**: The application works perfectly without any API keys. It defaults to using the Web Speech API, which is built into all modern browsers.

## Features Configuration

### TTS Provider Selection

The application provides multiple TTS options:

1. **Web Speech API** (Default)
   - No configuration needed
   - Browser-based, runs locally
   - Best for demos and testing
   - Limitations: No audio file export in some browsers

2. **ElevenLabs**
   - High-quality voices
   - Supports multiple languages and voice types
   - Requires API key (set ELEVENLABS_API_KEY)

3. **Hugging Face**
   - Free tier available
   - Various model options
   - Requires API key (set HUGGINGFACE_API_KEY)

### Duration Preferences

Users can select animation duration:

- **Short**: 30 seconds
- **Medium**: 60 seconds (default)
- **Long**: 120 seconds
- **Extended**: 300 seconds (5 minutes)

The system adjusts narration length and animation speed accordingly.

### Animation Styles

Three style options control visual complexity:

- **Minimal**: Clean, simple visualizations
- **Detailed**: Enhanced with labels and values
- **Cinematic**: Full effects and smooth animations

### Voice Options

- **Voice Types**: Narrator, Male, Female, Robotic
- **Languages**: English, Spanish, French, German, Italian, Portuguese, Japanese, Chinese
- **Providers**: Web Speech, ElevenLabs, Hugging Face

## Deterministic Animations

### Bernoulli's Principle

Triggered by text containing "bernoulli":

\`\`\`
Features:
- Streamline visualization
- Particle flow animation
- Airfoil shape
- Pressure differential display
- Formula: P₁ + ½ρv₁² = P₂ + ½ρv₂²
\`\`\`

### 4×4 Matrix Exponential

Triggered by text containing "4x4" or "matrix exponential":

\`\`\`
Features:
- 4×4 matrix grid
- Taylor series expansion visualization
- Term-by-term convergence
- Vector transformation
- Series: I + A + A²/2! + A³/3! + ...
\`\`\`

### Fallback Animations

Automatic selection based on keywords:

- **sine-wave**: Harmonic oscillation
- **bubble-sort**: Sorting algorithm visualization
- **vector-addition**: Head-to-tail vector composition
- **rotating-triangle**: Angular motion demonstration

## Recording and Export

### Video Recording

- Canvas recorded at 60 FPS
- Exports as WebM format (.webm)
- Includes audio if available
- ~2-10 MB for typical animations

### Audio Export

- Generated from TTS provider
- Format: MP3 or WAV depending on provider
- Can be downloaded separately

### Subtitle Export

- Format: SRT (SubRip)
- Compatible with video players and editors
- Can be burned into video using ffmpeg:

\`\`\`bash
# Burn subtitles into video
ffmpeg -i animation.mp4 -vf "subtitles=subtitles.srt" output.mp4

# Merge audio with video
ffmpeg -i animation.webm -i narration.mp3 -c:v copy -c:a aac output.mp4
\`\`\`

## Extending the Application

### Adding Custom Animations

1. **Create generator functions** in `lib/deterministic-generators.ts`:

\`\`\`typescript
export function myAnimationScenePlan(duration: string): SceneItem[] {
  return [
    {
      id: 'my-animation-1',
      name: 'My Custom Animation',
      description: 'Description...',
      duration: 30,
      animationType: 'my-type',
      parameters: {},
    },
  ];
}

export function myAnimationInstructions(): InstructionSet[] {
  return [/* ... */];
}

export function myAnimationVoiceScript(duration: string): string {
  return 'Detailed explanation...';
}
\`\`\`

2. **Add renderer** in `lib/animation-renderer.ts`:

\`\`\`typescript
export function renderMyAnimationAnimation(context: AnimationContext) {
  const { ctx, canvas, time, duration } = context;
  // Draw your animation here
}
\`\`\`

3. **Register in API** route (`app/api/generate/route.ts`):

\`\`\`typescript
const isMyAnimation = textLower.includes('my-keyword');
if (isMyAnimation) {
  scenes = myAnimationScenePlan(durationPreference);
  instructions = myAnimationInstructions();
  voiceScript = myAnimationVoiceScript(durationPreference);
}
\`\`\`

### Adding External TTS Provider

1. **Update** `lib/tts-generator.ts`:

\`\`\`typescript
export async function generateTTS(
  text: string,
  voiceType: string,
  provider: string,
  language: string
): Promise<TTSResult> {
  // ... existing code ...
  
  if (provider === 'my-provider' && process.env.MY_PROVIDER_API_KEY) {
    try {
      const response = await fetch('https://api.myprovider.com/tts', {
        method: 'POST',
        headers: {
          'Authorization': `Bearer ${process.env.MY_PROVIDER_API_KEY}`,
          'Content-Type': 'application/json',
        },
        body: JSON.stringify({ text, voice: voiceType }),
      });

      if (response.ok) {
        const arrayBuffer = await response.arrayBuffer();
        const audioUrl = 'data:audio/mp3;base64,' + Buffer.from(arrayBuffer).toString('base64');
        // Estimate duration
        const duration = Math.ceil((text.split(/\s+/).length / 150) * 60);
        return { audioUrl, duration };
      }
    } catch (error) {
      console.error('TTS provider error:', error);
    }
  }
  
  // Fallback...
}
\`\`\`

2. **Update frontend** in `components/control-panel.tsx`:

\`\`\`typescript
const TTS_PROVIDERS = [
  { value: 'web-speech', label: 'Web Speech' },
  { value: 'my-provider', label: 'My Provider' },
  // ... existing ...
];
\`\`\`

## Performance Tips

### Canvas Optimization

- Canvas resolution: 800×600 default (adjustable in `CanvasPanel`)
- Frame rate: 60 FPS (adjust in `captureStream()`)
- For longer animations, consider reducing resolution

### TTS Optimization

- Web Speech API is fastest (no network latency)
- ElevenLabs and Hugging Face add 1-5 second delay
- Cache results in production

### Memory Usage

- Large animations (>5 minutes) use ~100-300 MB
- Recording consumes additional memory (~50 MB per minute)
- Clear browser cache if running multiple sessions

## Browser Support

| Feature | Chrome | Firefox | Safari | Edge |
|---------|--------|---------|--------|------|
| Canvas  | ✓      | ✓       | ✓      | ✓    |
| Web Audio API | ✓ | ✓      | ✓      | ✓    |
| MediaRecorder | ✓ | ✓      | ✓      | ✓    |
| Web Speech API | ✓ | ✓ (limited) | ✓ | ✓ |
| offscreenCanvas | ✓ | ✓    | ✗      | ✓    |

## Troubleshooting

### Issue: Audio not playing

**Solution**:
\`\`\`javascript
// Check browser console for CORS errors
// Ensure crossOrigin="anonymous" on audio element
// Test Web Speech API first before external providers
\`\`\`

### Issue: Canvas not rendering

**Solution**:
- Check browser DevTools for errors
- Verify canvas element is mounted
- Test with fallback animations first
- Check animation type against available renderers

### Issue: Recording produces no video

**Solution**:
\`\`\`javascript
// Ensure MediaRecorder is supported
if (!window.MediaRecorder) {
  console.warn('MediaRecorder not supported');
}
// Check for CORS issues with audio
// Try recording without audio first
\`\`\`

### Issue: Subtitles misaligned

**Solution**:
- Verify TTS duration matches actual audio
- Check sentence splitting in `subtitle-generator.ts`
- Compare calculated duration with actual playback

### Issue: Memory leak on long recordings

**Solution**:
- Stop recording explicitly
- Clear canvas context regularly
- Reload page between multiple recordings

## Production Deployment

### Vercel Deployment

\`\`\`bash
# Push to GitHub
git push origin main

# Deploy on Vercel
vercel

# Set environment variables in Vercel dashboard
# Go to Settings > Environment Variables
# Add API keys for your TTS provider
\`\`\`

### Self-Hosted Deployment

\`\`\`bash
# Build for production
npm run build

# Start production server
npm start
\`\`\`

### Docker Deployment

\`\`\`dockerfile
FROM node:18-alpine
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
RUN npm run build
EXPOSE 3000
CMD ["npm", "start"]
\`\`\`

## API Rate Limits

### Web Speech API
- Unlimited (browser-based)

### ElevenLabs
- Free tier: 10,000 characters/month
- Pro tier: Based on subscription

### Hugging Face
- Free API: 30 requests/minute
- Paid API: Based on plan

## Support and Debugging

### Enable Debug Logging

Add to environment:
\`\`\`env
DEBUG=animation:*
\`\`\`

### Check Generated Data

\`\`\`javascript
// In browser console
console.log(window.animationData); // View last generated data
\`\`\`

### Reset Local State

\`\`\`javascript
localStorage.clear();
sessionStorage.clear();
location.reload();
\`\`\`

## License

MIT License - See LICENSE file for details
\`\`\`

```json file="" isHidden
