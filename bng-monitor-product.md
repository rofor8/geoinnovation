# BNG Monitor — Laser Product Definition

## One sentence

A dashboard that tells councils whether their BNG sites are keeping their promises.

## The one problem

Councils are legally required to monitor habitat gains on development sites for 30 years. They have 0.66 ecologists. They cannot do it. Sites will degrade unnoticed, £3bn of habitat investment becomes fiction, and councils carry the legal liability.

## The one customer

Local Planning Authority BNG leads. The person who signed off on the habitat management plan and is now personally responsible for 30 years of evidence they have no way to collect.

## The one question the product answers

**"Is this site maintaining the habitat gain that was promised — yes or no?"**

That's it. Red / amber / green per site. Evidence-grade.

---

## The core mechanic: Sightings + NGD Sites

We already have the **Sightings tool** — volunteers photograph green infrastructure,
select category and species, draw extents, log observations. Phone GPS, high-res
camera, even iPhone LiDAR for structure.

The product is: **bin every sighting into its NGD Site polygon.**

Every piece of land in England already has a polygon: **OS NGD Sites**. These are
functional sites — individual properties, businesses, schools, parks, car parks.
They are the same units that planning applications are made against.

```
Sighting → GPS locates it → falls within NGD Site polygon → site score updates
```

On the map, you don't see individual sightings. You see **NGD Site polygons**,
each coloured by their composite BNG score, with flags:

| Flag | Meaning | Effect |
|---|---|---|
| 🔍 **Undocumented** | No sightings yet for this site | Visible on map — incentivises volunteers to document it |
| 🕐 **Outdated** | Last sighting older than threshold (e.g. 6 months) | Prompts re-visit |
| ⚠️ **Needs attention** | Condition declining or issue flagged | Council notified |
| ✅ **Current** | Recently documented, condition acceptable | No action needed |

**Click any polygon** → expands to show all internal sightings, drawn polygons
(planting beds, hedgerows), individual species/condition data, and the overall
site score breakdown.

Why NGD Sites:
- Already exist nationally — no manual boundary drawing
- Map directly to planning applications (the BNG trigger)
- Each site is a functional unit someone is responsible for
- OS maintains them — boundaries update automatically
- AddressBase links each site to its address, owner, planning refs

For BNG-obligated sites, the score checks reality against the HMMP (Habitat
Management and Monitoring Plan). For all other sites, the score shows current
habitat condition — proactive intelligence, not just compliance.

**The key insight:** undocumented sites are visible. Every blank polygon on the
map is an invitation. People see their neighbourhood has gaps and fill them.
Coverage grows organically.

---

## Why smartphones, not satellites

Everyone in the country has a smartphone. A single person can document a site in
minutes: photograph all visible green infrastructure, select categories and species,
draw planting bed extents (with AR if needed), note fauna, log condition.

This gives us:

| Capability | Smartphone | Satellite (10m) |
|---|---|---|
| Resolution | Sub-centimetre photos | 10m pixels |
| Species identification | Yes — photos + human classification | No |
| Species per m² count | Yes — direct observation | No |
| Structural measurement | iPhone LiDAR | No (DEFRA VOM is single year only) |
| Condition assessment | Ground-level detail | Coarse vegetation indices |
| Fauna observation | Yes — insects, birds, mammals, fungi | No |
| Planting bed extents | AR-drawn GPS polygons | Sub-pixel, invisible |
| Indoor/rooftop access | Yes | Obstructed |
| Cost per observation | Free (volunteer time) | Free (but lower quality) |

**Satellite is supplementary.** Sentinel-2 NDVI provides a continuous background
signal between human visits — useful for temporal change detection and flagging
sites that need re-visiting. But the high-quality evidence comes from the ground.

---

## How the score works

Each NGD Site polygon gets a **composite BNG score** computed from multiple inputs:

### Input signals (ordered by importance)

