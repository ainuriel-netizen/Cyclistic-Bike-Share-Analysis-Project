# CYCLISTIC BIKE-SHARE ANALYSIS

## Cyclistic Bike-Share: User Behavior Analysis

## 📋 TABLE OF CONTENTS

Complete analysis of Cyclistic bike-share based on 11.1M rides (2024-2025).

## BUSINESS TASK

Identify key behavioral differences between casual riders and annual members of Cyclistic bike-share service (Chicago).

### Analysis Objective:
Using historical data from 2024-2025, uncover usage patterns, preferences, and behavioral insights that explain why casual riders would benefit from purchasing an annual membership.

### Target Audience:
- Cyclistic Marketing Team
- Company Leadership
- Business Development Strategy

## DATA PREPARATION (Prepare)

### Data Source

Monthly CSV files from Cyclistic bike-share for two complete years (24 files, 2024-2025). Each file contains 400k-600k rides with information about time, bike type, user type, and station coordinates.

### Data Structure (Original)

| Column | Type | Example |
|--------|------|---------|
| ride_id | String | ABC123XYZ |
| rideable_type | String | classic_bike, electric_bike, docked_bike |
| started_at | DateTime | 2024-01-15 08:30:00 |
| ended_at | DateTime | 2024-01-15 08:45:00 |
| start_station_name | String | Ellis Ave & 55th St |
| start_station_id | String | 13022 |
| end_station_name | String | Ellis Ave & 60th St |
| end_station_id | String | 13068 |
| start_lat / start_lng | String | 41.7823 / -87.6089 |
| end_lat / end_lng | String | 41.7756 / -87.6082 |
| member_casual | String | member, casual |

### Key Decisions Made During Preparation

**Time Period** Selected full 2024-2025 period → 11.4M rows, sufficient for seasonality analysis

**Data Types** Store coordinates as VARCHAR → avoid import errors in MySQL

## DATA IMPORT PROCESS

### Step 1: Import into MySQL

Used Python script with `pandas` and `mysql-connector-python` for safe import of all 24 CSV files into `rides` table.

**Import Results:**

| Metric | Value |
|--------|-------|
| **Total Rows** | 11,413,351 |
| **Start Date** | 2024-01-01 00:00:39 |
| **End Date** | 2025-12-31 23:53:24 |
| **Casual Riders** | 4,151,023 (36.37%) |
| **Annual Members** | 7,262,328 (63.63%) |

**Conclusion:**
✅ Data completeness — 100% (all months, both years)
✅ Sufficient volume for seasonality and behavioral pattern analysis
✅ Members 63.63%, casual 36.37% — ideal distribution for marketing analysis

### Step 2: Add Calculated Columns

Using Python script, added these calculated columns:
- `ride_length_minutes` — ride duration in minutes
- `day_of_week` — day of week (1=Sunday, 7=Saturday)
- `hour_of_day` — hour of ride start (0-23)
- `month` — month number
- `year` — year
- `week_of_year` — week of year

### Step 3: Identify and Handle Outliers

#### Cancelled Rides (< 1 minute)
- **Found:** 278,131 rides
- **Created separate table:** `cancelled_rides`
- **Distribution:**
  - Members: 134,564 (48.3%)
  - Casual: 144,367 (51.7%)
- **Insight:** Casual riders cancel more often — important for conversion strategy

#### Long Rides (> 24 hours)
- **Found:** 13,136 rides
- **Created separate table:** `long_rides`
- **Distribution:**
  - Casual: 10,790 (82%)
  - Members: 2,346 (18%)
- **Max duration:** 1,574 minutes (26.23 hours)
- **Insight:** Casual riders keep bikes much longer; requires additional analysis

### Step 4: NULL Value Check

**Critical fields (for main analysis) — 0 NULL:**
- ride_id: 0 NULL ✅
- rideable_type: 0 NULL ✅
- started_at: 0 NULL ✅
- ended_at: 0 NULL ✅
- member_casual: 0 NULL ✅
- ride_length_minutes: 0 NULL ✅
- All calculated columns: 0 NULL ✅

