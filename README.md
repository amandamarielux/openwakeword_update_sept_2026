# Custom Wake Word Trainer for Home Assistant

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/amandamarielux/openwakeword_update_sept_2026/blob/main/wakeword_trainer_HA.ipynb)

## Credits

- Based on [openwakeword-colab-2026](https://github.com/alfiedennen/openwakeword-colab-2026) by Alfie Dennen — the bulletproof Colab notebook this builds on. This version adds `.tflite` export and Home Assistant setup.
- [openWakeWord](https://github.com/dscripka/openWakeWord) by David Scripka.
- [onnx2tf](https://github.com/PINTO0309/onnx2tf) by PINTO0309.
- [piper-phonemize-fix](https://pypi.org/project/piper-phonemize-fix/).
- Home Assistant [openWakeWord add-on](https://github.com/home-assistant/addons/tree/master/openwakeword).

Train your own [openWakeWord](https://github.com/dscripka/openWakeWord) wake word in a single Colab notebook and use it with Home Assistant. **Run all → walk away → download `.tflite` → drop it into HA.**

Unlike most openWakeWord notebooks, this one outputs a **`.tflite`** file (as well as `.onnx`). That matters because the Home Assistant openWakeWord add-on **only loads `.tflite` custom models** — the `.onnx` files most notebooks produce are silently ignored. No separate conversion step.

---

## What you need

- A Google account (for Colab). Free tier works.
- Home Assistant OS with the **openWakeWord** add-on installed and running.
- A voice satellite (e.g. M5Stack Atom Echo, ESP32-S3-BOX, or any Wyoming satellite).
- ~90 minutes, and a computer that won't go to sleep during that time.

---

## Quick start

### 1. Train the model

1. Upload `wakeword_trainer_HA.ipynb` to [Google Colab](https://colab.research.google.com) (File → Upload notebook).
2. **Runtime → Change runtime type → T4 GPU** (free). L4 + High RAM on Colab Pro is ~2× faster but not required.
3. Find the config cell (**cell 21**) and edit two lines:

   ```python
   TARGET_PHRASE = ['hey snoop dogg', 'yo snoop dogg', 'snoop dogg']
   MODEL_NAME    = 'snoopdogg'
   ```

4. **Runtime → Run all.**
5. Walk away ~75–90 min. The last cell auto-downloads `<MODEL_NAME>.tflite` and `<MODEL_NAME>.onnx` to your computer.

### 2. Install it in Home Assistant

1. Open **Studio Code Server** (or the Samba add-on) on your HA box.
2. If it doesn't exist, create the folder `/share/openwakeword/`.
3. Copy `<MODEL_NAME>.tflite` into **`/share/openwakeword/`**.
4. **Settings → Add-ons → openWakeWord → Restart.**
5. Check the add-on **Log** tab — you should see:

   ```
   Found custom model <MODEL_NAME> at /share/openwakeword/<MODEL_NAME>.tflite
   ```

6. **Settings → Voice assistants →** open your assistant → scroll to **Streaming wake word engine** → select **openwakeword** → pick your model in the second dropdown → **Update**.
7. Say your wake word at a voice device to test.

> One assistant = one wake word. To run several wake words at once, assign each to a separate assistant/device.

---

## Choosing a good wake word

Wake word quality depends more on the phrase than the training:

- **Use 2–4 syllables.** One-syllable words (e.g. "snoop") trigger constantly on background noise.
- **Keep all `TARGET_PHRASE` variants a similar length.** Mixing "snoop dogg" with "hello there snoop dogg" teaches the model that a short sound is enough — more false fires. All variants train one model that fires on any of them.
- **Pick something uncommon** so normal conversation doesn't set it off.

Tuning after install (openWakeWord add-on → Configuration):

- **Threshold** (default `0.5`) — raise it for fewer false triggers, lower it if it's not catching you.
- **Trigger level** (default `1`) — number of activations before a detection registers.

---

## Why this notebook exists

The official openWakeWord training notebook has bit-rotted badly. On current Colab it fails out of the box for several reasons, most notably:

- **`piper-phonemize` / `piper-phonemize-cross` have no wheels** for current Colab Python (3.12/3.13). Fixed here by installing [`piper-phonemize-fix`](https://pypi.org/project/piper-phonemize-fix/) first (cell 1).
- **`torchaudio` 2.x removed `set_audio_backend`**, which older augmentation libs still call — patched at runtime.
- **openWakeWord's config schema and example YAML moved/renamed** across versions — this notebook builds a fully self-contained config instead of depending on the upstream `examples/` dir.
- **The stock export path only emits `.onnx`** — this notebook adds `.tflite` conversion via [`onnx2tf`](https://github.com/PINTO0309/onnx2tf) so the output works with Home Assistant directly.

Cells are written to be self-healing (they check their own outputs and re-create what's missing), so re-running after a disconnect is safe.

---

## Known limitations

- **It can't run 100% unattended.** Colab disconnects when your computer sleeps or after a period of idle, and the final `files.download()` needs the browser tab open. Keep the laptop awake and the tab in front.
- **No resume.** If the runtime dies mid-run, reopen and **Run all** again — it re-downloads everything from scratch (~90 min).
- **English only.** openWakeWord currently only trains reliable English models.
- **`.onnx` won't show up in the HA add-on** — that's expected; use the `.tflite`.

---

## Troubleshooting

| Symptom | Fix |
|---|---|
| `No matching distribution found for piper-phonemize-cross` | Harmless — `piper-phonemize-fix` (cell 1) already covers it. Look for "deps import cleanly." |
| `ModuleNotFoundError: No module named 'openwakeword'` | You ran a later cell before the install cells finished. Run all from the top. |
| Clip generation writes 0 clips | Same as above — openWakeWord wasn't installed yet. Run all in order. |
| Model isn't in the HA dropdown | It's `.onnx`, not `.tflite`, or it's in the wrong folder. Must be `/share/openwakeword/<name>.tflite`. Check the add-on Log. |
| Add-on Log never mentions the model | Wrong folder, or add-on wasn't restarted after adding the file. |
| Wake word over-triggers | Raise **threshold** in the add-on config; pick a longer phrase. |
| Runtime disconnected | Your computer slept. Reopen, set GPU, Run all again. |

---

## Files

- `wakeword_trainer_HA.ipynb` — the trainer notebook.
- `README.md` — this file.

## Credits

- [openWakeWord](https://github.com/dscripka/openWakeWord) by David Scripka.
- [onnx2tf](https://github.com/PINTO0309/onnx2tf) by PINTO0309.
- [piper-phonemize-fix](https://pypi.org/project/piper-phonemize-fix/).
- Home Assistant [openWakeWord add-on](https://github.com/home-assistant/addons/tree/master/openwakeword).
