<br/><br/>

**Business Requirement Document (BRD)  
Project KonKit**

Finalized with Business Forecast

<br/>

_"Stay visible, even when you're busy."_

## I. Project Overview

KonKit is a lightweight, ethical, and cost-efficient AI platform that helps small businesses consistently publish marketing content. It focuses on generating images and AI-stitched slideshow videos (with narration) from minimal user input, enabling creators to fill content gaps when human-produced material is not available or affordable.

## II. Product Philosophy & Positioning

KonKit is an AI filler content assistant designed for Indonesian and international small businesses aged 30+, often non-tech-savvy, who need easy, consistent brand visibility.

Core principles: Simplicity, Transparency, Ethics, Accessibility, Affordability, Localization.

## III. Core Features (Phase 1)

1\. Smart Prompt Refiner (GPT-4o / Claude 3 Sonnet)  
2\. Image Generation (Leonardo.ai / Adobe Firefly / SDXL fallback)  
3\. Script & Voice Narration (GPT-4o + ElevenLabs / Play.ht)  
4\. AI Video Slide Generator (FFmpeg / Remotion / MoviePy)  
5\. Scheduler & Randomizer  
6\. Asset Library  
7\. Identity Protection & Moderation  
8\. Dashboard & Transparency

## IV. Technical Flow

User Input (image + description + tone)  
↓  
Prompt Refiner → GPT-4o / Claude 3 (refine & safety checks)  
↓  
Image Prompt → Leonardo.ai / Firefly (generate visuals)  
↓  
Script Prompt → GPT-4o (generate narration + captions)  
↓  
Voice Synthesis → ElevenLabs / Play.ht (generate audio)  
↓  
Video Composer → FFmpeg / Remotion → MP4 (compose final video)  
↓  
Storage → Supabase R2 / Cloudflare R2  
↓  
Output → Download / Schedule / Save to Library

## V. Vendor & Cost Table (Estimates)

| Function | Vendor | Est. Cost |
| --- | --- | --- |
| Prompt Refinement | GPT-4o | \$0.002-\$0.01 per refine |
| Image Gen | Leonardo.ai | \$0.03-\$0.07 per image |
| TTS | ElevenLabs / Play.ht | \$0.015-\$0.03 per minute |
| Video Stitching | FFmpeg / Remotion | Free (CPU cost only) |
| Storage | Cloudflare R2 | ~\$0.005 per GB/month |

## VII. Business Forecast & Financial Calculations

Assumptions:  
\- \$1 ≈ IDR 15,000  
\- Per-generation cost: \$0.063 (GPT + image + storage)  
\- Free user: 5 gens/mo → \$0.315 cost  
\- Premium: 40 gens/mo → \$2.52 cost  
\- Premium price: \$10 → ~\$5 margin/user  
<br/>12-Month Projection (Conservative):

| Month | Paid Users | Revenue (IDR) | OPEX (IDR) | Profit (IDR) |
| --- | --- | --- | --- | --- |
| 1   | 20  | 3M  | 2M  | +1M |
| 3   | 60  | 9M  | 2.5M | +6.5M |
| 6   | 120 | 18M | 3M  | +15M |
| 12  | 400 | 60M | 5M  | +55M |

## VIII. Roadmap (12 Months)

Phase 1 (MVP, 10 weeks): Image + Script + TTS + Stitching + Scheduler + Identity Filter  
Phase 2 (3-6 months): Brand memory, multi-language, template enhancements  
Phase 3 (6+ months): Optional high-quality AI video rendering add-on

## IX. Closing Summary

KonKit balances practicality and aspiration - providing non-tech-savvy SMEs with a low-cost, ethical way to maintain an online presence. Its detailed cost transparency, AI orchestration, and focus on filler content make it a sustainable and trustworthy product for small business creators.