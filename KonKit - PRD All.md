# KonKit PRD - Module 3.1: Onboarding & Setup (Final Combined Production Version)

**Project:** KonKit

**Module:** 3.1 Onboarding & Setup

**Version:** vFinal Combined

**Status:** ✅ Locked for Design & Development

## Feature Overview

The **Onboarding & Setup module** is the user's entry point to KonKit. It captures core identity data (business, service, personal brand, or topic) and prepares the system for immediate AI-based content generation.

**Objective:** Allow non-technical small business owners or creators to complete onboarding in under 3 minutes and receive an instant preview of KonKit's capabilities.

**Philosophy:** "Show value instantly." Each step reduces user cognitive load while progressively building enough input for AI personalization.

## Feature Goals

Goal Description

**Rapid Personalization** Collect enough data (language, visuals, tone) for AI to

start generating useful content immediately.

**Simplicity & Clarity** Only one main action per screen, with tooltips or

Goal Description

inline helpers.

**Localization-First** Default to Bahasa Indonesia for local users with an

English toggle.

##### Instant Value Demonstration Resumable

**Experience**

End onboarding with a live generated sample.

Support pause/resume through persisted onboarding state.

## Scope

In Scope Out of Scope

Language & Authentication Subscription setup (handled in Billing) Upload assets + moderation Advanced brand presets (Phase 2) Tone and audience configuration Team collaboration onboarding Instant sample generation Dashboard metrics population

## User Stories

\# As a… I want to… So that…

- New user choose language I can understand the

process easily

- New user sign up quickly I can access KonKit

instantly

- Business owner upload my photos KonKit can personalize

visuals

- Creator describe my topic or service

The AI can tailor content ideas

- User choose tone and The posts fit my brand

\# As a… I want to… So that… audience style

- User preview a generated sample

I can trust the product's quality

- User resume onboarding I don't lose my progress

## s Functional Requirements

Step 0: Language Selection

- Auto-detect locale (set Bahasa Indonesia if region = ID).
- Manual toggle between EN/ID.
- Persist selection to users.language post-auth.

Step 1: Authentication

- Options: Google, Apple, Email + Password.
- On success → create user record with onboarding_status = in_progress.

Step 2: Upload Visual Assets

- Upload 5-10 images (product, service, or personal visuals).
- File formats: JPG, PNG ≤ 5MB each.
- Moderation via AI: reject flagged content instantly.
- Temporary storage under assets_temp table.

Step 3: Describe Brand/Topic

- Input fields:
  - name (required)
  - category (Product, Service, Personal Brand, Topic)
  - short_description (optional, GPT summary available)
- Save as content_bases_temp entry.

Step 4: Tone & Audience

- Tone (required): Friendly / Expert / Humorous / Inspirational / Serious.
- Audience (required): Buyers / Learners / Followers / Clients.
- Advanced toggle (optional): brand color, hashtags.

Step 5: Preview Sample

- Compose refined prompt → GPT-4o.
- Generate image → Leonardo.ai.
- Generate caption → GPT-4o.
- Return {image_url, caption, cost_estimate}.
- Allow 3 regenerations max.
- On "Finish," commit temp records to permanent tables and mark onboarding as complete.

##   Non-Functional Requirements

Category Requirement

**Performance** Onboarding <3 min total; preview generation <10s average.

**Localization** Bilingual support (EN/ID); JSON-based translation structure.

**Accessibility** Components ≥44px touch area; high contrast mode.

**Moderation** Auto-block on nudity/brand misuse; descriptive pop-up.

**Reliability** Save step progress in DB for resumability.

## User ↔ System Interaction Flow

flowchart TD

subgraph FE\[Frontend\] A1\[Start / Open App\] A2\[Language Selection\] A3\[Authentication\]

A4\[Step 1: Upload Visual Assets\] A5\[Step 2: Describe Brand/Topic\] A6\[Step 3: Tone & Audience\] A7\[Step 4: Preview Sample\] A8\[Finish → Dashboard\]

end

subgraph BE\[Backend\] B1\[Create User Record\] B2\[Upload to Storage\] B3\[Moderation Scan\] B4\[Create Temp Content Base\] B5\[Prompt Composer\]

B6\[AI Generation: GPT + Image\] B7\[Commit to Permanent Tables\]

end

A1 --> A2 --> A3 -->|new user| B1 --> A4

A4 -->|upload images| B2 --> B3 B3 -->|ok| A5

B3 -->|flagged| A4

A5 --> B4 --> A6 --> A7

A7 --> B5 --> B6 --> A7

A7 -->|Accept| B7 --> A8

## Page-by-Page Components & Specifications

Page 2.0 - Language Selection

| Component | Type | Mandatory | Behavior |
| --- | --- | --- | --- |
| LocaleDetector | Auto | No  | Detects locale → sets default language |
| LanguageToggle | Radio | No  | Updates translation in real-time |

Page 3.0 - Authentication

| Component | Type | Mandatory | Behavior |
| --- | --- | --- | --- |
| OAuthButtons | Buttons | No  | Google/Apple sign-in → /api/auth |
| EmailSignUpForm | Form | Yes | Validates & creates user |

Page 4.0 - Upload Visual Assets

Component Type Mandatory Behavior

UploaderGrid File Upload

Yes (≥5) Signed URL upload

\- triggers moderation

ModerationNotice Modal Auto Appears if file flagged

Page 5.0 - Describe Brand/Topic

| Component | Type |     | Mandatory |     | Behavior |
| --- | --- |     | --- |     | --- |
| NameInput | Text |     | Yes |     | Max 50 chars, |
|     | Field |     |     |     | stored in |
|     |     |     |     |     | content_bases_tem p |
| CategorySelect | Dropdow |     | Yes |     | Influences |
|     | n   |     |     |     | template & |
|     |     |     |     |     | generation type |
| ShortDescription | Textarea |     | No  |     | GPT suggestion |
|     |     |     |     |     | available via |
|     |     |     |     |     | /api/gpt/suggest- summary |
| Page 6.0 - Tone & Audience |     |     |     |     |     |
| Component | Type | Mandatory |     | Behavior |     |
| ToneSelector | Radio | Yes |     | Defines style of generated |     |
|     |     |     |     | content |     |

| Component | Type | Mandatory | Behavior |     |
| --- | --- | --- | --- |     |
| AudiencePicker | Dropdown | Yes | Adjusts AI vocabulary & tone |     |
| AdvancedToggle | Accordion | No  | Optional brand details |     |
| Page 7.0 - Preview Sample |     |     |     |     |
| Component | Type | Mandatory |     | Behavior |
| SampleCard | Preview | Yes |     | Displays generated sample (image + caption) |
| RegenerateButton | Button | No  |     | Requests new random template (max 3 times) |
| AcceptButton | Button | Yes |     | Commits<br><br>onboarding data |

## q API Endpoints Summary

Endpoint Method Description

/api/uploads/sign POST Signed URL for uploads

/api/assets/temp POST Registers temp asset

/api/moderation POST Moderates image

/api/content-base/temp POST Creates temp base

/api/content- base/temp/:id/preferen ces

/api/gpt/suggest- summary

PATCH Updates tone/audience

POST Suggests description

/api/generation/sample POST Generates sample post

/api/content- base/commit

/api/user/onboarding- status

POST Finalizes onboarding

GET Retrieves progress

## Data Model (Simplified)

Table Fields Description

users id, email, language, onboarding_status

assets_temp id, user_id, file_path, status

User identity and progress Temporary uploaded assets

content_bases_tem p

id, user_id, type, name, description

Temporary onboarding entry

preferences id, user_id, tone, audience

content_bases id, user_id, type,

name, tone

Default content style settings Permanent content base

## Edge Cases & Recovery

Scenario Expected Result

Upload interrupted Retry automatically twice; resume supported Moderation flag Block progress; explain violation

AI generation timeout Skip preview, still allow completion

<5 uploads Warning modal; allow continue with

missing_visuals = true

Browser closed mid-step Resume from last saved step

## Success Metrics

Metric Target

Onboarding completion rate ≥ 85%

Metric Target

Avg preview time ≤ 90s

First post generation post-setup ≥ 70% users Upload step drop-off < 10%

## Acceptance Criteria

- User completes onboarding in ≤3 minutes.
- Preview sample successfully generated for ≥80% of users.
- All uploads scanned by moderation with proper user feedback.
- Localization consistent across every step.
- Session recovery works after reload.
- 100% responsive on mobile.

## Design & UX Principles

Element Rule

Screen flow 1 main action per page

Copy tone Friendly, encouraging ("You're almost done!") Progress indicator Visible across all steps

Visual guidance Icons & small illustrations per step Layout Single-column vertical for mobile

Color feedback Green = success, Red = moderation error

**Status:** ✅ PRD Locked (Design + Dev Ready)

**Next Module:** Dashboard (3.2) - integrating sample content, transparency metrics, and quick generation actions.

# KonKit PRD - Module 3.2: Dashboard (Home)

**Project:** KonKit

**Module:** 3.2 Dashboard (Home)

**Version:** vFinal Combined

**Status:** ✅ Locked for Design & Development

## 1- Feature Overview

The **Dashboard (Home)** is the central control hub of KonKit, where users manage all activity after onboarding. It showcases the platform's philosophy of simplicity, transparency, and automation by summarizing key insights and offering one-tap actions.

