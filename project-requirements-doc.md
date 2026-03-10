**Role:** Senior Data Scientist & FRC Strategy Analyst  
**Objective:** Build Python-based pre-scouting and autonomous strategy analysis tools for FRC.

---

## Part A – Pre-Scouting Tool (2026 Las Vegas Regional)

### **1. Scope & Logic**
* **Target Event:** 2026 Las Vegas Regional (`2026nvlv`).
* **Target Data:** All teams registered for the `2026nvlv` event, **excluding our team (2375)**.
* **Scouting Logic:** For each team competing at the Vegas Regional, look up their match results from all **previous** 2026 events (Weeks 1–5, where TBA `week < 5`). Week 6+ events are excluded.
* **Metric:** Retrieve OPR and the following game-specific match statistics (averaged per team across all played matches) from their most recent 2026 competition. If a team hasn't competed yet in 2026, or no data is available, mark all stat values as `N/A`:
  `OPR`, `Hub Auto Fuel Count`, `Hub Teleop Fuel Count`, `Hub Endgame Fuel Count`, `Hub Total Fuel Count`, `Hub Transition Fuel Count`, `Hub Shift 1 Fuel Count`, `Hub Shift 2 Fuel Count`, `Hub Shift 3 Fuel Count`, `Hub Shift 4 Fuel Count`, `Hub First Active Shift Count`, `Hub Second Active Shift Count`, `Hub Uncounted`, `totalAutoPoints`, `totalTeleopPoints`, `endGameTowerPoints`, `totalTowerPoints`, `energizedAchieved`, `superchargedAchieved`, `minorFoulCount`, `majorFoulCount`, `foulPoints`, `rp`, `totalPoints`

### **2. Technical Specifications**
* **Environment:** Jupyter Notebook (`.ipynb`).
* **Language:** Python 3.10+.
* **Data Source:** The Blue Alliance (TBA) API v3.
* **Key Libraries:** `requests`, `pandas`, `numpy`, `python-dotenv`.
* **API Key Management:** TBA API key loaded from a `.env` file via `python-dotenv`. The `.env` file is git-ignored.

### **3. Functional Requirements**
1. **API Integration:** Use the appropriate endpoint (e.g., `/event/{event_key}/teams` for TBA) to get the list of teams attending Vegas.
2. **Team Exclusion:** Exclude our own team (team 2375) from the scouting list via a configurable `OUR_TEAM` constant.
3. **Historical Look-up:** For each team, fetch their 2026 event history to find their most recently completed event prior to the Vegas Regional. Filter to events where the TBA `week` field is < `MAX_PRIOR_WEEK` (set to 5).
4. **Statistics Extraction:**
    - **OPR** – fetched from `/event/{event_key}/oprs`.
    - **Component OPRs** – fetched from `/event/{event_key}/coprs`. Fields: `Hub Auto Fuel Count`, `Hub Teleop Fuel Count`, `Hub Endgame Fuel Count`, `Hub Total Fuel Count`, `Hub Transition Fuel Count`, `Hub Shift 1 Fuel Count`, `Hub Shift 2 Fuel Count`, `Hub Shift 3 Fuel Count`, `Hub Shift 4 Fuel Count`, `Hub First Active Shift Count`, `Hub Second Active Shift Count`, `Hub Uncounted`, `totalAutoPoints`, `totalTeleopPoints`, `endGameTowerPoints`, `totalTowerPoints`, `energizedAchieved`, `superchargedAchieved`, `minorFoulCount`, `majorFoulCount`, `foulPoints`, `rp`, `totalPoints`.
    - **Note on API key naming:** Hub-related fields use space-separated display names in the COPRS response (e.g., `Hub Total Fuel Count`), while other fields use camelCase (e.g., `totalAutoPoints`).
    - **Efficiency:** Cache OPR and COPRS data per event key so teams sharing the same prior event only trigger one API call each.
5. **Data Processing:** Create a Pandas DataFrame containing:
    * `Team Number`
    * `Team Name`
    * `Latest Event Played Before Week 6` (Name of the previous regional)
   * `Qualified for World Championships` (`Qualified` or `Not Qualified`, resolved from TBA advancement endpoints and status/event fallback checks)
    * `OPR` (Offensive Power Rating)
    * All 23 match-averaged score breakdown fields listed above
6. **Output:** Display a clean table within the Jupyter Notebook where teams are ranked from highest OPR to lowest OPR. `N/A` values should be pushed to the bottom of the table. All stat columns should be included in the output. Results are also exported to `2026nvlv_prescouting_stats.csv`.

### **4. Edge Case Handling**
* **Rookie Teams / First-Timers:** If a team is attending Vegas as their first 2026 event, ensure the OPR column shows `N/A` and the script continues without crashing.
* **Rate Limiting:** Include basic error handling for API requests to manage rate limits and failed connections (retry with exponential backoff).

---

## Part B – Autonomous Win Impact Analysis

