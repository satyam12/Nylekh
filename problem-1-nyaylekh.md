# Problem 1 — NyayLekh: dictation & drafting desk for an Allahabad High Court advocate

Status, 24 Oct 2026 (latest): Satyam: **"i wanted a free version not with google api or anything paid"**. Delivered:
- **free-test.html**: a microphone and a text box using the browser's own speech recognition. No key, account or payment.
- A rebuilt **index_01.html** in which the free engines come first and are the default.

He needs to upload both. The repo (checked 24 Oct by cloning) holds only the **4 Oct** index_01.html (index.html was deleted 4 Oct; no README). gemini-test.html was never uploaded, and the Gemini route is now optional or parked.

His earlier "microphone not allowed" on Android came from opening the downloaded file in a file/app preview (a WebView), not from the website. Still open: the Google OAuth Client ID for Drive sign-in (never provided).

## Brief
An advocate practising at the Allahabad High Court wants:
1. Hindi and English drafts transcribed with as few errors as possible.
2. Spelling checked with one click, English in UK standards.
3. A tab to format his content using his earlier templates.
4. Before finalising, a step-by-step recheck (party names, residue copied from the last template), then verify and release.

Use top-level speech recognition (OpenAI Whisper / Buzz for Windows, https://github.com/chidiwilliams/buzz/releases) and allow uploading sample templates. Templates carry confidential data, so it runs locally.

Decisions and requests over time:
- Decided with Satyam:
  - built-in Whisper plus Buzz support;
  - Hindi templates in both Kruti Dev/DevLys and Unicode;
  - handover = all three (Windows package, demo page with dummy data, user guide).
- 3 Oct: also a fully working browser version with the real microphone. Satyam published it on GitHub Pages (Claude artifacts cannot use the microphone).
- 3 Oct (v2), "no constraints like only this computer or browser — anyone can immediately access it on the go":
  - speech = the browser's built-in recognition (⚡ Instant), with Whisper kept as 🔒 Private;
  - sync = each user's own Google Drive ("Sign in with Google", scope drive.file);
  - Satyam registers the Google Cloud OAuth client (Web application, origin https://satyam12.github.io).
- 4 Oct: "should run on all devices seamlessly."
- 19 Oct, wanting WhisperX-level accuracy or better: chose Gemini 3.5 Transcribe, then asked for a simple Gemini test page.
- 20–24 Oct: Android microphone errors; "recheck for Android, iPad, PC and confirm running with testing an example."
- 24 Oct: wants it **free** — no Google API, nothing paid.

## Accuracy research (19 Oct 2026)
- **WhisperX gives no better words than Whisper.** It is faster-whisper plus VAD, wav2vec2 alignment and diarization.
- **Whisper is weak on Hindi.** Hindi WER figures:

  | Source | Model | Hindi WER |
  |---|---|---|
  | arXiv 2512.10967 | Gemini-2.5-pro | 18.5% |
  | arXiv 2512.10967 | Sarvam Saarika-2.5 | 39.0% |
  | arXiv 2512.10967 | Whisper-large-v3 | 71.7% |
  | IndicVoices | IndicConformer-600M | 14.4% |
  | IndicVoices | Sarvam saaras:v4 | 14.7% |

- **Gemini 3.5 Transcribe** costs about ₹25/h on the paid tier. Free-tier audio may be used to improve Google's products.
- **Free options:**
  - The browser's Web Speech API is free and keyless. The voice goes to Google (Chrome/Android), Apple (Safari) or Microsoft (Edge).
  - Whisper on the device (transformers.js) is free and private, but slower and weaker on Hindi.
  - Chrome's on-device Web Speech (processLocally) is desktop only, and Hindi support is unclear.

## Deliverables
- **Website**: repo https://github.com/satyam12/Nylekh (GitHub Pages, main/root). Satyam uploads from his phone or iPad. This session can read (clone) but not push. Deliver files named **index_01.html**, **free-test.html** and gemini-test.html; never a file named index.html. There is no OAuth Client ID in the builds yet.
- **free-test.html** (24 Oct, `web/free-test.html`, 30 KB, standalone, no external requests; link https://satyam12.github.io/Nylekh/free-test.html):
  - Hindi/English switch, big microphone button, live line, text box (kept in localStorage `freetest.`), Copy/Clear.
  - Voice commands in JS: full stop/पूर्ण विराम → . or ।, comma, new paragraph/line, brackets, question mark, etc. Also Hon'ble Court and sentence capitals.
  - "✓ Check this device": website/browser · speech recognition available · read a sentence aloud (% of words recognised). "Copy report" button.
  - Per-device handling:
    - Chrome on a computer: continuous recognition.
    - Android: one phrase per session, with restarts, to avoid repeated words.
    - Safari: commits the growing interim line after a 1.2 s pause, at session end and on Stop. A refused restart is shown as "paused".
  - Device-specific help when the microphone or recognition is refused. Banner for in-app WebViews and Firefox.
- **index_01.html** (24 Oct, free-first build):
  - Dictate dropdown: ⚡ Instant (free, the default) and 🔒 Private (free).
  - ✨ Gemini appears only after a key is saved. It sits in Settings › Speech engine under a folded "Optional, paid … not needed" section.
  - The engine pill reads "⚡ Instant (free)". Also includes Drive sync and device microphone help.
- **gemini-test.html v3** (`web/gemini-test.src.html` + `web/examples/`, built by `tools/build_gemini_test.py`) is the Gemini test page with a self-check. It needs a paid key, so it is now optional.
- Windows package `NyayLekh-1.0.0-windows.zip` (7.5 MB), which includes `NyayLekh-Web.html` (v1).
- Demo page (private until shared): https://claude.ai/artifact/GQCiqYov7iugMudwyWnV9d
- User guide (Claude Doc; describes v1 and needs updating): https://claude.ai/artifact/Mbqw6j66NpGwDvCGjjh7gE

## How it works (desktop)
- A local FastAPI server on 127.0.0.1:8765 with a plain-JS UI and no CDNs.
- Speech: faster-whisper (large-v3-turbo / large-v3 / small, plus an optional Hindi specialist) or the Buzz CLI.
- Text handling: amber marks on unsure words, names fed in as prompt/hotwords, voice commands, US → UK spelling, legal auto-corrections.
- Spelling: Hunspell (spylls), en_GB and hi_IN.
- Templates: .docx/.doc → fields and zones; Kruti Dev → Unicode.
- Verify & release: 7 steps; outputs a cleaned .docx, a Kruti Dev copy, a PDF and a certificate with SHA-256.

## How the browser version works
- The desktop Python code runs in Pyodide 314.0.7 in a module worker, routed by `app/webapi.py`. First start downloads about 15 MB. Built with `tools/build_web.py`.
- Engines are chosen per device:
  - ⚡ Instant: Web Speech API; free; the default.
  - 🔒 Private: Whisper via transformers.js; free.
  - ✨ Gemini: optional; uses the user's own key.
- Storage and sync: IndexedDB plus Google Drive sync in `web/sync.js` (drive.file).

## Verified (24 Oct)
- **free-test.html, Chrome engine** (`web/tests/e2e_free_test.py`, scripted recogniser `web/tests/fake_speech.js`): **46/46**.
  - Runs as a PC at 1280 px and as Android (Pixel UA, 412 px).
  - Text rules, self-check at 100%, report.
  - Dictation gives exactly "The applicant Ram Kumar was arrested on 12 March 2026.¶He is a resident of Soraon district Prayagraj, and the Hon'ble Court may kindly grant bail."
  - Continuous mode on PC and one phrase per session on Android.
  - Switching to Hindi mid-dictation (danda), text kept after reload, device-specific refused-microphone messages, no sideways scroll, WebView and Firefox banners.
- **free-test.html, Safari engine** (`web/tests/e2e_free_test_webkit.py`, WebKitGTK with iPad UA, Safari-style results that are never final): **9/9**.
  - Self-check at 100%; words written after each pause; last words written on Stop.
  - A silence leads to "paused" with the words kept; refused access → Safari steps.
- **Main app** `web/tests/e2e_hosted.py`: **63/63**.
  - Only free engines offered, with Instant first; "Instant (free)" pill.
  - Gemini hidden until a key is saved; Drive sync; Gemini with a fake API.
- Earlier: gemini-test 42/42 (Chrome) and 22/22 (WebKit); 127 pytest tests; desktop end-to-end.

## Open / not yet verified
- Real phones, the iPad and real browser speech services cannot be reached from the sandbox. The page's own "✓ Check this device" is the confirmation on each device; ask for the copied report.
- Free ⚡ Instant sends the voice to Google, Apple or Microsoft. For confidential matters use 🔒 Private or the Windows app.
- Desktop not yet run on real Windows hardware. Real accuracy with the advocate's own voice is unknown.

## Rebuild
- `python tools/build_web.py index_01.html [--client-id <ID>]`
- free-test.html needs no build; copy `web/free-test.html`.
- Tests: `python web/tests/e2e_free_test.py`; `xvfb-run -a /usr/bin/python3.12 web/tests/e2e_free_test_webkit.py`; `python web/tests/e2e_hosted.py` (build to /tmp/nl-web/index.html first); `python -m pytest tests`.
