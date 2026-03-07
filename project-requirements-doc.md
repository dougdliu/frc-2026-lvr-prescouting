**Role:** Senior Data Scientist & FRC Strategy Analyst  
**Objective:** Build a Python-based pre-scouting tool for the 2026 FRC Las Vegas Regional.

### **1. Scope & Logic**
* **Target Event:** 2026 Las Vegas Regional (`2026nvlv`).
* **Target Data:** All teams registered for the `2026nvlv` event.
* **Scouting Logic:** For each team competing at the Vegas Regional, look up their match results from all **previous** 2026 events (Weeks 1–5). 
* **Metric:** Retrieve OPR and the following game-specific match statistics (averaged per team across all played matches) from their most recent 2026 competition. If a team hasn't competed yet in 2026, or no data is available, mark all stat values as `N/A`:
  `OPR`, `Hub Auto Fuel Count`, `Hub Teleop Fuel Count`, `Hub Endgame Fuel Count`, `Hub Total Fuel Count`, `Hub Transition Fuel Count`, `Hub Shift 1 Fuel Count`, `Hub Shift 2 Fuel Count`, `Hub Shift 3 Fuel Count`, `Hub Shift 4 Fuel Count`, `Hub First Active Shift Count`, `Hub Second Active Shift Count`, `Hub Uncounted`, `totalAutoPoints`, `totalTeleopPoints`, `endGameTowerPoints`, `totalTowerPoints`, `energizedAchieved`, `superchargedAchieved`, `minorFoulCount`, `majorFoulCount`, `foulPoints`, `rp`, `totalPoints`

### **2. Technical Specifications**
* **Environment:** Jupyter Notebook (`.ipynb`).
* **Language:** Python 3.14.
* **Data Source:** The Blue Alliance (TBA) API v3 or the FIRST API. 
* **Key Libraries:** `requests`, `pandas`, `numpy`.

### **3. Functional Requirements**
1. **API Integration:** Use the appropriate endpoint (e.g., `/event/{event_key}/teams` for TBA) to get the list of teams attending Vegas.
2. **Historical Look-up:** For each team, fetch their 2026 event history to find their most recently completed event prior to the Vegas Regional.
3. **Statistics Extraction:**
    - **OPR** – fetched from `/event/{event_key}/oprs`.
    - **Component OPRs** – fetched from `/event/{event_key}/coprs`. Fields: `Hub Auto Fuel Count`, `Hub Teleop Fuel Count`, `Hub Endgame Fuel Count`, `Hub Total Fuel Count`, `Hub Transition Fuel Count`, `Hub Shift 1 Fuel Count`, `Hub Shift 2 Fuel Count`, `Hub Shift 3 Fuel Count`, `Hub Shift 4 Fuel Count`, `Hub First Active Shift Count`, `Hub Second Active Shift Count`, `Hub Uncounted`, `totalAutoPoints`, `totalTeleopPoints`, `endGameTowerPoints`, `totalTowerPoints`, `energizedAchieved`, `superchargedAchieved`, `minorFoulCount`, `majorFoulCount`, `foulPoints`, `rp`, `totalPoints`.
    - **Note on API key naming:** Hub-related fields use space-separated display names in the COPRS response (e.g., `Hub Total Fuel Count`), while other fields use camelCase (e.g., `totalAutoPoints`).
    - **Efficiency:** Cache OPR and COPRS data per event key so teams sharing the same prior event only trigger one API call each.
4. **Data Processing:** Create a Pandas DataFrame containing:
    * `Team Number`
    * `Team Name`
    * `Latest Event Played` (Name of the previous regional)
    * `OPR` (Offensive Power Rating)
    * All 23 match-averaged score breakdown fields listed above
5. **Output:** Display a clean table within the Jupyter Notebook where teams are ranked from highest OPR to lowest OPR. `N/A` values should be pushed to the bottom of the table. All stat columns should be included in the output.

### **4. Edge Case Handling**
* **Rookie Teams / First-Timers:** If a team is attending Vegas as their first 2026 event, ensure the OPR column shows `N/A` and the script continues without crashing.
* **Rate Limiting:** Include basic error handling for API requests to manage rate limits and failed connections.