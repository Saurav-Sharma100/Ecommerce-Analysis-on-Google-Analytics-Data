# 🛒 Ecommerce Analysis on Google Analytics Data

## 📌 Project Summary

This project builds an end-to-end **analytics engineering pipeline** on Google Analytics 4 ecommerce data from the Google Merchandise Store.

The raw GA4 export contains rich behavioral information, but its event-based structure, nested `event_params`, repeated `items` arrays, and absence of ready-made session and product tables make it difficult to use directly for business intelligence.

The solution begins with the business questions, uses a **BEAM (Business Event Analysis & Modeling) workshop** to translate those questions into a dimensional model, and then implements the model using **dbt Core and Google BigQuery**.

The resulting analytics layer powers three Power BI dashboards focused on **Acquisition, Purchase Funnel, and Product Performance**. A final **Vanna AI + Gemini** layer allows users to ask business questions in natural language and generate SQL against the curated BigQuery marts.

```text
Raw GA4 Events — BigQuery
            ↓
Data Exploration & UNNEST
            ↓
BEAM Workshop — 7Ws
            ↓
Dimensional Model Design
            ↓
dbt Core
Staging → Intermediate → Marts
            ↓
BigQuery Analytics Marts
            ↓
      ┌─────┴─────┐
      ↓           ↓
   Power BI    Vanna AI
               + Gemini
```

---

# 📊 Dashboard Preview

## Acquisition Performance

<img width="1332" height="752" alt="Image" src="https://github.com/user-attachments/assets/2cc51d4e-aecb-43ae-b56c-8fc7efff809c" />

---

## Purchase Funnel

<img width="1331" height="750" alt="Image" src="https://github.com/user-attachments/assets/0ac4f3e7-823d-4ec4-b9c7-1c6559fb4557" />

---

## Product Performance

<img width="1331" height="749" alt="Image" src="https://github.com/user-attachments/assets/c1b84f39-16d8-422f-a7e1-97e62d771cf6" />

---

# 🤖 Vanna AI Preview

<img width="1173" height="922" alt="Image" src="https://github.com/user-attachments/assets/7be6a3db-53c8-437a-97f1-687bd996f9fe" />

---

# 🧾 Project Snapshot

| Category | Details |
|---|---|
| **Domain** | Ecommerce / Digital Analytics |
| **Dataset** | Google Merchandise Store GA4 Sample Ecommerce |
| **Source** | BigQuery Public Dataset |
| **Data Structure** | Event-based GA4 export with nested and repeated fields |
| **Cloud Warehouse** | Google BigQuery |
| **Analytics Engineering** | dbt Core |
| **Modeling Methodology** | BEAM — Business Event Analysis & Modeling |
| **Data Model** | Dimensional / Star Schema |
| **Transformation Layers** | Staging → Intermediate → Marts |
| **Final Model** | 3 Fact Tables + 5 Dimensions |
| **BI Tool** | Power BI |
| **AI Analytics** | Vanna AI + Google Gemini + ChromaDB |
| **Programming** | SQL + Python |

---

# 🎯 Project Overview

The Google Merchandise Store collects website activity through **Google Analytics 4**, with raw GA4 events exported into BigQuery.

The dataset covers approximately three months of ecommerce activity from **November 2020 through January 2021**, including website sessions, product interactions, cart activity, checkout events, and completed purchases.

Although the raw data contains valuable behavioral information, it is not structured for direct business reporting.

Important information is distributed across:

- top-level event fields
- nested STRUCT fields
- `event_params` arrays
- `items` arrays
- ecommerce attributes
- device and geographic attributes
- user-level and session-level traffic information

The project therefore focuses on transforming the raw event stream into a reusable and documented analytics layer rather than writing isolated SQL queries for individual reports.

---

# 💼 Business Problem

Marketing, product, and leadership teams need consistent answers from ecommerce behavioral data.

