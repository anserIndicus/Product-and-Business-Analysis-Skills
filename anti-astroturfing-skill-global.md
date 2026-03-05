---
name: anti-astroturfing-global
description: >
  Credibility filter for global internet research. Automatically detects astroturfing, shilling,
  vote manipulation, and coordinated inauthentic behavior across X/Twitter, Reddit, Product Hunt,
  Hacker News, YouTube, LinkedIn, GitHub, Discord, TikTok, Mastodon, and other platforms.
  Scores each piece of content on three orthogonal dimensions and routes it into
  Organic Pool / Commercial Pool / Review Queue — never silently deletes anything.
  Use when: product research, sentiment analysis, competitive intelligence, market research,
  tech evaluation, or any task that collects public opinions from the global internet.
  Triggers: anti-astroturfing, filter shills, credibility check, shill detection, fake review filter,
  astroturf detection, coordinated inauthentic behavior, vote manipulation detection.
version: 1.0.0
author: DingWork
tags:
  - research
  - analysis
  - credibility
  - global
allowed_tools:
  - file_read
  - file_write
  - file_grep
  - file_glob
  - sandbox_execute
  - browser_use
  - local_chrome_use
  - get_url_contents
  - generate_image
---

# Anti-Astroturfing — Global Internet Edition

> During any research task (product sentiment, competitive analysis, market research, tech evaluation),
> **isolate and downrank** astroturfing, shilling, vote manipulation, and coordinated inauthentic behavior
> while preserving a full evidence chain for auditability.

---

## 1. Core Principles

1. **Triage, never trash.** Content flows into three pools — Organic / Commercial / Review Queue. Commercial content is not garbage; it reveals marketing angles and selling points, but must not be cited as organic sentiment.
2. **No single signal convicts.** A new account alone is not a shill. A link alone is not spam. Multiple signals must converge before risk escalates.
3. **Platform-specific baselines.** Each platform has different norms for tone, linking behavior, engagement ratios, and account metadata availability. Rules must adapt.
4. **Transparent evidence chain.** Every flag comes with an `Evidence` list — specific signals, their weights, and why they fired. The reader can always override.
5. **Cautious language.** Reports must say "suspected promotional content" or "likely coordinated," never accusatory certainties.

---

## 2. Three-Layer Detection Architecture

Score every piece of content on three orthogonal dimensions. This cleanly separates "the account is suspicious" from "this specific post is promotional" from "there's a coordinated campaign."

```
Account Layer  ──→  AccountRiskScore   (0-10)   Is this a sockpuppet / bot / shill account?
Post Layer     ──→  PostPromoScore     (0-10)   Is THIS post promotional, even if the account is real?
Coordination   ──→  CoordinationScore  (0-10)   Is this part of a coordinated campaign?
```

The three scores are computed independently and combined only at the routing stage.

### 2.1 Account Layer

Goal: Detect **sockpuppets, bot accounts, disposable shill accounts, and astroturf farms.**

| Signal | Description | Weight |
|--------|-------------|--------|
| Blank history + sudden activity | Near-zero post history, then a burst of on-topic commercial content | +4 |
| Topic entropy collapse | Account abruptly pivots from unrelated topics to a single product/niche | +3 |
| Mechanical posting pattern | Fixed intervals, on-the-hour timestamps, inhuman posting frequency | +3 |
| Engagement spike | Near-zero historical engagement, then sudden likes/replies/upvotes | +3 |
| Follower/following anomaly | Extreme ratio (follows thousands, followed by few) or vice versa | +2 |
| Username entropy | Random alphanumeric handle (e.g., `user_8f3k2x`, `john382847`) | +2 |
| Bio stuffed with CTAs | Profile/bio contains affiliate links, discount codes, or "DM for deals" | +2 |
| Lacks human texture (weak) | No personal anecdotes, no opinions outside one product, no humor | +1 |

> ⚠️ **New account ≠ shill.** Many legitimate users create accounts specifically to discuss a product they just bought. Account signals alone should push content to Review Queue, not Quarantine.

### 2.2 Post Layer

Goal: Detect **"this specific post is promotional / affiliate / paid"** — even if the author is a real person with a legitimate account.

**Strong signals (+4 to +8):**

