---
layout: page
title: Short-Term Solar Irradiance Forecasting and Weather Analysis using Gridded Data
display-order: 4
---
While solar energy has definite advantages over conventional power sources such as oil and coal, its
unpredictable nature poses a number of potential problems for the electrical grid. With conventional
generators, grid operators can relatively easily increase production to meet demand. However, the
unpredictable nature of solar energy can make it difficult to properly balance production and demand
which can lead to grid instability. Accurate solar irradiance forecasts can reduce the
unpredictability and help to ensure a stable grid.

In this paper[^paper] we present a method for short-term solar irradiance forecasting using gridded
global horizontal irradiance (GHI) data, estimated from satellite images. We use this data to first
create a simple linear regression model with a single predictor variable. We then discuss various
methods to extend and improve the model. We found that adding predictor variables and partitioning
the data to create multiple models both reduced prediction errors under certain circumstances.
However, both these techniques were outperformed by applying a data transformation before training
the linear regression model.

We also discuss a set of methods for identifying “rare weather events” by discretizing Global
Forecast System (GFS) data by transforming the data using either principal component analysis or a
discrete wavelet transform, then discarding and round the transformed data. While the current work
only investigates identifying past events, future work could investigate predicting these events and
using those predictions to improve solar irradiance forecasts.

[^paper]: This page is simply the abstract of the final report for my Master's degree project. The
    full report is available as a
    [PDF]({{ site.baseurl }}{% link projects/solar_irradiance_forecasting.pdf %}).
