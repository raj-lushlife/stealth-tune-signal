# Presence Beacon Testing Checklist

Use this checklist to validate the presence beacon system after any changes.

---

## Pre-Flight Checks

- [ ] Node.js and npm installed
- [ ] Repository cloned and dependencies installed (`npm install`)
- [ ] Dev server running (`npm run dev`) OR serving `public/` over HTTPS
- [ ] Desktop browser with Web Audio API support (Chrome/Safari/Firefox)
- [ ] iPhone with Safari for RX testing
- [ ] Speakers and microphone functional

---

## Local Run Validation

### Desktop TX + Desktop RX (Same Machine)

- [ ] Open `http://localhost:XXXX/presence-beacon.html` in Tab 1
- [ ] Click "▶ Start Audio" (grants audio context)
- [ ] Click "Start RX" → Observe "✅ Calibrated" after 5 seconds
- [ ] Open `http://localhost:XXXX/presence-beacon.html` in Tab 2
- [ ] Click "▶ Start Audio"
- [ ] Click "Start TX" → Observe console logs: `📡 TX @ symbol 0: code[0]=+1`
- [ ] Tab 1 (RX) shows "🔓 UNLOCKED" within 10-16 seconds
- [ ] Console shows: `🎉 FSM: CODE_LOCK → UNLOCKED (latched 45s)`

### Desktop TX + iPhone RX (Over-Air)

- [ ] iPhone: Open beacon page, "Start Audio" → "Start RX" → Wait for "✅ Calibrated"
- [ ] Desktop: "Start Audio" → "Start TX"
- [ ] Place iPhone 20-40 cm from desktop speakers
- [ ] iPhone shows "🔓 UNLOCKED" within 10-16 seconds
- [ ] Check iPhone console logs (if accessible via Safari Web Inspector) for:
  - `correlation >= 0.30`
  - `psr >= 7.5 dB`
  - `z >= 2.5` (key discriminator)

---

## Encoded vs Unencoded Test

### Encoded Audio (Should Unlock)

- [ ] RX device: Start RX, wait for "✅ Calibrated"
- [ ] TX device: Upload a music file to "Music Bed" input (optional, or use pink noise)
- [ ] Click "Start TX" (encodes in real-time)
- [ ] RX unlocks within 2 bursts (~16s)
- [ ] Console shows:
  - `corr >= 0.30`
  - `psr >= 7.5 dB`
  - `z >= 2.5` ✅

### Unencoded Audio (Should NOT Unlock)

- [ ] RX device: Start RX, wait for "✅ Calibrated"
- [ ] Play same music file WITHOUT TX running (no encoding)
- [ ] RX should remain "⏳ Waiting for Presence..."
- [ ] Console shows: `z < 2.5` ❌ (key discriminator)
- [ ] If `CODE_LOCK CANDIDATE` appears, verify it shows:
  - `corr=0.28-0.31` (marginal)
  - `psr=7-8 dB` (marginal)
  - `z=0.81-1.10 ❌` (fails threshold)

---

## False-Positive Battery

With RX running and TX stopped, test these inputs:

- [ ] **Speech**: Talk loudly near RX mic for 10 seconds → No unlock
  - Expected: z-score low (<2.5), lacks structured code
  - May trigger `PREAMBLE_LOCK` on transients, but fails `CODE_LOCK`

- [ ] **Claps**: Clap 3-5 times near microphone → No unlock
  - Expected: Fails PSR (broadband transient, low specificity)

- [ ] **Snaps**: Snap fingers 5-10 times → No unlock
  - Expected: Fails correlation (random spectrum)

- [ ] **Quiet Music (Unencoded)**: Play low-volume music → No unlock
  - Expected: As tested in logs, z < 2.5

- [ ] **Loud Music (Unencoded)**: Play high-volume music → No unlock
  - Expected: May trigger `PREAMBLE_LOCK`, but z < 2.5 blocks `CODE_LOCK`

**Expected Behavior**: None of these should reach `UNLOCKED` state. Monitor console for `CODE_LOCK CANDIDATE` entries and verify they show failing metrics (❌).

---

## Offline Encoder Test

- [ ] Upload a test audio file (e.g., 30s music clip)
- [ ] Set Spectral Beacon parameters:
  - Center: 12000 Hz
  - Spacing: 50 Hz
  - Duration: 60 ms
  - Repeats: 2
  - Gain: ±6.5 dB
  - Start: 2s