| Signal | Description | Weight |
|--------|-------------|--------|
| Affiliate / tracking parameters | URL contains `?ref=`, `?via=`, `?aff=`, `?tag=`, `utm_*`, `?coupon=`, `?shareid=` | +8 |
| Link shortener → e-commerce | Short URL (bit.ly, t.co, amzn.to, etc.) resolves to a storefront or product page | +6 |
| Referral / invite code in body | "Use my code SAVE20", "Sign up with my link for $10 off" | +6 |
| Explicit CTA cluster | ≥ 2 of: "link in bio", "DM me", "check my profile", "limited time", "use code", "click here" | +4 |
| Undisclosed sponsorship cues | Reads like ad copy but no `#ad` / `#sponsored` / `#partner` disclosure | +4 |

**Medium signals (+2 to +3):**

| Signal | Description | Weight |
|--------|-------------|--------|
| Disclosed sponsorship | Contains `#ad`, `#sponsored`, `#partner`, "this post is sponsored by" — not malicious, but must be downranked from organic sentiment | +3 |
| Unbalanced praise | All pros, zero cons, zero caveats, zero "it depends" — especially for a complex product | +2 |
| Press-release tone | Reads like marketing copy rather than personal experience — formal, superlative-heavy, feature-list style | +2 |
| Generic superlatives | "Best tool ever", "Game changer", "Must have" with no specifics | +2 |

> ⚠️ **Links ≠ ads.** Technical communities routinely link to docs, papers, repos, and RFCs. Classify by domain (information vs. commerce) and by parameter (tracking vs. clean).

### 2.3 Coordination Layer

Goal: Detect **organized campaigns** — astroturf farms, vote rings, review brigades. Individual actors can dodge single-point rules, but coordinated groups leave statistical footprints.

| Signal | Description | Weight |
|--------|-------------|--------|
| Near-duplicate text cluster | ≥ 3 accounts posting highly similar text (synonym swaps, emoji variations, same structure) | +6 |
| Shared destination cluster | ≥ 3 accounts linking to the same shortened URL or landing page | +6 |
| Mutual engagement ring | Tight cluster of accounts that exclusively like/upvote/retweet each other, minimal outside interaction | +5 |
| Time-window synchronization | Burst of same-topic posts/votes within a narrow time window (e.g., 20 posts in 30 minutes) | +4 |
| Cross-platform echo | Same text or same links appearing across multiple platforms simultaneously | +3 |
| Comment template flooding | Multiple accounts leaving near-identical positive comments under the same post | +4 |

---

## 3. Platform-Specific Rule Sets

The scoring engine is universal. Platform differences live in these rule tables — covering field availability, platform-specific shill patterns, and normal behavioral baselines.

### 3.1 X / Twitter

| Signal | Layer | Weight | Notes |
|--------|-------|--------|-------|
| Account age < 60 days + followers < 50 | Account | +4 | Disposable shill account |
| Username is random alphanumeric | Account | +3 | Bulk-registered |
| Following/follower ratio > 10:1 | Account | +2 | Mass-follow bot |
| Default or AI-generated profile picture | Account | +2 | Low-effort sockpuppet |
| Tweet history is exclusively retweets/promotions | Post | +5 | No original thought |
| Tweet reads like a press release | Post | +2 | Possible PR placement |
| Contains affiliate/tracking link | Post | +8 | Affiliate shill |
| Thread of praise with identical structure from multiple accounts | Coord | +6 | Coordinated campaign |
| Burst of likes/retweets from low-follower accounts in < 1 hour | Coord | +5 | Engagement farming |

**Baseline:** X is inherently noisy. Short, hyperbolic takes ("This is insane 🔥🔥🔥") are normal — flag on *substance absence + commercial intent*, not on tone alone.

### 3.2 Reddit

| Signal | Layer | Weight | Notes |
|--------|-------|--------|-------|
| Karma < 100 + account age < 90 days | Account | +4 | New throwaway |
| Post/comment history exclusively about one product | Account | +4 | Single-purpose shill |
| Only active in product-specific subreddits | Account | +2 | Possibly stakeholder |
| Contains affiliate link (`?ref=`, `?tag=`, Amazon associate) | Post | +8 | Affiliate spam |
| Tone is press-release formal, not conversational | Post | +3 | PR copy |
| "I'm not affiliated but…" + link | Post | +3 | Unprompted denial (Streisand signal) |
| Dismisses all competitors without specifics | Post | +2 | Possible astroturf |
| Multiple low-karma accounts praising same product in same thread | Coord | +6 | Vote brigade |
| Rapid upvote surge on a post in a small subreddit | Coord | +4 | Vote manipulation |

