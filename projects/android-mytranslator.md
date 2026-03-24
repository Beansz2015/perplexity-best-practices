# Android — MyTranslator

**Stack:** Kotlin, Android, Azure Cognitive Services Speech SDK
**Repo:** https://github.com/Beansz2015/MyTranslator

---

## Azure Speech SDK — Silence & Segmentation Tuning

Best-performing values for conversational real-time translation (as of March 2026):

```kotlin
// Apply in both buildManualRecognizer() and buildAutoRecognizer()
speechConfig.setProperty(PropertyId.SpeechServiceConnection_EndSilenceTimeoutMs, "900")
speechConfig.setProperty("speech.segmentation.mode", "custom")
speechConfig.setProperty("speech.segmentation.sentenceTimeoutMs", "750")
```

- `EndSilenceTimeoutMs` — silence after speech before cutting segment
- `speech.segmentation.mode = custom` — enables phonetic sentence-boundary detection instead of pure silence counting
- `speech.segmentation.sentenceTimeoutMs` — silence threshold within the segmentation model
- **Do NOT add `InitialSilenceTimeoutMs`** — only affects first utterance, adds unnecessary delay mid-conversation
- Values below 600ms for sentenceTimeout cause truncated utterances and poor translation quality

---

## Echo Guard (Same-Language Repeat Bug)

**Problem:** Azure occasionally misidentifies language on short/ambiguous utterances mid-conversation, causing source text to be "translated" back to itself.

**Fix:** Add in both `attachManualListeners()` and `attachAutoListeners()` after resolving `targetLang`:

```kotlin
val targetCode = targetLang.locale.split("-")[0]
if (detectedCode == targetCode) return@addEventListener  // discard misidentified segment
val translated = e.result.translations[targetCode]
```

---

## Thread Safety — Recognizer Lifecycle

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
    } catch (e: Exception) { /* swallow — best effort */ } finally {
        recognizerLock.unlock()
    }
}
```

All `buildManualRecognizer()` and `buildAutoRecognizer()` calls must also acquire this lock.

---

## Pre-Warm Pattern (Reducing First-Start Latency)

**Problem:** First Start press takes 4+ seconds because Azure WebSocket is built on demand.

**Fix:**
- Use `isWarmReady: Boolean` flag — only set to `true` after build **fully completes** (not just started)
- Call `preWarm()` in `onCreate()` to build connection in background while UI loads
- In `start()` / `startAutoMode()`: if `isWarmReady && !settingsChanged` → skip rebuild, activate immediately
- Use `destroy()` in `onDestroy()`, not `stop()` — `stop()` keeps WebSocket alive, `destroy()` fully tears down

---

## Auto-Detect Mode — Language Pair Narrowing

- Azure supports up to 10 candidate languages for auto-detect
- Once a foreign language is detected, rebuild recognizer with just 2 languages (default + detected) for better accuracy
- Rebuild must run on a background thread via `teardownRecognizer()` before `buildAutoRecognizer()`
- Checkbox ticks in Settings while IDLE should call `preWarmAutoMode()` to keep new shortlist warm