##### Objective

Deliver a lightweight, mobile-first interface that clearly communicates AI activity, usage transparency, and quick entry points to generate or schedule content.

## Feature Goals

Goal Description

**Quick Access** Enable users to reach core actions (Generate, Add

Base, Plan) in ≤ 2 taps.

**Transparency** Show AI usage and cost in real time. **Automation Visibility** Display scheduled and auto-generated posts. **Personalization** Adapt Dashboard widgets to user activity.

Goal Description

**Encouragement** Motivate users with contextual post ideas.

## Scope

In Scope Out of Scope

Dashboard UI & widgets Deep analytics (handled in Transparency

module)

Quick Actions Billing & plan upgrades (handled elsewhere)

Smart Suggestions Complex trend analysis (future)

Mini Calendar Preview Historical stats

AI Usage Display Admin-only monitoring

## User Stories

| #   | As a… | I want to… | So that… |
| --- | --- | --- | --- |
| 1   | User | view AI usage summary | I know my consumption and limits |
| 2   | User | generate content fast | I can stay active easily |
| 3   | User | view upcoming scheduled posts | I can ensure continuous visibility |
| 4   | User | get new post ideas | I never run out of inspiration |
| 5   | User | add new products or<br><br>bases | I can expand my<br><br>content catalog |

s Functional Requirements

### Dashboard Structure

Section Description

**Header Bar** Displays user name, plan badge, and settings icon.

**Quick Actions** \[Generate Post\] \[Add Content Base\] \[Auto Weekly Plan\]

**Smart Suggestion Card** AI-driven post idea based on user activity.

**Weekly Calendar** Mini calendar preview of scheduled posts.

**AI Usage Widget** Displays usage count and cost.

**Active Content Bases List** Shows product/service cards with quick

actions.

### Dynamic Behavior

Condition Behavior

First login post-onboarding Greet with "Welcome! Here's your

first post ready to go."

No bases found Show CTA: "Add your first content base."

Auto-plan active Highlight active days in calendar.

Idle user (7+ days) Suggest "Generate a random post to stay visible."

### Smart Suggestion Engine

Signal Description

**Last Generated Type** Suggests new content type if user repeats

same format.

**Tone** Keeps suggestions consistent with base

Signal Description

tone.

**Days Since Last Post** Re-engages inactive users.

**Engagement History (future)** Adjusts based on effective past content.

### Widgets & APIs

Widget API Function

AI Usage /api/usage/s ummary

Calendar /api/schedul e/preview

Smart Suggestion /api/dashboa rd/suggestio

n

Active Bases /api/content

\-base/list

User Info /api/user/in fo

Returns generation count and cost.

Shows upcoming 7 days of posts. Fetches context-aware idea.

Lists all content bases with next post.

Fetches name, plan, and localization.

##   Non-Functional Requirements

Category Requirement

**Performance** Load time ≤ 2s on 4G.

**Responsiveness** Optimized for mobile-first UI (min width 360px).

**Security** Authenticated API access; JWT-secured endpoints.

**Usability** All widgets scrollable within 2 screens.

**Reliability** Auto-refresh every 10 minutes.

## User ↔ System Interaction Flow

flowchart TD

subgraph FE\[Frontend\] A1\[Open Dashboard\] A2\[Request Usage Summary\]

A3\[Request Schedule Preview\] A4\[Display Content Bases\] A5\[Render Suggestion Card\] A6\[Trigger Quick Actions\]

end

subgraph BE\[Backend\] B1\[Fetch User & Plan Info\] B2\[Fetch usage_records\] B3\[Fetch schedules\] B4\[Fetch content_bases\] B5\[Run Suggestion Engine\]

end

A1 --> B1 --> A2 --> B2 --> A1

A1 --> A3 --> B3 --> A1

A1 --> A4 --> B4 --> A1

A1 --> A5 --> B5 --> A5

A6 -->|Click| Navigate to Module (Generate / Add Base / Scheduler)

## Component Breakdown

| Component | Type | Mandatory | Behavior |
| --- | --- | --- | --- |
| HeaderBar | Fixed | Yes | Shows user info & settings link. |
| QuickActionsRow | Buttons | Yes | Opens core modules. |
| SmartSuggestionCar d | Dynamic Widget | Optional | Displays content idea. |
| CalendarPreview | Mini Calendar | Yes | Shows upcoming posts. |
| AIUsageWidget | Info | Yes | Displays usage and |

| Component | Type | Mandatory | Behavior |
| --- | --- | --- | --- |
|     | Widget |     | cost. |
| ActiveBaseCard | List Compone nt | Yes | Lists content bases with status. |
| EmptyState | Fallback | No  | Appears if no<br><br>content base. |

q Data Model

Table Key Fields Description

users id, name, plan, language User metadata

usage_records id, user_id, type, cost, date AI generation logs

schedules id, user_id, base_id, post_date, status

content_bases id, user_id, name, type,

next_post_date

ai_suggestions id, user_id, text, type,

created_at

Scheduled posts Linked content bases Cached suggestions

## Edge Cases

Scenario Expected Behavior

No content base Show empty state CTA. API timeout Display last cached data.

Suggestion fail Show motivational fallback text. Offline mode Serve cached dashboard snapshot. Large user data Lazy-load lists beyond 20 entries.

## 1-1- Success Metrics

Metric Target

Dashboard load time ≤ 2s CTA click-through (Generate / Plan) ≥ 40% Daily active users ≥ 60%

API failure rate < 2%

## Acceptance Criteria

- All dashboard widgets render successfully within 2 seconds.
- Suggestion Card always returns a valid message or fallback.
- Calendar displays accurate post dates and statuses.
- AI Usage updates dynamically after generation.
- Fully responsive and mobile-optimized.

## Design & UX Guidelines

Element Rule

Layout Single scrollable layout with sticky header Visual Hierarchy Quick actions and usage widget on top

Colors Blue/Green palette consistent with KonKit identity Copy Tone Positive and short ("Keep your brand visible ✨") Animation Subtle fade-in for widgets

Accessibility Text alternatives for icons

**Status:** ✅ PRD Locked (Design + Dev Ready)

**Next Module:** 3.3 - Content Base (Enhanced Contextual Model)

# KonKit PRD - Module 3.3: Content Base (Create & Manage)

**Project:** KonKit

**Module:** 3.3 Content Base (Replaces Product Catalog)

**Version:** vFinal Enhanced Combined

**Status:** ✅ Locked for Design & Development

## 1- Feature Overview

The **Content Base** is the central repository of brand and product intelligence in KonKit. It represents the entity that AI uses to generate consistent, contextually rich, and brand-aligned content.

Each base combines **visual assets**, **brand tone**, **descriptive details**, and **contextual reference inputs** to help KonKit understand what the brand looks like, sounds like, and stands for.

##### Objective

Allow users to define clear visual and narrative context for their product, service, personal brand, or topic - empowering the AI to generate more consistent and realistic content.

## Feature Goals

Goal Description

**Multi-Layer Context** Capture logo, visuals, references, and metadata per

content base.

**Unified Structure** Support multiple entity types (product, service,

Goal Description

personal brand, topic).

**AI Consistency** Improve generation accuracy using structured

contextual data.

**Ease of Setup** Still achievable within 2-3 minutes for basic users.

**Automation Ready** Fully compatible with Scheduler & Random

Generator.

##### Tone & Audience Retention

Each base carries tone, audience, and descriptive cues.

## Scope

In Scope Out of Scope

CRUD for Content Bases Bulk import/export (future)

Asset upload (logo, product, content refs)

Asset tagging by AI (future)

Context-type differentiation Multi-user team access Metadata per base type Engagement analytics (future)

Integration with generation & scheduler

Brand memory persistence (Phase 2)

## User Stories

\# As a… I want to… So that…

- User create a new content base
- User upload different types of assets

I can generate consistent content for my brand

AI understands my brand's visuals and

| #   | As a… | I want to… | So that… |
| --- | --- | --- | --- |
|     |     |     | tone |
| 3   | User | add metadata and description | The AI knows my product/service context |
| 4   | User | set tone and audience | Posts align with my brand style |
| 5   | User | include content references | AI learns my preferred design and style |
| 6   | User | edit or delete bases | I can manage or update brand info<br><br>easily |

## s Functional Requirements

### Content Base Core Fields

Every Content Base stores these universal fields: - **Name** - brand, product, or topic name.

- **Description** - summary of what it is.
- **Type** - Product / Service / Personal Brand / Topic.
- **Tone** - Friendly / Expert / Humorous / Inspirational / Serious.
- **Audience** - Buyers / Learners / Followers / Clients.
- **Advanced Settings (optional):** color palette, hashtags, preferred phrases, watermark.

### Contextual Asset Layers (per Type)

Each type supports additional contextual uploads to provide better AI understanding.

#### Product-TypeBase

Asset Layer Input Purpose

**Logo Reference** 1-2 images Used for color palette and watermark embedding

**Product Images** 5-10 images Core product visuals for AI composition

**Content Reference** 3-5 screenshots

/ URLs

**Metadata Fields** Type, material, color, price range, tagline

For AI to learn post layout and framing

Helps generate accurate copy & tone