**Baseline:** Reddit culture is skeptical by default. Genuine users often include caveats ("it's not perfect but…"). Absence of any criticism in a review-style post is a meaningful signal here.

### 3.3 Product Hunt

| Signal | Layer | Weight | Notes |
|--------|-------|--------|-------|
| Voter account has only ever upvoted this one product | Account | +5 | Single-purpose upvote account |
| Voter account created within 7 days of product launch | Account | +3 | Launch-day sockpuppet |
| Voter has no maker/hunter badges and zero comments | Account | +2 | Empty account |
| Comment is vacuous ("Amazing product!", "Love it!", "Great job team!") | Post | +3 | No substance |
| Comment mentions features not yet shipped (insider knowledge) | Post | +2 | Possible team member |
| ≥ 20 upvotes in first 30 minutes from accounts matching above patterns | Coord | +7 | Organized launch manipulation |
| Voters follow each other but have no other PH activity | Coord | +5 | Upvote ring |
| Maker's social accounts show call-to-action to "support us on PH" | Coord | +3 | Mobilized network (gray area) |

**Baseline:** Product Hunt has an endemic vote-gaming culture. Makers routinely mobilize their networks. Distinguish between *organic community mobilization* (gray, lower risk) and *purchased upvote services* (high risk) by checking voter account quality.

### 3.4 Hacker News

| Signal | Layer | Weight | Notes |
|--------|-------|--------|-------|
| Account age < 30 days + zero prior comments | Account | +4 | Throwaway |
| Only comments are about one company/product | Account | +3 | Single-purpose |
| Comment reads like marketing copy, not HN's terse style | Post | +4 | Tone mismatch — HN norms are dry, technical, critical |
| Contains product link without technical substance | Post | +3 | Promo drop |
| Dismisses Show HN criticism without engaging technically | Post | +2 | Defensive shill |
| Burst of upvotes on a "Show HN" post within minutes of posting | Coord | +5 | Vote ring |
| Multiple new accounts defending same product in same thread | Coord | +6 | Coordinated defense |

**Baseline:** HN's culture is unusually critical and technical. Genuine comments typically include caveats, technical details, or comparisons. Unqualified praise stands out more here than on any other platform.

### 3.5 YouTube

| Signal | Layer | Weight | Notes |
|--------|-------|--------|-------|
| Channel has no prior videos in this niche | Account | +3 | One-off sponsored content |
| Channel subscriber count vs. view count anomaly | Account | +2 | Possible paid promotion |
| Video description contains affiliate links | Post | +6 | Affiliate content |
| "Link in description" / "Use code X" in video | Post | +5 | CTA-heavy |
| Sponsored but no disclosure (`#ad`, FTC-required) | Post | +4 | Undisclosed sponsorship |
| Disclosed sponsorship (`#ad`, "sponsored by") | Post | +3 | Legitimate but must be downranked from organic |
| Comment section is uniformly positive, no critical comments | Coord | +4 | Comment curation / bot comments |
| Multiple comments with identical phrasing | Coord | +6 | Bot comment farm |

**Baseline:** YouTube's influencer economy makes sponsorship ubiquitous. Disclosed sponsorships are not shilling — they go to Commercial Pool, not Quarantine. Focus on *undisclosed* promotion and *fake engagement*.

### 3.6 LinkedIn

| Signal | Layer | Weight | Notes |
|--------|-------|--------|-------|
| Profile appears auto-generated (stock photo, generic headline) | Account | +3 | Fake profile |
| Account only engages with one company's content | Account | +3 | Employee / stakeholder |
| Post is "thought leadership" that reads as product placement | Post | +3 | Corporate astroturf |
| "I'm not affiliated" + company praise + link | Post | +3 | Streisand signal |
| Engagement pod pattern: same group always likes/comments first | Coord | +5 | Engagement pod |
| Comments are all "Great post! 👏" / "So insightful!" with no substance | Coord | +3 | Pod behavior |

**Baseline:** LinkedIn's culture is inherently promotional — people routinely hype their own companies. Apply a *higher threshold* for PostPromoScore here. Focus on *coordinated* patterns and *fake profiles* rather than individual promotional posts.

### 3.7 GitHub Discussions / Issues

