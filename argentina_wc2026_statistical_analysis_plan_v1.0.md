# Statistical Analysis Plan v1.0 (locked for time-stamping) — Outline §7

This plan supersedes the earlier draft. All open decisions are now fixed (see the Decision log, §13). No confirmatory model may be run, and no specification below may be changed, after this document is time-stamped. Any change after registration must be logged as a deviation (§12).

## 0. Data freeze and disclosure of prior looks
- Analysis file: `argentina_wc2026_analysis_dataset.xlsx`, SHA-256 `5b93e88a87e73578150f0c1ffedfaba3f2e08d1cd9ebb7fb7eb7caa89d81aae6`, frozen 2026-10-22. 104 matches, 208 team-match rows, 48 teams, 68 logged VAR events (50 eligible for the nets).
- The tournament was complete before this plan was written. This is a time-stamped plan for a retrospective analysis, not an outcome-blind pre-registration.
- Prior looks (disclosed): during data audits the researcher and the assistants viewed per-team VAR net totals (Argentina had the highest summed net) and Argentina's individual VAR events. None of the tests below had been run. The statistics, thresholds and interpretation rules were fixed after those looks. Public controversy about Argentina's matches motivated the choice of focal team.
- Design facts taken from event counts and Elo only: Argentina has 6 eligible VAR events; the stronger team (by pre-match Elo) benefited in 25 of the 44 eligible events not involving Argentina (57%); Argentina was the stronger side in all 6 of its events.

## 1. Questions and hypotheses (all two-sided)
Focal team: Argentina, compared with the other 47 teams.
- **H1 (VDI):** Argentina's context-adjusted net of VAR-changed decisions equals its expectation given its strength.
- **H2 (SJI):** Argentina's cards-per-foul and fouls-awarded rates equal those of comparable teams, given match context and referee.
- **H3 (GDD):** Argentina's group opposition strength equals what the FIFA draw procedure gives a Pot-1 team by chance.
- **H4 (FPD):** Argentina's knockout-path opposition strength, given that it reached the final, equals what the bracket procedure gives by chance.
- **H5 (falsification):** the same pipeline applied to other prominent teams gives results as extreme as Argentina's.
Tier 1 = H1–H4. Tier 2 (reported, FDR q = 0.10): referee assignment, scheduling (rest days, travel, heat), added-time analysis. There is no Tier 3.

## 2. Units and eligibility
- Units: team-match (208), match (104), team (48), VAR event (68).
- A VAR event enters any net only if `used_in_nets = Yes` (confidence High or Medium, both teams identified). Excluded: 17 unidentified events, 1 Low-confidence event (M040 53'), and disputed events in the 3 `Conflict` matches.
- `var_log_status`: Complete (94), ESPN_only (7), Conflict (3). Primary analyses use all 104 matches.
- Language: never "referee bias". Use "decision disparity", "context-adjusted difference", "unusual officiating pattern". No causal or intent claims.

## 3. Estimands and primary tests

### 3.1 VDI (H1)
- Buckets: penalty (`penalty_var_net`), red card (`redcard_var_net`), other (`other_var_net` = goal + restart + card correction). Weights 0.50 / 0.30 / 0.20.
- Team value: VDI_t = (0.50·P_t + 0.30·R_t + 0.20·O_t) / (E_t/90), with P, R, O summed over the team's matches and E_t = 90 nominal minutes per match plus 30 per extra-time match. Report z-scores across the 48 teams descriptively.
- **Primary test (focal, context-adjusted).** Fit a baseline on eligible events NOT involving the focal team: Firth-penalized logistic regression, y_i = 1 if the benefiting team is the higher-Elo side; `logit P(y_i = 1) = α_bucket + δ·|ΔElo|/100`, where ΔElo is the pre-match Elo gap. For each focal event, π_i = P(y_i = 1) if the focal team is the stronger side, else 1 − P(y_i = 1). Statistic T = Σ w_b·s_i (s_i = +1 if the event favors the focal team, −1 if it harms it). Null: each focal event favors the focal team independently with probability π_i. Use 100,000 Monte Carlo draws, seed 2026. Two-sided p = P(|T* − E[T]| ≥ |T_obs − E[T]|). Report T_obs, E[T], T_obs − E[T] and the null 95% interval.
- The primary baseline uses pre-match Elo only. In-match xG, shots and possession are not used because they are partly consequences of VAR events (a VAR-awarded penalty adds xG for the beneficiary).
- Secondary: the unadjusted version (π_i = 0.5).
- **Rank and family-wise check.** Apply the same test to all 48 teams (baseline refitted leaving that team out). Report Argentina's rank on (T − E[T])/SD. Max-statistic p = P(max over the 48 teams of |z| ≥ Argentina's |z|), by simulating all eligible events jointly from a baseline fitted on all eligible events.
- **Stated power limit:** with 6 focal events and the stronger side benefiting about 57% of the time, the test cannot give a decisive p-value even if every focal event favored Argentina. H1 is reported as an estimate with an interval, not as a significance verdict.