| Source | What it captures | How it feeds the score |
|---|---|---|
| **Community sightings** | Three channels per submission: 1) Geotagged photos → ML-classified for species, condition, habitat type. 2) Manual GI type selection from structured list — human classification is more sophisticated than ML for edge cases and context. 3) Free text notes for additional observations (stress, damage, unusual activity, context). All three are further analysed: images via vision models, text via NLP, human labels as training signal. Covers the full biome — any sign of life or ecological activity: insects, birds, mammals, amphibians, nesting/breeding sites, pollinator counts, bat roosts, fungi/mushrooms, lichens, mosses, soil organisms, deadwood invertebrates. Every observation is a signal of habitat quality. Fungi on deadwood = healthy decomposition cycle. Lichen diversity = good air quality. Pollinator activity = functional food web. Nesting birds = structural habitat. A site with visible ecological activity scores higher than sterile green space. | Species richness, condition ground-truth, fauna as habitat quality indicator, nuanced context no sensor captures |
| **AR-drawn polygons** | Volunteers trace habitat patches in AR (planting beds, hedgerows, meadow edges, green walls) by walking the perimeter with phone camera — GPS polygon captured at ground level with photos embedded. Professionals can also use this for rapid site surveys. | Fine-grained habitat mapping at centimetre resolution, with visual evidence baked into each polygon |
| **iPhone LiDAR** | Structural measurement from phone — vegetation height, canopy volume, vertical diversity | Replaces DEFRA VOM for surveyed sites. Updatable, not a single frozen snapshot |
| **OS Enhanced Land Cover** | UKHAB habitat classification per site | Baseline habitat type, BNG metric 2.0 distinctiveness score |
| **Satellite spectral bands** | NDVI, chlorophyll, red-edge from Sentinel-2 every 5 days | Background change detection — flags sites for re-visit when vegetation signal changes |
| **DEFRA VOM** | National vegetation height/structure from LiDAR | Structural baseline for sites not yet surveyed by volunteers |
| **OS MasterMap** | Surface types, impermeable %, building footprints | Hard/soft surface ratio, change from pre-development |
| **Planning history** | HMMP commitments, BNG conditions, approval dates | What was promised, milestone deadlines |

### Score computation

The inputs fuse into a single site score via weighted composite index:

```
Site BNG Score = weighted combination of:
  - Species diversity            (community sightings — species count per m²)
  - Habitat condition            (community photos + classification + notes)
  - Fauna activity               (insect, bird, mammal, amphibian, fungi sightings)
  - Habitat structure            (iPhone LiDAR / VOM + AR polygons)
  - Vegetation cover & trend     (satellite spectral — background signal)
  - HMMP compliance              (reality vs plan milestones, if BNG-obligated)
```

Displayed as a colour on the map:

| Colour | Score | Meaning |
|---|---|---|
| Dark green | 80–100 | Excellent — thriving, diverse, exceeding targets |
| Green | 60–80 | Good — healthy habitat, on track |
| Yellow | 40–60 | Adequate — some vegetation but low quality/diversity |
| Amber | 20–40 | Degraded — declining or failing to establish |
| Red | 0–20 | Critical — sealed surface, dead planting, or no habitat |
| Grey | — | Undocumented — no data yet |

For BNG-obligated sites, this also maps to compliance:

| Status | Meaning | Action |
|---|---|---|
| Green | On track — habitat matching or exceeding HMMP milestones | No action needed |
| Amber | Watch — community flags concern or satellite detects decline | Council notified, verification requested |
| Red | Failing — habitat clearly not meeting HMMP conditions | Council must intervene, evidence logged |

**The key insight**: the same scoring works for every site, not just BNG-obligated
ones. Planners see habitat quality across their whole authority. A developer
checking a site before submitting plans can see exactly what the baseline is and
what BNG uplift is needed. Community groups see which streets are worst and
where action would have most impact.

---

## What the council sees

### Dashboard (the daily view)
- Map of authority — every NGD Site polygon coloured by BNG score
- Grey polygons = undocumented (incentivises coverage)
- BNG-obligated sites highlighted with a border/badge
- Summary: "47 BNG sites monitored. 41 green. 5 amber. 1 red. 204 undocumented."
- Filter by: developer, year approved, habitat type, ward, score range, flag status

