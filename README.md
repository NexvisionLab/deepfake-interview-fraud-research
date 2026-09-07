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

The physiological liveness signal at the core of our live-call research (see the companion report) is directly applicable here as one layer of a broader check: the same detection method that resists a real-time face-swap on a live call applies just as well to a screening interview.

A hiring-specific product would likely need to combine that with methods already standard in identity-verification (KYC) products, adapted to an interview context:

- **Liveness detection** (our physiological-signal research) — is there a real, present human being on camera at all, applied at interview time rather than continuously.
- **Document-to-face matching** — does the person on camera match the photo ID they registered with.
- **Cross-session consistency** — does the same candidate appear (voice, face) consistently across the screening call, technical interview, and any later check-ins, catching a scheme where one person interviews and a different person is actually hired to do the work.

## What we have not done yet

This report deliberately stops short of a product design document. Before that's warranted, we think three open questions need real answers:

1. **Does our liveness-detection research actually work on interview-style footage** — webcam quality, typical interview lighting and framing — as opposed to the synthetic data it's been validated against so far. See the companion live-call research report for the current state of that validation.
2. **What do buyers in this space (HR tech, background-check vendors, applicant-tracking-system providers) actually need**, and is a standalone product the right shape, or is this better delivered as a feature license into an existing hiring-verification platform.
3. **What's the realistic false-positive cost** in a hiring context — a live-call security tool interrupting a legitimate meeting is an inconvenience; a hiring-verification tool wrongly flagging a real candidate is a discrimination and liability risk that needs to be designed for explicitly, not treated as an afterthought.

## Next steps

1. Resolve the technical open questions in the companion live-call detection research first, since this product depends on that same core signal working reliably.
2. Talk to hiring-verification and background-check vendors about how this would need to integrate with existing workflows.
3. Scope a false-positive/fairness review before any product commitment, given the discrimination-liability stakes of a hiring-context false positive.

---

*This is a research summary, not a product announcement or a claim of production readiness.*

## Sources

- U.S. Department of Justice, press releases on North Korean IT worker scheme prosecutions (2024–2025)
- Federal Bureau of Investigation, public advisories on North Korean remote IT worker fraud
- Multi-national government advisory on real-time deepfake use in hiring fraud (August 2026)