#### Service-TypeBase

Asset Layer Input Purpose

**Logo / Identity** 1-2 images Branding consistency

**Environment Photos** 5-10 images Provide realistic context (e.g., salon, clinic)

**Service References** 3-5 URLs / images

**Metadata Fields** Industry, specialization, duration, target clients

Learn testimonial or explainer post style Feeds AI prompt

#### PersonalBrand-TypeBase

Asset Layer Input Purpose

**Portraits** 3-5 images Maintains personal visual identity

Asset Layer Input Purpose

**Logo / Signature Style** 1-2 images Ensures consistent

watermark or symbol

**Content References** 3-5 examples Mimic tone, caption length, and layout style

**Metadata Fields** Niche, preferred topics, catchphrases

Drives tone and post templates

#### Topic/EducationalBase

Asset Layer Input Purpose

**Topic Visuals** 3-5 diagrams or slides

Defines visual style of educational content

**Logo / Channel Brand** 1-2 images Consistent educational

brand identity

**Reference Materials** PDFs, URLs, slides

**Metadata Fields** Field, difficulty, learner level, preferred format

For AI to model teaching structure Context for content

variety

### Tone & Audience Logic

- - Each Content Base overrides global tone and audience defaults.
    - Used in every prompt composition and random generation.
    - Example: Base A (Friendly tone) and Base B (Expert tone) generate distinct captions even under the same template.

### Base Creation Wizard (Dynamic)

- Choose Base Type.
- Upload Logo (optional but recommended).
- Upload Main Visual Assets (required).
- Upload Content References (optional).
- Fill Metadata fields (dynamic per type).
- Add Name, Description, Tone, Audience.
- Preview sample → Save Base.

### AI Prompt Integration

The AI Prompt Composer dynamically selects relevant fields:

{

"name": "Lavender Soap",

"description": "Handmade lavender-infused soap for sensitive skin", "type": "Product",

"tone": "Friendly", "audience": "Buyers",

"metadata": {"color": "Purple", "material": "Natural oil", "price_rang

e": "Affordable"},

"assets": {"logo": \["logo1.png"\], "product_images": \["p1.png"\], "refer ences": \["ref1.png"\]}

}

Prompt Output Example: > "Generate a friendly Instagram post about Lavender Soap using brand logo, one of the uploaded product images, and the composition style from the reference images. Keep tone friendly and focus on daily self-care appeal."

##   Non-Functional Requirements

Category Requirement

**Performance** Base creation <3 minutes including upload. **Scalability** 100 bases per user, 10GB combined assets. **Reliability** Transactional save across assets + metadata. **Security** Assets stored privately; signed URL access only. **Moderation** Auto-scan before save (nudity, logos, faces).

## 7 User ↔ System Interaction Flow

flowchart TD

subgraph FE\[Frontend\] A1\[Add New Base\] A2\[Select Type\] A3\[Upload Logo & Assets\] A4\[Add References\]

A5\[Fill Metadata + Name + Description + Tone + Audience\] A6\[Save Base\]

A7\[View Base on Dashboard\] end

subgraph BE\[Backend\] B1\[Create Temp Base Record\] B2\[Store Assets by Type\] B3\[Run Moderation Scan\]

B4\[Persist Metadata & Preferences\] B5\[Commit Base + Link Assets\]

end

A1 --> A2 --> A3 --> B2 --> B3 --> A4 --> A5 --> B4 --> B5 --> A6 --> A

7

| 8 Component Breakdown |     |     |
| --- | --- |     | --- |
| Page Component | Type | Behavior |
| List BaseCard | Card | Displays logo, base |

Page Component Type Behavior

name, next post date

List AddBaseButton Floating Action

Opens creation wizard

Wizard TypeSelector Dropdown Determines dynamic

input flow

Wizard LogoUploader File Upload For logo reference

Wizard AssetUploader Grid Upload

Wizard ReferenceUploader File / URL

Input

Wizard MetadataForm Dynamic Fields

5-10 visuals, required Optional style

guidance

Adjusts by base type

Wizard DescriptionField Textarea Captures short brand

summary

Wizard ToneSelector Radio Sets tone Wizard AudiencePicker Dropdown Defines target

audience

Wizard SaveButton Action Commits creation

## q API Endpoints Summary

Endpoint Method Description

/api/content- base/create

POST Create new base with metadata and assets

/api/content-base/list GET Fetch user bases

/api/content-base/:id GET View or edit base

/api/content- base/:id/update

PATCH Update fields

Endpoint Method Description

/api/content- base/:id/delete

/api/content- base/:id/assets

DELETE Delete base

GET Retrieve categorized assets

/api/moderation POST Scan all uploaded files

## Data Model

Table Fields Description

content_bases id, user_id, name,

description, type, tone, audience, metadata (JSON)

content_assets id, base_id,

asset_type, url, status

Main entity

Categorized uploads

content_reference s

id, base_id, file_path or url, reference_type

Reference posts or inspiration

preferences base_id, color_palette, hashtags, phrases

Advanced style preferences

## 1-1- Edge Cases

Scenario Expected Result

Missing logo but valid assets Allow creation, mark logo as optional Reference upload flagged Warn user, exclude from AI prompt set Metadata incomplete Save with defaults, prompt user later

Delete base with active schedule

Confirmation modal required

## Success Metrics

Metric Target

Base creation success ≥ 95% Avg setup duration ≤ 180s

Upload failure rate < 3% Moderation false positives < 2%

## Acceptance Criteria

- Base creation and update works for all 4 context types.
- AI generation consistently links to logo, product, and references.
- Tone & audience correctly override defaults.
- Metadata and description enhance prompt reliability.
- Fully responsive and mobile-optimized.

## Design & UX Guidelines

Element Rule

Wizard Layout 5-step linear with

context-based fields

Upload Zone Grouped by category with icon labels

Visual Indicators ✅ Valid upload,

Element Rule

Flagged, ⏳ Pending moderation

Color Code Blue (default), Green

(saved), Red (error)

Copy Tone Friendly instructional

("Upload your product visuals here")

**Status:** ✅ PRD Locked (Enhanced for Contextual Input)

**Next Module:** 3.4 - Content Generation Flow (utilizing this structured context for prompt accuracy).

# KonKit PRD - Module 3.4: Content Generation Flow

**Project:** KonKit

**Module:** 3.4 Content Generation Flow

**Version:** vFinal Enhanced

**Status:** ✅ Locked for Design & Development

## 1- Feature Overview

The **Content Generation Flow** enables users to create AI-generated posts - images, videos, or scripts - either instantly ("Generate Now") or automatically through the **KonKit Scheduler**. The system now also supports **content type optimization** for multiple social platforms (e.g., Instagram Post, Story, TikTok Reels), ensuring outputs match the right format and dimension.

##### Objective

Provide users with fully optimized, brand-consistent, and platform-specific content that can be instantly downloaded or scheduled, leveraging structured base data and contextual AI.

## Feature Goals

Goal Description

**Platform Optimization** Allow users to choose content format for specific

social platforms.

**Unified Experience** Single flow for all types of generation (image, video,

script).

Goal Description

**Automation-Ready** Supports scheduled auto-generation by date and

frequency.

##### Randomization & Variety

**Minimization & Background Execution**

Ensures fresh yet consistent content via tone and template variation.

Users can leave the page while AI continues working.

**Persistent Library** All results stored and accessible by Base, Platform,

and Type.

## Scope

In Scope Out of Scope

Multi-platform content generation

Auto-posting to external apps (Phase 2)

Scheduled and ad-hoc modes Multi-user workflows

Progress tracking & minimize option

Custom branding layers (future)

Randomization logic Fine-tuning models

## User Stories

\# As a… I want to… So that…

- User choose the platform format before generating
- User generate content instantly

My content fits Instagram, TikTok, or other formats easily

I can create and use it right away

| #   | As a… | I want to… | So that… |
| --- | --- | --- | --- |
| 3   | User | schedule auto- generation | My brand stays active without manual work |
| 4   | User | preview my generated result by platform | I can ensure visuals fit properly |
| 5   | User | regenerate multiple times | I can perfect my post using history and variants |
| 6   | User | generate ad-hoc content | I can test ideas not tied to a base |
| 7   | User | see all generated content by platform | I can manage my creative library<br><br>effectively |

## s Functional Requirements

### Generation Modes

Mode Description

**Generate Now** User triggers immediate AI generation.

**Schedule Generation** Define specific future dates/frequency for KonKit

to auto-generate.

**Ad-hoc Generation** Create without a Content Base (manual description).

### Platform Type Selection (New)

Before generation, users choose **content platform** and **format preset.**

| Platform | Supported Formats | Output Dimensions |
| --- | --- | --- |
| **Instagram** | Post, Story, Reel | Post 1080x1080; Story |

| Platform | Supported Formats | Output Dimensions |
| --- | --- | --- |
|     |     | 1080x1920; Reel<br><br>1080x1920 |
| **TikTok** | Short Video | 1080x1920 |
| **Facebook** | Post, Story | Post 1200x1200; Story 1080x1920 |
| **YouTube** | Shorts | 1080x1920 |
| **Custom** | User-defined | Manual width × height<br><br>input |

##### Behavior

- The selected format dictates AI composition (e.g., caption length, aspect ratio).
- For videos, FFmpeg auto-stitches clips into correct orientation/resolution.
- For images, Leonardo prompt includes framing + cropping guidance (e.g., "portrait orientation for TikTok Reel").
- For copy-only generations, caption templates adapt by platform tone (hashtags for Instagram, call-to-action for Facebook, short hook for TikTok).

### Template Selection

Option Description

**Fixed Template** Use defined tone/style (e.g., Friendly Educational).

**Randomized Template** Random tone + structure at generation time.

**Templates:** Promo, Educational, Testimonial, Storytelling, Quote, Fun Fact.

### Generation Flow (Unified Pipeline)

- 1. User selects Base or Ad-hoc mode.
  - Selects Platform (Instagram, TikTok, etc.).
  - Chooses Generate Now or Schedule.
  - Chooses Template Mode (Fixed / Randomized).
  - KonKit composes prompts using base metadata, assets, and format.
  - AI Orchestration: Image → Script → TTS → Video Stitch → Moderation.
  - Preview result → Edit / Regenerate / Save / Schedule.
  - If long-running → minimize, continue in background.

### Scheduling Flow

Step Description

- Select Base / Ad-hoc + Platform
- Choose Frequency (Daily, 3x/Week, Weekly)
- Pick Calendar Range
- Choose Template Type (Fixed/Randomized)
- Confirm → Enqueue jobs into scheduled_generation_jobs

### Randomizer Logic

Weighted logic determines tone/template variety per generation, adapting to platform type.

Signal Weight Description

User's last tone +0.4 Repeat preferred tone

Recently used template

−0.3 Avoid repetition

Platform context +0.3 Match tone to platform (e.g., TikTok → fun; LinkedIn → professional)

Base type +0.2 Adjust template to content relevance

### Library Integration (Multi-Platform)

All outputs stored in the Library, filterable by: - Platform (Instagram, TikTok, etc.) - Type (Image, Video, Script) - Status (Processing, Done, Failed)

**Library Features:** \- Inline video/image preview.

- Platform-specific thumbnail (e.g., Instagram square icon).
- "Ready for \[Platform\]" label.
- Regeneration and variant history retained.
- Option to convert ad-hoc result → Content Base.

### Regeneration & History

- Unlimited regenerations.
- Stored under same task with timestamped variants.
- User can switch platform for regeneration (reframe content).
- Example: Reuse Instagram post as TikTok reel format.

### Dashboard & Progress Widgets

**Widgets:** \- Pending / In Progress / Done tasks (multi-platform).

- Upcoming Scheduled Generations.
- Recently Completed (Library preview).

**Progress UI:** \- 3-stage loader: Composing → Generating → Finalizing.

- Sticky footer with minimize button: "Working in background - check Library later."

##   Non-Functional Requirements

Category Requirement

**Performance** Generation ≤20s for images, ≤35s for videos.

**Scalability** 500 concurrent tasks; multiple platform optimization threads.

**Security** Signed URL per media asset.

**Moderation** Platform-compliant filters (e.g., TikTok sensitive content).

**Storage** Separate storage folders by platform & type.

**Reliability** Auto retry ×2 for generation errors.

## Data Model

Table Fields Description

generation_task id, user_id, base_id,

platform, type, tone, template, status, progress, scheduled_date

Core job tracking

generation_varian t

scheduled_generat ion_jobs

generation_librar y

id, task_id, media_url, caption, platform, created_at

id, user_id, base_id, platform, tone, template_type, frequency, start_date, end_date, next_run

id, task_id, platform, type, status, created_at

Variants by platform Automated job plan

For filtering and listing in Library

## API Endpoints

Endpoint Method Description

/api/generation/create POST Trigger manual or scheduled generation (includes platform param)

/api/generation/status

/:id

/api/generation/regene rate/:id

GET Retrieve task progress

POST Create new variant (platform adaptable)

/api/generation/list GET Fetch Library items

Endpoint Method Description

/api/schedule/create POST Create platform-aware scheduled jobs

/api/generation/:id/st ream

GET Get signed streaming URL

## q Acceptance Criteria

- Users can select platform and format type before generation.
- Generated outputs match target platform aspect ratio and tone.
- Scheduled jobs respect platform context (e.g., vertical for TikTok, square for Instagram).
- Library and Dashboard show platform tags and previews.
- Regeneration allows cross-platform adaptation.
- Background generation and notifications work seamlessly.

**Status:** ✅ PRD Locked (Includes Multi-Platform Content Optimization, Scheduler, and Randomization Enhancements)

**Next Module:** 3.5 - Random Generator (extends orchestration for auto- template variation).

# KonKit PRD - Module 3.5: Random Generator (Enhanced)

**Project:** KonKit

**Module:** 3.5 Random Generator (Automated AI Generation Engine)

**Version:** vFinal Enhanced

**Status:** ✅ Locked for Design & Development

## 1- Feature Overview

The **Random Generator** module enables KonKit to autonomously generate creative, contextually relevant, and platform-optimized content using stored user data (Content Bases, tone, templates, and preferences). It powers both **fully automatic content creation** and **randomized options** during manual generation (ad-hoc or base-bound).

##### Objective

Deliver continuous, brand-consistent yet diverse content for users by providing randomization as both a background automation and an interactive creative option.

## Feature Goals

Goal Description

##### Randomization in All Modes

Allow users to trigger random template/tone generation even in ad-hoc or base-bound content creation.

**Full Automation** Automatically generate random posts via scheduler or

Goal Description

idle triggers.

##### Diversity with Consistency

Ensure creative variation while maintaining brand tone.

**Smart Random Logic** Balance user preferences with fresh, varied

outcomes.

**Seamless Integration** Unified randomization experience across dashboard,

generation flow, and scheduler.

## Scope

In Scope Out of Scope

Randomized generation for ad- hoc, base-bound, and scheduled modes

AI model fine-tuning

Manual Surprise Me trigger Trend-based prompt prediction (Phase 2)

Random tone & template selection

Library and Scheduler integration

Cross-user training Social network posting

## User Stories

\# As a… I want to… So that…

- User click "Surprise Me" to auto-generate a post
- User select "Randomized" tone/template in manual generation

I can get quick content ideas.

I can create diverse content even when generating manually.

| #   | As a… | I want to… | So that… |
| --- | --- | --- | --- |
| 3   | User | have the system generate content automatically on schedule | I can maintain activity effortlessly. |
| 4   | User | trust random content to fit my brand tone and base | My output feels consistent but fresh. |
| 5   | User | view random history and regenerate variants | I can refine random outputs easily. |
| 6   | System | balance diversity and coherence | Generated content avoids repetition but<br><br>stays relevant. |

## 5 Functional Requirements

### Randomization Availability (Enhanced)

Randomized options appear wherever users select tone or template:

Context Location Behavior

**Ad-hoc Generation** Content Generation page

\- Template dropdown

Users can pick "Randomized" to let AI decide tone/template.

##### Base-Bound Generation

Base → Generate Now → Tone dropdown

"Randomized" available as tone option; randomizer applies to structure & style.

**Scheduler Job Setup** Create Plan → Template

behavior

Randomized option automates random creation at each

Context Location Behavior scheduled run.

##### UX Example

Dropdown label:

\> Template: \[Friendly\] \[Professional\] \[Educational\] \[Randomized \]

When selected → Random Generator assigns tone + archetype automatically during generation.

### Generation Triggers

Trigger Description

**Manual "Surprise Me"** Dashboard action to auto-generate a

random post.

##### Randomized Manual Generation

Tone/template set to randomized during ad-hoc or base generation.

**Scheduled Random Generation** Auto-triggered via random-enabled

scheduler jobs.

**Idle Trigger** Optional system-initiated generation during inactivity (future).

### Randomization Layers

| Layer | Description | Examples |
| --- | --- | --- |
| **Tone Layer** | Selects emotional style | Friendly, Expert, Inspirational, Playful |
| **Template Layer** | Chooses content archetype | Promo, Tip, Testimonial, Story, Quote |
| **Platform Layer** | Aligns tone + visual with selected | Instagram Story, |

Layer Description Examples platform TikTok Reel, etc.

### Weighted Randomization Logic

Factor Weight Description

Last used tone −0.3 Reduces repetition

Historical engagement (if available)

+0.4 Prioritize high-performing tones/templates

Base tone alignment +0.3 Keep outputs brand-consistent

Random seed +0.1 Adds controlled unpredictability

### Randomness Control

Mode Description

**Safe** Prioritizes brand tone; subtle randomization.

**Balanced** Mix of familiar and exploratory templates.

**Wild** Maximum creative variety; new tones, formats. Default: **Balanced**

### Integration Points

**Dashboard:** \- Surprise Me button triggers /api/random/generate. - Suggestion Card may auto-display randomized post ideas.

**Generation Flow (3.4):** \- Template dropdown includes Randomized option. - Random generator runs inline, reusing orchestration pipeline.

**Scheduler (3.6):** \- Jobs marked randomized=true use same logic at execution time.

**Library (5.0):** \- Randomized outputs tagged with icon + origin metadata.

### User Controls

Setting Options

Randomness Level Safe / Balanced / Wild

Allow Random Generation On / Off Notification on new random content Push / Email / None

Default Tone Manual / Randomized Accessible via Settings → AI Creativity Preferences.

##   Non-Functional Requirements

Category Requirement

**Performance** Randomized generation ≤25s avg.

**Scalability** 1,000+ concurrent random jobs.

**Reliability** 99% success rate with fallback prompt.

**Security** All random generations are user- isolated.

**UX** Dice animation or subtle shuffle effect for visual feedback.

## Data Model

Table Fields Description

Table Fields Description

random_generation

\_log

id, user_id, base_id, platform, tone, template, random_mode, created_at

Tracks random jobs across all contexts

generation_task Add field

is_randomized

(boolean)

settings random_mode, allow_random, notification_prefs

Marks manual generation with random tone/template

User-level control for randomness

## API Endpoints

Endpoint Method Description

/api/random/generate POST Trigger random content generation manually or via Surprise Me

/api/generation/create POST Accepts is_randomized=true param for inline randomization

/api/random/settings GET / PATCH Retrieve or update user

randomness preferences

/api/random/list GET Get random generation logs

## q Edge Cases

Scenario Expected Result

Randomized tone with no Fallback to system default tone weights.

Scenario Expected Result base data

Random generation produces duplicate

User disables randomization mid-job

Adjust random seed and retry. Respect new setting next run.

## Acceptance Criteria

- Randomized tone/template selection appears in every generation dropdown.
- Random generator integrates with manual, ad-hoc, and scheduled modes.
- Library marks randomized content with tag.
- Randomness level configurable in Settings.
- Balanced creative diversity with consistent brand tone.
- "Surprise Me" and randomized manual generations share same orchestration logic.

**Status:** ✅ PRD Locked (Includes Randomization Across All Modes, Dashboard Trigger, and Scheduler Integration)

**Next Module:** 3.6 - Scheduler (Job Management + Calendar Interface).

# KonKit PRD - Module 3.6: Scheduler (Job Management & Calendar Interface)

**Project:** KonKit

**Module:** 3.6 Scheduler

**Version:** vFinal Combined

**Status:** ✅ Locked for Design & Development

## 1- Feature Overview

The **Scheduler** module manages when and how KonKit generates new content automatically.

It allows users to define generation plans (for specific bases, platforms, and tones), visualize them on a calendar, and track execution status.

Unlike traditional schedulers that plan _publishing_, KonKit's scheduler plans **AI content creation itself** - ensuring users get a continuous stream of auto- generated, on-brand posts without manual effort.

##### Objective

Empower users to "set and forget" - by letting KonKit autonomously create randomized, platform-optimized content on predefined days or frequencies.

## Feature Goals

Goal Description

**Job Management** Create, edit, pause, or delete AI generation schedules.

**Calendar View** Visualize upcoming and completed generation jobs.

**Integration** Connect seamlessly with 3.4 (Generation Flow) and

Goal Description

3.5 (Random Generator).

##### Randomization Support Progress

**Transparency**

Generate unique tone/template per date automatically if randomization is enabled.

Show job status (Pending, Running, Completed, Failed).

**Notifications** Alert users when new content is auto-generated.

## Scope

In Scope Out of Scope

Scheduled generation of content Direct publishing to social media Recurring job management Multi-user team approval

Calendar UI Third-party calendar sync (Phase 2)

Notifications & logs Revenue tracking integration

## User Stories

| #   | As a… | I want to… | So that… |
| --- | --- | --- | --- |
| 1   | User | create a recurring schedule to auto- generate content | I can maintain a consistent posting plan. |
| 2   | User | visualize all upcoming and past generations | I can track what's planned and what's done. |
| 3   | User | pause or edit a generation plan | I can adjust strategy without deleting it. |
| 4   | User | receive notifications<br><br>when a job completes | I know when new<br><br>content is ready. |

\# As a… I want to… So that…

- User randomize tone/template for each job
- System run scheduled jobs reliably

My brand content remains varied and fresh.

KonKit maintains user trust and uptime.

## s Functional Requirements

### Job Creation Flow

- 1. User opens **Scheduler → Create Plan**
  - Selects:
    - Base or Ad-hoc Mode
    - Platform (Instagram, TikTok, etc.)
    - Content Type (Image, Video, Script)
    - Tone/Template (Fixed or Randomized)
  - Defines:
    - Frequency (Daily / 3× per week / Weekly / Custom days)
    - Start & End Dates
    - Time of Day (default: 9AM user local)
  - Clicks **"Confirm Plan"**
  - System creates scheduled_generation_job records for each date.

### Calendar Interface

View Description

**Month View** Shows all generation days with color-coded status.

**Week View** Highlights upcoming 7 days, scrollable.

**List View** Detailed job list with filtering (Base, Platform, Status).

**Color Code:** \- Completed

- In Progress
- Scheduled
- Failed

Clicking any date opens a job detail sidebar: - Base name

- Tone/Template (or Randomized)
- Type (Image/Video/Script)
- Platform
- Status
- \[Open in Library\] or \[Retry\]

### Job Status Lifecycle

| Status | Trigger | Action |
| --- | --- | --- |
| **Scheduled** | Created | Awaiting execution |
| **In Progress** | Worker triggered | Generation pipeline running |
| **Completed** | Job done | Library entry created |
| **Failed** | Error encountered | Retry or mark failed |
| **Paused** | User action | Excluded from next cycle |

- **Recurring Job Logic**

Frequency Description

**Daily** Run once per day

**3× per week** Automatically spaced (e.g., Mon-Wed-Fri)

**Weekly** Choose specific weekday

**Custom** User selects multiple specific days

**One-time** Single scheduled job

After each successful run: - System updates next_run_date. - If end date reached → mark plan as **Completed**.

### Integration with Random Generator

If **randomized = true**: - Tone, template, and sometimes platform (if user allowed multi-platform) are randomized each time. - The Scheduler passes this flag to /api/generation/create → pipeline auto-selects template + tone.

### Dashboard Integration

Add "Upcoming Schedule" Widget:

- Dashboard Widget

├── Next Scheduled Generation: \[EcoMag • Instagram Post • Tomorrow 9A

M\]

├── Auto Generation History (last 5)

└── Button: \[View Calendar\]

### Notifications & Alerts

| Event | Notification Type | Message Example |
| --- | --- | --- |
| Job Started | In-app toast | "Your EcoMag content is |

Event Notification Type Message Example being generated…"

Job Completed

Push / Email " New content for EcoMag is ready!"

Job Failed In-app + Email " Auto-generation for EcoMag failed - retry now."

Notifications configurable in Settings.

## Non-Functional Requirements

Category Requirement

**Performance** Handle up to 5k concurrent scheduled jobs/day.

**Reliability** 99% job execution success rate.

**Recovery** Retry failed jobs 2× automatically.

**Security** Jobs linked strictly to user_id; no shared access.

**Time Zone Awareness** Schedule based on user's local time (auto-detected).

**Scalability** Jobs processed in distributed worker queue (Redis/BullMQ).

## Data Model

Table Fields Description

scheduled_generat ion_jobs

id, user_id, base_id, platform, tone,

Core schedule data

Table Fields Description template,

randomized, frequency, start_date, end_date, next_run, time_of_day, status

job_executions id, job_id, run_date,

status, result_id, error_log

user_settings timezone, notifications_enabled, default_schedule_tim e

Logs every run of a schedule For user preferences

## API Endpoints

Endpoint Method Description

/api/schedule/create POST Create a new scheduled generation plan

/api/schedule/list GET Get all user schedules

/api/schedule/:id GET Get schedule detail

/api/schedule/:id/upda te

/api/schedule/:id/paus e

/api/schedule/:id/dele te

/api/schedule/:id/exec utions

PATCH Edit tone, frequency, or end date

PATCH Pause a plan

DELETE Delete schedule plan

GET Fetch execution history

/api/schedule/next GET Return next scheduled generation summary (for

Endpoint Method Description Dashboard)

## q Edge Cases

Scenario Expected Result

Missed job (server downtime) Requeue within 24-hour grace period Deleted base with active schedule Auto-cancel related jobs

User changes timezone All times auto-adjust to new zone Duplicate runs triggered Lock job via job_id + date hash Random generation fails Retry with fallback tone/template

## Success Metrics

Metric Target

Job success rate ≥ 99%

Missed job rate ≤ 0.5%

User retention (weekly active auto jobs) ≥ 70% Auto-generated post approval rate ≥ 85%

## 1-1- Acceptance Criteria

- Users can create, edit, and delete schedules.
- Calendar correctly reflects all job statuses.
- Randomized mode functions seamlessly with scheduler jobs.
- Notifications fire correctly for start/completion events.
- All scheduled outputs appear in Library automatically.
- Timezone alignment validated.
- Scheduler API stable under load (>5k daily jobs).

## 1-2 Design & UX Guidelines

Element Rule

**Calendar UI** Simplified mini-calendar,

color-coded by status

**Schedule Card** Shows Base, Platform, Frequency, Next Run

**Microcopy** "Let KonKit handle

tomorrow's post for you "

**Empty State** "No plans yet - schedule your first AI generation below."

**Animation** Subtle checkmark

animation on job completion

**Accessibility** Keyboard navigation for

date picker & calendar

**Status:** ✅ PRD Locked (Integrates Random Generator, Content Generation Flow, and Dashboard Widgets)

**Next Module:** 3.7 - Library (Content Management & Post-Generation Operations).

# KonKit PRD - Module 3.7: Library (Content Management & Post-Generation Operations)

**Project:** KonKit

**Module:** 3.7 Library

**Version:** vFinal Combined

**Status:** ✅ Locked for Design & Development

## 1- Feature Overview

The **Library** is the centralized hub for managing all generated content in KonKit.

It stores every AI-generated output (manual, random, or scheduled) and provides users with tools to **preview**, **edit**, **regenerate**, **download**, **reschedule**, and **analyze** performance (in future phases).

The Library acts as both a **repository** and a **launchpad** - allowing users to reuse existing content, view their creative history, and maintain brand consistency.

##### Objective

Create a clean, searchable, platform-aware content management system that connects the generation, scheduling, and base modules into a cohesive post- creation experience.

## Feature Goals

Goal Description

Goal Description

**Centralized Storage** Store all generated content, regardless of source

(manual, random, scheduled).

##### Multi-Platform Awareness

Organize outputs by platform type (Instagram, TikTok, YouTube, etc.).

**Version History** Track regenerations and keep variant history.

##### Post-Generation Actions

**Cross-Module Integration**

**Lightweight Analytics (Phase 2)**

Allow users to edit captions, download, or schedule reuse.

Connect with Scheduler, Content Base, and Dashboard widgets.

Track user interaction with saved outputs (views, downloads, reuse rate).

## Scope

In Scope Out of Scope

Storage, preview, and management of all generated content

Filtering and searching by attributes

Inline preview (image/video/script) Regeneration & rescheduling

actions

Direct posting to social media (Phase 2)

Cross-account collaboration AI insight scoring (future) Advanced analytics (future)

## User Stories

\# As a… I want to… So that…

| #   | As a… | I want to… | So that… |
| --- | --- | --- | --- |
| 1   | User | view all my generated content in one place | I can review, manage, and reuse posts. |
| 2   | User | filter my content by platform, type, or status | I can find specific outputs easily. |
| 3   | User | preview media inline | I can quickly review AI outputs. |
| 4   | User | edit captions or regenerate | I can fine-tune posts before reuse. |
| 5   | User | view generation history for a post | I can see what variations were made. |
| 6   | User | download optimized files | I can post them directly to social media. |
| 7   | User | reschedule existing content | I can let KonKit auto- create future variations. |
| 8   | System | auto-tag and store all generated content | I can keep data structured and<br><br>searchable. |

## s Functional Requirements

### Library Structure

Section Description

**Filters Toolbar** Filter by Base, Platform, Type (Image, Video, Script), Status, or Origin (Manual, Random, Scheduled).

**Content Grid** Card-based layout displaying thumbnail, platform icon, date, and action menu.

Section Description

**Search Bar** Keyword search across names, captions, and base references.

**Tags** Visual cues for origin ( Random, Scheduled, Manual).

### Content Card (Primary Unit)

Component Description

Thumbnail Image or video preview (auto aspect ratio based on platform).

Title Generated title or caption snippet.

Metadata Base name, platform, status, generation date.

Quick Actions \[View\] \[Download\] \[Regenerate\] \[Reschedule\] \[Delete\].

Status Indicator Processing / Ready / Failed.

### Content Detail Page

When user clicks **\[View\]**, opens full content detail view.

Section Description

Media Preview Inline display of image or video (HTML5 player).

Caption Editor Editable field for text-based outputs.

Variant History Carousel of regenerations with timestamps.

Metadata Panel Base name, tone, template, platform, cost, date.

Section Description

Actions \[Regenerate Variant\], \[Download\], \[Schedule Reuse\], \[Convert to Base\], \[Delete\].

### Variant Management

Each generation can have multiple versions (from re-generation or platform adaptation).

Feature Description

History Carousel Scroll through variant thumbnails.

Restore Button Replace active version with older variant.

Notes Field Optional user note on why a variant was kept.

Cross-Platform Reuse Convert existing output to new platform

dimension (reframe for Reels/Stories).

### Rescheduling (Integration with Scheduler)

Action Result

"Schedule Reuse" Opens Scheduler modal pre-filled with selected content and platform.

"Auto-Recreate Weekly" Creates repeating job using same

context.

"Smart Rotate" (future) Rotates top-performing posts in

intervals.

### Regeneration

Context Behavior

From Library Opens Generation Flow (3.4) pre-filled with existing content details.

With Randomized On AI regenerates using same tone but new template or structure.

Result Saved as new variant within same content group.

### File Handling & Export

Type Format Behavior

**Image** PNG / JPG (1080x1080 or platform preset)

**Video** MP4 (1080x1920, with audio if TTS used)

Ready for social upload. Inline playback & download.

**Script / Caption** Plaintext (.txt) Downloadable & copyable.

##   Non-Functional Requirements

Category Requirement

**Performance** Load ≤ 2s for up to 100 cards.

**Scalability** Store 10GB/user with efficient query caching.

**Reliability** Auto-save on every edit.

**Security** Signed URL for file access.

**Resilience** Auto-recover from broken thumbnail links.

**Usability** Mobile-first responsive grid.

## Data Model

Table Fields Description

generation_librar y

generation_varian t

id, user_id, base_id, platform, type, title, caption, status, origin, created_at, updated_at

id, library_id, media_url, caption, tone, template, platform, created_at

Core storage table

Version tracking

library_tags id, library_id, tag_type Tag relations for filters

library_notes id, variant_id, note_text

User notes per variant

## API Endpoints

Endpoint Method Description

/api/library/list GET Get all generated content with filters.

/api/library/:id GET Fetch specific content detail.

/api/library/:id/updat e

/api/library/:id/regen erate

/api/library/:id/delet e

/api/library/:id/sched ule

/api/library/:id/downl oad

PATCH Edit caption, tone, or title.

POST Create new variant.

DELETE Delete content & all variants.

POST Schedule reuse via Scheduler module.

GET Generate signed URL for file export.

## q Edge Cases

Scenario Expected Result

Missing thumbnail Auto-regenerate thumbnail or placeholder shown. Deleted base Content retained but tagged "Base Deleted." Large library Pagination + lazy loading applied.

User offline Cached local copy shown (read-only). Variant limit exceeded Older variants archived (retain last 10).

## Success Metrics

Metric Target

Average page load time ≤ 2 seconds

Content reuse rate ≥ 50% of generated posts reused Regeneration success rate ≥ 95%

User satisfaction (UX rating) ≥ 8/10

Variant retention rate ≥ 90% across sessions

## 1-1- Acceptance Criteria

- Library lists all generated content correctly (Manual / Random / Scheduled).
- Content cards display correct platform and tone icons.
- Detail view supports inline playback and caption editing.
- Regeneration adds new variants without overwriting old versions.
- Schedule Reuse opens Scheduler modal correctly.
- All files downloadable in correct format & resolution.
- Responsive across mobile and desktop.

## 1-2 Design & UX Guidelines

Element Rule

**Layout** Grid layout with sticky filter

bar.

**Preview Modal** Opens inline for media viewing.

**Icons** Platform-specific

(Instagram , TikTok , YouTube ►).

**Tags** Random, Scheduled,

Manual

**Color Palette** Consistent with brand

(Green = Ready, Blue = Processing, Red = Failed)

**Microcopy** "Here's what your AI has

crafted for you ✨"

**Accessibility** Captions and alt text

available for all media.

**Status:** ✅ PRD Locked (Integrates Generation, Scheduler, and Randomization Ecosystem)

**Next Step:** Begin drafting the **KonKit Phase 1 MVP Blueprint** for development sequencing and resource planning.

# KonKit PRD - Module 3.10: Transparency & Usage (AI Activity & Billing Overview)

**Project:** KonKit

**Module:** 3.10 Transparency & Usage

**Version:** vFinal

**Status:** ✅ Locked for Design & Development

## Feature Overview

The **Transparency & Usage** module visualizes how users' AI credits and system resources are consumed over time.

It combines **activity tracking**, **AI cost transparency**, and **usage analytics** into a single dashboard - helping users understand the value they're getting from KonKit.

##### Objective

Build user trust through visibility. Allow creators and businesses to see where credits go, monitor their generation frequency, and plan future usage efficiently.

## Feature Goals

Goal Description

**Clear Usage Insights** Show users how credits are spent by category and

time.

**Transparency in Cost** Display estimated equivalent cost per generation and

Goal Description

monthly total.

**Forecasting** Predict remaining credits vs. planned scheduler

activity.

**Behavioral Insights** Identify content type and tone usage trends.

**Encourage Upgrades** Provide upgrade nudges based on user activity.

## Scope

In Scope Out of Scope

Usage dashboard & metrics visualization

Actual payment gateway management

Forecasts & analytics Team-level cost splitting

Plan comparison & upgrade CTA Admin billing reporting (handled

separately)

## User Stories

| #   | As a… | I want to… | So that… |
| --- | --- | --- | --- |
| 1   | User | see how many credits I've used | I can manage my remaining balance wisely |
| 2   | User | understand where credits were spent | I can analyze my AI usage and focus on what works |
| 3   | User | view estimated costs and savings | I can see the ROI of my subscription |
| 4   | User | get alerts when nearing<br><br>credit limits | I can top-up or<br><br>upgrade in time |

\# As a… I want to… So that…

- User forecast next week's usage
- System collect usage data securely

I can plan my content generation accordingly

KonKit maintains transparency without exposing private info

## s Functional Requirements

### Dashboard Structure

Section Description

**Summary Cards** Quick overview of total credits, used %, remaining, and renewal date.

**Credit Breakdown** Donut chart by generation type (Image, Video, Script, Randomized, Scheduled).

**Usage Timeline** Line chart showing daily credit usage for the last 30 days.

**Scheduler Forecast** Predict upcoming credit usage from scheduled jobs.

**Cost Equivalent** Approximate AI infra cost vs. subscription spend.

**Upgrade Suggestion** Dynamic CTA: "You've used 80% of your

plan - upgrade for better value."

### Credit Breakdown Example

| Generation Type | Description | Metric |
| --- | --- | --- |
| Image + Caption | Standard post creation | 45% usage |
| Video | AI stitched video + TTS | 30% usage |

| Generation Type | Description | Metric |
| --- | --- | --- |
| Script | Caption or text-only generation | 10% usage |
| Random Generator | Auto or surprise generations | 10% usage |
| Scheduler | Auto jobs | 5% usage |

- **Detailed Logs (Advanced Tab)**

Field Description

Date When the generation occurred Task Type Image / Video / Script

Base / Context Linked base name

Credits Used Number of credits consumed Duration Time taken for generation

Platform (IG, TikTok, etc.)

Outcome Success / Failed / Moderated Cost Equivalent API + infra cost estimate

Users can export logs to CSV or view summarized charts.

### Forecast Logic

Forecast =

remaining_credits - (avg_daily_usage \* days_until_renewal) - scheduled_jobs_estimated_cost

If negative → highlight "Running Out Soon" banner.

### Upgrade & Top-Up Integration

- - Links to Pricing & Monetization (3.9).
    - Real-time sync with credit_wallet table.
    - Upgrade CTA shown dynamically at 70% usage threshold.
    - API pulls subscription_plans for cross-comparison.

##   Non-Functional Requirements

Category Requirement

**Performance** Dashboard loads <2s for 30-day data range.

**Security** All data fetched per authenticated user only.

**Data Retention** Keep logs 6 months; purge older for performance.

**Scalability** Handle 1M+ records efficiently using indexed queries.

**Accuracy** ±2% tolerance between displayed and actual credit count.

## Data Model

Table Fields Description

usage_records id, user_id, task_id,

type, credits_used, created_at,

Core usage tracking

Table Fields Description cost_estimate

credit_wallets current_balance,

last_updated

Real-time credit balance

credit_transactio ns

id, user_id, amount, source, reference_id

For accurate reconciliation

scheduler_jobs job_id, user_id,

next_run, estimated_cost

For forecast data

user_subscription s

plan_id, renewal_date For renewal and plan display

## API Endpoints

Endpoint Method Description

/api/usage/summary GET Returns total used, remaining, and breakdown by type

/api/usage/timeline GET Returns daily usage data (30 days)

/api/usage/logs GET Returns paginated activity logs

/api/usage/forecast GET Returns estimated consumption until renewal

/api/usage/cost GET Returns cost estimate vs. plan

/api/usage/export GET Exports logs as CSV

## q Edge Cases

Scenario Expected Result

No usage data Show empty state "No generations yet." Missing scheduler data Forecast excludes schedule jobs

Data desync (wallet mismatch)

Trigger sync and recalc from credit_transactions

Free plan user Hide cost comparison; show upgrade CTA High-usage spike Alert user via in-app and email notification

## Success Metrics

Metric Target

Usage dashboard load <2s

Forecast accuracy ≥90%

User trust rating (survey) ≥8/10 Upgrade conversion from dashboard ≥10%

Support tickets on billing confusion <2% total users

## Acceptance Criteria

- Dashboard correctly displays total and breakdown values.
- Logs match actual credit transactions.
- Forecast calculations accurate within margin.
- Cost estimate visible and intuitive.
- Upgrade flow functional and responsive.
- No personal data leak (only aggregated usage visible).

## Design & UX Guidelines

Element Rule

**Layout** Two-tab layout (Overview

/ Detailed Logs).

**Charts** Donut (type breakdown),

Line (timeline), Bar (forecast).

**Microcopy** "Here's how your creativity

credits have been used ✨."

**Color Code** Blue = Image, Purple =

Video, Yellow = Script, Gray

\= Randomized.

**Empty State** "No activity yet - start

creating your first content!"

**Upgrade CTA** Persistent floating button once usage >70%.

**Status:** ✅ PRD Locked (Integrated with Pricing, Scheduler & Library modules)\*\*

**Next Step:** Implement as part of Phase 1.5 (post-MVP analytics dashboard rollout).

# KonKit PRD - Module 3.9: Settings & Account Management (Final)

**Project:** KonKit

**Module:** 3.9 Settings & Account Management

**Version:** vFinal Combined

**Status:** ✅ Locked for Design & Development

## 1- Feature Overview

The **Settings & Account Management** module provides users control over their account information, AI generation preferences, language, tone defaults, notifications, and visual output options.

It centralizes personalization, security, and global system behavior in a single, intuitive interface.

##### Objective

Ensure that every generation, automation, and output in KonKit behaves consistently with user preferences - while allowing flexibility to override them during each operation.

## Feature Goals

Goal Description

##### Unified Preferences Hub

Provide one place to manage all AI, account, and visual settings.

**Pre-filled AI Defaults** Allow user preferences to auto-populate generation

and scheduler fields by default.

Goal Description

##### User Control & Flexibility

**Security & Localization**

**System-wide Integration**

Enable overrides anytime without permanent changes.

Handle authentication, sessions, and language toggles.

Ensure all modules respect user preferences.

## Scope

In Scope Out of Scope

Account management, AI preferences, language, watermark, and notifications

Security (sessions, password, data export)

Billing & payment gateway setup

Team role management (future)

Localization (EN/ID) Social platform linking (Phase 2)

## User Stories

| #   | As a… | I want to… | So that… |
| --- | --- | --- | --- |
| 1   | User | update my account info | My profile remains accurate. |
| 2   | User | set my preferred language | I can use KonKit comfortably. |
| 3   | User | define default AI behavior | My generations start prefilled with my preferences. |
| 4   | User | control AI creativity | I can balance |

| #   | As a… | I want to… | So that… |
| --- | --- | --- | --- |
|     |     |     | consistency vs exploration. |
| 5   | User | toggle watermark & logo | My visuals align with my branding. |
| 6   | User | manage notifications | I only get alerts I care about. |
| 7   | System | apply defaults globally | All content respects<br><br>user preferences. |

s Functional Requirements

### Settings Categories

Category Description

**Account Info** Manage personal info, email, password, plan, and sessions.

**Language & Localization** Toggle between English and Bahasa

Indonesia instantly.

**AI Preferences** Define tone, creativity, language, audience, and platform defaults.

**Visual Output Settings** Control watermark, brand logo, color

palette, and default formats.

**Notifications** Manage in-app, email, and push notification preferences.

**Security & Data** Handle sessions, password resets, and data export.

### AI Preference Settings (Clarified Function)

These act as **pre-filled default values**, not permanent rules.

Field Description Default

**Default Tone** Initial tone preselected in

generation flow.

**Default Audience** Pre-filled target audience

(buyers, learners, etc.).

**Creativity Mode** Governs AI's variability (Safe /

Balanced / Wild).