- [ ] Click "Process File" → Wait for encoding to complete
- [ ] Click "Play" on "Original" audio → Listen (no encoding, clean)
- [ ] Click "Play" on "Encoded" audio → Listen (with spectral beacon)
- [ ] At ±6.5 dB, encoding should be imperceptible (subtle or inaudible)
- [ ] Click "Download Encoded" → Save file for archival testing
- [ ] Optional: Use RX to detect the encoded file (play through speakers)

---

## Log Capture & Analysis

### Calibration Logs

- [ ] RX calibration shows:
  ```
  🎯 BASELINE CALIBRATION COMPLETE:
    ├─ Noise Mean Correlation: X.XXX (should be <0.4)
    ├─ Noise StdDev: X.XXX
    └─ Detection threshold: X.XXX (mean + 5σ)
  ```
- [ ] Document these values for your environment

### TX Logs

- [ ] TX shows output level: `🔊 TX OUTPUT LEVEL: X.X dBFS`
- [ ] Verify X.X > -30 dBFS (audible range)
- [ ] If < -30 dBFS, increase volume or TX gain

### Successful Unlock Logs

- [ ] Console shows:
  ```
  🔍 CODE_LOCK CANDIDATE:
    ├─ [match] t=X.XXs corr=X.XXX ✅ psr=X.XdB ✅ z=X.XX ✅
    ├─ [per-symbol] median|Δ|=X.XXdB ✅ energyDev=X.XdB ✅
    ├─ [guards] Lo: X.XdB ✅, Hi: X.XdB ✅ → ✅
    ├─ [vote] 2/2 within X.Xs
    ├─ [cadence] Δt=XXms (symbol≈60ms, repeat≈1860ms) ✅
  🎉 FSM: CODE_LOCK → UNLOCKED (latched 45s)
    └─ All criteria passed with proper cadence!
  ```

### Failed Attempt Logs (Unencoded)

- [ ] Console shows:
  ```
  🔍 CODE_LOCK CANDIDATE:
    ├─ [match] ... z=X.XX ❌  (where X.XX < 2.5)
  ⏱️ FSM: CODE_LOCK → IDLE (timeout after X.Xs, no valid code detected)
  ```
- [ ] Verify z-score is the failing metric (discriminator)

---

## Recalibration Test

- [ ] Start RX, wait for "✅ Calibrated"
- [ ] Click "Recalibrate Baseline" button
- [ ] Observe console:
  ```
  📊 Calibrating... X/70 samples
  🎯 BASELINE CALIBRATION COMPLETE: ...
  ```
- [ ] Verify detection still works after recalibration
- [ ] Ensure TX was NOT running during recalibration (else mean > 0.4 error)

---

## Checkpoint Validation

Verify the working configuration documented at lines 1802-1804 of `presence-beacon.html`:

- [ ] **Encoded audio**: Correlation ≥ 0.35, PSR ≥ 9 dB, **Z ≥ 4.0** → Should unlock
- [ ] **Unencoded audio**: **Z ≤ 1.5** → Should NOT unlock (z-score discriminates)

**Observed Values** (from logs):
- Encoded: `corr=0.410, psr=9.7dB, z=4.61` → UNLOCK ✅
- Unencoded: `corr=0.28-0.31, psr=7-8dB, z=0.81-1.10` → REJECT ✅

---

## Failure Mode Tests

### TX Running During Calibration

- [ ] Start TX first (intentional error)
- [ ] Start RX → Wait 5 seconds
- [ ] RX calibration should fail with:
  ```
  ⚠️ CALIBRATION FAILED: Baseline correlation too high!
    ├─ Measured baseline: X.XXX (>0.4)
    └─ Likely cause: TX was running during calibration
  ```
- [ ] Console shows: "Stop TX & recalibrate"
- [ ] Stop TX, click "Recalibrate Baseline" → Should succeed

### High Ambient Noise

- [ ] Start RX in noisy environment (e.g., near HVAC, traffic)
- [ ] Observe baseline stddev (σ) in calibration logs
- [ ] Detection may require higher SNR (increase TX gain or reduce distance)
- [ ] Guards may show higher deviations (normal, within threshold)

### Low TX Volume

- [ ] Start TX with low system volume or low "Output Gain"
- [ ] Check console: `🔊 TX OUTPUT LEVEL: X.X dBFS`
- [ ] If X.X < -30 dBFS, RX may not detect
- [ ] Solution: Increase system volume or TX "Output Gain" (line 1061 in code, UI adjustable)

### Guard Threshold Exceeded (Near-Field)

