# Quick Reference Guide

## Installation & Running

\`\`\`bash
npm install
npm run dev
# Open http://localhost:3000
\`\`\`

## File Quick Reference

| File | Purpose |
|------|---------|
| \`app/page.tsx\` | Main layout (3-column structure) |
| \`app/api/generate/route.ts\` | Backend API endpoint |
| \`lib/deterministic-generators.ts\` | Bernoulli, Matrix, Fallback animation generators |
| \`lib/animation-renderer.ts\` | Canvas drawing for each animation type |
| \`lib/subtitle-generator.ts\` | .srt file creation |
| \`lib/tts-generator.ts\` | Voice narration (Web Speech fallback) |
| \`components/control-panel.tsx\` | Left panel (inputs) |
| \`components/canvas-panel.tsx\` | Center panel (playback & recording) |
| \`components/output-panel.tsx\` | Right panel (outputs & highlighting) |

## API Endpoint

### POST /api/generate

\`\`\`javascript
const response = await fetch('/api/generate', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    text: "Explain Bernoulli's Principle",
    mode: 'simple',
    style: 'minimal',
    durationPreference: 'medium',
    voiceEnabled: true,
    voiceProvider: 'web-speech',
    voiceType: 'narrator',
    language: 'en',
    includeInstructions: true,
  }),
});

const data = await response.json();
// Returns: { success, scenes, instructions, voiceScript, voiceScriptDuration, subtitleSrt }
\`\`\`

## Keyboard Shortcuts (Future)

| Shortcut | Action |
|----------|--------|
| Space | Play/Pause |
| R | Record |
| S | Stop |
| D | Download |

## Environment Variables (Optional)

\`\`\`env
# Add to .env.local for better voice quality
ELEVENLABS_API_KEY=sk_xxx
HUGGINGFACE_API_KEY=hf_xxx
\`\`\`

## Test It Immediately

1. Click "Random Example" button
2. Click "Generate Animation"
3. Wait for generation (watch status messages)
4. Click "Play" to hear narration and see animation
5. Click "Record" to capture as WebM
6. Click "Download SRT" for subtitles

## Common Errors & Fixes

| Error | Solution |
|-------|----------|
| Canvas not showing | Check browser console (F12). Verify animation type matches renderer |
| Audio not playing | Enable voice in left panel. Try Web Speech if external provider fails |
| Recording not working | Use Chrome/Firefox. Try recording without audio first |
| Subtitles misaligned | Verify TTS duration in API response matches actual audio |

## Production Checklist

- [ ] Add API keys to `.env.local` (optional)
- [ ] Test in Chrome, Firefox, Safari, Edge
- [ ] Test on mobile
- [ ] Verify all animations generate correctly
- [ ] Test recording and export
- [ ] Deploy to Vercel: \`vercel\`

## Code Examples

### Add Custom Animation

\`\`\`typescript
// 1. Add to lib/deterministic-generators.ts
export function myAnimationScenePlan(): SceneItem[] {
  return [{ id: 'my-1', name: 'My Animation', duration: 60, animationType: 'my-type' }];
}

// 2. Add to lib/animation-renderer.ts
export function renderMyAnimationAnimation(ctx: CanvasRenderingContext2D) {
  // Draw your animation
}

// 3. Add keyword detection in app/api/generate/route.ts
if (textLower.includes('my-keyword')) {
  scenes = myAnimationScenePlan();
}
\`\`\`

### Test Animation Data

\`\`\`javascript
// In browser console
const data = await fetch('/api/generate', { /* ... */ }).then(r => r.json());
console.log(data); // View full response
console.log(data.voiceScript); // View narration
console.log(data.scenes); // View scene breakdown
\`\`\`

## Performance Tips

- **Faster API**: Web Speech fallback is instant (no network)
- **Smoother Animation**: Already optimized at 60 FPS
- **Lower Latency**: All processing happens locally by default
- **Mobile Friendly**: Canvas scales to viewport

## Deployment Platforms

### Vercel (Recommended)
\`\`\`bash
vercel
# Set env vars in dashboard
\`\`\`

### Docker
\`\`\`bash
docker build -t animation-generator .
docker run -p 3000:3000 animation-generator
\`\`\`

### Traditional Node.js
\`\`\`bash
npm run build
npm start
\`\`\`

## Support Resources

- **README.md** - Full feature overview
- **SETUP.md** - Detailed configuration
- **TESTING.md** - Testing procedures
- **ARCHITECTURE.md** - System design deep dive
- **README_GETTING_STARTED.md** - Beginner guide

## Key Features Checklist

- [x] Bernoulli's Principle animation
- [x] 4×4 Matrix Exponential animation
- [x] Fallback animations (5 types)
- [x] Web Speech API narration
- [x] Canvas recording (WebM)
- [x] Subtitle generation (.srt)
- [x] Auto-text highlighting
- [x] Responsive 3-column layout
- [x] Real-time status updates
- [x] Download exports

## Next Steps

1. **Try It**: Run and click "Random Example"
2. **Customize**: Add your own animation keywords
3. **Deploy**: Push to Vercel
4. **Enhance**: Add external TTS provider
5. **Extend**: Add new animation types

---

**Ready?** \`npm run dev\` and go to http://localhost:3000
\`\`\`