**Non-critical fields (stations and coordinates):**
- start_station_name: 2,100,788 NULL (18.9%)
- start_station_id: 2,100,788 NULL (18.9%)
- end_station_name: 2,135,646 NULL (19.2%)
- end_station_id: 2,135,646 NULL (19.2%)
- end_lat: 173 NULL (0.002%)
- end_lng: 173 NULL (0.002%)

### Final Cleaned `rides` Table:

| Metric | Value |
|--------|-------|
| **Total Rows** | 11,122,047 |
| **Casual Riders** | 3,996,198 (35.93%) |
| **Annual Members** | 7,125,849 (64.07%) |

**⚠️ ANALYSIS LIMITATION — Missing Station Data:**

~20% of station name and ID data missing (2.1M rows). Impacts:
- ❌ Popular station analysis
- ❌ Geographic patterns
- ❌ Member vs casual routes

However **does NOT impact** behavioral analysis:
- ✅ Ride length analysis
- ✅ Day of week distribution
- ✅ Hourly distribution
- ✅ Seasonal trends
- ✅ Bike type preferences
- ✅ Member vs casual comparison (100% complete)

**Approach:** Analysis focuses on behavioral patterns and temporal trends (100% complete data). Geographic station analysis conducted separately on 80% complete data with limitation noted.

## TECHNICAL CHALLENGES AND SOLUTIONS

### MySQL Timeout Issues (Error 2013)

**Problem:**
Working with 11.4M row table frequently triggered `Error 2013: Lost connection to MySQL server during query`. Occurred during:
- Large UPDATE queries on entire table
- Complex SELECT queries with aggregation
- GUI tool usage (MySQL Workbench)

**Root Causes:**
- Standard MySQL timeouts (30 sec) insufficient for 11M+ rows
- Network timeouts on large result sets
- GUI interfaces have their own connection limits

**Solutions Implemented:**

1. ✅ **Batch processing by month (Python)** — most reliable
   - Instead of UPDATE all 11.4M rows simultaneously
   - UPDATE each month separately (100k–600k rows per batch)
   - Result: reliable, timeout-free

2. ⚠️ **Increase MySQL timeouts** — partially helps
   - `SET SESSION max_execution_time = 3600000`
   - Helps but doesn't fully solve

3. ✅ **Create indexes** — improves SELECT queries
   - Indexes on frequently used columns
   - SELECT queries execute significantly faster
   - **Downside:** indexes consume additional space (~1 GB for 3.2 GB table)

### Current MySQL Configuration:

| Parameter | Value |
|-----------|-------|
| **Data Size** | 3,178.92 MB (~3.2 GB) |
| **Index Size** | 1,069.00 MB (~1 GB) |
| **Total Size** | 4,247.92 MB (~4.2 GB) |
| **Row Count** | 11,122,047 |
| **Indexes** | ride_length_minutes, member_casual |

**Conclusion:** MySQL has inherent limitations for large analytical operations on 10M+ row datasets. Solution: adopt specialized OLAP databases like ClickHouse.

## MIGRATION TO CLICKHOUSE

### Solution Architecture:

**MySQL** (OLTP):
- ✅ Reliable source data storage
- ✅ 11.1M clean records + 278k cancelled + 13k long rides
- ✅ Backup for data verification
- ❌ Slow analytics on large volumes (timeouts, index overhead)

**ClickHouse** (OLAP):
- ✅ Specialized for analytical queries (GROUP BY, aggregations)
- ✅ Handles 11M+ rows without timeouts
- ✅ Columnar storage with excellent compression
- ✅ Monthly partitions for fast queries

### Loaded Tables in ClickHouse:

| Table | Purpose | Rows |
|-------|---------|------|
| `cyclistic.rides` | Main user behavior analysis | 11,122,047 |
| `cyclistic.cancelled_rides` | Cancelled ride analysis | 278,131 |
| `cyclistic.long_rides` | Long ride analysis | 13,136 |

## PERFORMANCE COMPARISON: MySQL vs ClickHouse

### Real Test Results on 11.1M Row Dataset:

| Test | MySQL | ClickHouse | Speedup |
|------|-------|-----------|---------|
| **Test 1: Casual vs Member** | 9.406 sec | 0.204 sec | **46x faster** ⚡ |
| **Test 2: By Day of Week (with AVG)** | TIMEOUT (30s) | 0.238 sec | **∞ (MySQL failed)** |
| **Test 3: By Month (GROUP BY multiple)** | TIMEOUT (30s) | 0.174 sec | **∞ (MySQL failed)** |
| **Test 4: Hourly Patterns** | TIMEOUT (30s) | 0.428 sec | **∞ (MySQL failed)** |

### Storage Efficiency:

| Database | Rows | Size | Compression |
|----------|------|------|-------------|
| **MySQL** (table + indexes) | 11.1M | 4.2 GB | baseline |
| **ClickHouse** (columnar) | 11.1M | 0.87 GB | **4.8x smaller** 💾 |

### Conclusion:

ClickHouse is ** essential** for analytics on 10M+ rows. MySQL OLTP architecture is fundamentally unsuitable for GROUP BY operations at this scale.

## ANALYZE & INSIGHTS

### Key Findings from Dashboard

After loading 11.1M rides into ClickHouse and building interactive Power BI dashboards, several critical behavioral patterns emerged:

**1. Volume & Seasonality**

Members generate **64% of all rides** and maintain consistent usage year-round. Casual riders (36%) show a **sharp summer peak** (May-September) with winter usage dropping almost to zero.

**Key Insight:** Members = stable recurring revenue; Casual = seasonal volatility

**2. Ride Duration**

- Casual riders: 19-22 minutes average
- Members: 11-12 minutes average

**Casual riders stay 1.5-2x longer.** This indicates different use cases: Members = commuting (quick), Casual = leisure/sightseeing (extended rides).

**3. Days & Time Patterns**

- **Weekdays:** Members dominate 2:1 (work commute)
- **Weekends:** Casual more active (recreation)
- **Peak hours:** Afternoon (12:00-18:00) for both groups
- **Morning:** Casual show low activity, Members active from 07:00

**4. Route Patterns & Geography**

- **Members (A→B):** Point-to-point routes (home-work, transit hubs: Ellis Ave, University Ave). **Utility commuting.**
- **Casual (A→A):** Round-trip recreational routes (parks, waterfronts: DuSable Lake, Streeter Dr, Millennium Park). **Leisure scenario.**

## ACT: MARKETING RECOMMENDATIONS

Based on data-driven insights, recommend these three initiatives to convert Casual → Members:

### TOP-3 Actions

**1. Flexible Membership Passes**

**Action:** Introduce a "Weekend Pass" and "Summer Pass" to remove purchase barriers for users who don't ride in winter or prefer weekends.

**Why it works:**
- Casual riders show sharp summer peak and near-zero winter usage
- Weekends are significantly more active than weekdays for casual users
- Flexible pricing removes financial barrier for seasonal users

**Expected impact:** Casual Traffic Conversion

**2. Seasonal Targeting & Savings Triggers**

**Action:** Launch a spring campaign ("3 summer months cover the whole year") for Casuals and send push alerts with personal savings calculations during peak activity.

**Why it works:**
- Casual riders are active throughout summer (May-September) — enough to pay for annual membership
- Personalized savings calculations are stronger than generic offers
- Push at peak activity catches users with right emotional intent

**Expected impact:** Targeted Retargeting with high conversion

**3. Dynamic Pricing for Casual Passes**

**Action:** Introduce peak-hour and per-minute surcharges on single rides to make the fixed Annual Pass significantly more attractive.

**Why it works:**
- Casual riders take longer trips (19-22 min) → more minutes = higher cost
- Peak hours (afternoon) have demand → can charge more
- When a single ride costs $7-8, annual pass at $39/month looks more attractive

**Expected impact:** Membership Value Growth

### Notes

- Long rides (long_rides table) and cancelled rides (cancelled_rides table) remain for future analysis
- Station data analysis limited to 80% completeness (20% NULL locations) but doesn't impact behavioral insights
- All recommendations work within current subscription model

