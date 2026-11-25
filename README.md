# THESIS-MatrixTera (2015–2019) — ATOMO

**Author:** Jaurès Ratsimbazafiharivola  
**Also known as:** Rascof  
**Project:** ATOMO — dynamic thermal simulation and analysis of buildings

This repository brings together the calculation codes and software tools developed as part of my doctoral research (2015–2019), along with a substantially updated, open-source-oriented version of the original ATOMO tool.

The original research work focused on building thermal simulation, the use of weather and architectural data, sensitivity analysis, and the comparison of simulated results with measurements. The software has since been refactored and extended using open-source technologies and libraries. The current ATOMO application is a web-based tool built with PHP, JavaScript, and MySQL/MariaDB.

> **Research and software note:** ATOMO is a research and engineering tool. Its material properties, synthetic weather data, and simplified physical assumptions should be checked and calibrated against measurements before the results are used for design or predictive purposes.

## ATOMO at a glance

ATOMO supports the creation of a building model, the configuration of its envelope and site, and the simulation of its thermal behaviour over time.

Main capabilities include:

- **2D building design:** draw walls, rooms, doors, windows, and glazed openings.
- **3D visualization:** inspect the building, levels, roof geometry, textures, shadows, and simulated temperature fields.
- **Multilevel buildings:** create and manage floors, intermediate slabs, and different ceiling heights.
- **Dynamic thermal simulation:** simulate heat transfer through opaque walls, roofs, floors, windows, and doors.
- **Automatic multizone simulation:** model each valid enclosed room as a separate thermal zone when the plan contains at least two valid rooms. Single-room projects continue to use the historical single-zone engine.
- **Weather data:** use built-in synthetic climates for Analamanga, Boeny, and Haute Matsiatra, or import hourly weather data from CSV files.
- **Green roofs and rainfall:** configure green-roof properties, rainfall scenarios, test storms, and hourly water-balance calculations.
- **Thermal comfort indicators:** examine indoor and operative temperatures and comfort indicators, including results by room in multizone projects.
- **Sensitivity analysis:** use Latin Hypercube Sampling (LHS), regression, and a linear metamodel with validation.
- **Model recalibration:** compare simulation results with measured indoor data using indicators such as R², RMSE, bias, and Nash–Sutcliffe efficiency.
- **Data export:** export simulation time series and results to CSV.
- **Project management:** create an account and save building projects and imported weather datasets.

## What is included in this repository?

The current application is organized as follows:

```text
atomo/
├── assets/                 # Front-end JavaScript, CSS, 2D/3D visualization
├── examples/               # Example building projects and weather CSV files
├── inc/
│   ├── engine.php          # Historical single-zone thermal engine
│   ├── multizone.php       # Multizone thermal engine
│   ├── plan.php            # Server-side building-plan geometry
│   ├── sensitivity.php     # Sensitivity analysis and metamodel
│   └── weather.php         # Weather data and solar calculations
├── sql/
│   ├── schema.sql          # Database schema
│   └── seed.sql            # Initial materials, constructions, and glazing data
├── tools/                  # Command-line tests and utility scripts
├── api.php                 # Authenticated JSON API
├── config.php              # Database and session configuration
├── index.php               # Main application
├── install.php             # Database installer
├── login.php
├── logout.php
├── README.md
├── MISE_A_JOUR_MULTIZONE.md
└── SUIVI_PROJET.md
```

## Technical overview

- **Back end:** PHP (procedural architecture), with `mysqli` for MySQL/MariaDB access.
- **Front end:** JavaScript, HTML, and CSS.
- **Database:** MySQL or MariaDB.
- **3D rendering:** Three.js, loaded from a CDN by the browser.
- **Authentication and application security:** session-based authentication, CSRF tokens, prepared SQL statements, and user-scoped projects.
- **Simulation:** dynamic thermal model using an implicit Euler time integration scheme. The numerical engine uses a 15-minute calculation step and provides hourly results; the first three days are used for thermal warm-up.

### Thermal model — brief description

The model represents heat transfer through the building envelope using resistance-capacitance networks. It accounts for, among other things:

- conduction through opaque envelope elements;
- thermal mass of walls, floors, and internal partitions;
- window solar gains and heat transfer;
- solar geometry and radiation on façades and roof surfaces;
- indoor air temperature and internal gains;
- thermal coupling between rooms in multizone projects;
- green-roof water storage and evapotranspiration processes.

