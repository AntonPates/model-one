## Stock market prediction research

This project explores the effectiveness of machine learning models for stock price prediction by combining technical indicators and news sentiment analysis.

### Key Takeaways:  
- Incorporating news sentiment into forecasting models might improve the accuracy of forecast.  
- Hyperparameter tuning play a crucial role in enhancing model precision.  
- Despite the strong predictive accuracy, consistently outperforming the S&P 500’s Buy-and-Hold strategy remained a challenge emphasizing the importance of continuous model adaptation in real-world environments.


### Overview

- Trained 242 928 models for 482 S&P 500 companies
- Used LSTM neural networks with various configurations
- Incorporated technical indicators and news sentiment data
- Simulated trading strategies from Nov/2023 to Nov/2024

### Data Sources

- Stock prices: Yahoo Finance (yfinance library)
- News sentiment: Alpha Vantage API
- Technical indicators: Calculated from price data

### Technical Details

- Model architecture: 3 LSTM layers (128, 64, 20 neurons) + Dense output layer
- Training period: 2022-03-01 to 2024-11-01
- Prediction horizon: 5 days ahead
- Input window: 35 days of historical data

[Breif report](https://github.com/AntonPates/model-one/blob/main/report.md)

---
Some pieces of code are in [train.ipynb](https://github.com/AntonPates/model-one/blob/main/train.ipynb)