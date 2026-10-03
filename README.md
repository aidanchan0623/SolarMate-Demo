<div align="center">

# SolarMate ☀️

### Community solar sharing, explained through energy credits

**An engineering project by Chan Zhi Hong**<br>
Electrical Engineering, Universiti Malaya

**UM Technothon · Top 15 among 50+ teams · May 2026**

[Walkthrough](#project-walkthrough) · [Architecture](#architecture) · [Technical notes](docs/technical-walkthrough.md)

</div>

---

## About this showcase

This public repository presents SolarMate through screenshots and an engineering walkthrough. The application, backend and embedded source code remain in a separate private repository.

## The idea

A household can generate more solar energy than it uses, while a nearby business still relies on grid electricity. SolarMate explores how a community platform could account for that surplus and credit its value to consumers.

Producers follow their export commitments and earnings. Consumers see their solar credit, remaining grid imports and blended bill. An administrator follows the wider supply and demand picture.

This is a prototype for exploring energy sharing and its accounting. The prices and energy volumes in the walkthrough are illustrative.

## My contribution

I designed the competition platform and its generation, consumption and transaction dashboards. I also developed the Python/SQL backend for users, energy listings and trades. Our team reached the Top 15 among more than 50 teams at UM Technothon.

The private implementation has since developed into a React application with a FastAPI/SQLite backend and an ESP32 telemetry path. That continuation connects my interest in power systems with the practical work of building interfaces and handling data.

## Project walkthrough

**Note on my demo:** I captured these views from my standalone React demonstration on 3 October 2026 using mock data. I describe the backend and ESP32 work separately below. I still need to validate the complete workflow with physical meter readings and connected settlement records.

### 1. Producer view: an export commitment and its earnings

The producer follows a monthly export plan and compares illustrative earnings. This makes the relationship between exported energy and payment easy to follow.

<p align="center">
  <img src="docs/screenshots/prosumer-overview.jpg" alt="Producer portal showing a 300 kWh plan, 180 kWh exported and RM59.40 illustrative earnings" width="620">
</p>

### 2. Consumer view: solar credit and grid imports share one bill

The example customer uses **1,000 kWh**, with **472 kWh of solar credit** and **528 kWh of grid imports**. Under the prototype's rate assumptions, the blended bill is **RM498.42**, compared with **RM541.70** for grid electricity alone.

That is a **RM43.28 saving**, approximately **7.99% of the total comparison bill**.

<p align="center">
  <img src="docs/screenshots/billing-detail.jpg" alt="Bill breakdown showing RM202.96 for solar, RM275.46 for grid imports, RM20 fixed charge and RM498.42 total" width="860">
</p>

The settlement view divides the RM202.96 solar payment into RM155.76 for the producer, RM42.48 for the grid charge and RM4.72 for the platform. These amounts explain the prototype's accounting model.

### 3. Network view: matched exports and unmet demand

The illustrated network supplies **360,000 kWh** against **420,000 kWh** of subscribed demand. It matches all available export, leaving **60,000 kWh** of demand for grid supply. A 100% export matching rate describes the use of available exports, not the coverage of all demand.

<p align="center">
  <img src="docs/screenshots/matching-detail.jpg" alt="Illustrative chart comparing 360,000 kWh exported and matched with 420,000 kWh demand" width="860">
</p>

<details>
<summary><strong>More screenshots: allocation, full billing and network views</strong></summary>

#### Energy allocation

My expandable preview uses available export divided by subscribed demand. Note: I used separate mock scenarios for the 472 kWh credited total and the 857.1 kWh preview. My next step is to derive both from one settlement dataset with explicit periods.

![Consumer energy allocation](docs/screenshots/energy-allocation.jpg)

#### Full billing and settlement

![Complete billing and settlement view](docs/screenshots/billing-and-settlement.jpg)

#### Full network view

![Complete supply and demand view](docs/screenshots/supply-and-demand.jpg)

</details>

## Architecture

The frontend sends authenticated requests to FastAPI. The backend stores account and energy records in SQLite through SQLAlchemy. A separate HTTP path accepts ESP32 prototype readings.

```mermaid
flowchart LR
    ESP[ESP32 prototype sender] -->|JSON over Wi-Fi and HTTP| M[Meter API]
    UI[React dashboards] -->|HTTP and bearer token| API[FastAPI role APIs]
    M --> DB[(SQLite through SQLAlchemy)]
    API <--> DB
    API --> E[Energy accounting]
    E --> API
```

| Layer | Technology | Responsibility |
|---|---|---|
| Interface | React, JavaScript/TypeScript, Vite, TanStack Start | Producer, consumer and admin dashboards |
| API | Python, FastAPI, Pydantic | Account, role, meter, billing and wallet endpoints |
| Storage | SQLAlchemy, SQLite | Profiles, energy records, meter readings and wallet records |
| Embedded prototype | ESP32, Arduino C++, Wi-Fi, HTTP | Send voltage, current, power and accumulated energy |
| Accounting | JavaScript functions and Python energy helpers | Credit limits, grid imports, bills and earnings |

## Engineering details

### Energy allocation and billing

The standalone demo previews credit in proportion to available export and subscribed demand. It caps the preview at the package allocation. Billing caps credited energy at usage and calculates the remaining grid imports.

The backend additionally aggregates database records into monthly supply and demand summaries, limits matching against available supply and applicable quota, and records the corresponding payout and revenue totals.

All prices below are **prototype assumptions**, not a statement of current electricity tariffs:

| Example bill component | Calculation | Amount |
|---|---|---:|
| Solar energy | 472 kWh × RM0.43/kWh | RM202.96 |
| Grid imports | 528 kWh × RM0.5217/kWh | RM275.46 |
| Fixed charge | Monthly reference charge | RM20.00 |
| **Blended total** | Sum of the above | **RM498.42** |

### ESP32 telemetry

The sender calculates power using `P = V × I` and accumulates watt-hours using elapsed time. It posts a JSON reading to the meter API every five seconds. The API accepts the electrical quantities, associates the device with its producer profile and stores a timestamped reading.

Note: my current sketch **generates sample voltage and current values**. I have outlined INA219 integration, but I still need to implement and validate the sensor readings. I also scale small prototype energy values for the demonstration; measured energy must use the physical conversion `kWh = Wh / 1,000`.

### What I learned

An energy dashboard needs a clear definition behind every number. Generated energy, exported energy, customer credit and billed energy describe different stages of the system. Building SolarMate gave me practice connecting those stages, keeping their units clear and explaining how a change in one quantity affects the bill.

## My current scope and next steps

I have implemented application accounts, API calls, database models and a prototype telemetry path in the private repository. Note: my current demonstration uses local data and simulated wallet operations. I have not validated live payments, calibrated household metering or an operational energy market.

I plan to reconcile allocation previews and credited totals against one settlement dataset, then validate the telemetry path with measured sensor data.

## Explore the presentation

- [Detailed technical walkthrough and presentation script](docs/technical-walkthrough.md)
- [Screenshot gallery](docs/screenshots)

This showcase contains documentation and screenshots. The runnable application and setup instructions remain with the private implementation.

---

**Chan Zhi Hong**<br>
I welcome conversations about embedded systems, robot electronics and engineering projects.

[GitHub](https://github.com/aidanchan0623) · [LinkedIn](https://www.linkedin.com/in/zhi-hong-chan-821029313/)
