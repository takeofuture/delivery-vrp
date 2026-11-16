# Delivery VRP with Traffic, Driver Constraints, and 3D Packing

This repository contains the demonstration notebooks and supporting materials for my presentation at **XtremePython 2026**.

**Presentation:**  
*From Addresses to Efficient Deliveries: Solving Vehicle Routing Problems with Python*

**Conference:** XtremePython 2026  
**Date:** November 17, 2026  
**Event website / speakers:**  
https://xtremepython.dev/2026/speakers/

The presentation demonstrates how Python can connect geography, traffic data, vehicle routing optimization, driver constraints, and physical 3D cargo packing into a practical delivery-planning workflow.

The main goal is not simply to find the shortest route, but to create a more realistic delivery plan that considers:

- OpenStreetMap road networks
- TomTom real-time traffic data
- Vehicle Routing Problem (VRP) optimization with Google OR-Tools
- Customer delivery time windows
- Driver shifts and truck qualifications
- Soft driver-overtime penalties
- Truck payload and volume limits
- 3D cargo packing feasibility with `py3dbp`
- Iterative re-optimization when a proposed truck load cannot physically fit

---

## Demo Notebooks

### 07A1 — Base Scenario with TomTom

Base delivery scenario using:

- 20 customers
- 5 trucks
- driver schedules and qualifications
- customer time windows
- OpenStreetMap
- TomTom traffic
- OR-Tools VRP
- 3D packing validation
- soft driver-overtime optimization

This is the main notebook and contains the full workflow.

---

### 07A2 — Higher-Demand Scenario with TomTom

Uses essentially the same optimization model as 07A1, but increases the workload.

Changes include:

- additional customers
- a wider delivery area
- larger order quantities
- higher fleet utilization
- more packing conflicts and re-optimization

The purpose is to show how the same model reacts when demand increases.

---

### 07B1 — Public / Open Data Version

Returns to the base scenario but replaces the TomTom traffic layer with public and open transportation data.

Data sources include:

- OpenStreetMap
- City of Irvine GIS
- Caltrans CWWP2

The downstream VRP, driver constraints, and 3D-packing workflow remain essentially the same.

---

# Running in Google Colab

The notebooks are designed primarily for Google Colab.

Required Python packages are installed by the notebook itself.

The notebooks may also use Google Drive to store relatively expensive intermediate results such as:

- OpenStreetMap road-network caches
- downloaded TomTom traffic tiles
- merged OSM + traffic models
- travel-time matrices

---

## Google Drive Cache Directory

The example notebooks contain a Google Drive path similar to:

```python
CACHE_DIR_DRIVE = Path(
    "/content/drive/MyDrive/.../tomtom_cache"
)
```

Change this path to a directory in your own Google Drive.

For example:

```python
CACHE_DIR_DRIVE = Path(
    "/content/drive/MyDrive/delivery-vrp/tomtom_cache"
)
```

Google Drive is mounted with:

```python
from google.colab import drive

drive.mount(
    "/content/drive",
    force_remount=False
)
```

You do not have to use Google Drive.

For temporary experiments, you can instead use a local Colab directory such as:

```python
CACHE_DIR_LOCAL = Path(
    "/content/tomtom_cache"
)
```

However, files stored only under `/content` normally disappear when the Colab runtime is reset.

Google Drive is therefore recommended when you want to reuse downloaded traffic data or other cached results in a later session.

---

# TomTom API

The TomTom versions of this demo use the TomTom Traffic API.

You need a TomTom account and API key.

TomTom provides developer access to its APIs, and depending on the current product and account plan, free monthly requests or evaluation access may be available for testing small workloads.

Please check the current TomTom documentation and pricing before running a large number of requests.

Useful TomTom resources:

- TomTom MyTomTom
- TomTom API documentation
- TomTom Traffic API documentation
- TomTom API key setup documentation

---

## Configure the TomTom API Key in Google Colab

Do **not** hard-code your API key directly into the notebook, especially if you plan to upload the notebook to GitHub.

Instead, store the key in **Google Colab Secrets**.

Create a secret named:

```text
TOMTOM_API_KEY
```

The notebook retrieves the key from Colab Secrets when a TomTom API request is required.

The primary expected secret name is:

```text
TOMTOM_API_KEY
```

### Important

Never commit your real API key to GitHub.

For example, do not write:

```python
TOMTOM_API_KEY = "your-real-secret-key"
```

inside a public notebook.

---

# Traffic Cache

Downloading traffic data and rebuilding the traffic-aware road network can take time and may also consume API requests.

For this reason, the TomTom notebooks support caching.

The main user-facing switch is:

```python
REUSE_TRAFFIC_CACHE = True
```

or:

```python
REUSE_TRAFFIC_CACHE = False
```

---

## `REUSE_TRAFFIC_CACHE = False`

Use:

```python
REUSE_TRAFFIC_CACHE = False
```

when you want to obtain a **fresh TomTom traffic snapshot**.

The notebook will attempt to:

1. request current TomTom traffic data
2. download any required traffic tiles
3. apply the traffic data to the OSM road network
4. rebuild the traffic-aware travel-time model
5. save the downloaded and processed results to cache

