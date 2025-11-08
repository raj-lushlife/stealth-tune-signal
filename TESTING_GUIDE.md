# Audio Encoding Testing Guide

## Quick Reference: Test Cases

| Test # | Audio Source | TX Status | Expected Result |
|--------|-------------|-----------|-----------------|
| 1 | Microphone only | OFF | ❌ No unlock |
| 2 | White noise | ON | ✅ Unlock |
| 3 | Music/song | ON | ✅ Unlock |
| 4 | Music/song | OFF | ❌ No unlock |
| 5 | Speech | ON | ✅ Unlock |

---

## Setup (Do This Once)

1. Open `presence-beacon.html` in your browser
2. **Recommended Settings:**
   - Boost/Cut: **12dB** (start here for reliable detection)
   - Symbol Duration: **120ms**
   - Burst Interval: **8s**
   - Code Repeats: **2**

---

## Test 1: Microphone Only (Baseline Rejection)

**Goal:** Verify system rejects ambient noise without encoding

1. **DO NOT** click "Start TX"
2. Click "Start RX"
3. Make normal sounds (talk, clap, play music nearby)
4. **Expected:** Console shows rejections, NO unlock
5. Click "Stop RX"

✅ **Pass:** No unlock occurs  
❌ **Fail:** System unlocks (false positive - tighten thresholds)

---

## Test 2: Encoded White Noise

**Goal:** Confirm basic encoding works

1. Set audio source to "White Noise"
2. Click "Start TX" (wait 2 seconds)
3. Click "Start RX"
4. **Expected:** Within 3-10 seconds, see "🎉 UNLOCKED"
5. Click "Stop RX", then "Stop TX"

✅ **Pass:** System unlocks reliably  
❌ **Fail:** No unlock (check volume, device proximity)

---

## Test 3: Encoded Music/Song

**Goal:** Verify encoding works on real audio

1. Click "Upload Audio File" and select a music file
2. Set **Boost/Cut to 12dB** (use slider)
3. Click "Start TX" (wait 2 seconds)
4. Click "Start RX"
5. Listen to the audio - is the encoding audible?
6. **Expected:** Unlock within 3-10 seconds
7. Click "Stop RX", then "Stop TX"

✅ **Pass:** System unlocks, encoding barely/not audible  
⚠️ **Adjust:** If too audible, try reducing gain to 10dB → 8dB  
❌ **Fail:** No unlock (increase gain to 15dB and retry)

---

## Test 4: Unencoded Music (Critical False Positive Test)

**Goal:** Verify system rejects non-encoded audio

1. Keep your music file loaded
2. **DO NOT** click "Start TX" (or click "Stop TX" if running)
3. Click "Start RX"
4. Let it run for 30+ seconds
5. **Expected:** Console shows rejections, NO unlock
6. Click "Stop RX"

✅ **Pass:** No unlock occurs  
❌ **Fail:** System unlocks (CRITICAL - false positive detected)

---

## Test 5: Encoded Speech (Optional)

**Goal:** Test with voice/podcast audio

1. Upload a speech/podcast audio file
2. Set **Boost/Cut to 15dB** (speech needs higher gain)
3. Follow same steps as Test 3
4. **Expected:** Unlock within 3-10 seconds

---

## Troubleshooting

### No Unlock on Encoded Audio
- **Check volume:** TX output level should be > -30 dBFS (shown in console)
- **Increase gain:** Try 15dB or 18dB
- **Check distance:** Move phone closer to speaker
- **Hard refresh:** Browser may have cached old code

### False Positives (Unlock Without TX)
- **Lower thresholds:** Reduce correlation/PSR/Z-score in RX code
- **Check TX status:** Ensure TX is completely stopped
- **Verify logs:** Look for "TX validation: ✅ TX was ON"

### Encoding Too Audible
- **Lower gain:** Try 8dB → 6dB → 5dB
- **Check frequency:** 12kHz should be near-inaudible (age-dependent)
- **Audio type matters:** Music masks better than silence/speech

---

## Understanding Console Output

```
🔍 CODE_LOCK CANDIDATE @ t=3.12s:
├─ [correlation] 0.574 (≥0.3) ✅    ← Pattern match strength
├─ [PSR] 11.4dB (≥6) ✅            ← Peak-to-sidelobe ratio
├─ [Z-score] 4.74 (≥3) ✅          ← Statistical significance
├─ [median|Δ|] 1.60dB (≥0.5) ✅   ← Differential strength
└─ [guards] Lo=13.1dB, Hi=11.7dB ✅ ← Noise rejection
✅ VALID DETECTION! Vote 1/2
```

**All checks must pass for unlock.**

---

## Optimal Settings Found

After testing, record your optimal settings here:

- **Music Gain:** _____ dB (imperceptible + reliable)
- **Speech Gain:** _____ dB
- **White Noise Gain:** _____ dB
- **Max Reliable Distance:** _____ cm/inches

---

## Success Criteria

✅ Test 1 (Mic only): No unlock  
✅ Test 2 (White noise encoded): Unlock  
✅ Test 3 (Music encoded): Unlock + imperceptible  
✅ Test 4 (Music unencoded): No unlock  
✅ Test 5 (Speech encoded): Unlock

**All 5 tests passing = System validated!** 🎉
