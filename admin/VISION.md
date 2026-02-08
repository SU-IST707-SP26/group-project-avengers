# VISION.md

## Current Vision

### Project: NYC Yellow Taxi Analytics - Trip Duration and Congestion Pricing Prediction

**Primary Stakeholder:** 
- City transportation agencies responsible for managing congestion and evaluating transportation policies
- Taxi drivers and fleet operators whose earnings and operational decisions depend on trip duration and pricing

**Secondary Stakeholders:** 
- Riders and city residents affected by congestion pricing and travel times
- Urban researchers and planners studying mobility patterns and policy impacts
- Local businesses understanding foot traffic patterns

**Problem Statement:**
New York City residents, visitors, taxi drivers, and transportation planners currently lack predictive insights into taxi trip duration and the factors influencing congestion pricing exposure. While aggregate statistics exist at the precinct level, there is limited ability to predict trip outcomes based on temporal, spatial, and trip-specific factors. With the newly introduced Central Business District (CBD) congestion pricing fee in January 2025, understanding when and where these charges apply has become critical for both operational efficiency and policy evaluation.

**Solution:**
We will build machine learning models that predict taxi trip duration and identify conditions under which congestion pricing is applied at the trip level in NYC, incorporating temporal patterns, spatial features, and socioeconomic factors. The models will provide predictions using baseline approaches (linear regression for trip duration, logistic regression for congestion pricing) and advanced machine learning methods (random forests and gradient boosting) to capture non-linear relationships. The system will help stakeholders understand urban mobility patterns, evaluate the early impacts of congestion pricing policy, and make more informed decisions about travel planning and resource allocation.

**Data Source:**
NYC Yellow Taxi Trip Records for January 2025, published by the New York City Taxi and Limousine Commission, containing millions of trip records with temporal features (pickup/dropoff times), spatial features (location IDs), trip characteristics (distance, passenger count), fare components, and the newly introduced CBD congestion fee.

Key Outcomes:
- Regression model predicting trip duration (evaluated using RMSE and MAE)
- Classification model predicting congestion pricing exposure (evaluated using precision, recall, and ROC-AUC)
- Feature importance analysis identifying key drivers of trip duration and congestion pricing
- Interpretable insights for policy evaluation and operational decision-making
---

## Version History
No previous versions - Initial project vision established February 2025
