# Cyclistic Bike-Share Analysis Project

Complete end-to-end analysis of 11.1M bike-share trips to identify behavioral differences between casual riders and annual members.

## 🚀 Quick Start

**For Russian Speakers (Русский язык):**

Open RU/cyclistic_report.md

Read sections: БИЗНЕС-ЗАДАЧА → ПОДГОТОВКА → ПРОЦЕСС → АНАЛИЗ → ДЕЙСТВИЯ

View interactive dashboard: RU/Cyclistic Rus.pdf

**For English Speakers:**

Open EN/cyclistic_report.md

Read sections: BUSINESS TASK → PREPARATION → DATA CLEANING → ANALYSIS → RECOMMENDATIONS

View interactive dashboard: EN/Cyclistic En.pdf

## 📊 Key Findings

✅ Members = 64% of rides, stable year-round

✅ Casual = 36% of rides, summer peak, winter near-zero

✅ Casual stay 1.5-2x longer (leisure vs commuting)

✅ MySQL timeout problem solved with ClickHouse (46x faster)

✅ 3 actionable recommendations for member conversion

## 🛠 Technologies Used

Component	Technology

Data Processing	Python, Pandas, MySQL

Analytics	ClickHouse (OLAP), Power BI

Documentation	Markdown (bilingual)

Dashboard	Power BI Desktop

## 📈 Analysis Stages

**1. Ask (Бизнес-задача)**

Identify behavior differences between casual riders and members

Goal: Develop conversion strategy

**2. Prepare (Подготовка)**

Source: 24 monthly CSV files (2024-2025)

Volume: 11.4M bike trips

Data validation: ROCCC framework

**3. Process (Процесс)**

Import to MySQL

Add calculated columns (ride_length, day_of_week, etc)

Handle outliers (cancelled rides, long rides)

Check NULL values (18-20% in station data)

**4. Analyze (Анализ)**

Migrate to ClickHouse (performance 46x-infinite improvement)

Create interactive Power BI dashboards

Identify behavioral patterns:

Seasonality

Duration differences

Route patterns (commuting vs leisure)

**5. Share (Визуализация)**

3-page interactive Power BI dashboard

Bilingual documentation

Executive-ready insights

**6. Act (Рекомендации)**

TOP-3 strategies for casual → member conversion

Seasonal campaigns

Targeted retargeting

Membership value growth

## 💡 Technical Highlights

Problem Solved: MySQL Timeouts

Issue: Error 2013 on 11M+ row queries

Solution: Migrate to ClickHouse OLAP

Result: 46x-infinite speed improvement

Data Completeness

✅ Critical fields: 100% complete

⚠️ Station data: 80% complete (20% NULL)

Impact: No effect on behavioral analysis

File Sizes

Original MySQL: 4.2 GB (3.2 GB data + 1 GB indexes)

ClickHouse: 0.87 GB (4.8x compression)

## 🎯 Recommendations Summary

1. Flexible Membership Passes

Introduce "Weekend Pass" and "Summer Pass" to remove purchase barriers for seasonal users

2. Seasonal Targeting & Savings Triggers

Launch spring campaign ("3 summer months cover the whole year") with personalized savings calculations

3. Dynamic Pricing for Casual Passes

Introduce peak-hour and per-minute surcharges to make annual pass more attractive

## This project demonstrates:

End-to-end data analysis (Ask → Act)
Real-world problem solving (MySQL timeouts)
Large-scale data handling (11M+ rows)
Executive communication (dashboards + recommendations)
Bilingual technical documentation

Last Updated: August 2026
