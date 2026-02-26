# BNG Monitor — Technical Scoring Approach

## The official BNG formula

```
Habitat Units = Area × Distinctiveness(0-8) × Condition(0-3) × Strategic Significance(1.0-1.15)
```

Four factors. We need to compute or approximate all four per NGD Site polygon.

---

## Factor by factor: what we can derive

### 1. Area ✓ (from NGD Sites)

- NGD Site polygons give us the area in m²/hectares
- This is the most expensive data (OS licence via Geovation)
- Already computed per polygon — no processing needed

### 2. Distinctiveness ✓ (from OS Enhanced Land Cover → UKHAB lookup)

OS Enhanced Land Cover classifies every piece of land using UKHAB codes. Each
UKHAB code maps directly to a BNG distinctiveness score via a fixed lookup table:

| UKHAB Habitat | Distinctiveness | Score |
|---|---|---|
| Developed land / sealed surface | Very Low | 0 |
| Modified grassland | Low | 2 |
| Vegetated garden | Low | 2 |
| Arable field margins | Low | 2 |
| Other neutral grassland | Medium | 4 |
| Mixed scrub | Medium | 4 |
| Ponds (non-priority) | Medium | 4 |
| Other broadleaved woodland | Medium | 4 |
| Lowland mixed deciduous woodland | High | 6 |
| Lowland heathland | High | 6 |
| Reedbeds | High | 6 |
| Lowland meadows | Very High | 8 |
| Blanket bog | Very High | 8 |
| Ancient woodland | Very High | 8 |

**This is a simple join.** OS Enhanced Land Cover UKHAB code → lookup table → distinctiveness score. Free to compute, authoritative, national coverage.

### 3. Strategic Significance ✓ (from LNRS mapping)

Once Local Nature Recovery Strategies are published:
- Sites within LNRS priority areas → 1.15 (High)
- Sites not in LNRS areas → 1.0 (Low)

LNRS data will be open. Binary spatial join. Free to compute.

### 4. Condition — SMARTPHONES CAN DO THIS

Official condition assessment requires counting:
- Vascular plant species per m² (6-8 for Moderate, 10+ for Good in grassland)
- Sward height variation
- Bare ground coverage %
- Invasive species cover
- Native species diversity (woodland)
- Age structure (woodland)
- Deadwood presence

**Satellite can't do this.** 10m pixels can't count species per m².

**But a person with a smartphone can.** In minutes. The official condition criteria
are fundamentally ground-level observations — exactly what our Sightings tool
captures:

| Official condition criterion | How a volunteer captures it |
|---|---|
| Species per m² | Photograph and classify all visible species in a patch |
| Sward height variation | iPhone LiDAR scan, or photo with reference object |
| Bare ground % | Photo of ground cover, manual estimate or ML classification |
| Invasive species cover | Photo + species identification (manual or ML) |
| Native species diversity | Count of unique native species photographed |
| Age structure (woodland) | Photos of tree sizes + structural notes |
| Deadwood presence | Photo observation — yes/no/extent |

**This is the insight.** The BNG condition criteria that make satellite useless
are precisely the things a volunteer with a phone excels at. Our Sightings tool
was designed for this — it just didn't know it was computing BNG condition.

---

## The BNG Monitor Condition Score

### Primary source: community observations (smartphones)

The three-channel observation (photos + manual classification + free text) gives us
the highest-quality condition data available at scale:

**What we can derive per polygon from sightings:**
- Species richness (count of unique species observed) — **directly maps to BNG condition bands**
- Species per m² estimates (count species in photographed area)
- Condition keywords from NLP on free text
- Photo-based condition classification from vision models
- Presence of indicator species (both positive and negative)
- Fauna diversity as functional ecosystem indicator (insects, birds, mammals, fungi)
- Ground-level issues invisible from space (pollution, damage, fly-tipping)
- Bare ground estimates from photos
- Structural diversity from iPhone LiDAR or photo analysis

