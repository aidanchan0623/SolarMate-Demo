# SolarMate technical walkthrough

## A short presentation script

“SolarMate began with a simple question: how could surplus household solar generation benefit local business consumers? I worked on the platform architecture and dashboards, alongside the Python/SQL backend for users, listings and trades.

“The producer sees an export commitment and earnings. The consumer sees credited solar energy and the grid imports they still need. The admin sees supply and demand across the network.

“In the demonstration shown here, the customer uses 1,000 kWh and receives 472 kWh of solar credit. The remaining 528 kWh comes from the grid. Under the prototype's price assumptions, that produces a RM498.42 bill. The settlement breakdown explains where the solar payment goes.

“The private implementation also connects React to FastAPI and SQLite, with an ESP32 meter endpoint. The included sender calculates power and accumulates energy over time, although it generates sample readings by default. My next step would be to reconcile allocation and billing data, then validate the telemetry path with sensor measurements.”

## Implementation reference

The following file names refer to the private implementation. This public showcase does not include those source files.

| Topic | Implementation |
|---|---|
| Session and role selection | `src/solarmate/App.jsx` |
| API requests and bearer tokens | `src/solarmate/api/client.js` |
| Frontend calculations | `src/solarmate/utils/calculations.js` |
| Route registration | `backend/main.py` |
| SQLite connection | `backend/database.py` |
| Account, energy and wallet models | `backend/models.py` |
| Monthly matching and payout totals | `backend/platform_summary.py` |
| Backend energy units and bills | `backend/energy.py` |
| Reading validation | `backend/schemas.py` |
| Meter ingestion | `backend/routers/meter.py` |
| Prototype scaling and reading status | `backend/meter_utils.py` |
| ESP32 energy and HTTP sender | `esp32/solarmate_esp32_sender.ino` |

## Allocation preview

The standalone demonstration uses available export divided by subscribed demand. Package allocation multiplied by that ratio gives a preview, capped at the package amount. A zero-demand guard prevents division by zero. This is proportional allocation, not a network power-flow or dispatch solver.

The captured page has two independent demonstration values: 472 kWh already credited and an 857.1 kWh preview based on an 85.7% ratio. A complete settlement path should derive them from one dataset with explicit time periods.

The backend monthly-summary logic has a different scope. It aggregates recorded exports and consumer credits, considers active monthly capacities, limits the matched quantity, and stores a monthly summary. It is not identical to the standalone preview algorithm.

## Billing and settlement

The calculation limits credit to usage and treats the remainder as grid imports. All prices are prototype assumptions.

| Item | Example amount |
|---|---:|
| Solar charge: 472 kWh × RM0.43 | RM202.96 |
| Grid charge: 528 kWh × RM0.5217 | RM275.46 |
| Fixed charge | RM20.00 |
| **Blended bill** | **RM498.42** |
| Grid-only comparison | RM541.70 |
| **Saving** | **RM43.28, approximately 7.99%** |

The interface's approximately 17.6% figure describes the solar unit-rate discount. It is not the reduction in this example's entire bill.

The RM202.96 solar payment splits into RM155.76 producer payout, RM42.48 grid charge and RM4.72 platform revenue, using RM0.33, RM0.09 and RM0.01 per kWh respectively.

## Embedded telemetry

The ESP32 sender calculates `power_w = voltage_v × current_a` and accumulates `energy_wh += power_w × elapsed_hours`. It sends JSON every five seconds over HTTP.

Its voltage and current functions currently generate sample values. INA219 access appears as suggested replacement code in comments, so the sketch does not establish real sensor measurement.

The backend validates nonnegative electrical fields, finds the producer profile and stores a timestamped reading. For the designated ESP demonstration device, it replaces previous meter-reading rows and also maintains a separate LCD demonstration record. This is not an append-only raw telemetry archive.

The backend uses `energy_wh × 2.0` as a demonstration quantity labeled in kWh. This amplifies the small prototype for presentation. The physical unit conversion is `kWh = Wh / 1,000`.

## Screenshot provenance

Captured on 3 October 2026 from the standalone React demo used for the portfolio. Its mock session and data differ from the private implementation's backend-connected session flow. These images do not represent physical meter tests or real customer transactions.

| File | View |
|---|---|
| [prosumer-overview.jpg](screenshots/prosumer-overview.jpg) | Complete producer overview |
| [energy-allocation.jpg](screenshots/energy-allocation.jpg) | Consumer credit and allocation preview |
| [billing-and-settlement.jpg](screenshots/billing-and-settlement.jpg) | Complete billing and settlement |
| [supply-and-demand.jpg](screenshots/supply-and-demand.jpg) | Complete network scenario |
| [billing-detail.jpg](screenshots/billing-detail.jpg) | Unretouched billing crop |
| [matching-detail.jpg](screenshots/matching-detail.jpg) | Unretouched network crop |

The walkthrough combines captured UI behaviour with source inspection. It documents the API and embedded implementation without claiming hardware validation during this documentation update. The competition result and individual contribution follow the portfolio's résumé record.
