# PG&E Baseline Territory Daily kWh Allowances

Source: PG&E tariff data, effective June 1, 2022 – Present  
(Used for E-1 tiering and E-TOU-C baseline credit)

---

## How to Use

1. Find the user's **Baseline Territory** code (letter P–Z) on their bill under "Service Information"
2. Find their **Heat Source** code:
   - Code **H** = All-Electric (electric heat / heat pump) → use "ALL-ELEC." row
   - Code **B** = Basic Electric (gas heat) → use "BASIC ELEC." row
3. Look up the daily kWh for the appropriate season
4. Monthly baseline = daily kWh × number of billing days

**Default if unknown:** Territory X, All-Electric (H) is a reasonable Bay Area default.

---

## ALL-ELECTRIC Customers (Heat Source Code H)
*(e.g., electric heat pump, resistance heating)*

| Territory | Winter Daily (kWh) | Summer Daily (kWh) |
|-----------|-------------------|-------------------|
| P | 26.0 | 15.2 |
| Q | 26.0 | 8.5 |
| R | 26.7 | 19.9 |
| S | 23.7 | 17.8 |
| T | 12.9 | 7.1 |
| V | 19.1 | 10.4 |
| W | 19.0 | 22.4 |
| **X** | **14.6** | **8.5** |
| Y | 24.0 | 12.0 |
| Z | 15.7 | 6.7 |

*Winter = October–May; Summer = June–September*  
*Territory X is most common for San Jose / South Bay area.*

---

## BASIC ELECTRIC Customers (Heat Source Code B)
*(e.g., gas furnace, other non-electric primary heat)*

| Territory | Winter Daily (kWh) | Summer Daily (kWh) |
|-----------|-------------------|-------------------|
| P | 11.0 | 13.5 |
| Q | 11.0 | 9.8 |
| R | 10.4 | 17.7 |
| S | 10.2 | 15.0 |
| T | 7.5 | 6.5 |
| V | 8.1 | 7.1 |
| W | 9.8 | 19.2 |
| **X** | **9.7** | **9.8** |
| Y | 11.1 | 10.5 |
| Z | 7.8 | 5.9 |

---

## Notes

- These baselines apply to individually-metered residential service (E-1, E-TOU-C, E-TOU-D, EV2-A, E-ELEC)
- EV-B and EM/EM-TOU use different baseline structures — EV-B has no baseline credit
- For E-TOU-C: the baseline credit (–$0.08140/kWh) is applied to kWh up to the baseline quantity, regardless of when that energy is used
- Medical baseline (higher allowance) is available for qualifying medical conditions — if user mentions this, note they may have an elevated allowance

## Example Calculation

User in Territory X, Heat Source H, billing period March 16 – April 9 (25 days), all winter:
- Daily baseline = 14.6 kWh
- Monthly baseline = 14.6 × 25 = 365 kWh
- If usage = 178.3 kWh → entirely within Tier 1 (E-1) or eligible for full baseline credit (E-TOU-C)

---

## Region → Territory Code Lookup

Use this table to map a user's city/area answer to a territory code when they don't know their code. Territory boundaries are defined by climate zone, not county lines — when in doubt, default to **X**.

| User's Area | Territory Code | Notes |
|-------------|---------------|-------|
| San Jose, Santa Clara, Sunnyvale, Cupertino, Mountain View, Milpitas, Campbell | **X** | South Bay floor / valley |
| San Jose hills, Los Gatos, Saratoga (higher elevation) | **X** or **Q** | Check elevation; above ~1,500 ft → Q |
| San Francisco, Daly City, South San Francisco, Pacifica | **X** | Coastal, mild climate |
| Oakland, Berkeley, Alameda, Emeryville | **X** | East Bay coastal |
| Fremont, Union City, Hayward, San Leandro | **X** | East Bay |
| Livermore, Pleasanton, Dublin (inland East Bay) | **R** | Hotter inland valley |
| Concord, Walnut Creek, Pleasant Hill, Martinez | **R** | Contra Costa inland |
| Antioch, Brentwood, Pittsburg (far East Bay) | **R** | Hot summer inland |
| San Mateo, Redwood City, Palo Alto, Menlo Park | **X** | Peninsula, mild |
| Half Moon Bay, Pescadero (coastal Peninsula) | **X** | Coastal |
| Santa Cruz (below 1,500 ft), Capitola, Watsonville | **Q** | Coastal |
| Santa Cruz (above 1,500 ft), Ben Lomond, Boulder Creek | **Q** | Mountain |
| Monterey, Pacific Grove, Carmel, Salinas | **T** | Coastal/Monterey |
| Sacramento, Elk Grove, Roseville, Folsom | **S** | Hot Central Valley |
| Stockton, Lodi, Tracy | **P** | San Joaquin Valley |
| Modesto, Turlock, Merced | **P** | San Joaquin Valley |
| Fresno, Clovis, Visalia, Bakersfield | **Y** | Hot Central/South Valley |
| Napa, Sonoma, Santa Rosa, Petaluma | **V** | North Bay |
| Marin County (San Rafael, Novato) | **V** | North Bay |
| Eureka, Arcata, Humboldt Coast | **T** | Far North Coast |
| Redding, Chico, Red Bluff | **W** | Northern interior, hot summers |
| South Lake Tahoe, Truckee, High Sierra | **Z** | High elevation, cold winters |

**When user provides a city not listed:** Ask for their zip code and use the climate description (mild coastal = X, hot inland summer = R/S/P/Y, mountain = Z/Q) to make the best match. Always note the assumption and suggest the user verify on their PG&E bill under "Service Information → Baseline Territory."
