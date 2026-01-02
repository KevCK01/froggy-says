# Audio Issues & Fixes (Froggy Says)

## Problems encountered
- **Mobile audio blocked (iOS Safari/Chrome)**: AudioContext starts in `suspended` until a user gesture; oscillators failed silently.
- **Web Audio unreliable on iOS**: Even after resume, oscillators sometimes produced no sound.
- **HTMLAudio reuse limits**: Repeated plays could stall if too many audio elements were created/held.
- **Missing assets earlier**: Initially using generated tones only; later replaced with real `mp3` assets.

## Solutions implemented
1. **Explicit “Tap to Start” unlock**
   - Create + resume `AudioContext` inside the tap handler.
   - Play a short audible blip immediately in the same gesture.
   - Hide the overlay only after the unlock attempt.

2. **Prefer real audio assets**
   - Added `c.mp3`, `e.mp3`, `g.mp3`, `b.mp3`, `buzzer.mp3`, `fanfare.mp3`, `grow.mp3`.
   - Web Audio oscillators kept as a fallback.

3. **Per-sound HTMLAudio pooling (to avoid iOS limits)**
   - For each sound key, create a small pool (size 6) of `HTMLAudioElement` instances.
   - On play: grab a free/finished instance, reset `currentTime`, mark `inUse`, and play.
   - `onended` releases to the pool; a 5s interval also frees stuck entries.
   - Prevents “stops responding after many plays” caused by hitting element limits.

4. **Resume on every play call**
   - Before any Web Audio playback, call `audioContext.resume()` if suspended.

## Current playback order of preference
1. Try pooled HTMLAudio for that sound key (real mp3).
2. If that fails, fall back to Web Audio oscillator.

## Files touched
- `froggy-says.html` (main game + audio logic)
- `index.html` (synced copy for GitHub Pages)
- MP3 assets in the project root.

## Testing tips
- On iOS: hard refresh → tap “🔊 Tap to Start!” → expect a short beep, then play pads.
- If silent: check hardware mute/volume; reload and tap again.