However, querying raw GA4 data directly creates several problems:

- Nested fields require specialized SQL
- Sessions must be reconstructed from events
- Product information is embedded inside repeated arrays
- User-level and session-level attribution have different meanings
- Different analysts can implement metrics differently
- Raw event tables are difficult to consume directly in BI tools

The objective was to create a **single source of truth** that converts raw GA4 events into business-ready fact and dimension tables.

---

# ❓ Business Questions

The entire analytical model was designed around three questions:

### 1. Acquisition

> **Which channels bring users — and which convert them to purchase?**

### 2. Purchase Funnel

> **At which funnel step do we lose the most users?**

### 3. Product Performance

> **Which products have high interest but low conversion?**

These questions drive the BEAM workshop, fact-table grains, dbt transformations, Power BI dashboards, and Vanna AI training examples.

---

# 🛠️ Tools & Technologies

| Tool | Role in the Project |
|---|---|
| **Google BigQuery** | Raw GA4 source, cloud warehouse and dbt transformation target |
| **dbt Core** | Modular SQL transformation, testing, documentation and lineage |
| **SQL** | GA4 exploration, UNNEST operations, session reconstruction and business logic |
| **BEAM** | Business-event-driven dimensional modeling methodology |
| **Power BI** | Semantic model, DAX and business dashboards |
| **Vanna AI** | Natural-language-to-SQL conversational analytics |
| **Google Gemini** | LLM used by the Vanna implementation |
| **ChromaDB** | Local vector store for Vanna training information |
| **Python** | Vanna configuration, training and query application |
| **Git / GitHub** | Version control |

---

# 🗂️ Dataset Overview

The project uses:

```text
bigquery-public-data.ga4_obfuscated_sample_ecommerce
```

GA4 stores each day's events in separate wildcard tables:

```text
events_YYYYMMDD
```

which can be queried together using:

```text
events_*
```

Each row represents an event generated by a pseudonymous user.

Relevant ecommerce events include:

```text
session_start
      ↓
view_item
      ↓
add_to_cart
      ↓
begin_checkout
      ↓
purchase
```

Other events such as `page_view`, `scroll`, `first_visit`, and `user_engagement` provide additional behavioral context but were not modeled as independent business facts.

---

# 🧩 GA4 Data Engineering Challenges

## Nested `event_params`

GA4 stores many event attributes as key-value pairs inside the repeated `event_params` array.

Instead of having a simple column such as:

```text
session_engaged
```

the value may be stored inside:

```text
event_params[
    {
        key: "session_engaged",
        value: {...}
    }
]
```

The project therefore uses BigQuery `UNNEST()` to convert these nested parameters into relational rows that can be transformed downstream.

## Repeated `items` Array

Product information is stored inside the repeated `items` array.

A single ecommerce event can contain multiple products, with attributes such as:

```text
item_id
item_name
item_brand
item_category
item_variant
price
quantity
item_revenue
```

The staging layer unnests this array to create **one row per item per event**.

## Sessions Must Be Reconstructed

The GA4 export does not provide a ready-made analytical session table.

A session identifier is constructed using:

```text
user_pseudo_id + ga_session_id
```

Events sharing the same session key are then aggregated to derive session-level metrics including:

- session start
- session end
- session duration
- engagement
- event count
- page views
- product views
- add-to-cart activity
- checkout activity
- purchases
- conversion flag

This becomes the foundation for `fct_sessions`.

---

# 🧠 BEAM Workshop — Business Event Analysis & Modeling

One of the central parts of this project was completing a **BEAM workshop before implementing the dbt models**.

Instead of designing tables around the structure of the GA4 source, BEAM starts with:

> **What business events actually matter?**

The ecommerce journey was mapped from traffic acquisition through purchase:

```text
Traffic Source
      ↓
Customer Visits
      ↓
Views Product
      ↓
Adds to Cart
      ↓
Begins Checkout
      ↓
Completes Purchase
```