| Signal | Layer | Weight | Notes |
|--------|-------|--------|-------|
| Account has zero repos, zero contributions, created recently | Account | +4 | Throwaway |
| Only activity is starring/praising one repo | Account | +3 | Star-farming account |
| Issue/discussion is a feature comparison that always favors one product | Post | +3 | Astroturf comparison |
| Links to commercial product as "solution" to issue | Post | +4 | Promo disguised as help |
| Burst of stars from empty accounts in short window | Coord | +6 | Star manipulation |

**Baseline:** GitHub is a technical platform. Genuine contributions include code, reproduction steps, or technical analysis. Pure praise without technical substance is anomalous.

### 3.8 Discord (Public Servers)

| Signal | Layer | Weight | Notes |
|--------|-------|--------|-------|
| Account joined server < 24 hours ago + immediately posts promo | Account | +4 | Drive-by shill |
| Account only posts in one channel, always about same product | Account | +3 | Single-purpose |
| Message contains invite link / referral code | Post | +6 | Referral spam |
| Message is a wall of text praising product, out of context | Post | +4 | Copy-paste promo |
| Multiple accounts posting similar messages across channels | Coord | +6 | Spam wave |

### 3.9 TikTok

| Signal | Layer | Weight | Notes |
|--------|-------|--------|-------|
| Account has < 5 videos, all about same product/brand | Account | +4 | Single-purpose promo account |
| Bio contains storefront link / "DM for collab" | Account | +2 | Influencer / potential shill |
| Video is unboxing/review with affiliate link in bio | Post | +5 | Affiliate content |
| "Link in bio" / "Use my code" without #ad disclosure | Post | +5 | Undisclosed sponsorship |
| Disclosed #ad / #sponsored | Post | +3 | Legitimate, downrank from organic |
| Comment section flooded with identical positive comments | Coord | +6 | Bot comments |
| Duet/stitch chain with identical praise from low-follower accounts | Coord | +5 | Coordinated campaign |

**Baseline:** TikTok's algorithm-driven discovery means even genuine users can go viral with zero followers. Account age/follower count is a *weaker* signal here than on other platforms. Focus on *content signals* and *coordination patterns*.

### 3.10 Mastodon / Fediverse

| Signal | Layer | Weight | Notes |
|--------|-------|--------|-------|
| Account on a single-user or tiny instance, created recently | Account | +3 | Possible sockpuppet instance |
| Only boosts/posts about one product | Account | +3 | Single-purpose |
| Post contains tracking links or affiliate parameters | Post | +7 | Commercial intent |
| Tone mismatch with Fediverse norms (too polished, too corporate) | Post | +2 | Astroturf |
| Same post boosted across multiple small instances simultaneously | Coord | +5 | Cross-instance coordination |

**Baseline:** The Fediverse skews anti-corporate. Overtly promotional content stands out sharply. However, the decentralized nature means anyone can spin up an instance — account provenance is harder to verify.

---

## 4. Scoring & Decision Routing

### 4.1 Labels & Thresholds (Configurable)

| Label | Trigger | Meaning |
|-------|---------|---------|
| `SockpuppetLikely` | AccountRiskScore ≥ 7 | Suspected fake / disposable account |
| `PromoLikely` | PostPromoScore ≥ 8 | Suspected promotional / affiliate content |
| `CoordinatedLikely` | CoordinationScore ≥ 7 | Suspected coordinated campaign |
| `StakeholderLikely` | Account is employee / investor / partner | Biased but not a shill — flag, don't quarantine |
| `DisclosedSponsorship` | Post contains `#ad` / `#sponsored` / `#partner` | Legitimate commercial, route to Commercial Pool |
| `NeedsReview` | Any score in borderline range (5–7) | Insufficient evidence, needs human judgment |

### 4.2 Operating Modes

| Mode | Use Case | Behavior |
|------|----------|----------|
| **Strict** | Quantitative analysis, formal reports | Borderline content excluded; maximize signal purity |
| **Balanced** | Exploratory research, early-stage discovery | Borderline content enters Review Queue; preserve more data |

Default: **Balanced**. Switch via instruction: "use strict mode" → Strict.

### 4.3 Routing Logic (Three-Pool Triage)

