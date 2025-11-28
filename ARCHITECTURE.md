# Architecture Overview

## System Design

This application uses a three-layer architecture:

### Frontend Layer (React Components)
- **Page Layout** (`app/page.tsx`): Three-column responsive layout
- **Control Panel** (`components/control-panel.tsx`): User input and settings
- **Canvas Panel** (`components/canvas-panel.tsx`): Animation playback and recording
- **Output Panel** (`components/output-panel.tsx`): Results display and text highlighting

### API Layer (Next.js Route Handlers)
- **Generate Endpoint** (`app/api/generate/route.ts`): Orchestrates animation generation

### Core Logic Layer (TypeScript Libraries)
- **Deterministic Generators**: Produce consistent scenes, instructions, voice scripts
- **Animation Renderers**: Canvas drawing functions for each animation type
- **Subtitle Generator**: SRT file creation with timing
- **TTS Generator**: Voice narration (server-side with fallback)
- **Test Helpers**: Verification utilities
- **Utilities**: Mathematical computations and helpers

## Data Flow

\`\`\`
User Input (Control Panel)
    ↓
[POST /api/generate]
    ↓
Generate API Route
    ├→ Detect animation type (Bernoulli, Matrix, Fallback)
    ├→ Generate Scene Plan
    ├→ Generate Instructions
    ├→ Generate Voice Script
    ├→ Call TTS Provider (or estimate duration)
    └→ Generate Subtitles
    ↓
Response: {scenes, instructions, voiceScript, duration, audioUrl, subtitleSrt}
    ↓
Canvas Panel receives data
    ├→ Render animation based on scene type
    ├→ Sync with audio (or Web Speech)
    ├→ Update timeline
    └→ Broadcast time updates
    ↓
Output Panel receives time updates
    └→ Highlight current sentence in voice script
\`\`\`

## Component Hierarchy

\`\`\`
Home (app/page.tsx)
├── ControlPanel
│   ├── TextInput
│   ├── ModeSelect
│   ├── StyleSelect
│   ├── DurationSelect
│   ├── InstructionsToggle
│   ├── VoiceSettings
│   │   ├── VoiceProviderSelect
│   │   ├── VoiceTypeSelect
│   │   └── LanguageSelect
│   ├── ActionButtons
│   │   ├── GenerateButton
│   │   ├── RandomExampleButton
│   │   └── ResetButton
│   └── StatusMessage
│
├── CanvasPanel
│   ├── Canvas
│   ├── Timeline
│   │   └── RangeSlider
│   ├── PlaybackControls
│   │   ├── PlayButton
│   │   ├── PauseButton
│   │   ├── StopButton
│   │   └── RecordButton
│   ├── DownloadButtons
│   │   ├── DownloadAudioButton
│   │   └── DownloadSRTButton
│   └── HiddenAudioElement
│
└── OutputPanel
    ├── SceneBreakdown
    │   └── Accordion (per scene)
    ├── InstructionJSON
    │   └── Details (collapsible)
    ├── TextExplanation
    │   └── SentencesList (with highlighting)
    └── SubtitlesInfo
\`\`\`

## State Management

The application uses React local state with event broadcasting:

1. **Global State**: Minimal use - animation data managed at Home level
2. **Local State**: Each panel manages its own state (playing, currentTime, etc.)
3. **Event Broadcasting**: `audioTimeUpdate` event dispatches current playback time to all components
4. **URL State**: (Future) Could use URL params for shareable links

## Performance Considerations

### Rendering
- Canvas renders at 60 FPS via `requestAnimationFrame`
- Uses `audio.currentTime` as single source of truth for timing
- Fallback to manual time tracking when Web Speech is used

### Memory
- Animations stored in GPU (via canvas)
- Voice script kept in memory (typically < 100 KB)
- Recording chunks collected in array (cleared after download)

### Network
- API endpoint responds in < 500ms (no external API calls by default)
- Web Speech API is local (no network latency)
- External TTS adds 1-5 seconds (configurable)

## Error Handling

\`\`\`
Try-Catch Hierarchy:
├── API Route
│   ├── Validation errors → 400
│   ├── Generator errors → 500 with fallback
│   └── TTS errors → Continue with duration estimate
├── Canvas Panel
│   ├── Render errors → Log and clear canvas
│   └── Recording errors → Graceful fallback
└── Output Panel
    └── Parse errors → Show empty state
\`\`\`

## Browser APIs Used

- **Canvas 2D Context**: Animation rendering
- **HTMLAudioElement**: Audio playback
- **Web Audio API**: Audio analysis (optional)
- **MediaRecorder API**: Video recording
- **Speech Synthesis API**: Voice narration fallback
- **Fetch API**: Network requests
- **localStorage**: (Optional future use)

## Security Considerations

- User input sanitized before display
- No eval() or dangerous DOM manipulation
- CORS handled via `crossOrigin="anonymous"`
- API keys stored in environment only (not in frontend code)
- No personal data collection

## Scalability Paths

### To handle more users:
1. Add server-side caching for generation results
2. Implement database for user sessions
3. Add CDN for static assets
4. Implement API rate limiting

### To support more animation types:
1. Add new generator functions
2. Create corresponding render function
3. Add keyword detection
4. Register in API route

### To support more TTS providers:
1. Implement provider SDK
2. Add config in `tts-generator.ts`
3. Update frontend provider list
4. Add provider credentials handling

## Testing Strategy

### Unit Testing
- Test generators produce valid data
- Test renderers don't crash on invalid input
- Test subtitle timing calculations
- Test utility functions with edge cases

### Integration Testing
- Test full API response validation
- Test audio sync across components
- Test recording export

### End-to-End Testing
- User flow: input → generate → record → download
- Fallback behaviors when APIs unavailable
- Browser compatibility

## Deployment

### Development
\`\`\`bash
npm run dev  # http://localhost:3000
\`\`\`

### Production Build
\`\`\`bash
npm run build
npm start
\`\`\`

### Hosting Options
- **Vercel**: Recommended (auto-deploys from GitHub)
- **Docker**: For self-hosted
- **Traditional Node.js**: Works on any Node.js host

## Future Enhancements

1. **Database Integration**: Save/load animations
2. **User Accounts**: Track user creations
3. **Collaborative Editing**: Multiple users editing
4. **Advanced Animations**: 3D support (Three.js)
5. **Mobile App**: React Native version
6. **AI-Generated Scripts**: Use AI to improve voice scripts
7. **Custom Fonts**: Upload custom fonts for rendering
8. **Background Music**: Add background tracks
9. **Transitions**: Smooth scene transitions
10. **Analytics**: Track popular animation types

## License

MIT License