Three analytical business events were selected as the foundation of the model:

1. **Purchase**
2. **Product Interaction**
3. **Session**

---

# 🔎 The BEAM 7Ws Framework

Each business event was analyzed using the **7Ws**.

| W | Business Question | Modeling Result |
|---|---|---|
| **Who?** | Who performed the event? | `dim_users` |
| **What?** | What product was involved? | `dim_products` |
| **When?** | When did it occur? | `dim_date` + timestamps |
| **Where?** | Device / location context? | `dim_device` + geographic attributes |
| **Why?** | Which source or channel drove it? | `dim_traffic_source` |
| **How Many?** | Revenue, quantity, events, activity? | Fact-table measures |
| **How?** | How is the event identified/classified? | Natural keys and event attributes |

The workshop converts business questions into concrete modeling decisions before SQL transformation begins.

---

# 🧱 Dimensional Modeling Decisions

## Purchase Event

**Grain:**

> One row per completed transaction.

Implemented as:

```text
fct_purchases
```

## Product Interaction Event

Product funnel events share the same fundamental context:

```text
view_item
add_to_cart
begin_checkout
purchase
```

Rather than creating separate fact tables for every funnel step, they are consolidated into:

```text
fct_product_interactions
```

with a `funnel_step` attribute.

**Grain:**

> One row per item per product event.

## Session Event

Website activity is reconstructed into:

```text
fct_sessions
```

**Grain:**

> One row per reconstructed user session.

This supports acquisition, engagement, device, and session-conversion analysis.

---

# ⚙️ dbt Core — Analytics Engineering Layer

dbt Core is the primary transformation framework used in this project.

Rather than maintaining large standalone SQL scripts, the transformation pipeline is divided into modular models with explicit dependencies.

The project follows a three-layer architecture:

```text
Raw GA4
   │
   ▼
STAGING
   │
   ▼
INTERMEDIATE
   │
   ▼
MARTS
```

| Layer | Purpose | Materialization |
|---|---|---|
| **Staging** | Clean, rename, type and flatten source data | View |
| **Intermediate** | Apply joins, reconstruction and business logic | View |
| **Marts** | Expose business-ready facts and dimensions | Table |

This separation keeps source cleanup, analytical logic, and final reporting models independent from one another.

---

# 🚦 dbt Development & Production Targets

Separate dbt targets are used for development and production.

```text
DEV
dbt_ga4_dev
```

is used as the development sandbox.

```text
PROD
dbt_ga4
```

contains the clean models intended for downstream analytics.

The default target is development, while production models are built explicitly using the production target.

---

# 🧹 dbt Staging Layer

The staging layer establishes a clean contract with the raw GA4 source.

Three primary staging models are created:

```text
stg_ga4__events
stg_ga4__event_params
stg_ga4__items
```

## `stg_ga4__events`

Maintains one row per GA4 event while:

- creating a surrogate `event_id`
- converting timestamps
- parsing dates
- constructing session IDs
- flattening device attributes
- flattening geography
- exposing user-level traffic attribution
- exposing ecommerce fields

A surrogate key is generated using `dbt_utils`:

```sql
{{ dbt_utils.generate_surrogate_key(
    ['user_pseudo_id', 'event_timestamp']
) }}
```

## `stg_ga4__event_params`

Uses:

```sql
UNNEST(event_params)
```

to create one row per parameter per event.

Values stored across GA4's different parameter data types are standardized into a common field using `COALESCE`.

## `stg_ga4__items`

Uses:

```sql
UNNEST(items)
```

to create one row per product per event.

This exposes product-level attributes for downstream funnel and revenue analysis.

---

# 💰 Cost-Aware dbt Development

Development runs intentionally restrict the raw GA4 wildcard source to a small date range:

```sql
{% if target.name == 'dev' %}

WHERE _TABLE_SUFFIX
BETWEEN '20201101' AND '20201107'

{% endif %}
```

