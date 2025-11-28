# Testing Guide

## Unit Tests

The application includes test helpers to verify generated animation data:

### Verify Animation Response

\`\`\`typescript
import { verifyAnimationResponse } from '@/lib/test-helpers';

const response = await fetch('/api/generate', {
  method: 'POST',
  body: JSON.stringify({ text: 'Explain Bernoulli...' }),
});

const data = await response.json();
const { valid, errors } = verifyAnimationResponse(data);

if (!valid) {
  console.error('Validation errors:', errors);
}
\`\`\`

### Verify Bernoulli Animation

\`\`\`typescript
import { verifyBernoulliAnimation } from '@/lib/test-helpers';

const { valid, errors } = verifyBernoulliAnimation(data);
console.assert(valid, 'Bernoulli animation should be valid', errors);
\`\`\`

### Verify Matrix Exponential Animation

\`\`\`typescript
import { verifyMatrixExpmAnimation } from '@/lib/test-helpers';

const { valid, errors } = verifyMatrixExpmAnimation(data);
console.assert(valid, 'Matrix animation should be valid', errors);
\`\`\`

### Verify Subtitle Timing

\`\`\`typescript
import { verifySubtitleTiming } from '@/lib/test-helpers';

const { valid, errors, warnings } = verifySubtitleTiming(
  data.subtitleSrt,
  data.voiceScript,
  data.voiceScriptDuration
);

errors.forEach(err => console.error(err));
warnings.forEach(warn => console.warn(warn));
\`\`\`

### Verify Animation Duration

\`\`\`typescript
import { verifyAnimationDuration } from '@/lib/test-helpers';

const { valid, estimatedDuration } = verifyAnimationDuration(
  data.voiceScript,
  data.voiceScriptDuration
);

console.log(`Estimated duration: ${estimatedDuration}s`);
\`\`\`

## Manual Testing Checklist

### Bernoulli Animation
- [ ] Text input: "Explain Bernoulli's Principle"
- [ ] Verify scene breakdown shows 3 scenes
- [ ] Verify instructions JSON contains bernoulli parameters
- [ ] Verify voice script mentions pressure and velocity
- [ ] Verify animation renders streamlines and airfoil
- [ ] Verify duration is appropriate (30-300s based on selection)
- [ ] Verify subtitles generated and downloadable
- [ ] Verify audio plays or Web Speech fallback activates

### Matrix Exponential Animation
- [ ] Text input: "Explain 4x4 matrix exponential"
- [ ] Verify scene breakdown shows 3 scenes
- [ ] Verify instructions JSON contains matrix parameters
- [ ] Verify voice script mentions Taylor series
- [ ] Verify animation renders matrix grid
- [ ] Verify series expansion visualized
- [ ] Verify recording works at 60 FPS
- [ ] Verify downloaded WebM video plays in player

### Fallback Animations
- [ ] Test "sine wave"
- [ ] Test "bubble sort"
- [ ] Test "vector addition"
- [ ] Test "rotating triangle"

### UI/UX Tests
- [ ] All controls responsive to input
- [ ] Status messages display correctly during generation
- [ ] Canvas updates in real-time
- [ ] Play/Pause/Stop controls work
- [ ] Timeline slider syncs with playback
- [ ] Auto-highlighting tracks audio time
- [ ] Text is clickable to seek
- [ ] Right panel scrolls independently

### Recording Tests
- [ ] Record without audio (Web Speech)
- [ ] Record with audio (if available)
- [ ] Downloaded WebM opens in video player
- [ ] Audio synced in recorded video
- [ ] Multiple recordings don't cause issues

### Export Tests
- [ ] Download audio (if available)
- [ ] Download SRT subtitles
- [ ] Download WebM video
- [ ] All files can be opened in respective programs

## Browser Compatibility

Test these browsers:
- [ ] Chrome/Chromium (latest)
- [ ] Firefox (latest)
- [ ] Safari (latest)
- [ ] Edge (latest)
- [ ] Mobile Chrome
- [ ] Mobile Safari

## Performance Tests

### Metrics to Monitor

Use Chrome DevTools Performance tab:

1. **Page Load Time**: Should be < 2 seconds
2. **Animation FPS**: Should maintain 60 FPS during playback
3. **Memory Usage**: Should not exceed 300 MB during recording
4. **CPU Usage**: Should stay below 80% during playback

### Long Animation Test

- Generate extended (300s) animation
- Record full duration
- Monitor memory usage
- Verify no crashes or freezes

## Accessibility Tests

- [ ] Keyboard navigation works (Tab, Enter, Spacebar)
- [ ] Screen reader reads labels and descriptions
- [ ] Color contrast meets WCAG standards
- [ ] Touch targets are >= 48x48 pixels
- [ ] All interactive elements have focus indicators

## Network Tests

### With API Keys

If configuring external TTS providers:

\`\`\`bash
# Test ElevenLabs
curl -X POST https://api.elevenlabs.io/v1/text-to-speech/default \
  -H "xi-api-key: $ELEVENLABS_API_KEY" \
  -d "{ \"text\": \"test\" }"

# Test Hugging Face
curl -X POST https://api-inference.huggingface.co/models/gated/... \
  -H "Authorization: Bearer $HUGGINGFACE_API_KEY" \
  -d "{ \"inputs\": \"test\" }"
\`\`\`

### Offline Mode

- [ ] Works without internet (Web Speech only)
- [ ] Gracefully falls back when API unavailable
- [ ] Error messages are helpful

## Edge Cases

- [ ] Empty text input (should show validation error)
- [ ] Very long text (>10,000 words)
- [ ] Special characters in text
- [ ] Rapid API calls (should not crash)
- [ ] Browser close during recording
- [ ] Tab loss of focus during playback
- [ ] Memory warnings on mobile
- [ ] Low battery mode on mobile

## Debugging

### Enable Verbose Logging

Add to browser console:

\`\`\`javascript
// Log all API calls
const originalFetch = window.fetch;
window.fetch = (...args) => {
  console.log('[FETCH]', args[0], args[1]);
  return originalFetch(...args);
};

// Log animation events
window.addEventListener('audioTimeUpdate', (e) => {
  console.log('[TIME]', e.detail);
});

// Log errors
window.addEventListener('error', (e) => {
  console.error('[ERROR]', e.error);
});
\`\`\`

### Common Issues

**Issue**: Canvas not rendering
- Check: `canvas.getContext('2d')` returns context
- Check: Animation type matches renderer
- Check: Scene data is present in API response

**Issue**: Audio not playing
- Check: Browser console for CORS errors
- Check: `crossOrigin="anonymous"` on audio element
- Check: TTS provider is configured (if using external)

**Issue**: Recording not working
- Check: MediaRecorder API supported
- Check: Canvas stream captured
- Check: MIME type supported (video/webm)

**Issue**: Subtitles misaligned
- Check: Voice script duration calculation
- Check: Sentence splitting logic
- Check: Subtitle parsing

## Performance Optimization Tests

\`\`\`javascript
// Measure API response time
console.time('generate');
const response = await fetch('/api/generate', { ... });
console.timeEnd('generate');

// Measure render time per frame
let frameCount = 0;
const start = performance.now();
// ... animation plays ...
const end = performance.now();
console.log('FPS:', frameCount / ((end - start) / 1000));

// Measure memory before/after
const memBefore = performance.memory.usedJSHeapSize;
// ... generate animation ...
const memAfter = performance.memory.usedJSHeapSize;
console.log('Memory used:', (memAfter - memBefore) / 1024 / 1024, 'MB');
\`\`\`

## Regression Testing

After code changes, verify:

1. Bernoulli animation still generates correctly
2. Matrix exponential visualization works
3. Recording exports valid WebM
4. Subtitles stay synchronized
5. All export formats work
6. UI remains responsive

## CI/CD Integration

For automated testing in deployment:

\`\`\`yaml
# Example GitHub Actions workflow
name: Test
on: [push, pull_request]
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
        with:
          node-version: '18'
      - run: npm install
      - run: npm run build
      - run: npm run lint
      - run: npm test (when available)
\`\`\`
\`\`\`
