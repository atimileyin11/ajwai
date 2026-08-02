# Ajwai

**AI freshness and demand intelligence for perishable food distribution.**

Live site: [ajwai.site](https://ajwai.site) · Live demo: [ajwai.site/demo.html](https://ajwai.site/demo.html)

---

## The problem

A food distributor buys hundreds of crates a week with no reliable read on the condition they arrived in. Quality is judged by eye, in seconds, on the loading bay. Stock is dispatched roughly in the order it was stacked.

By the time a crate is visibly bad, the money is gone — and the buyer who would have taken it two days earlier bought elsewhere. Every discarded crate is paid-for stock, paid-for transport, and paid-for storage written off at once, on categories where margins are already thin.

## What Ajwai does

Turns intake into data. A phone or fixed camera photographs each lot; the system grades it, estimates remaining sell window from that grade plus storage and transit conditions, and ranks the whole warehouse by urgency. Demand forecasting then matches the most urgent stock to the buyers most likely to take it now.

| Capability | Status |
|---|---|
| Image-based lot grading | Demo working (classical CV) |
| Shelf-life estimation | Demo working |
| Dispatch ranking | Demo working |
| Trained CNN grader | In development — needs labelled data |
| Demand forecasting | Specified, not started |
| Buyer matching | Specified, not started |

## Current status

**MVP in development.** This repository contains the marketing site and a working browser-based grading demo. The demo is a real measurement pipeline, not a mockup — it produces different results for different photographs.

We are recruiting Lagos depots as design partners to contribute labelled intake imagery. Model training begins once that dataset exists.

## Repository contents

```
index.html     Main site
demo.html      Browser-based grading demo (no backend, no upload)
privacy.html   Privacy policy
terms.html     Terms of use
thanks.html    Form confirmation page
```

No build step. Static files, deployed to Netlify. Clone and open `index.html` to run locally.

## How the demo works

Everything runs client-side in the browser. No image leaves the device.

1. **Downsample** the image to a canvas, max 480×360.
2. **Convert** every pixel from RGB to HSV.
3. **Segment** produce from background using saturation and value thresholds. If the resulting mask covers less than 8% of the frame, the demo refuses to guess and asks for a cleaner shot.
4. **Measure** within the mask:
   - hue distribution across green / yellow / red bands
   - defect fraction from dark pixels (`V < 0.28`) and brown pixels (`H ∈ [10°, 50°]`, low saturation, low value)
   - circular hue standard deviation as a lot-consistency metric
5. **Score** a quality index (0–100) penalised by defect load, colour spread, and over-maturity, mapped to grades A / B / C / below spec.
6. **Estimate shelf life** from a per-crop baseline, reduced by maturity and defect load, then scaled for temperature using a Q10 coefficient of 2 — respiration rate roughly doubles per 10°C rise.
7. **Rank** graded lots by remaining hours into a dispatch order.

### What this is not

This is classical computer vision, not the trained grading model. It exists to prove the measurement pipeline and decision output work end to end. It is approximate, and cluttered backgrounds confuse it. The CNN grader is the next milestone and requires labelled intake imagery we do not yet have.

## Planned architecture

| Layer | Technology |
|---|---|
| Image storage | Amazon S3, lifecycle rules to cold tiers |
| Application data | Amazon RDS |
| Inference API | AWS Lambda / ECS |
| Model training & serving | Amazon SageMaker |
| Report generation | Amazon Bedrock |
| Asset delivery | Amazon CloudFront |
| Auth | Amazon Cognito |
| Monitoring | Amazon CloudWatch |
| Model acceleration | CUDA, TensorRT, Triton Inference Server |
| Edge grading | NVIDIA Jetson (low-connectivity depots) |

Grading must return a result while the truck is still being unloaded, which puts inference latency on the critical path — hence GPU inference rather than CPU.

## Roadmap

- [x] Phase 0 — Site, demo pipeline, positioning
- [ ] Phase 1 — Intake capture app, lot records, per-crop baselines
- [ ] Phase 2 — Design-partner depots, labelled data collection, CNN training
- [ ] Phase 3 — Demand forecasting and buyer matching
- [ ] Phase 4 — AWS production deployment, retraining pipeline, RBAC
- [ ] Phase 5 — GPU inference at scale, edge grading
- [ ] Phase 6 — Multi-depot enterprise tier, additional markets

## Team

- **Adeola Timileyin** — Founder & technical lead. Software developer and computer science student at Kwara State University. Leads architecture, the grading pipeline, and the shelf-life model. [LinkedIn](https://www.linkedin.com/in/adeola-timileyin-4599a2325)
- **Abdulfaruk** — AI engineer. Leads model development: grading, defect detection, shelf-life estimation.
- **Samuel William** — Operations lead. Leads depot engagement, design-partner recruitment, and product operations.

## Contact

[hello@ajwai.site](mailto:hello@ajwai.site)

---

© 2026 Ajwai. All rights reserved. This repository is public for transparency; the code is not licensed for reuse.