This allows transformation logic to be developed against a smaller slice of the dataset before running the full production model.

---

# 🔄 dbt Intermediate Layer

The intermediate layer contains reusable business logic that should not be duplicated across marts.

Three intermediate models are used:

| Model | Responsibility |
|---|---|
| `int_ga4__product_events` | Combines event and item data and assigns funnel steps |
| `int_ga4__session_traffic` | Extracts session-level source, medium and campaign |
| `int_ga4__sessions` | Reconstructs complete sessions and engagement metrics |

## Product Funnel Logic

`int_ga4__product_events` standardizes the four modeled product events:

```text
1 → view_item
2 → add_to_cart
3 → begin_checkout
4 → purchase
```

This model feeds both:

```text
fct_product_interactions
fct_purchases
```

so the event-to-item transformation logic is defined once rather than duplicated.

## Session-Level Attribution

An important GA4 modeling distinction is made between:

### User-Level Attribution

```text
traffic_source
traffic_medium
traffic_campaign
```

These describe how the user was originally acquired.

### Session-Level Attribution

```text
session_source
session_medium
session_campaign
```

These describe the source responsible for an individual session.

The intermediate layer reconstructs the session-level attribution from `session_start` event parameters.

---

# 🏗️ dbt Mart Layer

The mart layer contains the final business-ready tables consumed by Power BI and Vanna AI.

### Fact Tables

```text
fct_purchases
fct_product_interactions
fct_sessions
```

### Dimension Tables

```text
dim_users
dim_products
dim_date
dim_device
dim_traffic_source
```

The marts are materialized as tables to provide a stable consumption layer for downstream analytics.

---

# ⭐ Data Model — Star Schema

<img width="1259" height="738" alt="Image" src="https://github.com/user-attachments/assets/15a9fec9-fc57-4488-b26f-ece063951e85" />

---

The final model contains **3 fact tables and 5 dimensions**, each designed around a defined business grain.

| Table | Grain |
|---|---|
| `fct_purchases` | One row per transaction |
| `fct_product_interactions` | One row per item per product event |
| `fct_sessions` | One row per session |
| `dim_users` | One row per pseudonymous user |
| `dim_products` | One row per product |
| `dim_date` | One row per calendar date |
| `dim_device` | One row per device / OS / browser combination |
| `dim_traffic_source` | One row per source / medium combination |

---

# ✅ dbt Testing & Documentation

dbt YAML files document model purpose, column meaning, grain, and relationships.

Data-quality tests include checks such as:

```yaml
tests:
  - unique
  - not_null
```

and accepted-value validation for controlled fields such as funnel events.

```yaml
accepted_values:
  values:
    - view_item
    - add_to_cart
    - begin_checkout
    - purchase
```

Testing helps validate assumptions such as:

- primary keys are populated
- session IDs are unique where expected
- funnel events contain supported values
- required identifiers are not null

---

# 🔗 dbt Lineage

```text
GA4 events_*
│
├── stg_ga4__events
├── stg_ga4__event_params
└── stg_ga4__items
        │
        ▼
INTERMEDIATE
│
├── int_ga4__product_events
├── int_ga4__session_traffic
└── int_ga4__sessions
        │
        ▼
MARTS
│
├── fct_purchases
├── fct_product_interactions
├── fct_sessions
├── dim_users
├── dim_products
├── dim_date
├── dim_device
└── dim_traffic_source
        │
        ├───────────────┐
        ▼               ▼
     Power BI        Vanna AI
```

---

# 📈 Dashboard 1 — Acquisition Performance

### Business Question

> **Which channels bring us users — and which convert them to purchase?**

The page contains:

- **Sessions by Channel Over Time**
- **Conversion Rate by Channel**
- **Sessions vs Conversion Scatter**

The core conversion measure is:

```dax
Conversion Rate =
DIVIDE(
    SUM(fct_sessions[converted]),
    COUNTROWS(fct_sessions)
)
```

The scatter plot helps distinguish between:

```text
High Traffic + High Conversion
High Traffic + Low Conversion
Low Traffic + High Conversion
Low Traffic + Low Conversion
```

rather than evaluating acquisition channels on traffic volume alone.

---

# 🛒 Dashboard 2 — Purchase Funnel


### Business Question

> **At which funnel step do we lose the most users?**

The finished funnel shows approximately:

```text
47K  Product Views
 ↓
13K  Add to Cart
 ↓
 7K  Begin Checkout
 ↓
 4K  Purchase
```

The largest absolute loss occurs between **product view and add to cart**.

The overall view-to-purchase conversion displayed in the funnel is approximately **9.3%**.

The page also compares funnel participation across:

- desktop
- mobile
- tablet

and plots conversion performance across the analysis period.

### Funnel DAX

```dax
Step 1 Views =
CALCULATE(
    DISTINCTCOUNT(fct_product_interactions[user_pseudo_id]),
    fct_product_interactions[event_name] = "view_item"
)
```

```dax
Step 2 Cart =
CALCULATE(
    DISTINCTCOUNT(fct_product_interactions[user_pseudo_id]),
    fct_product_interactions[event_name] = "add_to_cart"
)
```

```dax
Step 3 Checkout =
CALCULATE(
    DISTINCTCOUNT(fct_product_interactions[user_pseudo_id]),
    fct_product_interactions[event_name] = "begin_checkout"
)
```

```dax
Step 4 Purchase =
CALCULATE(
    DISTINCTCOUNT(fct_product_interactions[user_pseudo_id]),
    fct_product_interactions[event_name] = "purchase"
)
```

Additional measures calculate:

```text
Cart to View %
Checkout to Cart %
Purchase to Checkout %
Overall Conversion %
```

---

# 📦 Dashboard 3 — Product Performance

### Business Question

> **Which products have high interest but low conversion?**

The page contains:

- **View-to-Purchase Rate**
- **Top Products by Revenue**
- **Category-Level Funnel**

Product interest and purchases are calculated separately:

```dax
Product Views =
CALCULATE(
    DISTINCTCOUNT(fct_product_interactions[session_id]),
    fct_product_interactions[event_name] = "view_item"
)
```

```dax
Product Purchases =
CALCULATE(
    DISTINCTCOUNT(fct_product_interactions[session_id]),
    fct_product_interactions[event_name] = "purchase"
)
```

```dax
View to Purchase Rate =
DIVIDE(
    [Product Purchases],
    [Product Views]
)
```

```dax
Total Revenue =
CALCULATE(
    SUM(fct_product_interactions[revenue]),
    fct_product_interactions[event_name] = "purchase"
)
```

This separates **product interest** from **actual purchasing behavior**, allowing high-view but low-conversion products to be identified as potential optimization opportunities.

---

# 🤖 Vanna AI — Conversational Analytics Layer


The final layer of the project extends the analytics model beyond predefined dashboards.

**Vanna AI** is connected to the curated BigQuery marts so a user can ask a business question in natural language and receive generated SQL and query results.

```text
Business Question
      ↓
Vanna AI
      ↓
ChromaDB Context
      +
Google Gemini
      ↓
Generated SQL
      ↓
BigQuery dbt Marts
      ↓
Query Results
```

This allows questions such as:

```text
"What is the conversion rate by channel?"
```

to be translated into SQL against the governed analytics layer rather than directly against the raw GA4 event export.

---

# 🧠 Training Vanna AI

The Vanna implementation is trained using three types of context.

### 1. DDL

Vanna receives the structure and data types of:

```text
fct_purchases
fct_sessions
fct_product_interactions
```

### 2. Business Documentation

Plain-English documentation explains:

- table grain
- intended use
- metric definitions
- attribution logic
- funnel behavior