Friendly Buyers Balanced

##### Language for Captions

Determines output language for captions or scripts.

Follows UI language

**Default Platform** Used to auto-fill first platform

choice.

Instagram

##### Randomization Default

Controls whether 'Randomized' ON tone/template is auto-enabled.

**Functional Clarification:** \- These defaults **auto-populate generation & scheduler forms** (Modules 3.4-3.6).

- **Users can override** during generation; overrides apply to that session only.
- **Global defaults persist** unless explicitly updated here.
- Preference hierarchy applied system-wide: 1. Manual User Input (highest priority)

- Content Base Defaults (if defined)
- User Preferences (from Settings)
- System Defaults (fallback)

### Visual Output Customization

Option Description

Option Description

**Watermark Toggle** Enable or disable watermark overlay. **Brand Logo Upload** Used for watermark or corner overlay. **Color Palette** Influences image prompt style.

**Default Aspect Ratio** Set preferred generation ratio (1:1, 9:16,

16:9).

**Font Style (future)** For text-overlay visuals.

### Notification Settings

Event Type Default

Job Started In-App ON Job Completed Push/Email ON Job Failed Email ON AI Tips & Insights In-App OFF

### Account & Security Management

Function Description

**Edit Profile** Update personal info.

**Change Password** Secure form with confirmation.