For multizone projects, each enclosed room is represented by an independent air-temperature node. Shared partitions exchange heat between adjacent zones, and rooms connected by internal doors can exchange air according to user-defined scenario parameters.

## Installation

### Requirements

- PHP **7.4 or later** with the `mysqli` extension;
- MySQL or MariaDB;
- a web server such as Apache;
- a modern browser with JavaScript enabled;
- an internet connection for the Three.js library used by the 3D viewer.

### Local installation (XAMPP, WAMP, or LAMP)

1. Clone or download this repository.
2. Copy the `atomo/` directory into your web server document root (for example, `htdocs/` with XAMPP).
3. Edit `atomo/config.php` and set your database host, username, password, and database name.
4. Open `http://localhost/atomo/install.php` in your browser. The installer creates the database tables and loads the initial reference data.
5. Open `http://localhost/atomo/login.php?mode=register` to create your account.
6. Sign in and start a project.

**Important:** after installation, remove or restrict access to `install.php`. Before updating an existing installation, back up both the application directory and the database. Do not overwrite a customized `config.php` without preserving its settings.

## Example projects and weather data

The `examples/` directory contains sample building projects illustrating different construction systems and configurations, including examples associated with Analamanga, Boeny, and Haute Matsiatra, as well as a multi-room house and a two-level building.

It also includes sample weather CSV files. The rainfall example demonstrates hourly rainfall data, including dry hours and intense rainfall events.

For imported weather data, the application expects hourly records. The CSV importer recognizes columns for air temperature, wind speed, solar radiation, and optionally rainfall (`pluie_mm_h` is the recommended rainfall header). See the in-app import guidance and the project documentation for accepted formats and missing-value handling.

## Running the tests

From the `atomo/` directory, run the available checks with PHP and Node.js:

```bash
php tools/test-engine.php
php tools/test-multizone.php
node tools/test-plan.js
node tools/test-rain.js
```

These tests exercise the numerical engine, multizone geometry and heat exchanges, plan geometry, and rainfall visualization logic. A successful automated test run does **not** replace validation against independent simulation software or in-situ measurements.

## Current multizone model: scope and limitations

The multizone engine is activated automatically when a project contains at least two valid enclosed rooms. The historical single-zone engine is retained for single-room projects. The current implementation supports up to 40 zones per project.

The following limitations are important when interpreting results:

- Interzone air-exchange rates are prescribed scenario parameters; the model does not solve a pressure network or calculate wind-driven and stack-effect flows.
- Internal doors use the configured opening schedule.
- Occupant and equipment gains are distributed among rooms according to floor area rather than assigned individually to each room.
- Sensitivity-analysis outputs in multizone mode are currently aggregated at building level, not calculated separately for every zone.
- Synthetic climates and material properties are indicative and are not presented as calibrated datasets.
- The 3D visualization mesh is for display; it is not the numerical calculation mesh.
- The current update is not, by itself, an experimental validation. Comparison with an independent tool (for example, EnergyPlus) and measurements collected in real buildings are still needed before claiming predictive accuracy.

Further implementation details and the multizone update are documented in [`MISE_A_JOUR_MULTIZONE.md`](MISE_A_JOUR_MULTIZONE.md) and [`SUIVI_PROJET.md`](SUIVI_PROJET.md).

## Research context and evolution

The repository is connected to the doctoral research carried out between **2015 and 2019**. The current ATOMO application preserves the research-oriented purpose of the original tool while extending its software architecture and user interface.

The updated version brings together the historical calculation approach and newer capabilities such as:

- a browser-based 2D building editor and 3D viewer;
- a dynamic PHP simulation engine;
- multizone thermal modelling;
- expanded weather and rainfall handling;
- green-roof modelling;
- sensitivity analysis and model recalibration tools.

The exact scientific scope and conclusions of the thesis should be taken from the thesis manuscript and associated publications; this README describes the software currently included in the repository.

## Acknowledgements and open-source approach

I chose to build on open-source technologies and libraries to make the code easier to maintain, inspect, and extend. Please consult the licences of the third-party libraries and assets used by the project before redistributing them.

## Author

**Jaurès Ratsimbazafiharivola**  
Alias: **Rascof**

This repository documents software developed during my doctoral research and its subsequent updates.

## Licence

MIT License

Copyright (c) 2022 Jaurès RATSIMBAZAFIHARIVOLA