This mode may make TomTom API requests.

Use it when you want current traffic conditions.

For example:

```python
REUSE_TRAFFIC_CACHE = False
```

is appropriate when refreshing the data shortly before an experiment or presentation.

---

## `REUSE_TRAFFIC_CACHE = True`

Use:

```python
REUSE_TRAFFIC_CACHE = True
```

when you want to reuse traffic data that was already downloaded.

If a compatible cache exists, the notebook can reuse it without downloading the same TomTom data again.

This is useful when:

- rerunning the notebook shortly after a previous run
- restarting a Colab runtime
- testing the VRP code repeatedly
- debugging other parts of the notebook
- preparing a live demo
- avoiding unnecessary TomTom API requests

For example:

```python
REUSE_TRAFFIC_CACHE = True
```

is normally convenient when the traffic snapshot is still recent enough for your experiment.

---

# Cache Strategy

The TomTom notebooks use several levels of cache.

## 1. Merged OSM + TomTom Cache

This is the fastest option.

If a compatible merged cache exists, the notebook can reuse:

- the traffic-applied OSM graph
- customer travel-time matrices
- related road-model results

In this case, no new TomTom request may be necessary.

---

## 2. Raw TomTom Traffic Cache

If a merged result is unavailable, previously downloaded TomTom traffic tiles can still be reused.

The notebook attempts to load cached traffic data before requesting missing data from TomTom.

Only missing traffic tiles need to be downloaded.

---

## 3. OpenStreetMap Cache

The base OSM road network is also cached.

Because the OSM road graph is not treated as real-time traffic data in this demo, it can normally be reused across repeated runs.

The default setting is:

```python
FORCE_REFRESH_OSM = False
```

---

## 4. Rebuild the Merged Road Model

If necessary, the notebook combines:

```text
OSM road network
        +
TomTom traffic
        ↓
Traffic-aware road model
        ↓
Travel-time matrix
```

The resulting model can then be cached again for later reuse.

---

# Recommended Workflow

For a fresh experiment:

```python
REUSE_TRAFFIC_CACHE = False
```

Run the notebook once and save the traffic snapshot and merged results.

For subsequent runs performed soon afterward:

```python
REUSE_TRAFFIC_CACHE = True
```

Reuse the saved traffic data instead of requesting it again.

A typical workflow is therefore:

```text
First run
    ↓
REUSE_TRAFFIC_CACHE = False
    ↓
Download fresh TomTom traffic
    ↓
Build traffic-aware road model
    ↓
Save cache
    ↓
Later run
    ↓
REUSE_TRAFFIC_CACHE = True
    ↓
Reuse cached traffic/model
    ↓
Run VRP and packing experiments
```

---

# Cache Compatibility

The notebook creates a scenario signature based on the current customer/configuration data.

This helps prevent a merged cache created for one customer configuration from being accidentally reused for a different scenario.

For example, the base scenario and higher-demand scenario may require different merged caches even when they use the same underlying TomTom traffic snapshot.

---

# Optimization Workflow

The overall workflow is:

```text
Customers + Orders
        ↓
Trucks + Drivers
        ↓
OpenStreetMap Road Network
        ↓
Traffic Data
        ↓
Traffic-Aware Travel-Time Matrix
        ↓
OR-Tools VRP
        ↓
Customer / Truck Assignment
        ↓
Visit Order
        ↓
Driver and Time Constraints
        ↓
Weight / Volume Constraints
        ↓
3D Packing Check
        ↓
        ├── FIT → Final Delivery Plan
        │
        └── NOT FIT
              ↓
         Add Packing Cut
              ↓
         Re-optimize VRP
```

Routing-related decisions are optimized together inside each VRP solve.

The 3D packing model is connected through an iterative feedback loop rather than being solved as one monolithic optimization model.

---

# Hard and Soft Constraints

Examples of hard constraints in the current demo include:

- every customer must be served
- customer delivery time windows
- truck payload limits
- truck cargo-volume limits
- driver availability
- driver-to-truck qualification
- shift start time

Driver overtime is modeled as a **soft constraint**.

Overtime is allowed when necessary, but it receives a strong penalty in the objective function.

The current objective can therefore be summarized as:

```text
Minimize traffic-aware travel time
+
Strongly penalize driver overtime
```

Physical 3D packing feasibility is checked after each VRP solution.

If packing fails, a new packing constraint is added and the VRP is solved again.

---

# Important Notes

This repository is a demonstration / experimental implementation.

The input customers and orders are mock data rather than production delivery records.

The current model does not yet fully optimize:

- route-aware loading order
- loading and unloading effort
- irregular cargo shapes
- fragile-item handling
- stackability restrictions
- center of gravity
- refrigerated / frozen transportation
- spoilage risk
- full fuel and labor cost
- maintenance and depreciation
- revenue and profit
- complete operational risk

These are possible future extensions.

---

# Security

Do not publish:

- TomTom API keys
- private Google Drive credentials
- private customer data
- proprietary delivery records

Use Colab Secrets or another secure secret-management mechanism for API credentials.

---

# License

Add the license appropriate for your project before redistributing or reusing the repository.