**Manage Sessions** View and revoke active logins.

##### Two-Factor Authentication (future)

Optional added security.

**Data Export** Download data archive per GDPR compliance.

### Integration Summary

Module Integration

- 1. **Content Generation** Reads tone, audience, and creativity

defaults.

- 1. **Random Generator** Applies creativity level and tone weight bias.
  - **Scheduler** Reads timezone and default notification settings.
  - **Library** Applies watermark and logo preferences.

##   Non-Functional Requirements

Category Requirement

**Performance** Instant save and reflect (<500ms).

**Reliability** Persistent storage via user_id mapping. **Security** Auth required for all writes; encrypted fields. **Localization** Full EN/ID coverage.

**Accessibility** WCAG 2.1 AA compliance.

## Data Model

Table Fields Description

users id, name, email, language, plan_id, created_at

user_preferences id, user_id, tone,

audience, creativity, language, watermark_enabled, logo_url,

Core user identity.

AI & output defaults.

Table Fields Description color_palette,

default_platform

notification_pref erences

id, user_id, job_started, job_completed, job_failed, tips_enabled

User alert control.

user_sessions id, user_id, device, ip,

last_active, revoked

Login sessions.

## API Endpoints

Endpoint Method Description

/api/user/info GET Fetch user details.

/api/user/update PATCH Update account fields.

