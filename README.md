# FRC 2026 Las Vegas Regional – Pre-Scouting Tool

A Jupyter Notebook-based pre-scouting tool for the **2026 FRC Las Vegas Regional** (`2026nvlv`). Retrieves OPR and game-specific Component OPR statistics for every registered team based on their most recent prior 2026 competition.

## Data Source

All data is fetched from [The Blue Alliance API v3](https://www.thebluealliance.com/apidocs/v3).

## Metrics Collected

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

## Setup

1. **Create a virtual environment and install dependencies:**
   ```bash
   python -m venv venv
   source venv/bin/activate
   pip install requests pandas numpy
   ```

2. **Get a TBA API key** at https://www.thebluealliance.com/account and set it in the notebook's configuration cell.

3. **Run the notebook** `frc_2026_lvr_prescouting.ipynb` — cells execute sequentially and produce a ranked table plus a CSV export (`2026nvlv_prescouting_stats.csv`).

## Output

Teams are ranked by OPR (descending). Teams with no prior 2026 competition data show `N/A` and are pushed to the bottom of the table. Results are also exported to `2026nvlv_prescouting_stats.csv`.

## Project Structure

```
frc_2026_lvr_prescouting.ipynb   # Main notebook
project-requirements-doc.md      # Detailed requirements
2026nvlv_prescouting_stats.csv   # Generated output
```
