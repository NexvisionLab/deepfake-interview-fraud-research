# Deepfake Hiring Fraud: Market & Product Research

**Status: early-stage product research. Not a shipped product.**

This report summarizes research into a distinct but related problem from live-call authentication: verifying that the person a company is *hiring* — often for a fully remote role, over a video interview — is who, and what, they claim to be.

## The problem, at scale

This is not a hypothetical risk. U.S. federal prosecutors and the FBI have documented a sustained, large-scale scheme in which North Korean state-linked operatives obtain remote IT jobs at Western companies using stolen identities, AI-generated personas, and — increasingly — **real-time deepfake video during the interview itself**, specifically to defeat identity checks a hiring manager would otherwise catch. The FBI has identified over 300 U.S. companies unknowingly affected; DOJ actions in 2024 and 2025 led to multiple guilty pleas and indictments, including operators of U.S.-based "laptop farms" that let overseas operatives remotely control company-issued equipment while appearing to work domestically. An estimated $800 million in wages was funneled to North Korea through this scheme in 2024 alone. In August 2026, eleven countries issued a joint warning specifically about the use of real-time deepfake tooling to beat hiring-verification checks.

The consequences aren't limited to salary diversion. Companies in this scheme have had source code, credentials, and in at least one documented case over $900,000 in cryptocurrency stolen by operatives who were hired — and passed interviews — under a false identity.

## Why this is a different product from live-call security, not the same one

Our related research into [real-time deepfake detection for live video calls](https://github.com/NexvisionLab/deepfake-call-detection-research) targets a continuous-monitoring problem: is *this ongoing call, right now*, being manipulated. Hiring fraud is a different shape of problem:

- **A gate, not a stream.** A hiring decision happens at a small number of discrete checkpoints (screening call, technical interview, offer, onboarding) rather than needing continuous monitoring of every subsequent meeting.
- **A different buyer.** The customer for this is HR, recruiting, and identity-verification/background-check vendors — not the security or IT teams who'd buy live-call protection.
- **Identity, not just liveness.** Detecting that a candidate isn't using a real-time deepfake is necessary but not sufficient — the harder and more valuable question is whether the underlying identity itself is genuine, which points toward document verification and cross-session consistency checks as well as pure liveness signals.

## Proposed technical direction

The physiological liveness signal at the core of our live-call research (see the companion report) is directly applicable here as one layer of a broader check — but a 2026-09-08 literature review (below) found this layer is weaker against the realistic threat model than our original framing assumed, which changes which layer actually carries the weight.

A hiring-specific product would likely need to combine that with methods already standard in identity-verification (KYC) products, adapted to an interview context:

- **Liveness detection** (our physiological-signal research) — is there a real, present human being on camera at all, applied at interview time rather than continuously. **Weaker than assumed against face-swap specifically — see below.**
- **Document-to-face matching** — does the person on camera match the photo ID they registered with. Now looks like the load-bearing layer, not a supporting one.
- **Cross-session consistency** — does the same candidate appear (voice, face) consistently across the screening call, technical interview, and any later check-ins, catching a scheme where one person interviews and a different person is actually hired to do the work.

## Technical feasibility scoping (2026-09-08)

Real research against each of the three layers above, done after our companion live-call report reached its own first real-footage milestone (see that repo for the validation details). No code was written for this product - this is scoping, not a build.

### Liveness layer: a real finding that changes the architecture

The literature is clear on a point our original framing glossed over: **face-swap deepfakes - the realistic real-time attack this fraud scheme actually uses - partially preserve the source video's own physiological signal.** A face swap replaces the visible face but the underlying video substrate (the operator's own real, live skin, off-camera or at the swap's edges) is still there, so the CHROM-style pulse signal our companion research validates can still detect "a real human is present" - it just can't tell you WHICH real human. The operator's own genuine pulse can leak straight through the swap. This is the opposite problem for talking-face/fully-synthetic avatars (no real underlying video exists at all, so a real physiological signal genuinely shouldn't be there) - but the DOJ prosecutions this report opened with describe live human operators using real-time face-swap tooling, not audio-driven synthetic avatars. Against the actual documented threat, liveness detection alone answers "is someone real on this call" when the question that matters is "is THIS PERSON real" - a real human, just the wrong one, passes it cleanly.

This doesn't make the liveness layer worthless (it still catches fully-synthetic avatar attacks, and it's one signal among three), but it means **document-to-face matching is doing the actual identity-verification work in this architecture, not liveness** - worth being explicit about rather than letting "we have a liveness detector" imply more than it can deliver.

One partially reassuring, unrelated finding for the companion report's own open compression question: earlier work (Ciftci et al.-adjacent literature on rPPG for deepfake detection) found heart-rate-band frequency content can still contribute meaningfully to detection even in compressed video - not a substitute for testing it directly against real call-bitrate footage, but not a reason to assume the signal is destroyed by compression either.

### Document-to-face matching: technically proven, licensing needs a real conversation

InsightFace/ArcFace is the realistic open-source technical path - proven at national scale (used for e-ID deduplication across well over 100 million portraits in multiple countries) and the standard baseline in published ID-photo-to-selfie matching research. But there's a real licensing wrinkle worth flagging now, before any cost estimate assumes this is free: **the InsightFace code itself is MIT, but its own pretrained recognition models (buffalo_l, antelope - the ones any real deployment would actually use) are licensed for non-commercial research only**, with commercial use requiring a paid license negotiated directly with InsightFace. A hiring-verification product is unambiguously commercial, so this is a real cost/timeline item for any actual build, not a detail to discover later.

