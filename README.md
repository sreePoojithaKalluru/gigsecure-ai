# AI-Insurance-Swiggy
# AI-Powered Income Protection for Swiggy Delivery Partners

## 1. Introduction

Swiggy delivery partners play an important role in our daily lives. However, their income is highly dependent on external conditions such as weather, pollution, and local disruptions. On days with heavy rain, extreme heat, or poor air quality, delivery activity drops significantly, leading to loss of income.

Currently, there is no proper system to support delivery partners during such situations. This project aims to solve that problem by providing a simple and automated income protection system.

---

## 2. Problem Statement

Delivery partners often lose 20–30% of their income due to conditions beyond their control, such as:

* Heavy rainfall
* Extreme temperatures
* High pollution levels
* Sudden local restrictions

These events directly affect their ability to work, and there is no financial safety net available for them.

---

## 3. Proposed Solution

We propose an AI-powered parametric insurance platform designed specifically for Swiggy delivery partners.

The system automatically monitors external conditions such as weather and air quality. When certain predefined conditions are met, it triggers a payout to the delivery partner without requiring any manual claim process.

The main goal is to ensure that delivery partners receive compensation for income loss in a simple, fast, and transparent way.

---

## 4. Target User (Persona)

Name: Ramesh
Occupation: Swiggy Delivery Partner
Location: Hyderabad

Ramesh works daily to earn his income through deliveries. On days with heavy rain or extreme heat, he is unable to work efficiently or sometimes cannot work at all. As a result, he loses a significant portion of his daily earnings.

This system is designed to support users like Ramesh by providing financial protection during such conditions.

---

## 5. System Workflow

The working of the system is as follows:

1. The delivery partner registers on the platform
2. The system collects basic details such as location and work pattern
3. AI-based risk analysis is performed using historical and real-time data
4. A suitable weekly premium is assigned to the user
5. The system continuously monitors external data (weather, AQI, etc.)
6. If a predefined threshold is crossed, a claim is automatically triggered
7. The payout is processed instantly and sent to the user

This ensures a seamless and zero-effort experience for the delivery partner.

---

## 6. Weekly Premium Model

The pricing model is designed on a weekly basis to match the earning cycle of gig workers.

The premium varies depending on the risk level of the area:

* Low Risk Area: ₹20 per week
* Medium Risk Area: ₹35 per week
* High Risk Area: ₹50 per week

The system dynamically adjusts the premium based on factors such as weather history, environmental conditions, and location-based risks.

---

## 7. Parametric Triggers

The platform uses predefined conditions to automatically trigger payouts. These include:

* Rainfall exceeding a certain limit (e.g., 50 mm)
* Temperature crossing extreme levels (e.g., above 45°C)
* Air Quality Index (AQI) reaching hazardous levels (e.g., above 300)

When any of these conditions are met, the system automatically initiates the payout process.

---

## 8. AI/ML Integration

Artificial Intelligence is used in the following ways:

* Risk Assessment: Analyzing historical and real-time data to identify high-risk areas
* Dynamic Pricing: Adjusting weekly premiums based on risk levels
* Fraud Detection: Preventing misuse through location validation and activity checks

This makes the system more accurate, efficient, and reliable.

---

## 9. Technology Stack

Frontend: React.js
Backend: Node.js / FastAPI
Database: MongoDB
APIs: Weather API, AQI API
Payment Integration: Razorpay (Test Mode)

---

## 10. Development Plan

The development will be carried out in phases:

* Phase 1: Ideation, planning, and system design
* Phase 2: Core development including registration, pricing, and claims
* Phase 3: Advanced features such as fraud detection and dashboards

---

## 11. Conclusion

This project focuses on providing a practical solution to a real-world problem faced by Swiggy delivery partners. By using AI and automation, the system ensures quick and fair compensation for income loss caused by external conditions.

The approach is simple, scalable, and designed to create a reliable safety net for gig workers.