```
                         ┌─ All scores < 5
                         │   → ✅ KeepAsOrganic (Organic Pool)
                         │   Use for: core conclusions, sentiment statistics
                         │
Scoring done ──→ Route ──┼─ PostPromo ≥ 8 (regardless of account risk)
                         │   → 📦 MoveToCommercialPool
                         │   Use for: analyzing selling points, marketing angles, pricing signals
                         │   NOT for: proving organic sentiment
                         │
                         ├─ Any score in borderline range (5–7)
                         │   → 🔍 MoveToReviewQueue
                         │   Action: flag for human review, include in report appendix
                         │
                         ├─ AccountRisk ≥ 7 AND PostPromo < 5
                         │   → ⚠️ Downrank (cite with caveat, note account risk)
                         │
                         └─ AccountRisk ≥ 7 AND (PostPromo ≥ 8 OR Coordination ≥ 7)
                             → 🔴 Quarantine (exclude from analysis, list in appendix)
```

> **Two-channel evidence base:** The Organic Pool drives conclusions. The Commercial Pool reveals *how* the product is being marketed. Both are valuable — different questions, different pools.

---

## 5. Workflow

### 5.1 Integrated Research Mode (Recommended)

This skill activates as a filter layer during any research task:

**Step 1 — Ingest:** Collect content from target platforms. Record full metadata: author info, timestamps, engagement metrics, URLs, thread context.

**Step 2 — Normalize:** Unify fields across platforms. Resolve shortened URLs. Extract tracking parameters.

**Step 3 — Score:** Compute AccountRiskScore / PostPromoScore / CoordinationScore independently for each item. Build Evidence chain.

**Step 4 — Route:** Apply routing logic to assign each item to Organic Pool / Commercial Pool / Review Queue / Quarantine.

**Step 5 — Report:**
- Main analysis draws conclusions **only from Organic Pool** content.
- Commercial Pool content may be cited for marketing strategy analysis.
- Appendix includes Astroturf Audit Summary (see §6.2).

### 5.2 Standalone Audit Mode

Retroactively audit an existing research report:

1. Read the existing report, extract each cited opinion/post.
2. Apply three-layer scoring based on available information.
3. Where metadata is incomplete, annotate: "Assessed with limited information — confidence reduced."
4. Output an audit report with revised conclusions if the sentiment balance shifts significantly.

---

## 6. Output Formats

### 6.1 Inline Annotation (Embedded in Research Report)

Add a `Credibility` column to the evaluation table:

```markdown
| Source | Key Opinion | Engagement | Credibility |
|--------|-------------|------------|-------------|
| u/genuine_user on r/product | "Solid tool, but the API rate limits are frustrating. Switched from X because..." | 45↑ 12💬 | ✅ Organic |
| u/new_acc_2026 on r/product | "Best product ever! Totally changed my workflow!" | 3↑ 0💬 | 🔍 Review [Acct: new+4, Post: no specifics+2] |
| @shill_handle on X | "🔥 Game changer! Use my code SAVE20 → link" | 2♥ 0🔁 | 📦 Commercial [Post: affiliate+8, CTA+4] |
| @bot_8f3k on X | "Amazing tool!!!" (1 of 15 identical tweets) | 0♥ | 🔴 Quarantine [Acct: random name+3, age<7d+4; Coord: 15 duplicates+6] |
```

### 6.2 Astroturf Audit Summary (Report Appendix)

```markdown
## Astroturf Audit Summary

### Overview
- **Total samples collected:** N
- **Organic Pool:** M (X%) ← used for core conclusions
- **Commercial Pool:** P (Y%) ← used for marketing analysis
- **Review Queue:** Q (Z%)
- **Quarantined:** R (W%)

### Platform Breakdown
| Platform | Samples | Organic | Commercial | Review | Quarantined |
|----------|---------|---------|------------|--------|-------------|
| Reddit | 40 | 28 (70%) | 5 (12%) | 4 (10%) | 3 (8%) |
| X/Twitter | 35 | 20 (57%) | 8 (23%) | 5 (14%) | 2 (6%) |
| Product Hunt | 20 | 8 (40%) | 3 (15%) | 4 (20%) | 5 (25%) |
| Hacker News | 15 | 13 (87%) | 1 (7%) | 1 (7%) | 0 (0%) |
| ... | ... | ... | ... | ... | ... |

### Risk Type Distribution
- SockpuppetLikely: N1 items
- PromoLikely: N2 items
- CoordinatedLikely: N3 items
- StakeholderLikely: N4 items
- DisclosedSponsorship: N5 items

### Impact Assessment
After excluding astroturf / commercial content, the positive:negative sentiment ratio
shifted from X:Y to X':Y'. Core conclusions [were / were not] materially affected.

### Quarantined Content Detail
| # | Platform | Content Summary | Acct | Post | Coord | Signals Hit |
|---|----------|-----------------|------|------|-------|-------------|
| 1 | X | "Amazing tool!!!" (×15 identical) | 7 | 2 | 8 | random name, age<7d, 15 near-duplicates |
| 2 | PH | "Love it! Great job!" | 6 | 3 | 7 | single-product voter, launch-day account, upvote ring |
```