### **1. Scope & Logic**
* **Years Analysed:** 2024, 2025, and 2026 (each in its own notebook).
* **Target Data:** All official FRC matches across the entire season for each year, including regular weeks, DCMP, and CMP/Einstein.
* **Auto Score Field (per year):**
  - **2024:** `autoPoints` (top-level field in score breakdown)
  - **2025:** `autoCoralPoints` (top-level field in score breakdown)
  - **2026:** `hubScore.autoPoints` (nested inside the `hubScore` object; excludes tower points)
* **CMP Handling:** TBA returns `week=None` for Championship events. These are assigned the next sequential week number after the last regular-season week.
* **Week Labels:** Automatically detected from event types — regular weeks, DCMP weeks, and CMP weeks are labelled accordingly (e.g., "Week 1", "DCMP (Wk 5)", "CMP (Wk 7)").

### **2. Technical Specifications**
* **Environment:** Jupyter Notebooks (one per year).
* **Language:** Python 3.10+.
* **Data Source:** The Blue Alliance (TBA) API v3.
* **Key Libraries:** `requests`, `pandas`, `numpy`, `matplotlib`, `python-dotenv`.
* **API Key Management:** Same `.env` file as Part A.

### **3. Analysis Steps (per notebook)**
1. **Fetch Events:** Retrieve all official events for the year. Filter to `event_type` in `{0, 1, 2, 3, 4, 5, 6}`. Assign CMP events a computed week number.
2. **Discover Score Breakdown:** Print a sample match's full score breakdown JSON to confirm the correct field names.
3. **Fetch Match Data:** Iterate through all events, extracting per-match: event key, match key, match type (qual vs playoff), week, red/blue auto points, red/blue total scores, red/blue foul points, and auto winner (red/blue).
   - **Foul Adjustment:** The match winner is determined from *foul-adjusted* totals rather than raw scores. For each alliance: `adjusted_total = total_score − foulPoints` (where `foulPoints` are the points awarded to that alliance due to the opposing alliance's fouls). The alliance with the higher adjusted total is recorded as the winner.
   - **Tie Handling:** Actual ties and matches that become ties after foul adjustment are both excluded from the analysis entirely.
   - **Auto Tie Exclusion:** Matches where both alliances scored equal auto points are excluded at collection time. Only matches with a decisive auto winner are retained.
4. **Auto Win Percentage:** For all collected matches (auto ties and match ties already excluded), calculate the % of the time the alliance that won autonomous also won the overall match (using foul-adjusted totals). Produce a table broken down by week plus an overall row.
5. **Winning Auto Score Distribution:**
   - Percentile table (0%–100% at 10% intervals) by week + overall.
   - Per-week histograms (bin = 1 pt) with 3 columns per week: All Matches, Qualification, Playoff.
   - Overall histograms (3-panel: All / Qual / Playoff).
   - All histograms include vertical reference lines at 10th, Q1 (25th), Median (50th), Q3 (75th), and 90th percentiles.
6. **Auto Win Margin Distribution:**
   - Same percentile table and histogram structure as the winning score distribution, but for the margin between the winning and losing alliance's auto scores.
7. **100th Percentile Match Lookup:** Identify the specific match(es) at the maximum winning auto score and maximum auto margin, printing match key, event, week, alliance, and scores.

### **4. Histogram Specification**
* **Weekly charts:** Grid of `n_weeks` rows × 3 columns (All Matches / Qualification / Playoff).
* **Overall charts:** 1 row × 3 columns.
* **Bin width:** 1 point.
* **Reference lines:** 10th (orange, dash-dot), Q1 (purple, dashed), Median (black, dashed), Q3 (blue, dashed), 90th (green, dash-dot). Each labelled with the percentile name and value.
* **Colors:** All Matches = steelblue, Qualification = seagreen, Playoff = tomato.

### **5. Edge Case Handling**
* **Empty subsets:** If a week has no playoff matches, the subplot shows a "no data" title.
* **Missing score breakdown:** Matches without score breakdown data are skipped.
* **Foul-adjusted ties:** Matches where alliances are tied after subtracting foul points are excluded, even if the raw scores differ.
* **Auto ties:** Matches where both alliances scored equal auto points are excluded at data collection time.
* **Rate Limiting:** Same retry/backoff strategy as Part A.

---

## Part C – Project Infrastructure

### **1. Dependencies (`requirements.txt`)**
```
requests==2.32.5
pandas==3.0.1
numpy==2.4.2
python-dotenv==1.2.2
matplotlib
ipykernel==7.2.0
notebook==7.5.4
```

### **2. Environment Configuration**
* TBA API key stored in `.env` file (git-ignored).
* `python-dotenv` loads the key via `load_dotenv()` at the top of each notebook.
* Instructions for creating `.env` provided for Linux/macOS, Windows CMD, and Windows PowerShell.

### **3. Documentation**
* `README.md` — Setup instructions (venv, dependencies, .env), notebook descriptions, and usage for all platforms.
* `project-requirements-doc.md` — This document; detailed functional and technical requirements.