### 3. Question / SQL Examples

The training set contains **10 business question-to-SQL pairs** covering:

```text
Acquisition
Funnel
Product Performance
```

This grounds the conversational layer in the same curated marts used for BI reporting.

---

# 🧬 Vanna Technical Architecture

The Python implementation combines:

```python
class GA4Vanna(ChromaDB_VectorStore, GeminiChat):
```

### ChromaDB Vector Store

Stores training information used to retrieve relevant context for a new question.

### Gemini

Acts as the language model responsible for interpreting the question and generating SQL using retrieved context.

The Vanna instance then connects directly to BigQuery.

This creates a conversational analytics interface without exposing business users to:

- raw GA4 arrays
- UNNEST logic
- session reconstruction
- dbt dependencies
- dimensional joins

---

# 🐍 Separate Python Environments

The project uses separate virtual environments for the transformation and AI layers:

```text
dbt_env
   ↓
dbt Core + BigQuery transformations

vanna_env
   ↓
Vanna AI + Gemini + ChromaDB
```

This isolates package dependencies between the analytics engineering and conversational AI components.

Credentials and generated artifacts are excluded from version control through `.gitignore`.

---

# ✨ Technical Highlights

- **Business-first dimensional modeling** using BEAM before implementation
- **Nested GA4 transformation** using BigQuery `UNNEST()`
- **Session reconstruction** from event-level GA4 data
- **Multi-grain fact modeling** across sessions, transactions and product interactions
- **Modular dbt architecture** using staging, intermediate and marts
- **Development / production separation** using dbt targets
- **Data quality testing** and YAML documentation
- **dbt lineage** for model dependency visibility
- **Shared analytics marts** powering both Power BI and Vanna AI
- **Conversational analytics** using Vanna AI, Gemini and ChromaDB

---

# 💡 Business Insights

## 1. The largest funnel loss occurs early

The finished funnel contains approximately **47K product viewers but only 13K users reaching add-to-cart**, making the view-to-cart transition the largest absolute drop in the modeled purchase journey.

## 2. Checkout completion is stronger than initial product progression

Approximately **7K users begin checkout and 4K reach purchase**, indicating that a substantial amount of funnel loss occurs before checkout rather than exclusively at the final transaction stage.

## 3. Channel volume and channel efficiency are different questions

The acquisition dashboard shows differences in session volume between traffic sources while conversion rates also vary. A channel generating the most traffic is therefore not automatically the most efficient acquisition source.

## 4. Device context matters to funnel analysis

Desktop generates the largest funnel volumes, followed by mobile, while tablet represents a much smaller share of activity.

## 5. Product popularity does not guarantee conversion

The product dashboard separates views, purchases, conversion rate, and revenue, making it possible to distinguish between products attracting attention and products converting that attention into purchases.

---

# 🏁 Conclusion

This project demonstrates how raw behavioral data can be transformed into a structured analytics system rather than treated as a collection of one-off queries.

The process begins with business questions and a **BEAM workshop**, where meaningful ecommerce events and their grains are identified before the warehouse model is designed.

Raw GA4 data is then transformed through **dbt Core**, with staging models handling nested source structures, intermediate models implementing reusable business logic and session reconstruction, and marts exposing tested fact and dimension tables in BigQuery.

Those same curated models serve two different consumption patterns: **Power BI** provides structured acquisition, funnel, and product-performance dashboards, while **Vanna AI with Gemini and ChromaDB** provides a conversational interface for natural-language analytics.

```text
Business Questions
        ↓
BEAM Modeling
        ↓
Raw GA4 / BigQuery
        ↓
dbt Transformation
        ↓
Dimensional Model
        ↓
Tested Analytics Marts
        ↓
Power BI + Vanna AI
```

The key outcome is a consistent analytics layer where dashboards, SQL analysis, and AI-generated queries are all built on the same governed business logic.