### Site detail (click any polygon)
- All sightings within the polygon: photos, species, condition, dates
- AR-drawn habitat polygons: planting beds, hedgerows, green walls
- BNG score breakdown: which inputs are driving the number
- For BNG sites: HMMP summary, current status vs promise, next milestone
- Data freshness: "Last sighting: 2 weeks ago. Satellite: 2 days ago."
- Trend: arrow showing direction of travel over time

### Annual report (the compliance output)
- PDF per site or portfolio: "Here is the evidence that BNG obligations are being met"
- Includes community photos, species lists, condition assessments, satellite trend
- Suitable for audit, DEFRA reporting, committee papers
- This is what the council actually needs to discharge their legal duty

---

## What it doesn't do (yet)

- Does NOT suggest interventions (v2 feature — ranked actions to improve score)
- Does NOT replace professional ecology assessments (supplements them)
- Does NOT handle BNG credit trading / offsetting (that's Environment Bank / marketplaces)
- Does NOT require the council to change any existing process — sits alongside

## Why this wins

| Test | Answer |
|---|---|
| Is the pain acute? | Yes — legal obligation, no staff, accumulating liability |
| Is the buyer obvious? | Yes — BNG lead at the LPA, one person per council |
| Can they pay? | Yes — from BNG monitoring fees charged to developers |
| Is the alternative worse? | Yes — £300-800/day ecologist visits per site, or doing nothing and hoping |
| Does it need OS data? | Yes — NGD Sites polygons, Enhanced Land Cover, MasterMap surfaces |
| Can 2 people build it? | Yes — community app exists, satellite pipeline exists, need the dashboard |
| Can it be sold in 6 months? | Yes — Bristol pilot, show the traffic lights, charge £250/month |
| Does it get better with scale? | Yes — more volunteers = more ground-truth, more sites = better ML training |

---

## MVP: what ships first

### Month 1-2 (Geovation starts)
- Load NGD Sites polygons for Bristol — every functional site on the map
- Bin existing Sightings data into NGD Site polygons — sites immediately get scores
- Grey/undocumented sites visible — motivates volunteer coverage
- Overlay BNG-obligated sites from planning data
- Manual HMMP data entry for BNG sites (council provides the plans)

### Month 3-4
- Site flag system: undocumented, outdated, needs attention
- AR polygon capture: volunteers walk perimeter of planting beds / habitat patches
- iPhone LiDAR integration for structural measurement
- Satellite NDVI background monitoring — auto-flags sites with declining signal
- OS Enhanced Land Cover integration (UKHAB habitat type per site)

### Month 5-6
- Annual compliance report generator (PDF)
- Second council onboarded
- HMMP data structured and searchable
- Demo to 5+ South West councils

### After Geovation
- HMMP auto-parsing (AI reads the PDF plans)
- Intervention suggestions for red/amber sites (the vision feature)
- Developer self-service portal (they pay to prove compliance)
- National rollout

---

## Revenue from day one

Councils already charge developers **BNG monitoring fees** as part of planning conditions. The money exists — it's just spent on occasional consultant visits. BNG Monitor replaces sporadic expensive checks with continuous cheap monitoring.

- Council pays: £250/month (£3,000/year) for dashboard + monitoring
- Developer pays: £500-2,000 per site assessment (replaces £2-5k consultant visit)
- The council can fund BNG Monitor from fees they're already collecting

---

## The competitive moat restated

**Verna** tells councils which sites need monitoring (paperwork).
**BNG Monitor** tells councils whether those sites are actually healthy (reality).

They're complementary, not competing. Verna is the to-do list. We're the evidence.

No one else has community ground-truth. Parliament says it's needed. We have 143
assets proving it works. Every smartphone is a sensor. Every volunteer is an
ecologist-in-training. No satellite can count species per square metre — but a
person with a phone can do it in minutes.
