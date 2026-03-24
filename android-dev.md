# Android Development Best Practices

## Azure Speech SDK — Translation & Language Detection

### Silence & Segmentation Tuning (as of March 2026)

Current best-performing values for conversational real-time translation:

```kotlin
// In both buildManualRecognizer() and buildAutoRecognizer()
speechConfig.setProperty(PropertyId.SpeechServiceConnection_EndSilenceTimeoutMs, "900")
speechConfig.setProperty("speech.segmentation.mode", "custom")
speechConfig.setProperty("speech.segmentation.sentenceTimeoutMs", "750")
```

- `EndSilenceTimeoutMs` controls silence after speech before cutting the segment
- `speech.segmentation.mode = custom` enables phonetic sentence-boundary detection
- `speech.segmentation.sentenceTimeoutMs` is the silence threshold within the segmentation model
- Do NOT add `InitialSilenceTimeoutMs` — only affects first utterance, causes unnecessary delay
- Values below 600ms for sentenceTimeout cause truncated utterances and poor translation quality

---

### Echo Guard (Same-Language Repeat Bug)

**Problem:** Azure occasionally misidentifies language on short/ambiguous utterances mid-conversation, causing the source text to be "translated" back to itself (same language echo).

**Fix:** Add this guard in both `attachManualListeners()` and `attachAutoListeners()` after resolving `targetLang`:

```kotlin
val targetCode = targetLang.locale.split("-")[0]
if (detectedCode == targetCode) return@addEventListener  // discard misidentified segment
val translated = e.result.translations[targetCode]
```

---

### Thread Safety — Recognizer Lifecycle

**Problem:** Calling `recognizer.close()` on a background thread while Azure SDK fires `recognized` callbacks causes `IllegalStateException` crash.

**Fix:** Use `ReentrantLock` around ALL recognizer create/close operations:

```kotlin
private val recognizerLock = ReentrantLock()

private fun teardownRecognizer() {
    recognizerLock.lock()
    try {
        isWarmReady = false
        recognizer?.stopContinuousRecognitionAsync()
        recognizer?.close()
        recognizer = null
    } catch (e: Exception) { /* swallow */ } finally {
        recognizerLock.unlock()
    }
}
```

All `buildManualRecognizer()` and `buildAutoRecognizer()` calls must also hold this lock.

---

### Pre-Warm Pattern (Reducing First-Start Latency)

**Problem:** First press of Start takes 4+ seconds because WebSocket connection to Azure is built on demand.

**Fix:**
- Use `isWarmReady: Boolean` flag (set to `true` only after build **fully completes**)
- Call `preWarm()` in `onCreate()` to build the connection in the background at app launch
- In `start()` / `startAutoMode()`: if `isWarmReady && !settingsChanged` → skip rebuild, activate immediately
- Use `destroy()` (not `stop()`) in `onDestroy()` — `stop()` keeps WebSocket alive, `destroy()` fully tears down

---

### Auto-Detect Mode — Language Pair Narrowing

- Azure supports up to 10 candidate languages for auto-detect
- Once a foreign language is detected, rebuild recognizer with just 2 languages (default + detected) for better accuracy
- The rebuild must happen on a background thread using `teardownRecognizer()` before `buildAutoRecognizer()`
- Checkbox ticks in Settings while IDLE should call `preWarmAutoMode()` to keep the new shortlist warm