/api/preferences/get GET Retrieve AI & visual preferences.

/api/preferences/updat e

PATCH Save new defaults.

/api/notifications/get GET Fetch current notification settings.

/api/notifications/upd ate

PATCH Update user notification settings.

/api/user/sessions GET List sessions.

/api/user/sessions/rev oke

POST Revoke specific session.

/api/user/export POST Request data export archive.

## Acceptance Criteria

- - Defaults prefill generation and scheduler forms correctly.
    - Overridden values do not change global preferences.
    - All updates save instantly and persist.
    - Logo/watermark visible in generated outputs when enabled.
    - Language toggling refreshes UI instantly.
    - Notifications respect preferences and delivery channels.
    - All API endpoints authenticated and reliable.

**Status:** ✅ PRD Locked (Final Version with Default Behavior Clarified)

##### Next Step: Compile the KonKit Phase 1 MVP Build Plan & Tech Spec (ERD + API Integration Map)

KonKit PRD - Module 3.10: Pricing & Monetization (Final)

**Project:** KonKit

**Module:** 3.10 Pricing & Monetization **Version:** vFinal Simplified Credit Model **Status:** ✅ Locked for Design & Development

## Feature Overview

The **Pricing & Monetization** module defines how users pay for and consume AI-powered content generation in KonKit.

