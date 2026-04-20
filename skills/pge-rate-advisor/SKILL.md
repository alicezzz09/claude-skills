---
name: pge-rate-advisor
description: >
  Analyzes PG&E residential electricity usage to recommend the optimal rate plan (E-1, E-TOU-C, E-TOU-D, E-ELEC, EV2-A, EV-B) and estimate potential savings. Use this skill whenever a user asks about PG&E electricity bills, rate plans, TOU plans, electricity costs, or wants to know if they could save money on their PG&E plan. Trigger for phrases like "which PG&E plan is best", "should I switch rate plans", "compare electricity rates", "am I on the right plan", "save on electricity bill", or any mention of PG&E rate comparison. Also trigger when a user uploads a PG&E bill, PG&E usage CSV, or asks about time-of-use pricing. Always use this skill proactively when electricity rate optimization is even tangentially relevant.
---

# PG&E Residential Rate Plan Advisor

This skill helps users find the best-value PG&E residential electricity rate plan by analyzing their usage patterns and comparing estimated costs across all applicable plans.

---

## Rate Plans Covered (effective March 1, 2026)

### Tiered Plans
| Plan | Description | Tier 1 (≤100% baseline) | Tier 2 (>100% baseline) |
|------|-------------|------------------------|------------------------|
| **E-1** | Standard tiered | $0.32561/kWh | $0.40702/kWh |

**E-1 fixed charges:** Base Services Charge = Income Tier 3: $0.79343/day (~$24/mo), Tier 2: $0.39688/day, Tier 1: $0.19713/day

### Time-of-Use Plans (TOU)
All TOU plans have the same Base Services Charge tiers as E-1.

| Plan | Peak Window | Summer Peak | Summer Off-Peak | Winter Peak | Winter Off-Peak | Baseline Credit |
|------|------------|-------------|-----------------|-------------|-----------------|-----------------|
| **E-TOU-C** | 4–9pm every day | $0.52240 | $0.39940 | $0.39757 | $0.36757 | –$0.08140 applied to baseline usage |
| **E-TOU-D** | 5–8pm weekdays only | $0.47708 | $0.34212 | $0.38747 | $0.34886 | None |
| **E-ELEC** *(requires EV, battery, or heat pump)* | 4–9pm every day | $0.55214 | $0.33358 | $0.32063 | $0.28468 | None; partial-peak rates apply 3–4pm & 9pm–midnight |
| **EV2-A** *(requires EV)* | 3–4pm + 4–9pm daily | $0.53809 peak / $0.42760 partial-peak | $0.22558 off-peak | $0.41099 / $0.39428 | $0.22558 | None |
| **EV-B** *(requires EV)* | 2–9pm weekdays | $0.62131 peak / $0.37720 partial-peak | $0.26465 off-peak | $0.43878 / $0.30677 | $0.23504 | None; no Base Services Charge (meter charge $0.04928/day instead) |

**Season definitions:**
- E-1, E-TOU-C, E-TOU-D, E-ELEC, EV2-A: Summer = June–Sep, Winter = Oct–May
- EV-B: Summer = May–Oct, Winter = Nov–Apr

For full TOU period details → read `references/tou-periods.md`  
For baseline territory quantities → read `references/baseline-territories.md`

---

## Step 1: Collect User Information via `ask_user_input`

Use the `ask_user_input` tool to gather the following. Ask in 2–3 rounds maximum; batch related questions together.

### Round 1 — Bill & Plan Basics
```
Q1: Do you have a recent PG&E bill?
  Options: Yes, I can share it | No, I'll estimate usage

Q2: Do you have an EV, home battery, or electric heat pump?
  Options: Yes (EV) | Yes (battery or heat pump) | Both | None of these

Q3: Are you currently on a time-of-use (TOU) plan?
  Options: Yes | No (tiered E-1) | Not sure
```

### Round 2 — Usage Details (if no bill/CSV provided)
```
Q4: What's your approximate monthly electricity usage?
  Options: Under 300 kWh | 300–600 kWh | 600–1000 kWh | Over 1000 kWh

Q5: When do you use most of your electricity?
  Options: Mostly daytime (9am–4pm) | Mostly evenings (4–9pm) | Spread evenly | Mostly overnight (9pm–7am)
```

### Round 3 — Home & Baseline
```
Q6: What is your primary heating source?
  Options: Electric (heat pump or resistance) | Gas | Other/don't know

Q7: Where do you live? (This helps determine your energy baseline allowance)
  Options: San Jose / South Bay / Santa Clara Valley | San Francisco / Peninsula / Coastal Bay Area | Sacramento / Central Valley inland | Fresno / Bakersfield / Hot Central Valley | Monterey / Santa Cruz / Coast | Stockton / Modesto / Inland Valley | Other / Not sure (enter city or zip)
```

After the user answers Q7, map their region to a territory code using `references/baseline-territories.md`. If they pick "Other / Not sure", ask for their city or zip code and look it up; if still ambiguous, default to Territory X with a note.

**PRIVACY NOTE:** If the user shares a PG&E bill or usage CSV, extract the relevant figures (kWh usage, billing period, baseline territory, heat source code) but **do not repeat or display** the account number, full address, or full name in your analysis output. Reference the user only as "your account."

---

## Step 2: Parse Usage Data

### Option A — PG&E 15-minute interval CSV
The user may export usage from: PG&E portal → Menu → Usage & Rates → Energy Usage Details