### 3.2 SJI (H2)
Fitted on the 208 team-match rows with count mixed models: a random intercept for referee (and, in the fouls model, for the opponent) and a fixed focal-team indicator. Each team in turn is the focal team, with the other 47 forming the reference pool; team-level random intercepts are not used because they would compete with the focal effect. Family: negative binomial, unless the Pearson dispersion of the Poisson fit with Argentina as focal is below 1.1, in which case Poisson. Decide this once on the Argentina fit and apply it to all 48 focal fits. "Standardized Elo gap" = (team Elo − opponent Elo) divided by its standard deviation across the 208 rows.
- **Cards:** `yellow_cards`, offset log(`fouls_committed`); covariates: standardized Elo gap, `is_knockout`, `heat_index_c`. Favorable = lower rate.
- **Fouls awarded:** the opponent's `fouls_committed`; covariates: the team's `possession_pct` and `shots`, standardized Elo gap, `is_knockout`, `heat_index_c`, `days_rest` (with `is_first_match`); random intercept for the opponent. Favorable = more fouls awarded. xG is excluded (penalties inflate it).
- Report each focal effect as a rate ratio with a 95% CI. **SJI primary p-value** = min(1, 2·min(p_cards, p_fouls)).
- SJI composite (description only): 0.59·z(−β_cards) + 0.41·z(β_fouls), where β are each team's focal effects (log rate ratios) from the 48 focal fits and z is taken across the 48 teams. The weights are the original 0.50/0.35 renormalized; added time is out of the composite.
- **Added time (Tier 2):** regress announced second-half added time on `goals_h2`, `subs_h2`, `cards_h2`, `var_changes_h2` with a referee random intercept. Judge the residual's direction against the team's game state at 90:00 (`added_time_gamestate`).

### 3.3 GDD (H3)
- Opponent Strength Composite per team: OSC = 0.40·z(Elo) + 0.30·z(FIFA points) + 0.30·z(log `title_prob_devig`), z across the 48 teams.
- Null: 100,000 simulated full group draws. Pots from the `teams` sheet. Hosts fixed in their actual groups (Mexico A, Canada B, USA D). The other nine Pot-1 teams assigned at random to the remaining groups. Pots 2, 3, 4 assigned in sequence, one team per group per pot, with the confederation rule (no two teams from one confederation in a group, except UEFA up to two), using random sequential assignment with restart on dead ends. Bracket-path constraints for the top seeds are not simulated (documented simplification).
- **Validity check:** the actual draw (groups in the `teams` sheet) must satisfy all simulated constraints.
- GDD = (null mean of the mean group-opponent OSC − realized mean OSC) / null SD, positive = easier draw. Two-sided p = 2·min(F, 1 − F), F the null CDF at the realized value. Also report the same quantity for the other 11 Pot-1 teams.

### 3.4 FPD (H4)
- Inputs: FIFA's published Round-of-32 slot structure, the knockout bracket through the final, and the third-place allocation table (Annex C of the FIFA regulations, 495 combinations). Obtain them from the FIFA regulations and cross-check against a second source (e.g., the Wikipedia knockout-stage page).
- **Validation (mandatory):** applying the table and bracket to the actual group results must reproduce every actual Round-of-32 pairing and the whole actual bracket in the `matches` sheet. If it cannot, FPD is dropped from Tier 1 and reported descriptively.
- Simulation: 100,000 full tournaments, seed 2026, starting from the simulated group draw. Group matches: goals ~ Poisson with means μ_A = 2.7·We, μ_B = 2.7·(1 − We), where We = 1/(10^(−ΔElo/400) + 1) from fixed pre-tournament Elo. Group ranking by points, goal difference, goals scored, then random. Third-placed teams ranked the same way. Knockouts: team A advances with probability We.
- Statistic: among simulation runs in which Argentina plays 8 matches (reaches the final), the mean OSC of its first 8 opponents; the realized value is Argentina's actual mean. Report the percentile, a two-sided p, and the number of qualifying runs.

### 3.5 OAI and DDI
Descriptive only: OAI = (z_VDI + z_SJI)/2, DDI = (z_GDD + z_FPD)/2. Not tested.

