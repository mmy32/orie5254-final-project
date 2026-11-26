# Problem Definition — ORIE 5254 Final Project, Group 8

Sources: `research/ORIE 5254 Final Project Instructions.pdf` (assignment), `research/final_strike_selection_corrected.xlsx` rows 24–26 (Group 8 parameters), `research/opra-pillar-20260722.cbbo-1m.csv.zst` (data). Data checks in §5 were run on 2026-11-25.

**Due: 2026-12-07.** In-class work sessions: 2026-11-30.

---

## 1. The problem

Each group member owns one SPX option. For every one-minute snapshot on **2026-07-22**, rebuild the option's pricing inputs from that minute's quotes alone, then attribute the minute-to-minute change in the option's mid price to its Greeks:

```
ΔPrice ≈ Δ·ΔF + ½·Γ·ΔF² + Vega·Δσ + Θ·ΔDays + ρ·Δr + residual
```

The Greeks are taken at the start of each minute and the contributions are summed over the day. The task is graded on the **share of tick PnL the attribution explains**, measured against a personal required threshold (§3).

### 1.1 Pipeline, per minute *t*, using only that minute's quotes

| Step | Output | Method |
|---|---|---|
| 1. Box rate | `D(t)`, `r(t)` | A box on strikes K₁<K₂ pays K₂−K₁ at expiry, so its price is D·(K₂−K₁). Regress box price on width across strike pairs to get D; then r = −ln(D)/τ. |
| 2. Forward | `F(t)` | Put-call parity: C − P = D·(F − K) is linear in K. The x-intercept is F; the slope −D cross-checks step 1. |
| 3. Implied vol | `σ(t)` | Option mid ÷ D gives the undiscounted price; invert Black-76 with **py_lets_be_rational** (provided). No hand-rolled bisection. |
| 4. Greeks | Δ, Γ, Vega, Θ, ρ | Black-76 Greeks **with respect to F**, not spot. |

The fits in steps 1–2 are done **per expiry**: each member fits the box and forward on their own option's expiry.

### 1.2 Conventions the assignment fixes

- Mid = (bid + ask)/2 everywhere.
- Work off the fitted forward F. Spot is never observed in the file. Dividends and borrow are absorbed into F.
- Greeks are taken at the start of each minute and applied to that minute's changes ΔF, Δσ, Δr, ΔDays.

### 1.3 Metric

The PDF defines:

```
% PnL Explained = ( Σ|tickPnL| − Σ|tickPnLExplained| ) / Σ|tickPnL|
```

Read literally, this scores 0% when the attribution is perfect (explained equals actual). The intended metric is almost certainly the residual form:

```
% PnL Explained = 1 − Σ|tickPnL − tickPnLExplained| / Σ|tickPnL|
```

**Open question for the instructor:** confirm the residual form (§6, item 1).

---

## 2. Data

- **File:** `opra-pillar-20260722.cbbo-1m.csv.zst`. OPRA consolidated BBO for SPX options on one trading day, 2026-07-22, sampled every minute. About 95 MB compressed, about 4M rows.
- **Columns used:** `ts_recv` (minute timestamp, UTC), `bid_px_00`, `ask_px_00`, `symbol`.
- **OSI symbol:** `SPX   YYMMDD{C|P}KKKKKKKK`. Characters 7–12 are the expiry, character 13 is C or P, characters 14–21 are strike × 1000.
- **Observed coverage:** 450 distinct minutes, 13:31–21:00 UTC (09:31–17:00 ET). Each option appears only in minutes when it was quoted, so the number of ticks differs by contract.

---

## 3. Group 8 parameters

Valuation date: **2026-07-22**. TTE is in **calendar days** from the valuation date, as the spreadsheet computes it.

### 3.1 Member options

| Member | Strike K | Expiry | TTE (days) | Required % PnL explained |
|---|---:|---|---:|---:|
| Row 24 — *name TBD* | **20,000** | 2031-12-19 | 1,976 | **52.1%** |
| Row 25 — *name TBD* | **2,800** | 2027-04-16 | 268 | **42.5%** |
| Row 26 — *name TBD* | **3,400** | 2028-12-15 | 877 | **58.9%** |

Call or put is **not recorded in the sheet** and must be chosen and stated (§4).

### 3.2 Group and class averages

| | Strike | TTE (days) |
|---|---:|---:|
| Class average (35 options, 11 groups) | 7,485.71 | 276.20 |
| Group 8 average | 8,733.33 | 1,040.33 |

### 3.3 How the required % is built

```
Required = 0.25·(1 − |K̄_grp − K̄_cls| / max(K̄_grp, K̄_cls))     group vs class, strike
         + 0.25·(1 − |T̄_grp − T̄_cls| / max(T̄_grp, T̄_cls))     group vs class, TTE
         + 0.25·(1 − |K_me − K̄_grp| / max(K_me, K̄_grp))        me vs group, strike
         + 0.25·(1 − |T_me − T̄_grp| / max(T_me, T̄_grp))        me vs group, TTE
```

(√(x²) in the PDF is |x|.)

| Term | Row 24 (K 20,000) | Row 25 (K 2,800) | Row 26 (K 3,400) |
|---|---:|---:|---:|
| Group vs class, strike | 0.857 | 0.857 | 0.857 |
| Group vs class, TTE | 0.265 | 0.265 | 0.265 |
| Me vs group, strike | 0.437 | 0.321 | 0.389 |
| Me vs group, TTE | 0.526 | 0.258 | 0.843 |
| **Required (¼ × sum)** | **0.521** | **0.425** | **0.589** |

