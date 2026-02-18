

# DGPS Basemap & HMM Map-Matching Framework

This repository contains a specialized geospatial pipeline designed to transform raw, high-precision **DGPS (Differential Global Positioning System)** survey data into a topological road network and subsequently align noisy vehicle trajectories to that network using **Hidden Markov Models (HMM)**.

---

## 📂 Project Workflow & File Structure

The project is executed in three distinct phases: **Basemap Construction**, **Map-Matching Execution**, and **Validation**.

### Phase 1: High-Precision Basemap Building

* **`Script for base map building.ipynb`**: The entry point for raw survey data.
* **Input**: Raw DGPS Excel file (e.g., `Path wise sorted DGPS Kotturpuram.xlsx`).
* **Logic**: Converts Easting/Northing (UTM Zone 44N) into WGS84 coordinates, preserves survey metadata (`DES` column), and builds a directed graph with `from_id`, `to_id`, and native `bearing` calculations.
* **Outputs**: `base_map_nodes.shp` and `base_map_edges.shp`.
* **Data**: There are two data files with names, Dec6_Kotturpuram_naviconly_10.43 am trip.xls, etc.


### Phase 2: Map-Matching (MM) Algorithms

Three levels of algorithms are provided, ranging from basic snapping to advanced probabilistic modeling:

1. **`1) MM_Topological algo.ipynb`**:
* *Basic*: Snaps GPS points to the single nearest road segment. High risk of "road jumping" in dense areas.


2. **`2) MM_weight_DGPS.ipynb`**:
* *Intermediate*: Uses a composite score of **Spatial Distance + Heading Agreement** to select the best road.


3. **`3) MM_HMM_For_DGPS.ipynb`**:
* *Advanced*: The primary research algorithm. Uses a Hidden Markov Model and the Viterbi algorithm to find the most probable path across a sequence of points.



### Phase 3: Visualization & Validation

* **`Animation of GPS Coordinates.ipynb`**: Converts final output into an interactive HTML animation using `folium`. This allows playback of the "matched" path to verify algorithm performance.

---

## Phase 1: DGPS Data Requirements

The **Basemap Building** script requires a specific format to ensure topological integrity:

* **Mandatory Columns**:
* `Easting` & `Northing`: Raw metric coordinates from the DGPS station.
* `ID`: An incremental integer defining the physical sequence of the road survey.
* `DES`: Metadata descriptions (e.g., `E1GM`, `BASESESTN`) used to identify road types or landmarks.


* **Transformation**: The script automatically projects coordinates from **UTM Zone 44N (EPSG:32644)** to **WGS84 (EPSG:4326)** for compatibility with standard GPS data.

---

## Phase 2: Technical Analysis of the HMM Algorithm

The **HMM Map-Matching** is the core of this research, designed to overcome the limitations of standard GPS noise in urban canyons.

### How it Works:

1. **Emission Probability**: For every noisy GPS point, the algorithm identifies candidates within a **50m Search Radius**. It calculates the likelihood of the vehicle being on that road based on a Gaussian distribution of the distance.
2. **Transition Probability**: The algorithm evaluates the physical possibility of moving from the previous road to the current one. High weights are given to connected segments in the DGPS basemap; near-zero weights are given to "teleporting" between unconnected roads.
3. **Viterbi Decoding**: The algorithm builds a **Trellis Graph** of all possibilities and solves for the Maximum Likelihood Path. This ensures the vehicle trajectory remains smooth and topologically sound.

---

## Outputs & Visualization

### Algorithm Outputs:

The final processed file (e.g., `final_hmm_commercial_grade.csv`) includes:

* Original Latitude/Longitude.
* **`hmm_matched_lat` / `hmm_matched_lon**`: High-accuracy coordinates snapped to DGPS road segments.
* **`matched_segment_id`**: The exact DGPS edge ID, identifying specific road segments for congestion alerts.

### Visualization:

The animation script produces an HTML file with a play/pause interface, showing vehicle movement along the DGPS-defined path.


* **Dynamic Speed Estimation**: Future iterations can integrate `length_m` from the basemap with `datetime` stamps to calculate real-time segment speeds.
* **Orientation-Aware HMM**: Fusing `bearing` data from the basemap with vehicle heading will further improve accuracy at acute-angle intersections.

---
