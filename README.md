# Vanguard Digital Experience Analysis

## Table of Contents

- [Team Members](#team-members)
- [Executive Summary](#executive-summary)
- [Business Overview](#business-overview)
- [Business Questions](#business-questions)
- [Dataset](#dataset)
- [Data Preparation](#data-preparation)
- [Python Analysis](#python-analysis)
- [Key Findings](#key-findings)
- [Business Recommendations](#business-recommendations)
- [Limitations](#limitations)
- [Presentation](#presentation)
- [Tableau Dashboard](#tableau-dashboard)
- [Repository Structure](#repository-structure)
- [How to Run the Project](#how-to-run-the-project)

## Team Members

**Vanguard CX Analytics Team**

- Telma Guerreiro
- Erzsebet Fuzes
- Marta Atencia

## Executive Summary

This project evaluates Vanguard’s redesigned website through an A/B test, examining session completion, navigation friction, client age, and performance across different time periods.

The redesign increased session completion by **2.95 percentage points**, below the **5-point target**. High-friction journeys were more common in Test, with the largest increase in navigation friction at **step_1**. Clients aged 48 and above were less likely to record a successful visit, and the observed age-related performance gap remained with both interfaces.

We recommend investigating the difficulties at step_1, improving instructions and access to guidance, and monitoring whether these changes benefit different client groups. A gradual rollout would allow Vanguard to assess completion, friction, and support demand before expanding the redesign to all clients.

## Business Overview

Vanguard is a US-based investment management company. Its Customer Experience (CX) team wanted to understand whether an updated website would make an existing online process easier for clients to complete. The redesign combined a more intuitive interface with prompts shown during the process to help clients understand what to do next.

Vanguard ran an A/B test from **March 15 to June 20, 2017**. Clients were randomly assigned to one of two groups. **Control** used the original website experience, and **Test** used the redesigned interface. Both groups completed the same five-step process:

`start → step_1 → step_2 → step_3 → confirm`

As the CX Analytics team, our goal is to help Vanguard decide **whether the redesign should replace the original experience**. The project brief asks whether clients complete the process more often, how long it takes them, how they navigate the steps, and where they still get stuck. The event logs show each recorded action, so we reconstruct sessions before measuring these outcomes.

We also look at who experiences difficulty and when. The age analysis helps us identify clients who may benefit from clearer guidance, while the time-of-week analysis helps us understand whether the results are consistent across market hours, off-hours, and weekends.

## Business Questions

### H1 — Did the redesign improve completion and reduce friction?

- Did the redesigned website increase the session completion rate by at least 5%
- Did the redesign reduce the proportion of high-friction journeys?
- At which steps were clients more or less likely to skip, repeat, or move backward in Test compared with Control?
- Which steps should the CX team investigate before a wider rollout?

### H2 — How does age relate to client success?

- Among clients who succeeded at least once, did clients under 48 need fewer recorded visits to reach their first success than clients aged 48 or above?
- How likely was each age group to record at least one successful visit during the experiment?
- Did both age groups see higher success rates in Test? Was the observed age gap narrower or wider on the redesigned interface?

### H3 — Does performance vary by time of week?

- How is session traffic distributed across weekday market hours, weekday off-hours, and weekends?
- How do completion, session duration, and navigation friction compare between Test and Control within each period?
- When did the redesign show its largest observed completion gain, and what could that mean for the timing of a phased rollout?

## Dataset

We used the Vanguard experiment files supplied with the Ironhack project brief.

| Source file | Contents |
|---|---|
| `df_final_web_data_pt_1.txt` and `df_final_web_data_pt_2.txt` | Web events with client, visitor, and visit identifiers, process steps, and timestamps |
| `df_final_experiment_clients.txt` | Test and Control assignments by client |
| `df_final_demo.txt` | Client demographics, tenure, account count, account balance, calls, and logons |

The experiment assigned groups at the **client level** using `client_id`. We identified sessions using `client_id` and `visit_id` together. `visitor_id` identifies a browser or device associated with a client. A client can use several devices and contribute multiple sessions.

## Data Preparation

1. **Combine the web logs.** Stack the two web-event files and join the experiment assignments on `client_id`. Join those assignments to the client profiles for the demographic analysis.
2. **Clean and export the inputs.** Remove duplicate web events and records with missing fields. Remove client profile rows without an experiment assignment or client ID. Standardise column names and export `cleaned_web_data.csv` and `cleaned_client_data.csv`.
3. **Build the base session table.** Group the cleaned events by client and visit, count the recorded events per visit, and export one row per session to `sessions_data.csv`.
4. **Reconstruct the journeys.** Sort events chronologically within each visit. Record the start and end steps, measure elapsed time, and count skips, repeats, and backward moves. Export the enriched table as `sessions_data_master.csv`.
5. **Prepare the H1 export.** Add the H1 completion and friction measures to the session data and export `h1_sessions.csv` for Tableau.

## Python Analysis

| Notebook | Role |
|---|---|
| `data_cleanup.ipynb` | Combines and cleans the source files |
| `Cleaned_session_data.ipynb` | Creates the base session table |
| `demo_data_analysis.ipynb` | Profiles clients and compares Test and Control demographics, account characteristics, and prior engagement |
| `Session_data_analysis_day_1.ipynb` | Explores event timing, step activity, and where recorded sessions end |
| `journey_status.ipynb` | Explores confirmation events and navigation order |
| `master_session_data.ipynb` | Builds and exports the master session table |
| `H1_analysis.ipynb` | Tests the completion target and compares overall and step-level friction |
| `H2_complete.ipynb` | Compares client success and attempts to first success by age and interface |
| `H4_investor_pschology.ipynb` | Compares journey outcomes across time periods |

### H1 — Session completion and friction

A session qualified as completed if its first recorded step was `start`, its last recorded step was `confirm`, and its duration was **360 seconds or less**. The threshold was based on the **75th percentile of completion times**. We divided qualifying sessions by all recorded sessions in each experiment group.

We used a one-sided test at the 5% significance level and accounted for repeat visits from the same client when estimating uncertainty.

- **H₀** — The Test completion rate is at least 5 percentage points higher than the Control rate.
- **Hₐ** — The Test minus Control difference is below 5 percentage points.

We classified a session as **high-friction** if it met any of these conditions:

- It lasted **more than 300 seconds**, a threshold based on the mean session duration.
- Its first recorded step was something other than `start`.
- It included at least one skip, repeat, or backward move.

Each visit counted once in the overall friction rate, even if it met multiple conditions. A session could qualify as both completed and high-friction. For example, a visit that started at `start`, ended at `confirm`, and lasted 330 seconds met both definitions.

For step-level navigation friction, we calculated the share of visits that reached a step and made at least one irregular move from it. Each visit counted once per step. The `confirm` row captures moves recorded after a confirmation within the same visit.

### H2 — Age and attempts to first success

H2 uses the same session completion rule as H1. We ordered each client’s visits chronologically, counted each visit as one attempt, and identified the first visit that met the completion criteria.

We split clients at the sample median age of **48**. The younger group was under 48, and the older group was 48 or above. The **client success rate** includes all clients with recorded ages and measures the share with at least one successful visit. The **attempts-to-success comparison** includes clients who succeeded at least once.

We used a one-sided Welch’s t-test to assess whether younger successful clients needed fewer attempts on average. We also compared success rates and attempts by age group within each interface and explored account balances and recorded calls descriptively.

### H3 — Market hours, off-hours, and weekends

H3 uses the first recorded event in each visit to group sessions into **weekday market hours** (09:30 to 16:00), **weekday off-hours**, and **weekends**. The analysis assumes the timestamps are in US Eastern Time based on the observed hourly traffic pattern.

We compared completion, friction, and median session duration across the three periods. For the final comparison, H3 will use the same strict completion rule as H1 and H2. The current H3 output counted a confirmation anywhere in the visit toward strict completion. The segmented completion rates will be recalculated using `confirm` as the last recorded step, and the friction and backtracking figures will be checked against the same master session data.

## Key Findings

### H1 — Session completion and friction

### H1 — Session completion and friction

| Metric | Control | Test | Test minus Control |
|---|---:|---:|---:|
| Session completion rate | 28.72% | 31.67% | +2.95 percentage points |
| High-friction journey rate | 50.67% | 60.15% | +9.48 percentage points |
| Navigation friction from `step_1` | 12.08% | 22.00% | +9.92 percentage points |
| Navigation friction from `step_3` | 19.94% | 18.33% | −1.61 percentage points |
| Median Effective Time | 114 seconds | 112 seconds | −2 seconds |

Test recorded approximately **three additional completed sessions per 100 sessions**, but the improvement fell short of the **5-percentage-point target**. The one-sided test accounted for multiple visits from the same client and gave **p = 8.527 × 10⁻⁹**, supporting a completion-rate difference below the target.

Higher completion came alongside more high-friction journeys. **60.15% of Test sessions** met at least one friction criterion, compared with **50.67% in Control**. Backward moves showed the largest descriptive increase (**+6.57 percentage points**), followed by repeats (**+4.36**) and skips (**+0.94**).

The largest step-level navigation gap occurred at **`step_1`**, where **22.00% of Test visits** experienced friction compared with **12.08% of Control visits**. This makes it the clearest starting point for investigating navigation difficulties. **`step_3`** was the only step with lower observed navigation friction in Test.

Effective Time, calculated by summing intervals associated with forward moves of exactly one step, was slightly lower in Test. Its median decreased by **two seconds**, while its mean decreased from **171.18 to 163.27 seconds**. Because this comparison includes incomplete sessions, lower Effective Time may reflect faster progression or fewer steps completed. These timing and friction comparisons are descriptive.

Together, the findings support refining the redesign, particularly at `step_1`, and monitoring completion alongside friction before expanding the rollout.

### H2 — Success and attempts by age

| Metric | Under 48 | Age 48 or above |
|---|---:|---:|
| Clients with at least one successful visit | 48.97% | 34.35% |
| Mean attempts among successful clients | 1.149 | 1.250 |
| Median attempts among successful clients | 1 | 1 |

Younger clients were **14.62 percentage points** more likely to record a successful visit. Among successful clients, they needed about **0.10 fewer attempts** on average. This difference was statistically significant overall (**p = 1.36 × 10⁻³²**) and within both Test and Control.

Both age groups had higher success rates and fewer average attempts in Test. However, the observed age gap in success rates widened from **14.04 percentage points in Control** to **15.04 in Test**. Whether the redesign changed this gap was not tested directly.

Among successful clients, median account balances were generally higher for those requiring more attempts, suggesting that Vanguard could investigate whether these clients would benefit from additional digital assistance.

Older clients recorded slightly more calls on average, although the median was **three across all groups**. These historical calls cannot be directly linked to difficulties during the digital process.

### H3 — Performance by time period

### H3 — Performance by time period

| Time period | Sessions | Share of sessions |
|---|---:|---:|
| Weekday market hours | 28,530 | 41.08% |
| Weekday off-hours | 26,069 | 37.54% |
| Weekend | 14,848 | 21.38% |

The redesign’s largest observed completion gains occurred during **weekday market hours**, the busiest period. Strict completion increased from **31.48% to 35.22%**, a difference of **3.74 percentage points**, while overall completion increased from **53.10% to 62.14%**, a difference of **9.04 points**. The reported share of sessions with backward moves fell from **1.57% to 0.40%**, a relative reduction of approximately **75%**.

Test also recorded higher completion during weekday off-hours (**+2.48 percentage points strict and +8.81 points overall**) and weekends (**+2.28 points strict and +7.73 points overall**). Reported backward-move rates decreased by approximately **47%** and **41%**, respectively. Median session duration increased most on weekends (**15 seconds**, compared with **6 seconds** during market hours). Longer durations could reflect more browsing, more progress, or other differences in session behavior.

Composite friction was higher in Test across all periods (**+6.31 to +7.63 percentage points**). Visits beginning at a later step were also more common, increasing from approximately **4% to 10.43%–12.07%**. These visits may include resumed journeys, although confirming that explanation requires linking them to earlier incomplete visits.

Average and median ages were similar between Test and Control within each period, providing reassurance about age balance. These descriptive findings support considering a phased rollout, beginning during lower-traffic periods to monitor stability and journey performance before expanding to market hours.

## Business Recommendations

1. **Refine the redesign before expanding the rollout.** Investigate the steps contributing most to navigation friction and assess targeted improvements through usability research, performance monitoring, and controlled testing where useful. Agree on acceptable completion and friction rates before expanding access.

2. **Make step_1 easier to understand and navigate.** Use transition heatmaps and usability sessions to identify where clients hesitate, repeat actions, or move backward. Provide clearer instructions, explain what information is required, and make the next action and access to help easy to find. Test when guidance is most useful so additional prompts support progress without overwhelming clients.

3. **Make it easier for clients to reach their first success.** During the experiment, **34.35% of clients aged 48 and above** recorded at least one successful visit, compared with **48.97% of clients under 48**. The median among clients who succeeded was one attempt in both groups. Include clients aged 48 and above in usability testing, and test clearer instructions, a visible progress indicator, or optional help after a repeated step. Separately, check whether visits that begin midway through the process follow an earlier incomplete visit. If that pattern reflects clients returning to finish, test a “Continue where you left off” option.

4. **Measure whether better guidance reduces support costs.** Clients aged 48 and above had slightly more historical calls to Vanguard in both groups. In the next A/B test, capture support contacts related to the online process and track the number of contacts per 100 clients, average handling time, and cost per contact. Use those measures alongside completion rates to see whether clearer instructions and easier access to help improve first-time success and reduce the cost of supporting clients.

5. **Use a phased rollout if the follow-up test meets the success criteria.** Start with a limited release during a lower-traffic period to check stability and confirm that completion and friction are tracking as expected. Then validate the experience during weekday market hours, which accounted for **41.08% of recorded sessions** under the Eastern Time assumption. Review both metrics before expanding to all clients.

6. **Investigate difficulties among clients with higher account balances.** Explore whether personalised guidance could improve their digital experience.

7. **Monitor performance by age group.** Track success rates, first-attempt success, average attempts among successful clients, and support requests to assess whether future improvements benefit different customer segments.

## Limitations

- **Reasons for navigation patterns.** Event logs show recorded actions but cannot explain why clients repeat steps, move backward, or leave. These behaviors may reflect difficulty, intentional navigation, or tracking issues.
- **Time measurements and thresholds.** Completion and friction thresholds were based on the available data. Conclusions may change under different thresholds. Time between events can include inactivity, so Effective Time estimates time associated with forward progression.
- **Session and client outcomes.** Session completion rates give more weight to clients who make more visits. They answer a different question from the percentage of clients who succeed at least once.
- **Attempts to success.** Average attempts include only clients who succeeded during the observation period. Earlier attempts and successes after the experiment ended are unobserved.
- **Age groups.** The age split at 48 reflects this dataset’s median. It simplifies a continuous relationship and may hide differences within each group.
- **Account balances and historical calls.** Vanguard balances may represent only part of a client’s wealth. Calls from the previous six months cannot be directly linked to the online process or used to establish savings from the redesign.
- **Timezone assumption.** The time-of-week analysis assumes US Eastern Time, which could not be confirmed. Its market-hours categories also use weekday opening times without accounting for market holidays.
- **Statistical coverage.** The friction comparisons and changes in the age gap are descriptive. Separate tests within groups do not establish that the redesign’s effect differs between age groups or time periods.
- **Causal interpretation and generalisation.** Associations between client characteristics and performance do not establish causation. The findings describe clients and behavior recorded during the 2017 experiment, so their relevance to other populations or current website use would need assessment.

## Presentation

The project presentation is available in the `slides/` folder. It summarises the A/B test, highlights the results from H1, H2, and H3 using graphs from the analysis notebooks, and presents the key takeaways and recommendations for Vanguard’s Customer Experience team.

Tableau visualisations of client demographics and other exploratory findings are available in the `figures/` folder.

## Tableau dashboard

Link to Tableau dashboard: https://public.tableau.com/app/profile/telma.guerreiro/viz/ExperimentOverview/ExperimentOverview

## Repository Structure

```text
vanguard-project/
├── data/
│   ├── raw/
│   │   ├── df_final_web_data_pt_1.txt      # Original web event log
│   │   ├── df_final_web_data_pt_2.txt      # Original web event log
│   │   ├── df_final_experiment_clients.txt # Original Test & Control assignments
│   │   └── df_final_demo.txt               # Original client demographics
│   └── clean/
│       ├── cleaned_web_data.csv            # Cleaned web events
│       ├── cleaned_client_data.csv         # Cleaned client data
│       ├── sessions_data.csv               # Base session table
│       └── sessions_data_master.csv        # Combined session data
├── notebooks/
│   ├── data_cleanup.ipynb                  # Cleans & combines the source files
│   ├── Cleaned_session_data.ipynb          # Creates the base session table
│   ├── demo_data_analysis.ipynb            # Explores client demographics
│   ├── Session_data_analysis_day_1.ipynb   # Explores timing & session activity
│   ├── journey_status.ipynb                # Checks confirmations and navigation order
│   ├── master_session_data.ipynb           # Builds the master session table
│   ├── H1_analysis.ipynb                   # Analyses H1
│   ├── H2_complete.ipynb                   # Analyses H2
│   └── H3_investor_psychology.ipynb        # Analyses H3
├── figures/                                # Tableau charts
├── slides/                                 # Project presentation
├── config.yaml                             # Project configuration
├── pyproject.toml                          # Python project dependencies
└── README.md                               # Project overview and instructions
```

## How to Run the Project

1. Clone the repository and open the `vanguard-project/` folder in Jupyter Notebook.
2. To reproduce the analysis using the prepared data in `data/clean/`, open the following notebooks in `notebooks/` and run all cells.

   | Notebook | Analysis |
   |---|---|
   | `H1_analysis.ipynb` | Session completion, navigation friction, and the five-percentage-point completion target |
   | `H2_complete.ipynb` | Client success and attempts to first success by age group |
   | `H3_investor_psychology.ipynb` | Session outcomes during market hours, off-hours, and weekends |

3. To rebuild the prepared data from the original Vanguard files in `data/raw/`, run `data_cleanup.ipynb`, `Cleaned_session_data.ipynb`, and `master_session_data.ipynb` in that order before running the analysis notebooks.