### 3.4 Grade thresholds in the sheet

The sheet multiplies the required % by 0.9, 0.8, 0.7 and 0.6 and labels the results A, B, C and D Threshold:

| Member | Required | A (×0.9) | B (×0.8) | C (×0.7) | D (×0.6) |
|---|---:|---:|---:|---:|---:|
| K 20,000 | 52.1% | 46.9% | 41.7% | 36.5% | 31.3% |
| K 2,800 | 42.5% | 38.3% | 34.0% | 29.8% | 25.5% |
| K 3,400 | 58.9% | 53.0% | 47.1% | 41.2% | 35.3% |

---

## 4. Contract choice: call or put

The SPX level on the day is roughly 7,500, based on the K = 2,800 call quoting near 4,745. Every Group 8 strike is far from the money. For each strike, the out-of-the-money leg has more quotes and a much tighter spread:

| Contract | Moneyness | Minutes quoted | Two-sided | Avg spread | Avg spread / mid |
|---|---|---:|---:|---:|---:|
| `SPX 270416P02800000` | OTM put | 426 / 450 | 426 | 0.35 | 4.2% |
| `SPX 270416C02800000` | deep ITM call | 324 / 450 | 324 | 20.77 | 0.4% |
| `SPX 281215P03400000` | OTM put | 406 / 450 | 406 | 2.68 | 4.6% |
| `SPX 281215C03400000` | deep ITM call | 158 / 450 | 156 | 49.55 | 1.1% |
| `SPX 311219P20000000` | deep ITM put | 404 / 450 | 403 | 300.15 | 3.6% |
| `SPX 311219C20000000` | far OTM call | 56 / 450 | 55 | 3.66 | 15.7% |

**Proposed choices:**
- **K 2,800 → put, Apr-2027.** Two-sided in every minute it appears, with a 35-cent spread.
- **K 3,400 → put, Dec-2028.** Two-sided in every minute it appears.
- **K 20,000 → open decision.** Both legs are weak:
  - The **call** has only 56 quoted minutes and a 16% relative spread, so the series is sparse and noisy.
  - The **put** has 404 quoted minutes, but it is about 12,500 points in the money with a 300-point spread. Its price is almost entirely intrinsic value, so σ is poorly identified and mid noise will feed into the residual.

  The instructions ask for "a liquid, reasonably near-the-money strike". Neither leg qualifies, so the write-up must justify whichever leg is chosen.

---

## 5. Data-specific risks

1. **Thin box and forward fits for the 2031 expiry.** Strikes with a two-sided call *and* put in the same minute:

   | Expiry | Avg pairs per minute | Min | Max |
   |---|---:|---:|---:|
   | 2027-04-16 | 206.6 | 53 | 214 |
   | 2028-12-15 | 110.7 | 28 | 135 |
   | 2031-12-19 | 29.5 | **3** | 35 |

   For the 2031 expiry, D and F can swing on a handful of quotes. Minute-to-minute noise in F and r then shows up as delta and rho PnL, or as residual. The fit needs a robustness rule, such as a minimum pair count or outlier rejection, and a plan for minutes that fail it.
2. **Long tenors make rho and the discount factor matter.** At τ ≈ 0.7–5.4 years, small errors in r move D noticeably. With F held fixed, Black-76 rho is −τ·Price.
3. **Some quotes fall outside regular trading hours.** Quotes run to 21:00 UTC (17:00 ET), past the 16:15 ET close. Decide whether to keep these minutes.
4. **Missing minutes.** If an option is not quoted in a minute, its PnL step spans more than one minute. Decide how to handle the gaps (§6).

---

## 6. Open decisions

| # | Decision | Notes |
|---|---|---|
| 1 | Form of the % PnL Explained metric | Confirm with the instructor (§1.3). |
| 2 | Call or put for K 20,000 | §4 |
| 3 | Member names for rows 24–26 | Not in the sheet. |
| 4 | τ convention for Black-76 | Calendar ACT/365 vs trading days. Root `SPX` monthlies are AM-settled at the Friday open, which affects τ at the minute level. ΔDays per minute must use the same units as Θ. |
| 5 | Box-spread pair selection and fit robustness | Which strike pairs, any weighting, minimum pair count. |
| 6 | Handling gaps and after-hours minutes | Carry forward, skip, or span the gap. |

---

## 7. Deliverables (due 2026-12-07)

1. **Excel file.** The input CSV rows for the option plus columns for: Total PnL, PnL Explained, each fit (D, r, F, plus fit diagnostics), σ, every Greek, and the per-Greek PnL contributions. The instructor will distribute a short example of the format.
2. **Time-series plot** of each PnL contribution (delta, gamma, vega, theta, rho, residual) against timestamp.
3. **Write-up of ½–1 page in trading language.** Which Greek drove the PnL and why, what F and σ did, and why the residual is what it is. Report each Greek's total, their sum, the actual open-to-close change, and the residual.
4. **Code** that runs top to bottom and reproduces the table and the figure.

## 8. Success criteria

- [ ] For each member's option, D, r, F and σ are computed at every quoted minute using only that minute's quotes.
- [ ] Greek contributions sum to the actual open-to-close change, with an explicit residual.
- [ ] % PnL Explained ≥ the member's required value: 52.1% for K 20,000, 42.5% for K 2,800, 58.9% for K 3,400.
- [ ] The Excel file, plot, write-up and code are all produced by one top-to-bottom run.
