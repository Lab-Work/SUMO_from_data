## Creating SUMO simulation scenarios with real-world traffic volume data

This project provides tools to create SUMO (Simulation of Urban MObility) traffic scenarios using real-world traffic volume data. It processes traffic count data and generates SUMO-compatible simulation files. This is a project associated with ICCPS 2025 Poster submission. If you use the code and/or data in this project, please cite the following paper:

@inproceedings{zhang20251000daysim,
  title={1000DaySim: Open-Source Traffic Simulation With Real Data Over Long Time Horizons},
  author={Zhang, Zhiyao and Zhang, Yuhang and Qui{\~n}ones-Grueiro, Marcos and Barbour, William and Biswas, Gautam and Work, Daniel},
  booktitle={Proceedings of the ACM/IEEE 16th International Conference on Cyber-Physical Systems (with CPS-IoT Week 2025)},
  pages={1--2},
  year={2025}
}

## Project Structure

```
.
├── data/
│   └── sampled_intersections_volume_2022-2025.csv  # Traffic turning movement counts (TMC)
├── SUMO_files/
│   ├── SR1-3.net.xml          # Road network definition
│   ├── SR1-3.rou.xml          # Base routes definitions
│   ├── SR1-3_timing.add.xml   # Traffic signal timing and plan (actuated type)
│   ├── SR1-3.sumocfg          # Simulation configuration
│   ├── SR1-3-NEMA.net.xml     # Road network definition for NEMA-standard signal control
│   ├── SR1-3_timing-NEMA.add.xml   # Traffic signal timing and plan (NEMA type)
│   ├── SR1-3-NEMA.sumocfg     # Simulation configuration for NEMA-standard signal control
│   ├── SR1-3_volume.xml        # Volume data converted from TMC csv file
│   ├── SR1-3_volume_trips.xml  # Vehicle trips generated from volume and routes by routeSampler.py
│   └── routeSampler.py        # Route sampling utility function
├── csv_to_volume_xml.py       # Data conversion script. Convert from TMC csv file to SUMO volume and trips files during selected time period
├── daily_vehicle_count.py     # Daily vehicle count visualization
├── TOD_volume.py              # Time of day volume visualization
└── requirements.txt           # Python dependencies
```
* the two .net.xml files are identical except that preloaded signal type is NEMA for SR1-3-NEMA.net.xml.

## Data Processing Pipeline

1. **Data Conversion**: `csv_to_volume_xml.py` processes the raw data and generates SUMO volume and trips files. Use command line arguments to run the script:

Input:
```bash
python csv_to_volume_xml.py \
  --csv_input data/sampled_intersections_volume_2022-2025.csv \ # TMC csv file
  --route_file SUMO_files/SR1-3.rou.xml \ # SUMO input route file
  --output SUMO_files/SR1-3_volume.xml \ # SUMO output volume file
  --start "2024-01-01 00:00:00-05:00" \ # Start date and time
  --end "2024-01-01 23:59:59-05:00" \ # End date and time
```
*Note: We recommend the time period to be less than 1 week in each scenario. The processing time and simulation speed will be affected as the time period increases.*

Output:
```bash
SUMO_files/SR1-3_volume.xml # SUMO volume file
SUMO_files/SR1-3_volume_trips.xml # SUMO trips file
```
Besides, following configuration files are updated with the new volume and trips data:
```bash
SUMO_files/SR1-3.sumocfg  # actuated signal configuration
SUMO_files/SR1-3-NEMA.sumocfg  # NEMA-standard signal configuration
```

2. **Route Sampling**: The script automatically calls `routeSampler.py` to generate realistic vehicle trips based on the volume data. The route sampler:
- Matches traffic counts from real data
- Preserves turn ratios at intersections
- Generates time-distributed vehicle trips

## Requirements

- Python 3.10+ (tested in Python 3.12.4)
- SUMO 1.20.0
- Python packages listed in `requirements.txt`

Install dependencies:
```bash
pip install -r requirements.txt
```

## Running the Simulation

1. Process the volume data:
```bash
python csv_to_volume_xml.py [arguments]
```

2. Run SUMO with either actuated or NEMA configuration:
```bash
# actuated configuration
sumo-gui -c SUMO_files/SR1-3.sumocfg

# NEMA-standard configuration
sumo-gui -c SUMO_files/SR1-3-NEMA.sumocfg
```

## Configuration

The simulation can be configured through either `SR1-3.sumocfg` or `SR1-3-NEMA.sumocfg` (for NEMA-standard signal control), which specify:

### Input Files
- `SR1-3.net.xml` or `SR1-3-NEMA.net.xml`: Road network definition
- `SR1-3_volume_trips.xml`: Generated vehicle trips based on volume data. This is automatically updated by `csv_to_volume_xml.py` after new trips file is generated.
- `SR1-3_timing.add.xml`: Traffic signal timing plans

### Output Files
- `SR1-3-output-stats.xml` or `SR1-3-NEMA-output-stats.xml`: Traffic performance metrics including:
  - Vehicle counts
  - Average speed
  - Wait time and travel time
  - Departure delay

## References
For SUMO traffic signal types, please refer to:
https://sumo.dlr.de/docs/Simulation/Traffic_Lights.html
https://sumo.dlr.de/docs/Simulation/NEMA.html

For more information about route sampling, please refer to:
https://sumo.dlr.de/docs/Tools/Turns.html