It follows a **simple credit-based model** with equal feature access across all tiers - only the amount and value of credits differ.

##### Objective

Provide a transparent, fair, and scalable pricing system that aligns with KonKit's low-friction philosophy while ensuring positive margins over API and infrastructure costs.

## Feature Goals

Goal Description

##### Equal Features for All Tiers

No locked features - all users access the same generation tools.

**Credit-Based Usage** Each generation consumes credits depending on

complexity.

**Simple Pricing Tiers** Tier differences based only on credit allocation and

Goal Description

price efficiency.

##### Transparent Cost Feedback Sustainable Profit

**Margin**

Users always see remaining credits and estimated cost before generation.

Maintain positive margin (~50-60%) after API and infra expenses.

## Scope

In Scope Out of Scope

Credit wallet, subscription management, top-ups

Tax & invoice generation (billing provider handles)

Usage tracking & pre-checks Multi-currency (future) Credit-based generation limits API key billing integration

## Pricing Model Overview

KonKit uses a **credit-based subscription** system: - Each generation (image, video, caption, etc.) consumes credits.

- Every plan provides monthly credits that reset on renewal.
- Additional credits can be purchased anytime via top-ups.

## s Functional Requirements

### Subscription Tiers (All-Feature Access)

| Tier | Price (USD) | Monthly Credits | Effective Value/Credit | Notes |
| --- | --- | --- | --- | --- |
| **Free** | \$0 | 20  | -   | For onbo |

| Tier | Price (USD) | Monthly Credits | Effective Value/Credit | Notes<br><br>ardin |
| --- | --- | --- | --- | --- |
|     |     |     |     | g, water mark appli ed |
| **Star ter** | \$9 | 120 | \$0.075 | Entry tier for small creat ors |
| **Gro wth** | \$19 | 300 | \$0.063 | Best value for<br><br>activ |
|     |     |     |     | e users |
| **Pro** | \$39 | 700 | \$0.056 | For agen cies or heav y creat<br><br>ors |

##### All tiers include

✅ Unlimited generations until credit limit

✅ Access to all AI tools (Image, Script, Video)

✅ Scheduler, Randomizer, Library, Settings

✅ Watermark removal for paid tiers

### Credit Consumption Rules

Generation Type Credit Cost Notes

Image + Caption 1 Standard Instagram/F acebook- style post

Script Only 0.5 Caption or script text generation

Short Video 2 Includes visuals + voice-over (up to 20s)

Regeneration 50% of original Encourages refinement

Random/Auto Generation Same as equivalent

manual type

Consistent pricing logic

##### Behavior

- Credits deducted **only after successful generation**.
- Scheduled & random jobs pre-check available balance before execution.
- Failed or aborted jobs do not consume credits.

### Top-Up Packs

Pack Price Credits Validity

| Pack | Price | Credits | Validity |
| --- | --- | --- | --- |
| Small | \$5 | 70  | 90 days |
| Medium | \$10 | 150 | 180 days |

Top-ups extend usage beyond the subscription allocation. Credits apply instantly upon purchase.

### Usage Tracking & Dashboard Integration

Metric Description

**Total Credits Used** Lifetime generation count.

**Remaining Credits** Real-time balance.

**Credit Breakdown** Shows usage per content type.

**Upcoming Forecast** Estimated scheduler consumption next 7 days.

Displayed in: - **Dashboard (3.2):** "Usage Widget" - **Transparency (3.8):**

Detailed usage and billing overview

### Credit Management Logic

Behavior Description

**Low Balance Warning** Triggered when balance < 20%.

**Auto Pause** Scheduler jobs pause when balance = 0.

**Upgrade Prompt** Friendly nudge to upgrade or top-up.

**Auto Top-Up (optional)** Re-purchase last top-up pack

automatically.

### Upgrade / Downgrade Flow

Action Description

Action Description

**Upgrade Plan** Immediate activation; prorated credit adjustment.

**Downgrade Plan** Applies next billing cycle; existing credits retained.

**Cancel Plan** Converts account to Free tier with watermark enabled.

### System Integrations

Module Integration Detail

**Dashboard (3.2)** Usage widget, credit status color indicators.

**Generation Flow (3.4)** Pre-check credit before generation.

**Scheduler (3.6)** Deducts credits on execution day. **Random Generator (3.5)** Auto-checks before random job runs. **Settings (3.9)** Toggle for auto top-up and credit alerts. **Transparency (3.8)** Aggregates all usage + cost logs.

##   Non-Functional Requirements

Category Requirement

**Performance** Credit check latency <100ms.

**Reliability** All credit transactions atomic and idempotent.

**Scalability** Handle 1M+ generation records/month. **Security** Billing via secure gateway (Stripe/Paddle). **Transparency** Real-time sync between wallet & dashboard.

## Data Model

Table Fields Description

subscription_plan s

user_subscription s

id, name, price, credits_included, credit_value

id, user_id, plan_id, start_date, end_date, auto_renew

Plan definitions Active plan record

credit_wallets id, user_id,

current_balance, total_used

Credit balances

credit_transactio ns

id, user_id, amount, type (debit/credit), reference_id, source, created_at

Ledger transactions

topup_purchases id, user_id, credits,

price, expiry_date

Top-up purchase log

## API Endpoints

Endpoint Method Description

/api/plans GET Retrieve all subscription plans.

/api/subscribe POST Activate or change plan.

/api/credits/balance GET Return user credit balance.

/api/credits/deduct POST Deduct credits for generation.

/api/credits/history GET Fetch usage transactions.

/api/topup POST Purchase a credit top-up pack.

/api/credits/forecast GET Show upcoming consumption estimate.

## q Edge Cases

Scenario Expected Result

Credit deduction fails mid- process

Rollback and retry transaction.

Insufficient credits before job Prompt upgrade/top-up. Downgrade after high-usage Retain credits, adjust plan next cycle.

Free plan user hits video generation

Block and display upgrade suggestion.

## Success Metrics

Metric Target

Credit accuracy 100% transactional consistency Subscription renewal rate ≥ 80% monthly

Free → Paid conversion ≥ 12%

Avg. credit usage/month ≥ 70% of allowance

Gross margin ≥ 50% after infra/API costs

## Acceptance Criteria

- All plans share same features, differing only in credit allocation.
- Credit deductions atomic and accurate.
- Dashboard reflects real-time usage updates.
- Top-ups apply instantly.
- Scheduler pauses gracefully when credits depleted.
- Upgrade/downgrade flow transitions smoothly.

## Design & UX Guidelines

Element Rule

**Plan Cards** Emphasize simplicity: price,

credits, per-credit value.

**Usage Widget** Green = healthy, Yellow = low, Red = 0.

**Microcopy** "Create more, worry less -

your credits power your imagination ⚡ ."

**Top-Up Modal** Quick checkout with clear credit-to-price ratio.

**Upgrade Prompt** Friendly call-to-action: "Running low? Add more creativity fuel ."

**Status:** ✅ PRD Locked (Feature-Equal, Credit-Based Pricing Model Integrated Across Modules)\*\*

##### Next Step: Include in Phase 1 MVP Blueprint & Technical Specification (ERD + API Map)