### Cross-session consistency: partially already built, different legal footing than our crawler work

The voice half of this is a near-direct reuse of a technique already shipped elsewhere in our portfolio: the OSINT platform's own speaker voice-fingerprinting (speechbrain/spkrec-ecapa-voxceleb ECAPA-TDNN embeddings, cosine similarity) answers exactly the "same voice across sessions" question this product needs. The face half would need the same InsightFace/ArcFace embedding stack as the document-matching layer above - one licensing conversation would cover both.

Worth being explicit about a real distinction from that other platform's own policy: its face-detection work deliberately never computes a face embedding, specifically because it processes crawled dark-web images of people who never consented to any biometric processing - identifying/re-identifying them would cross a real legal line (GDPR/BIPA-style biometric-privacy rules). A hiring candidate's face embedding, computed with their knowledge as an explicit condition of a job application they chose to submit, sits on completely different legal ground (informed consent, a clear and disclosed purpose). This product does need consent-and-disclosure design work of its own, but it is not the same problem the other platform deliberately avoided - the two shouldn't be conflated just because both involve face embeddings.

### Fairness: a real, quantified, currently-unaddressed risk

Our original report flagged discrimination/liability risk as something needing explicit design attention rather than being treated as an afterthought. Real numbers now exist to back that up, and they're not small: published rPPG accuracy studies show mean absolute heart-rate error roughly **triples** for the darkest skin tones (Fitzpatrick VI) compared to lighter ones - one comparison found CHROM-based extraction (the exact method our companion research uses) going from 5.2 bpm error on Fitzpatrick I-III subjects to 14.1 bpm on Fitzpatrick V-VI, and a separate meta-analysis found errors as high as 13.6 bpm on the darkest category versus 2-4 bpm on lighter ones. The mechanism is physical, not a training artifact: darker skin absorbs more of the light whose subtle reflectance changes carry the pulse signal, weakening the signal-to-noise ratio the whole method depends on. Compounding this: the public dataset our companion research just validated against (UBFC-rPPG) is itself only about 5% dark-skinned subjects, and this is typical - a survey of 100 rPPG studies found darker skin tones significantly underrepresented across the field's own benchmark datasets. Concretely, this means the real-footage validation milestone the companion report just reached does not, on its own, say anything about how this would perform across the actual demographic range of a real hiring pool - and a liveness signal that's measurably worse at confirming a real human is present for candidates with darker skin is exactly the kind of design flaw that turns into a discrimination claim, not a footnote.

## What we have not done yet

This report deliberately stops short of a product design document. Three open questions from before are now partially answered by the scoping above; what's still genuinely open:

1. ~~Does our liveness-detection research actually work on interview-style footage~~ - **partially answered**: the companion report's first real-footage validation (webcam-quality, controlled indoor lighting, front-facing - a reasonable proxy for a screening-call setup) ran end-to-end and produced a plausible result. Still open: whether it survives real video-call compression specifically, and now also whether it's worth relying on at all against face-swap given the finding above.
2. **What do buyers in this space (HR tech, background-check vendors, applicant-tracking-system providers) actually need**, and is a standalone product the right shape, or is this better delivered as a feature license into an existing hiring-verification platform - unchanged, still needs real conversations, not more research.
3. **What's the realistic false-positive cost** in a hiring context - now backed by real, quantified skin-tone-accuracy numbers showing this is not a hypothetical risk to design around later.

## Next steps

1. Get real interview-style footage (webcam quality, varied lighting, varied skin tones - not just one more UBFC-rPPG-style clip) to actually test the fairness gap quantified above, rather than reasoning about it from literature alone.
2. Talk to hiring-verification and background-check vendors about how this would need to integrate with existing workflows, and whether they already license a comparable identity-verification stack (Persona, Onfido, Jumio, Veriff, and similar KYC vendors already do document-to-selfie matching commercially - worth understanding what they don't yet do that this would add, given liveness alone doesn't add much against face-swap).
3. Get a real quote from InsightFace for commercial model licensing before assuming the document-matching layer is a free, off-the-shelf component.
4. Scope a false-positive/fairness review as an actual design requirement, not a later add-on, given the quantified skin-tone gap above - this should shape the product from the first architecture decision, not be bolted on after a prototype exists.

---

*This is a research summary, not a product announcement or a claim of production readiness.*

## Sources

- U.S. Department of Justice, press releases on North Korean IT worker scheme prosecutions (2024–2025)
- Federal Bureau of Investigation, public advisories on North Korean remote IT worker fraud
- Multi-national government advisory on real-time deepfake use in hiring fraud (August 2026)
- Published research on physiological signals as a forensic modality for deepfake detection, and on face-swap vs. fully-synthetic (talking-face) deepfakes' differing relationship to source-video physiology (2026)
- Published research on rPPG signal use in deepfake detection under video compression (ICIAP 2023)
- Published research on demographic/skin-tone bias in remote photoplethysmography, including CHROM-specific accuracy figures across the Fitzpatrick skin-type scale, and on dataset demographic composition across the rPPG research field
- InsightFace project documentation, on ArcFace/InsightFace's use in national-scale ID-deduplication deployments and its own model licensing terms