## 4. Inference and interpretation
- α = 0.05, two-sided. Tier 1 p-values (H1–H4) adjusted with Holm. Tier 2 p-values adjusted with Benjamini–Hochberg, q = 0.10. Report effect sizes and 95% intervals throughout.
- Bayesian companion for VDI and SJI: partial-pooling hierarchical models (team effects; weakly informative normal(0, 1) priors on log rate ratios; 4 chains, report R-hat and effective sample size). Report the posterior probability that Argentina's effect is in the favorable direction and the 95% credible interval.
- **Interpretation rubric (mechanical).** An index is described as an "unusual officiating pattern" only if ALL hold: (a) Holm-adjusted focal p < 0.05; (b) Argentina is rank 1 or 2 of 48 (extreme tail) on that index; (c) the same sign and p < 0.10 in at least 80% of the sensitivity variants; (d) the pattern is not shared by a majority of the comparison teams in §7. Otherwise: "not distinguishable from normal variation", with the interval stated and the effect size the data could and could not have detected.

## 5. Sensitivity analyses (Outline §7.3)
S1 include Low-confidence and Conflict events; S2 assign directions to the 17 unidentified events at random (1,000 replications) plus best-case and worst-case assignments for Argentina; S3 exclude the 7 ESPN_only matches; S4 drop restart events from "other"; S5 unweighted counts (each event ±1); S6 alternative VDI weights (1/3 each; 0.6/0.3/0.1); S7 per-match instead of per-90 exposure; S8 exclude extra-time minutes; S9 add in-match possession and shots to the VDI baseline; S10 drop M001's unverified xG.

## 6. Robustness checks (Outline §7.4)
Poisson vs negative binomial; logistic vs ordinal event models; with and without in-match covariates; excluding the three hosts; excluding retractable-roof venues from heat covariates; leave-one-referee-out.

## 7. Falsification (Outline §7.6)
Run the VDI and SJI pipelines for all 48 teams and report the full distribution. Named comparison set (11 teams): the three hosts (Mexico, Canada, USA) and the 8 highest pre-tournament Elo teams other than Argentina (Spain, France, England, Brazil, Portugal, Colombia, Netherlands, Ecuador).

## 8. Missing data, outliers, model selection
No imputation of outcomes. Blank by design: `days_rest` and `distance_traveled_km` for first matches (use `is_first_match`). No outliers removed; extra-time matches stay in with exposure adjustment. Model forms are fixed here; no data-driven selection. Alternatives appear only as robustness checks.

## 9. Software and reproducibility
Count models: R (`glmmTMB`, negative binomial) preferred; Python/PyMC acceptable. State the engine used and do not switch after seeing results. Firth logistic: `logistf` (R) or `firthlogist` (Python). Versions pinned in a lock file; random seed 2026; one script regenerates every table and figure from the frozen file; archive the code together with both hashes.

## 10. Calibration checks (required before reporting results)
1. Re-run the VDI focal test 1,000 times on the real events with directions re-drawn from the fitted baseline; the rejection rate at α = 0.05 should be near or below 5%.
2. Apply the SJI models after randomly reassigning the focal label among teams (500 replications); the rejection rate should be near 5%.
3. Show the simulation constraint checks for the draw and the bracket (§3.3, §3.4).

## 11. Limitations declared in advance
Very few Argentina VAR events (6) and 50 eligible events overall; 17 unidentified events excluded; ESPN as the primary VAR instrument with a measured miss rate from the verification subset; 7 matches unverified; weather is outdoor ERA5 weather at retractable-roof venues; the team-level odds input is one pre-tournament market snapshot; one tournament; observational data, so context-adjusted associations, not causal effects, and no inference about intent.

## 12. Deviations
Any deviation from this plan (an error found in the plan, a data problem, a model that cannot be fit) must be reported in a deviations log with the date, what changed, why, and whether any result had been seen. Silent changes are not permitted. Analyses not in this plan are labeled exploratory and are never reported as confirmatory.

## 13. Decision log
- SJI composite: 0.59 cards / 0.41 fouls; added time moved to a Tier 2 analysis.
- FPD: kept in Tier 1, conditional on the mandatory validation in §3.4.
- Interpretation rubric: as in §4.
- Falsification set: as in §7.
- Changes from the original outline: variables cut (venue, player protection, injuries, media, psychological, historical baseline; referee career rates; ten performance variables); match-level odds replaced by a team-level title probability; added-time deviation replaced by raw inputs and a Tier 2 analysis; VDI categories refined (goal, restart, card correction) with `other` as their sum; VDI adjustment uses pre-match Elo rather than in-match performance.

## 14. Registration
Register this file and both hashes on OSF (Registries has a template for analyses of data that already exist) or commit them to a public repository, and record the date. State in the paper that the plan was time-stamped after collection and after the looks disclosed in §0.