CSV format:
```
TYPE, DATE, START TIME, END TIME, USAGE (kWh), COST, NOTES
Electric usage, 2026-03-18, 00:00, 00:14, 0.31, $0.06
```

Parse by:
1. Skip metadata rows (Name, Address, Account Number, Service) — **do not display these**
2. Group records by date and hour
3. Flag which 15-min slots fall in peak/partial-peak/off-peak windows for each plan
4. Sum kWh by TOU period per day → aggregate to monthly

### Option B — Bill information only
Use daily average kWh from the bill. Apply a reasonable usage shape assumption:
- If user says "mostly evenings": 40% of usage in 4–9pm window
- If user says "mostly daytime": 70% off-peak
- If user says "overnight heavy": 60% overnight off-peak
- Otherwise: use typical residential shape (~30% peak, 70% off-peak in winter)

---

## Step 3: Determine Baseline

Look up the user's daily baseline kWh from `references/baseline-territories.md`:
- Use their territory code (from bill's "Baseline Territory" field, default X if unknown)
- Use their heat source code (H = All-Electric, B = Basic Electric/Gas heat)
- Apply summer baseline for June–Sep, winter baseline for Oct–May
- Monthly baseline = daily kWh × days in billing period

For E-TOU-C: the baseline credit (–$0.08140/kWh) applies to baseline usage in all TOU periods.
For E-1: Tier 1 rate applies up to 100% of baseline; Tier 2 above that.

---

## Step 4: Estimate Costs Per Plan

Calculate estimated monthly cost for each applicable plan:

### E-1 Cost Formula
```
cost = (base_services_charge_per_day × days)
     + (min(total_kwh, baseline_kwh) × 0.32561)
     + (max(0, total_kwh - baseline_kwh) × 0.40702)
```

### TOU Cost Formula (E-TOU-C example)
```
cost = (base_services_charge_per_day × days)
     + (peak_kwh × peak_rate)
     + (offpeak_kwh × offpeak_rate)
     - (min(total_kwh, baseline_kwh) × 0.08140)   ← baseline credit
```

### EV-B Cost Formula (no Base Services Charge)
```
cost = (0.04928 × days)   ← meter charge
     + (peak_kwh × peak_rate)
     + (partpeak_kwh × partpeak_rate)
     + (offpeak_kwh × offpeak_rate)
```

Apply eligibility filters:
- E-ELEC and EV2-A: only if user has EV, battery, or heat pump
- EV-B: only if user has EV

---

## Step 5: Output Format

Present results in this structure:

### 1. Your Current Plan Estimate
Show estimated cost on their current plan for the billing period.

### 2. Comparison Table (Summer & Winter)
| Plan | Est. Summer Bill | Est. Winter Bill | Annual Est. | vs. Current |
|------|-----------------|-----------------|-------------|-------------|
| E-1  | $X | $X | $X | baseline |
| E-TOU-C | $X | $X | $X | ±$X |
| E-TOU-D | $X | $X | $X | ±$X |
| E-ELEC* | $X | $X | $X | ±$X |
| EV2-A* | $X | $X | $X | ±$X |
| EV-B* | $X | $X | $X | ±$X |

*Only show if user is eligible

### 3. Recommendation
- **Best plan overall** with reasoning
- **Best plan if usage shifts** (e.g., "If you charge your EV overnight, EV-B saves ~$X/year")
- Note any assumptions made

### 4. Important Notice — Your Actual Bill Will Be Higher
**Always include this notice explicitly in the response, worded clearly for the user:**

> ⚠️ **These estimates cover PG&E delivery charges only.** Most Bay Area customers also pay a separate electricity generation charge from a Community Choice Aggregator (CCA) — for example, San Jose Clean Energy (SJCE) for San Jose residents, or Peninsula Clean Energy, Marin Clean Energy, etc. for other areas. This charge typically adds **$10–$20/month** (or more for high usage) on top of the delivery estimates shown above.
>
> The good news: CCA generation charges are the **same regardless of which PG&E rate plan you choose**, so the savings comparisons above are still accurate — you'd save the same relative amount on any plan. Your real total bill = the estimates above + your CCA generation charge + local taxes/surcharges.

Also note:
- Base Services Charge tier depends on income; this analysis assumes Tier 3 (most common) unless the user's bill shows otherwise
- Encourage user to verify with PG&E's official rate comparison tool at pge.com

---

## Key Notes

- **SJCE / CCA customers**: Many San Jose / Bay Area customers have generation charges from a Community Choice Aggregator. These are billed separately and roughly equivalent across plans — analysis focuses on delivery charges where plan choice matters most.
- **Base Services Charge**: Implemented March 1, 2026. Income Tier 1 = $0.19713/day, Tier 2 = $0.39688/day, Tier 3 = $0.79343/day. Most customers are Tier 3. EV-B has no Base Services Charge (meter charge instead).
- **CARE discount**: 35% discount on volumetric charges. If user mentions CARE, apply discount to energy charges.
- **California Climate Credit**: Applied in March and October bills (~$36.18 credit). Mention but don't include in monthly estimate unless analyzing those months.

---

## References

- `references/tou-periods.md` — Exact peak/partial-peak/off-peak hour windows for all plans
- `references/baseline-territories.md` — Daily baseline kWh by territory and heat source code
