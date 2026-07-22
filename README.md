# Invest.Data

**Investment and Policy Trends and Insights**  
*SME and Enterprise Development, Policy & Regulations Unit (WKPTS) — World Bank Group*

> 🔒 **For World Bank Group internal use only. Do not distribute externally.**

---

## Overview

Invest.Data is an interactive investment data dashboard built with [Shiny for Python](https://shiny.posit.co/py/). It provides a dynamic, evidence-based view of global investment trends across economies, regions, and income levels, designed to support WBG operational work and client engagement.

The dashboard covers foreign capital inflows, FDI trends, bilateral investment relationships, the business environment, and investment linkages. It is updated as new data becomes available, with the Investment Highlights section intended to be refreshed biannually (each H1 and H2).

---

## Dashboard Tabs

| Tab | Description |
|---|---|
| **Landing** | Overview map with FDI concentration by economy |
| **FDI Concentration Map** *(inactive)* | Interactive ipyleaflet choropleth map of FDI inflows by economy (2000–2024), with a draggable year slider and click-to-inspect popups. Requires `ipyleaflet` and `ipywidgets`. Currently not wired into `app.py`. |
| **Investment Highlights** | Timely narrative analysis with charts; new edition each semester |
| **Foreign Capital** | Foreign capital inflows by type (FDI, portfolio, other); filterable by economy, region, or income group |
| **FDI Trends** | FDI inflow/outflow trends with IQR benchmarking across peer groups |
| **Bilateral Trends** | Bilateral FDI flows and stocks between economy pairs (inward/outward) |
| **Business Environment** | Regulatory and institutional indicators benchmarked across economies |
| **Linkages** | Foreign investment linkages indicators with time-series and bar chart views |
| **WKPTS Hub** | External link to the WBG SharePoint site for the unit |
| **About** | Methodology, data sources, and appendices |

---

## Project Structure

```
invest-data/
│
├── app.py                          # Main Shiny app entry point
├── shared.py                       # Data loading and shared objects
│
├── landing_panel.py                # Landing tab (UI + server)
├── highlights_panel.py             # Investment Highlights tab
├── foreign_capital_panel.py        # Foreign Capital tab
├── fdi_trends_panel.py             # FDI Trends tab
├── bilateral_trends_panel.py       # Bilateral Trends tab
├── business_environment_panel.py   # Business Environment tab
├── linkages_panel.py               # Linkages tab
├── about_panel.py                  # About tab (UI only)
├── fdi_concentration_map.py        # FDI Concentration Map (inactive — not yet wired into app.py)
│
├── data_preparing.py               # General data pipeline (run before each dashboard update)
├── invest_data_insights.py         # Highlights figures pipeline (run before each new edition)
│
├── data/
│   ├── foreign_capital.csv
│   ├── fdi_trends.csv
│   ├── fdi_iqr.csv
│   ├── fdi_panel.parquet           # Geospatial data for landing map
│   ├── fdi_legend.html
│   ├── bilateral_inflow.csv
│   ├── bilateral_outflow.csv
│   ├── bilateral_instock.csv
│   ├── bilateral_outstock.csv
│   ├── economies_bilateral.csv
│   ├── business_environment_averages.csv
│   ├── linkages_averages.csv
│   ├── 2026_1_Fig1.csv             # Investment Highlights figures (per edition)
│   ├── 2026_1_Fig2a.csv
│   ├── 2026_1_Fig2b.csv
│   ├── 2026_1_Fig3a.csv
│   └── 2026_1_Fig3b.csv
```

---

## Architecture

Each tab follows a consistent modular pattern:

- **`<panel>_ui()`** — returns a `ui.nav_panel()` with a hero card, an "About this Series / How to use" info box, sidebar filters, and chart outputs.
- **`<panel>_server()`** — contains all reactive logic, chart rendering, and download handlers for that tab.
- **`shared.py`** — loads all data files once at startup and exposes shared DataFrames and dropdown lists used across modules.

Panels import from `shared.py` inside a `try/except ImportError` block, which allows each module to be developed or tested in isolation.

---

## Data Pipeline

The project uses two separate pipeline scripts, each serving a distinct purpose.

---

### `data_preparing.py` — General dashboard pipeline

Prepares all the data consumed by the main dashboard tabs (Foreign Capital, FDI Trends, Bilateral Trends, Business Environment, Linkages, and the Landing map). It reads from raw source files and outputs the processed CSVs and Parquet files loaded by `shared.py`.

**Run before each biannual dashboard update:**

1. Obtain updated source data files and place them in the expected input folders.
2. Run `data_preparing.py` from top to bottom.
3. Verify that all output files in `data/` have been refreshed.
4. Restart the Shiny app.

---

### `invest_data_insights.py` — Investment Highlights figures pipeline

Prepares the charts and underlying datasets for each new **Investment Highlights** edition. It reads from raw investment data sources (UNCTAD, IMF, fDi Markets) and produces the edition-specific figure CSVs (e.g., `2026_1_Fig1.csv`, `2026_1_Fig2a.csv`, etc.) consumed by `highlights_panel.py`.

It also handles analytical tasks specific to each edition, such as:
- EMDE / Advanced Economy classification based on the latest Global Economic Prospects.
- Stacked bar chart preparation for foreign capital inflow components (FDI, portfolio, other).
- Bubble chart data for digital and climate investment CAPEX analysis.
- Inflow/outflow growth comparisons by region and income group.

**Run before each new Highlights edition:**

1. Obtain updated source data (UNCTAD FDI stats, IMF balance of payments, fDi Markets exports).
2. Run `invest_data_insights.py` from top to bottom.
3. Verify that all figure CSVs for the new edition are written to `data/`.
4. Register the new CSVs in `shared.py` and add the new accordion panel in `highlights_panel.py`.
5. Restart the Shiny app.

---

## Deployment

The app is deployed on the organization's **Posit Connect** server. The main Python packages required are:

```
shiny
shinywidgets
plotly
pandas
geopandas
numpy
pyarrow
kaleido             # required by invest_data_insights.py for static chart export
ipyleaflet          # required by fdi_concentration_map.py (inactive)
ipywidgets          # required by fdi_concentration_map.py (inactive)
```

---

## Key Technical Notes

- **Dropdown menus:** Economy selectors use a grouped dict-based approach with non-selectable separator headers (regions, income levels). They are initialized with an empty selection.
- **Selectize padding:** Visual indentation padding from selectize inputs is stripped server-side using `.strip("\u00a0").strip()` before filtering.
- **Color tokens:** All panels share the same color system — `NAVY (#0a2d45)`, `BLUE (#3f9dd4)`, `BG_LIGHT (#f5f9fc)`, `BG_BLUE (#e8f4fb)`, `BORDER (#d4e4ef)`.
- **`pyarrow`** must be installed for `shared.py` to load the `fdi_panel.parquet` file correctly.

---

## Updating Investment Highlights

Each semester a new Highlights edition is added. To add a new edition (e.g., `2026 H2`):

1. Prepare the new figure CSVs (e.g., `2026_2_Fig1.csv`, etc.) via `invest_data_insights.py`.
2. Add the new data loads to `shared.py`.
3. Add a new `accordion_panel` block inside `_highlights_accordion()` in `highlights_panel.py`, following the existing `2026 H1` structure.
4. Add new `@render_widget` server functions for each figure inside `highlights_server()`.

---

## Contributing

This project is maintained by the **SME and Enterprise Development, Policy and Regulations unit**. For questions, issues, contributions, or training, contact Arlan Brucal ([abrucal@worldbank.org](mailto:abrucal@worldbank.org)), Francisco Aguilar Cisneros ([faguilacisneros@worldbank.org](mailto:faguilacisneros@worldbank.org)), or Kanako Nannichi ([knannichi@worldbank.org](mailto:knannichi@worldbank.org)).

When modifying a panel, follow the existing architecture:
- Keep the UI layout (`_ui()`) and reactive logic (`_server()`) for each panel together in its own dedicated file — for example, everything related to the Foreign Capital tab lives in `foreign_capital_panel.py`. Do not split a panel's code across multiple files.
- Import from `shared.py` inside a `try/except ImportError` block.
- Reuse the established color tokens — do not introduce new hex values.
- Test the panel file independently before integrating into `app.py`.

---

## License

Internal World Bank Group tool. All rights reserved. Not for public distribution.
