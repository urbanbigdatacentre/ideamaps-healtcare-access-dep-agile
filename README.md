---
title: Maternal Care Access Deprivation — Kano, Nigeria
authors: [Diego Pajarito Grajales - Diego.PajaritoGrajales@glasgow.ac.uk, Xingyi Du - xingyi.du@glasgow.ac.uk]
category: Our Data
tags: [Maternal health care, Emergency obstetric care, Accessibility]  
---

## Overview

Maternal care access deprivation dataset is a dataset that depicts how difficult is for women in slums and other deprived areas to **access emergency maternal care**. **Mortality** among pregnant women and newborns strongly affects vulnerable communities and has been **prioritised by the communities** participating in the IDEAMAPS project. The team considered community priorities and analysed the different phases of maternity: antenatal, intrapartum or delivery, and postnatal, then decided to **focus on intrapartum or delivery phase as being the most critical**. The intertwined relationship between maternal health care and urban deprivation has been documented and described in the literature [(Abascal et al., 2022)](https://doi.org/10.1016/j.compenvurbsys.2022.101770). Therefore, analysing such conditions implies gathering data and analysing how vulnerable communities relate to emergency maternal care (EmOC) in the cities of Kano and Lagos in Nigeria. To do so, the team built a model that stands on factors such as offer, demmand and access.

## Definitions of Deprivation Levels

The dataset relates the offer of emergency obstetric care (i.e., health care facilities offering EmOC), their service levels (i.e., comprehensive or basic care) and relative costs (i.e., private facilities charging higher relative costs that public facilities); the demmand represented as female population in childbearing age; and the phisical accesibility represented in travel time (i.e., travel times that also include delays of waiting for a vehicle, high traffic and difficult road conditions). Together, these values serve to estimate deprivation access based on the enhanced two-step floating catchment area (i.e., access deprivation as the inverse of accessibility)  —> **Low, Medium, or High.**

### Low
<blockquote > My neighbourhood offers a wide range of places to provide maternal care and handle obstetric emergencies. The places are nearby with a good mix of public and affordable options as well as private and more expensive ones for those who prefer them. </blockquote>

<img src="image-examples/emergency-maternal-care-access-deprivation-low.png" alt="example-low">

### Medium
<blockquote> There are a couple of places offering maternal care in my neighbourhood, some of which can handle emergencies. Women face a mixed scenario with some options to access suitable and affordable obstetric care, and others requiring either to travel long distances, pay relatively high fees or have private insurance to access the required obstetric care. </blockquote>

<img src="image-examples/emergency-maternal-care-access-deprivation-medium.png" alt="example-medium">

### High
<blockquote > It is difficult to find adequate maternal care in my neighbourhood, especially during an emergency. Women definitely require a long trip (i.e., more than 30min by car) to reach a suitable and affordable facility offering obstetric care.</blockquote>

<img src="image-examples/emergency-maternal-care-access-deprivation-high.png" alt="example-high">

## Access deprivation levels based on the accessibility score provided by the E2SFCA method

We use the enhanced two-step floating catchment area (E2SFCA) method to estimate accessibility to emergency maternal care. The E2SFCA method combines information about the supply of healthcare services (i.e., health care facilities offering emergency obstetric care) and the demand for these services. The result is a numeric value called the accessibility score ranging between 0 and 1. To define the deprivation levels (i.e., Low, Medium, High), we apply city-specific thresholds to classify the accessibility scores into three categories. These thresholds are the result city-specific analysis detailed in the next section.

---

## 🛠️ Setup

### 1. Clone the repository

```bash
git clone https://github.com/urbanbigdatacentre/ideamaps-healthcare-paper-analysis.git
cd ideamaps-healthcare-paper-analysis/data_preparation
```

### 2. Create a virtual environment

```bash
python -m venv .venv
source .venv/bin/activate 
```

### 3. Install dependencies

```bash
pip install -r requirements.txt
```

### 4. Set up the Jupyter kernel (for VS Code or Jupyter Notebooks)

```bash
pip install -U ipykernel
python -m ipykernel install --user --name=.venv
```

---

## 📊 Data Sources

- [IDEAMAPS study areas](https://github.com/urbanbigdatacentre/ideamaps-models/tree/dev/docs/study-areas)
- [Women of childbearing age population counts from WorldPop](https://hub.worldpop.org/geodata/summary?id=18447)
- [Open buildings from Overture Map Foundation](https://overturemaps.org/)
- [Road network data from OpenStreetMap via the Open Route Service API](https://openrouteservice.org/)

---

## 🚀 Running the Model

Execute notebooks in the following order:

`data_preparation.ipynb`
Loads and preprocesses raw data, generates population centroids, and prepares inputs for the OD matrix computation.

`E2SFCA_Analysis.ipynb`
Runs the full E2SFCA model. Key steps:

**Step 1 — Aggregate population data**  
Disaggregate WorldPop 1km raster to 100m × 100m using Google Building Footprints as weights.

**Step 2 — Compute the OD Matrix**  
Calculate travel times and distances from each grid centroid to EmOC facilities using the [OpenRouteService (ORS) Matrix API](https://openrouteservice.org/).

- **Option A: Public ORS API**
  ```bash
  OPENROUTESERVICE_API_KEY = 'your_api_key'
  api_key = os.getenv('OPENROUTESERVICE_API_KEY')
  client = openrouteservice.Client(key=api_key)
  ```

- **Option B: Local ORS Instance (Docker)**  
  See [ORS documentation](https://github.com/GIScience/openrouteservice/tree/main) for setup instructions.

**Step 3 — Apply the E2SFCA Method**  
- Define catchment areas per facility using a travel-time threshold
- Compute supply-to-demand ratios for each facility
- Aggregate accessibility scores for each demand grid cell

`comparison_analysis.ipynb`
Compares accessibility outcomes across deprivation categories and produces summary statistics.

---

## Methodology: Enhanced Two-Step Floating Catchment Area (E2SFCA)

The E2SFCA method improves on the traditional 2SFCA by incorporating a **distance decay function**, so that closer populations have a stronger impact on accessibility scores.

**Step 1 — Supply-to-demand ratio at each facility $i$:**

$$R_i = \frac{S_i}{\sum_{k \in \{d_{ik} \leq d_0\}} W(d_{ik}) \, P_k}$$

**Step 2 — Accessibility score at each demand location $j$:**

$$A_j = \sum_{i \in \{d_{ij} \leq d_0\}} W(d_{ij}) \, R_i$$

| Symbol | Description |
|---|---|
| $A_j$ | Accessibility score for demand location $j$ |
| $R_i$ | Supply-to-demand ratio at facility $i$ |
| $S_i$ | Facility capacity/weight at location $i$ |
| $P_k$ | Female population of childbearing age at location $k$ |
| $d_{ij}$ | Travel time between locations $i$ and $j$ |
| $d_0$ | Maximum travel time threshold |
| $W(d)$ | Distance decay weight function (Gaussian/exponential/stepwise) |

Accessibility scores are normalised to [0, 1] and classified into three deprivation levels using **city-specific thresholds**.

---

## City-Specific thresholds for Emergency Maternal Care access deprivation

City-specific thresholds were applied to classify emergency maternal care accessibility deprivation based on the distribution of standardized accessibility scores derived from the enhanced two-step floating catchment area (E2SFCA) method. Threshold variability across cities reflects differences in functional urban area scale, building density, population density (e.g., women of childbearing age), the spatial distribution of the 3 closest healthcare facilities and local mobility conditions. Thresholds also adapt to based on the quality of the available datasets and the distribution of the accessibility scores. Accessibility scores are interpreted such that higher values indicate better access and lower levels of deprivation. The table below summarises the thresholds applied in each city for classification.

If you require further information about the data sources and methodology used to derive these thresholds, please email us via the [IDEAMAPS network](mailto:admin@ideamapsnetwork.org) or [Dr Diego Pajarito Grajales](mailto:diego.pajaritograjales@glasgow.ac.uk) at the [Urban Big Data Centre](https://ubdc.ac.uk), University of Glasgow.

- ### Kano, Nigeria

[Functional Urban Area (FUA)](https://human-settlement.emergency.copernicus.eu/): ~ 1409 km², FUA population: approximately 4,821,779 (2015)

| Deprivation level | Accessibility Score Range |
|-------------------|---------------------------|
| High (2)          | 0.000001 < score ≤ 0.005  |
| Medium (1)        | 0.005 < score ≤ 0.02      |
| Low (0)           | score > 0.02              |

---

## 📎 Outputs

| File | Description |
|---|---|
| `deprivation-classification.gpkg` | Deprivation classification grid (Low/Medium/High) |
| `travel_matrix_3_closest_EmOC.gpkg` | Travel matrix to 3 nearest EmOC facilities |

For additional outputs including image examples and dataset metadata, see the [model outputs folder](https://github.com/urbanbigdatacentre/ideamaps-models/blob/dev/models/emergency-maternal-care/kano).

---

## Limitations and assumptions

As with any model, there are limitations that emerge from the multiple decisions made and constraints imposed by the available datasets. The following are the main limitations identified by the team.

- Population counts might not reflect the most recent changes in demographics or how people are spread out in different areas.

- The health care facilities we have chosen might not cover all available options, meaning some places that offer emergency obstetric care (EmOC) might be overlooked, while others that do not provide it currently might be included.

- To estimate travel times, we used a standard routing service where the vehicle speeds were not tested on the ground, which can lead to inaccuracies.

- In some areas, the building footprints dataset might not fully capture all structures, especially informal ones, which can affect how we estimate where women of childbearing age live.

- There are some roads that are not captured in the dataset used to calculate routes and travel times.

- We did not consider public transport to estimate travel times.

- Using synthetic indexes adds complexity, making it hard to explain the process and difficult to provide feedback on the results.
