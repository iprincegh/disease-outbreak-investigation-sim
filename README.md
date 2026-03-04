# Outbreak Investigation — Veronika's Party

Retrospective cohort study of a gastroenteritis outbreak that followed a dinner party on 26 February 2026. The analysis walks through the full outbreak investigation — from descriptive epi through to identifying the food vehicle and narrowing down the pathogen.

This was done as part of a field epidemiology training exercise, following the EPIET/RKI outbreak investigation framework.

## What happened

Veronika hosted a dinner party with 56 guests. By the next day, most of them were sick — vomiting, nausea, diarrhoea. We sent out a structured questionnaire asking about symptoms, timing, and which of the 12 food items each person ate. The goal: figure out which food made everyone sick, and what pathogen was likely responsible.

**Short version of the results:** 42 out of 56 guests fell ill (75% attack rate). Tiramisu had the strongest association with illness by a wide margin — 34 of 35 people who ate it got sick. The clinical picture (vomiting-predominant, ~30h median incubation, self-limiting) points to norovirus.

## Repo structure

```
outbreak/
├── README.md
├── outbreak_analysis.Rmd       # the full analysis
├── outbreak_analysis.html      # knitted report
└── data/
    └── Outbreak_data.csv       # linelist
```

The entire analysis lives in a single RMarkdown file. The CSV is semicolon-delimited with Latin-1 encoding (German date formats and food names like Räucherlachsrolle and Kräuterquark).

## What the analysis covers

The Rmd is structured as a narrative that follows the standard outbreak investigation steps. Here's roughly what each section does:

**Descriptive epi** — who got sick, broken down by age and sex. Spoiler: everyone got sick at roughly the same rate regardless of demographics, which is what you'd expect from a foodborne common-source event.

**Epidemic curve** — the main 3-hour bin epicurve uses EPIET-style stacked squares with a two-level x-axis (time ticks on top, dates below, vertical midnight separators). There's also a daily epicurve, one stratified by sex, and a cumulative incidence curve. The shape is textbook point-source: one sharp peak, no secondary wave.

**Incubation period** — calculated as hours from party start (19:00) to symptom onset. Median around 30 hours, range 16–83h.

**Symptoms** — vomiting was universal (100%), followed by nausea (98%) and diarrhoea (93%). No bloody stool at all. Median illness duration around 38 hours. There's a histogram of duration and a Gantt-style timeline showing each person's illness window.

**Food-specific attack rates** — this is the core analytical piece. For each of the 12 foods, we build a 2×2 table (ate/didn't eat × ill/well), compute attack rates, risk ratios with 95% CIs, and Fisher's exact p-values using `epitools`. Tiramisu dominates: RR well above 1, p < 0.001. There's a forest plot and a stacked bar chart to visualise it.

**Tiramisu × strawberry co-consumption** — strawberries also showed a significant RR, so we check whether that's real or just confounding. A heatmap and proportion bar show that nearly all strawberry eaters also ate tiramisu, so the strawberry association is most likely an artefact.

**Dose–response** — attack rate by total number of food items consumed. People who ate more items were more likely to be sick, consistent with tiramisu being a widely consumed dish.

**Stratified analysis** — tiramisu RR holds up when you stratify by potato salad or sex, ruling out those as confounders.

**Household clustering and secondary transmission** — all household members attended the party together, so we can't compute a classical secondary attack rate (no unexposed household contacts). Instead we look at within-household onset timing: if someone in a household got sick much later than their family members, that could suggest person-to-person spread. We plot onset times colour-coded by household, with a 48-hour reference line. The section also explains why R₀ doesn't really apply to a foodborne point-source outbreak (it's a person-to-person concept), with the formulas from the Karch lecture material.

**Pathogen hypothesis** — putting together the incubation, symptoms, attack rate, and epi curve shape to do a differential. Norovirus fits best; S. aureus and B. cereus are too fast, Salmonella doesn't match the symptom profile.

## Running it

You need R (tested on 4.5.2) with these packages:

```r
install.packages(c("tidyverse", "lubridate", "epitools", "incidence2", "scales"))
```

Then either open the Rmd in RStudio and hit Knit, or from the terminal:

```bash
Rscript -e 'rmarkdown::render("outbreak_analysis.Rmd")'
```

The data path in the Rmd is `data/Outbreak_data.csv` — should work out of the box with this repo layout.

## Data notes

The linelist has 79 rows (56 attendees + 23 incomplete/non-attendee records that get filtered out). It's semicolon-delimited, Latin-1 encoded. Dates are in German format (`DD.MM.YYYY HH:MM`). The 12 food exposure columns are binary (0/1). There's a household ID for clustering analysis and fields for travel history and prior illness to check for confounders.

One known quirk: Margarete Schubert's `symp_stop` has a truncated year (`28.02.202` instead of `28.02.2026`), which gets caught and fixed in the cleaning step.

## Team
Sara, George, Christiane & Prince

## References

The methodology follows materials from:

- Karch — *Dynamics of Infection* (R₀, Rₙ, secondary attack rates), Clinical Epidemiology Spring 2026
- Wilking (RKI) — *Infectious Diseases Outbreak Investigations*, Clinical Epidemiology Spring 2026
- Jäger — *Outbreak Practical: Introduction*, Clinical Epidemiology Spring 2026
- The [`incidence2`](https://cran.r-project.org/package=incidence2) R package documentation