**AR-drawn polygons give us:**
- Exact extents of planting beds, hedgerows, green walls
- Habitat mapping at centimetre resolution (vs satellite's 10m)
- Visual evidence baked into each polygon boundary
- Area calculations for fine-grained habitat patches

### Secondary source: satellite (temporal gap-fill)

Sentinel-2 (free, every 5 days, 10m) provides the **background monitoring signal**
between volunteer visits:

| Band/Index | What it indicates | Role |
|---|---|---|
| NDVI (B4/B8) | Vegetation cover & vigour | Flags sites where vegetation is declining — triggers re-visit |
| Red-edge (B5/B6/B7) | Chlorophyll content, vegetation stress | Early stress detection before visible decline |
| SCL (Scene Classification) | Bare soil, vegetation, water, sealed | Detects land cover change — did vegetated area shrink? |

**What satellite does for us:**
- Continuous temporal signal between human visits (every 5 days vs whenever a volunteer goes)
- Automated change detection alerts ("NDVI dropped 20% on this site — needs re-visit")
- Seasonal phenology baseline (is vegetation growing when expected?)
- National coverage for sites not yet visited by volunteers

**What satellite cannot do:**
- Count species per m² (the core BNG condition criterion)
- Identify invasive species
- Assess habitat quality below 10m resolution
- Detect ground-level condition (damage, pollution, fly-tipping)
- Measure fauna activity

### Tertiary source: DEFRA VOM (structural baseline)

The Vegetation Object Model gives us vegetation height/structure from LiDAR:
- Canopy height statistics (mean, max, std dev)
- Structural diversity score (multiple height layers = more complex habitat)
- Baseline structure for sites not yet surveyed

**Limitation:** single composite year only. iPhone LiDAR from volunteers provides
updatable structural data that supersedes VOM for surveyed sites.

---

## The composite: how it all combines

```
BNG Monitor Score = f(
  Distinctiveness_official,        ← from Enhanced Land Cover (fixed lookup, certain)
  StrategicSignificance_official,  ← from LNRS (binary, certain)
  Area_official,                   ← from NGD polygon (measured, certain)
  Condition(                       ← our innovation (ground-truth + satellite supplement)
    community_species_richness,    ← count of species observed per site (PRIMARY)
    community_condition_signals,   ← ML on photos + NLP on text (PRIMARY)
    community_fauna_activity,      ← ecological activity indicators (PRIMARY)
    community_structural_data,     ← iPhone LiDAR + AR polygons (PRIMARY)
    satellite_change_detection,    ← NDVI trend flags declining sites (SUPPLEMENT)
    VOM_structural_baseline,       ← height layers for unvisited sites (FALLBACK)
    expected_condition_for_type    ← what condition SHOULD be for this UKHAB type
  )
)
```

### The condition scoring

For each polygon, condition is scored 0-3 approximating official categories:

| Score | Maps to | How derived |
|---|---|---|
| 2.5-3.0 | Good | 10+ species observed, positive fauna (insects/birds/fungi), structural diversity confirmed, no stress signals, healthy photos |
| 2.0-2.5 | Fairly Good | 6-9 species, some fauna activity, reasonable structure, stable satellite signal |
| 1.5-2.0 | Moderate | 4-5 species, limited structure, minimal fauna, satellite stable |
| 1.0-1.5 | Fairly Poor | 2-3 species, flat structure, no fauna signals, satellite declining |
| 0.5-1.0 | Poor | <2 species, no structure, negative indicators, satellite low/declining |
| 0-0.5 | N/A | Sealed surface, no vegetation signal |

**For sites with community data:** score derived primarily from species counts,
photo analysis, and fauna observations — the same criteria an ecologist uses.

**For sites without community data:** score derived from satellite NDVI + VOM
structure — a lower-confidence proxy that flags the site for volunteer visit.

**The key:** we're not claiming this IS the official condition score. But our
community observations capture the same ground-level signals an ecologist looks
for. Between formal 5-yearly assessments, this gives continuous monitoring — and
the community data may be closer to official condition than any satellite proxy.

---

## What this actually costs to compute

| Data source | Cost | Coverage | Update |
|---|---|---|---|
| Community sightings | Free (volunteer network) | Where volunteers are active | Continuous |
| Smartphone cameras | Free (volunteers' own phones) | Universal | Every visit |
| iPhone LiDAR | Free (built into modern iPhones) | Where iPhone users visit | Every visit |
| Sentinel-2 imagery | Free (Copernicus Open Access) | National | Every 5 days |
| Sentinel-2 processing | <£40/month serverless (existing pipeline) | Bristol → national | Automated |
| DEFRA VOM | Free (Open Government Licence) | National | Single composite (future updates TBC) |
| OS Enhanced Land Cover | Via Geovation (PSGA) | National | Updated periodically |
| OS NGD Sites | Via Geovation (PSGA) | National | Maintained by OS |
| ML model training | One-time compute cost | N/A | Retrain periodically |

**Total ongoing cost for national computation: <£40/month + OS licence**

The OS licence is the critical dependency that Geovation provides.

---

## What the time series graph shows

When you click a polygon, you see a chart. X-axis is time, Y-axis is the composite score.

```
Score
3.0 ┤                                    ╭──── Good
    │                              ╭─────╯
2.5 ┤                        ╭─────╯         Fairly Good
    │                  ╭─────╯
2.0 ┤            ╭─────╯                     Moderate ← target by Year 5
    │      ╭─────╯
1.5 ┤╭─────╯                                 Fairly Poor
    ││
1.0 ┤│ ← development complete, bare ground   Poor
    │
    └──┬──┬──┬──┬──┬──┬──┬──┬──┬──┬──┬──→ Time
      Y0 Y1 Y2 Y3 Y4 Y5 Y6 Y7 Y8 Y9 Y10
```

The graph shows:
- **Green line**: actual score over time (from community observations + satellite)
- **Dashed line**: expected trajectory from HMMP (what was promised)
- **Red zone**: below the expected trajectory = potentially failing
- **Volunteer dots**: when someone visited and documented the site (large, high-confidence)
- **Satellite dots**: automated readings between visits (small, lower-confidence)
- **Score breakdown**: hover over any point to see which factors are driving it

---

## Honest limitations (what we tell councils)

1. **Not a replacement for formal ecology assessment.** The official condition
   score requires standardised methodology. Our community observations capture
   similar signals but aren't performed by certified ecologists. The official
   assessment still has a role — but we bridge the 5-year gap between them.

2. **Community coverage depends on volunteers.** Sites in high-population areas
   get documented first. Rural BNG sites may have fewer observations. Satellite
   provides the background signal everywhere; community data enriches where available.
   The "undocumented" flag incentivises coverage.

3. **VOM is a single year.** Until DEFRA updates it, we can't track structural
   change from national LiDAR. iPhone LiDAR from volunteers and satellite NDVI
   bridge the gap for visited sites.

4. **Cloud cover affects satellite.** Sentinel-2 is optical — no data through
   clouds. UK averages ~60% cloud cover. This is why satellite is supplementary,
   not primary. Community observations work in any weather.

5. **Species identification accuracy.** Volunteer species IDs vary in quality.
   ML classification on photos helps. Three-channel approach (photo + manual
   selection + text) provides cross-validation. Accuracy improves as the training
   dataset grows.

---

## The positioning: what we say to Geovation

"BNG Monitor doesn't replace the ecologist. It's the continuous monitoring layer
between their 5-yearly visits. Community volunteers with smartphones capture the
ground-level evidence councils need. Satellite provides the temporal signal between
visits. The ecologist confirms it officially. Without our layer, councils are
flying blind between formal assessments — and 30 years is a long time to fly blind."

The formula we compute is a **proxy for the official BNG formula** using the same
structure (Area × Distinctiveness × Condition × Strategic Significance) but with
condition derived primarily from community smartphone observations, supplemented
by satellite change detection. Three of four factors are identical to the official
metric. The fourth (condition) is our innovation — crowd-sourced ground-truth that
captures the same signals an ecologist looks for, continuously, at national scale,
for free.
