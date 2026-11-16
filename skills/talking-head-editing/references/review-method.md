# 🔍 Failure mechanisms and review criteria

## 🧩 Why transcript-only editing fails

The following observed failure classes require different evidence. Treat all as candidates at each suspicious moment; do not stop after finding one plausible explanation.

| Failure class | Mechanism | Detection and repair |
| --- | --- | --- |
| Hidden repetition | Recognition cleans up an audible repeated word or abandoned start. | Compare overlapping short windows, source context and actual audio. Inspect inside retained takes. Retain the complete intended attempt, not a leftover word from an earlier one. |
| Chopped onset or release | Word timestamps are approximate; a cut lands inside a quiet consonant or trailing syllable. | Compare waveform/speech evidence with nearby source and audiovisual playback; expand or move the boundary, or choose another complete take. |
| Lingering silence | A transient divides one speech gap and a loop only processes its first silence interval. | Examine the entire gap, classify the transient, join only non-speech subdivisions and verify remaining duration after trimming. |
| Incoming idle expression | Clip begins before actual speech while mouth, gaze or expression changes. | Locate actual onset, remove unnecessary idle lead-in, retain natural preparatory movement and quiet phonemes. |
| Outgoing abandoned articulation | Mouth begins the next discarded word before the cut; the next clip resets the mouth. | End after the retained word completes but before discarded articulation; consider another take if no clean boundary exists. |
| Discontinuous gesture | Head, eyes, hands or objects jump between source moments. | Compare moving sequences, choose compatible phases or adjust framing where justified; a crop cannot conceal every discontinuity. |
| Wrong diagnosis | A reported timestamp is mapped to an old export or only a visual hypothesis is considered. | Identify viewed version, inspect a neighborhood and map through its real manifest. Check wording, sound, timing and visuals together. |

## 🎞️ Boundary review procedure

For every join, inspect outgoing and incoming context in sequence. A useful starting window is roughly 1.5 seconds on each side, widened for meaning or gesture completion. Inspect finer frames near suspicious articulation; sparse stills can miss a blink or mouth opening.

Use these independent review dimensions:

- **Meaning and wording:** complete thought, correct take, no unintended repetition or abandoned word, no agent-directed aside. Preserve intentional rhetorical repetition.
- **Word integrity:** full incoming onset and outgoing release, including quiet unvoiced consonants; no mid-word cut hidden by a transcript.
- **Rhythm:** natural breath and sentence cadence, no accidental waiting or rushed removal of an intentional beat.
- **Visual action:** mouth phase, gaze, eyelids, head, hands and held objects. Looking away or blinking is not automatically defective.
- **Composition:** deliberate crop changes, sensible headroom and sufficient source resolution; no arbitrary alternating zoom pattern.
- **Synchronization and sound:** lips match the retained source audio, no click or abrupt ambience change, intelligible speech at consistent perceived level.

Waveform thresholds are candidate generators, not semantic decisions. Noise gating can make starts appear sharp; quiet consonants may be mistaken for silence. A lead-in around 60–100 ms was helpful in some recordings, while other references use wider padding. Derive boundaries from the actual utterance rather than enforcing those numbers. Apply short audio fades only where they do not attenuate retained speech.

## ✂️ Choosing the repair

First repair take selection and word boundaries. If the candidate is incomplete, search another complete attempt before trying to rescue it with more trimming. Next preserve natural cadence and choose compatible visual action. Finally decide whether framing, an authorized relevant insert, or a different cut improves the remaining discontinuity.

A zoom on every cut and zero pause after every cut are not general rules. Intentional jump cuts can suit talking-head content. Crop changes may improve emphasis or mask small repositioning, but cannot repair duplicated words or cut syllables. J/L cuts can help transitions between different views; do not detach a visible speaker's lips from their own audio. Morphing can introduce artifacts and needs specific visual verification.

## ✅ Regression and output checks

Use prior failure classes as regression cases on each new piece: hidden duplicate word, clipped initial consonant, silent facial lead-in, discarded outgoing mouth movement, multi-part pause, intentional repetition and replacement complete take. Do not transplant old timestamps or crop choices.

Re-recognition of the rendered audio can find discrepancies but may repeat the original recognizer's omissions. A second recognizer is corroboration, not ground truth. Keep intended text, detected discrepancies and actual evidence separate. Compare all joins and suspicious mid-take spans, plus the opening and ending. Verify the rendered media, not merely the plan.

Record a review ledger with source/output version, time range, defect hypothesis, evidence by modality, change and result. Distinguish confirmed defects, stylistic choices and unresolved cases. Full decode, frame count, waveform correlation and loudness measurement establish technical properties; they do not establish coherent wording or perceptual naturalness.

When final pieces will be assembled later, retain edit decisions and audio before delivery compression. A later edit invalidates affected timing maps, transcript checks and potentially programme loudness. Label verification files by the actual output they checked; never transfer one version's measurement to another without remeasurement.
