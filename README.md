# FRC 2026 Las Vegas Regional – Pre-Scouting & Auto Analysis

A collection of Jupyter Notebooks for **FRC pre-scouting** and **autonomous strategy analysis**.

- **Pre-scouting notebook** — Retrieves OPR and game-specific Component OPR statistics for every team registered at the 2026 Las Vegas Regional (`2026nvlv`) based on their most recent prior 2026 competition, and marks whether each team is already qualified for the 2026 World Championships.
- **Auto analysis notebooks (2024, 2025, 2026)** — Analyse how winning autonomous correlates with match wins, broken down by week, with percentile distributions and histograms.

## Data Source

[The Blue Alliance API v3](https://www.thebluealliance.com/apidocs/v3)

## Notebooks

| Notebook | Description |
|---|---|
| `frc_2026_lvr_prescouting.ipynb` | LVR team stats: OPR + Component OPR from each team's most recent prior event (weeks 1–5). Team 2375 is excluded (our team). |
| `frc_2026_auto_analysis.ipynb` | 2026 autonomous win impact analysis using `hubScore.autoPoints` (auto fuel points, excludes tower). |
| `frc_2025_auto_analysis.ipynb` | 2025 autonomous win impact analysis using `autoCoralPoints`, per-week breakdown through CMP. |
| `frc_2024_auto_analysis.ipynb` | 2024 autonomous win impact analysis using `autoPoints`, per-week breakdown through CMP. |

### Pre-Scouting Metrics (2026)

| Category | Fields |
|---|---|
| **Power Rating** | OPR |
| **Hub Fuel** | Hub Auto/Teleop/Endgame/Total/Transition Fuel Count |
| **Hub Shifts** | Hub Shift 1–4 Fuel Count, Hub First/Second Active Shift Count |
| **Hub Other** | Hub Uncounted |
| **Points** | Total Auto Points, Total Teleop Points, Endgame Tower Points, Total Tower Points, Total Points |
| **Achievements** | Energized Achieved, Supercharged Achieved |
| **Fouls** | Minor Foul Count, Major Foul Count, Foul Points |
| **Ranking** | RP |
| **Championship Status** | Qualified for World Championships (`Qualified` / `Not Qualified`) resolved via TBA advancement endpoints and status signals |

### Auto Analysis Outputs (per year)

Each auto analysis notebook produces:

1. **Auto win percentage table** — For matches where one alliance decisively won auto, what % of the time did that alliance also win the overall match? Broken down by week.
2. **Winning auto score distribution** — 10% percentile table + histograms (bin = 1 pt) per week, split into All Matches / Qualification / Playoff.
3. **Auto win margin distribution** — Same breakdown for the margin between the winning and losing alliance's auto scores.
4. **Overall charts** — Aggregate histograms across all weeks, split by match type.
5. **100th percentile match lookup** — Identifies the specific match(es) at the maximum values.

All histograms include vertical reference lines at the 10th, Q1, Median, Q3, and 90th percentiles.

## Prerequisites

- **Python 3.10+**
- **A TBA API key** — sign up at https://www.thebluealliance.com/account


## Setup

1. **Clone the repository:**
   ```bash
   git clone <repo-url>
   cd frc_2026_pre_scouting
   ```

2. **Create and activate a virtual environment:**

   **Linux / macOS:**
   ```bash
   python3 -m venv venv
   source venv/bin/activate
   ```

   **Windows (Command Prompt):**
   ```cmd
   python -m venv venv
   venv\Scripts\activate
   ```

   **Windows (PowerShell):**
   ```powershell
   python -m venv venv
   venv\Scripts\Activate.ps1
   ```

3. **Install dependencies:**
   ```bash
   pip install -r requirements.txt
   ```

4. **Create a `.env` file** in the project root with your TBA API key:

   **Linux / macOS:**
   ```bash
   echo 'TBA_API_KEY=your_tba_api_key_here' > .env
   ```

   **Windows (Command Prompt):**
   ```cmd
   echo TBA_API_KEY=your_tba_api_key_here > .env
   ```

   **Windows (PowerShell):**
   ```powershell
   'TBA_API_KEY=your_tba_api_key_here' | Set-Content -Path .env
   ```

   Or simply create a file named `.env` with a text editor and add:
   ```
   TBA_API_KEY=your_tba_api_key_here
   ```

   Replace the placeholder with your real key from https://www.thebluealliance.com/account

   > The `.env` file is listed in `.gitignore` and will **not** be committed.
   > The `python-dotenv` package (included in `requirements.txt`) reads this file automatically.

## Running the Notebooks

1. Make sure your virtual environment is activated:

   **Linux / macOS:**
   ```bash
   source venv/bin/activate
   ```

   **Windows (Command Prompt):**
   ```cmd
   venv\Scripts\activate
   ```

   **Windows (PowerShell):**
   ```powershell
   venv\Scripts\Activate.ps1
   ```

2. Open any notebook in **VS Code** (with the Jupyter extension) or launch Jupyter:
   ```bash
   jupyter notebook
   ```

3. **Run all cells in order** (top to bottom).

### Pre-Scouting Notebook

`frc_2026_lvr_prescouting.ipynb` will:
- Fetch every team registered for the 2026 Las Vegas Regional (excluding team 2375).
- Look up each team's most recent prior 2026 event (week 5 or earlier).
- Retrieve OPR and Component OPR statistics from that event.
- Resolve Worlds qualification using TBA advancement endpoints and status/event fallback checks.
- Display a ranked table and export results to `2026nvlv_prescouting_stats.csv`.

### Auto Analysis Notebooks

Each auto analysis notebook (`frc_2024_auto_analysis.ipynb`, `frc_2025_auto_analysis.ipynb`, `frc_2026_auto_analysis.ipynb`) will:
- Fetch all official events for that season (including CMP/Einstein for 2024 and 2025).
- Collect match data from every event.
- Compute auto win percentages, score distributions, and margin distributions.
- Generate per-week histograms split by match type (All / Qualification / Playoff).

> **Note:** The first run of each notebook makes many API calls and may take several minutes.

## Output

- **Pre-scouting:** Teams ranked by OPR (descending). Teams with no prior 2026 data show `N/A` and are pushed to the bottom. Results exported to `2026nvlv_prescouting_stats.csv`.
- **Auto analysis:** Tables and charts rendered inline in each notebook.

## Project Structure

```
frc_2026_lvr_prescouting.ipynb   # Pre-scouting notebook (2026 LVR)
frc_2026_auto_analysis.ipynb     # Auto analysis – 2026 (hubScore.autoPoints)
frc_2025_auto_analysis.ipynb     # Auto analysis – 2025 (autoCoralPoints)
frc_2024_auto_analysis.ipynb     # Auto analysis – 2024 (autoPoints)
requirements.txt                 # Python dependencies
project-requirements-doc.md      # Detailed project requirements
2026nvlv_prescouting_stats.csv   # Generated output (after running pre-scouting)
.env                             # TBA API key (not committed, git-ignored)
.gitignore                       # Git ignore rules
```