---

## 7. Common Pitfalls & Safeguards

1. **New account ≠ shill.**
   Many people create accounts specifically to discuss a product they just bought or a problem they just hit. Account age is a medium signal (+3–4) that must stack with promotional or coordination evidence to escalate.

2. **Links ≠ ads.**
   Technical communities (HN, GitHub, Reddit programming subs) routinely link to documentation, papers, repos, and RFCs. Classify by *domain type* (informational vs. commercial) and *URL parameters* (tracking vs. clean).

3. **Enthusiasm ≠ astroturf.**
   Some people genuinely love products and express it effusively. The reliable signals are *commercial intent* (affiliate links, referral codes, CTAs) and *coordination patterns* (duplicate text, vote rings), not enthusiasm alone.

4. **Stakeholder ≠ shill.**
   Employees, investors, and partners have real bias but are not astroturfers. Label them `StakeholderLikely` — downrank from organic pool but do not quarantine. Their perspective can be valuable context.

5. **Disclosed sponsorship ≠ deception.**
   Content marked `#ad` or `#sponsored` is playing by the rules. Route to Commercial Pool (not Quarantine). The real threat is *undisclosed* paid promotion.

6. **Platform culture matters.**
   - HN: Terse, critical, technical. Unqualified praise is anomalous.
   - Reddit: Skeptical by default. Absence of caveats is a signal.
   - X: Hyperbolic by nature. "🔥🔥🔥" is normal — flag on substance absence + commercial intent.
   - LinkedIn: Inherently promotional. Raise thresholds.
   - Product Hunt: Vote gaming is endemic. Focus on voter account quality.
   - TikTok: Algorithmic virality means low-follower accounts can be genuine. Weaken account-age signals.

7. **Never use accusatory language.**
   Reports must say "suspected promotional content," "likely coordinated activity," or "flagged for review" — never "this is a shill" or "this is fake."

---

## 8. Link & Keyword Lexicons

### 8.1 Tracking / Affiliate Parameters
`ref` `aff` `via` `tag` `utm_source` `utm_medium` `utm_campaign` `utm_content` `utm_term` `coupon` `promo` `discount` `shareid` `invite` `referral` `partner` `associate` `clickid` `subid` `irclickid`

### 8.2 Common Affiliate Domains
`amzn.to` `bit.ly` `tinyurl.com` `t.co` `geni.us` `shrsl.com` `rstyle.me` `shopstyle.it` `go.skimresources.com` `howl.me` `linktr.ee` (when linking to storefronts)

### 8.3 CTA / Promotional Keywords (English)
"link in bio", "use my code", "use code", "sign up with my link", "check the link", "DM me", "DM for details", "limited time", "exclusive offer", "discount code", "swipe up", "click here", "grab it now", "don't miss out", "last chance", "giveaway entry"

### 8.4 Sponsorship Disclosure Markers
`#ad` `#sponsored` `#partner` `#gifted` `#collab` `#brandpartner` `#paidpartnership` "sponsored by" "in partnership with" "thanks to [brand] for sponsoring"

---

## 9. Quality Checklist

Before delivering any research report, verify:

- [ ] Organic Pool contains zero affiliate/tracking links
- [ ] High-risk samples are quarantined and excluded from core sentiment statistics
- [ ] Key conclusions are cross-validated by multiple low-risk sources (cross-platform, cross-author, cross-time)
- [ ] Every "suspected astroturf" flag traces back to specific Evidence entries
- [ ] Report appendix includes the Astroturf Audit Summary
- [ ] Impact assessment states whether excluding flagged content materially changed the sentiment balance
- [ ] Language throughout uses "suspected" / "likely" — never accusatory certainties