- [ ] Place RX within 10 cm of TX speakers
- [ ] Start TX and RX
- [ ] Observe console: `[guards] Lo: X.XdB ❌` or `Hi: Y.YdB ❌` where X or Y > 10.0
- [ ] Solution: Increase distance to 20-40 cm, or increase `GUARD_THRESHOLD` (line 1810)

### Preamble Timeout

- [ ] Observe FSM state in UI: "PREAMBLE_LOCK"
- [ ] If it times out after 2s without transitioning to CODE_LOCK:
  ```
  ⏱️ FSM: PREAMBLE_LOCK → IDLE (timeout after 2.0s)
  ```
- [ ] Cause: Insufficient sign changes in preamble
- [ ] Solution: Lower threshold at line 1830 (0.6 → 0.5)

### Code Lock Timeout

- [ ] Observe FSM state in UI: "CODE_LOCK"
- [ ] If it times out after 5s without unlocking:
  ```
  ⏱️ FSM: CODE_LOCK → IDLE (timeout after 5.0s, no valid code detected)
  ```
- [ ] Review logs for which metrics are failing (correlation, PSR, z-score, guards, etc.)
- [ ] Solution: Increase `VOTE_WINDOW` (line 1813) to 7.0s, or relax thresholds

---

## Acceptance Criteria

### Minimum Pass Requirements

- [x] All parameters derived from `public/presence-beacon.html` with exact line numbers
- [x] No invented values - all thresholds traced to code
- [x] Working configuration documented from actual logs
- [x] Reproducible on desktop + iPhone
- [ ] **False-positive battery passes** (pending your testing)
- [x] Clear tuning-only controls identified (Tier 0 in handoff doc)
- [x] Code-change paths outlined (Tier 1, Tier 2 in handoff doc)

### Documentation Complete

- [x] `SPREAD-SPECTRUM-PRESENCE-BEACON.md` created with comprehensive handoff
- [x] `README.md` updated with Quick Start and link to handoff doc
- [x] `CHECKLIST.md` (this file) with concrete test procedures

### Performance Benchmarks

- [ ] **Encoded audio detection**: 100% success rate (10/10 tests)
- [ ] **Unencoded audio rejection**: 100% success rate (10/10 tests)
- [ ] **False-positive battery**: 0% false unlock rate (speech, claps, snaps, music)
- [ ] **Detection latency**: < 16 seconds (within 2 bursts)
- [ ] **Near-field operation**: Works at 20-40 cm distance
- [ ] **Far-field limit**: Detect at 1-2 meters (optional, depends on environment)

---

## Test Log Template

Copy this template for each test run:

```
### Test Run: [DATE] [TIME]

**Environment**:
- TX Device: [Desktop/Laptop, OS, Browser]
- RX Device: [iPhone/Android, OS, Browser]
- Distance: [XX cm]
- Ambient Noise: [Quiet/Moderate/Loud]

**Test Case**: [Encoded Audio / Unencoded Audio / False-Positive Battery]

**TX Settings**:
- Center Freq: [12000] Hz
- Spacing: [50] Hz
- Symbol Duration: [60] ms
- Gain: [±6.5] dB
- Burst Interval: [8] s

**RX Calibration**:
- Baseline Mean: [X.XXX]
- Baseline StdDev: [X.XXX]
- Detection Threshold: [X.XXX]

**Result**: [PASS / FAIL]

**Metrics** (if detected):
- Correlation: [X.XXX] (≥0.30)
- PSR: [X.X] dB (≥7.5)
- Z-Score: [X.XX] (≥2.5)
- Median |Δ|: [X.XX] dB (≥0.5)
- Energy Dev: [X.X] dB (≤1.0)
- Guards: Lo [X.X] dB, Hi [X.X] dB (≤10.0)
- Cadence: Δt=[XX] ms

**Notes**: [Any observations, issues, or anomalies]
```

---

## Troubleshooting Quick Reference

| Issue | Check | Fix |
|-------|-------|-----|
| No unlock | Console logs for z-score | If z < 2.5, increase TX gain or reduce distance |
| Calibration fails | TX running? | Stop TX, recalibrate |
| Guards fail | Distance too close? | Move to 20-40 cm or increase GUARD_THRESHOLD |
| Timeout in CODE_LOCK | Votes collected? | Check logs for failing metrics, adjust thresholds |
| No TX output | Check dBFS | Verify > -30 dBFS, increase volume |

---

**Last Updated**: 2025-01-07
**Status**: Ready for validation testing
