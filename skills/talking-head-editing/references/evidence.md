# 📚 Evidence and limits

These sources informed the method. They are practitioner documentation and product/tool descriptions, not independent proof that an agent can edit arbitrary footage perfectly. Reopen current sources before relying on changing tool features or installation instructions. The local failure mechanisms in the companion reference are operational lessons, not universal numeric editing laws.

## 🧰 Agent editing workflows

- [Video Use instructions](https://github.com/browser-use/video-use/blob/main/SKILL.md) describe transcript-led work with visual inspection and rendered-boundary checks. The useful principle is to combine word timing, waveform and image context. Its cloud transcription path may conflict with local-only requirements; adapting these checks does not mean the whole tool was installed or run. Padding values are starting heuristics, not a quality guarantee.
- [Proofcut](https://github.com/tydude001/proofcut), its [trial documentation](https://github.com/tydude001/proofcut/blob/main/docs/TRIAL.md) and [local workflow notes](https://github.com/tydude001/proofcut/blob/main/docs/plans/LOCAL.md) document attempts at verification and limitations of local recognition/review. Reported trials are developer evidence, not independent validation. Check its license before describing it as open source; source availability alone does not establish an OSI-approved license.

## 🎥 Editing and perception references

- [Adobe jump-cut overview](https://www.adobe.com/creativecloud/video/post-production/cuts-in-film/jump-cut.html): visible discontinuity can be an intentional pacing choice. Do not classify every detectable cut as a defect.
- [Adobe Morph Cut overview](https://helpx.adobe.com/premiere/desktop/add-video-effects/apply-video-transitions/morph-cut-overview.html): smoothing depends on suitable footage and can introduce artifacts; it is not a substitute for good boundary selection.
- [Adobe J/L cut documentation](https://helpx.adobe.com/ca/premiere/desktop/edit-projects/trim-clips/perform-j-cuts-and-l-cuts.html): sound and image transitions can occur at different times. Preserve lip synchronization wherever the speaking face remains visible.
- [MediaPipe Face Landmarker](https://developers.google.com/edge/mediapipe/solutions/vision/face_landmarker): facial landmarks and expression signals can support candidate detection for mouth, eyelid and head changes. This is an optional implementation direction, not an implemented editor or proof of natural cuts. Such signals still require temporal and semantic interpretation.

## ⚖️ Claims the evidence does not support

No cited source establishes an always-perfect, no-review agent workflow, a mandatory zoom at every cut, a universal silence threshold, or ASR agreement as proof of correct speech. If direct audio perception is unavailable, say so and distinguish the checks that were actually possible. Preserve this limitation in the project handoff rather than letting summaries imply that a full listening review occurred.
