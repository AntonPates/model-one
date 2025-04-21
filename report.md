### Numbers and facts

#### Training and inference
- 242 928 models were trained and evaluated for 482 companies from S&P 500. Several companies were not used because of lack of data or they were not in S&P 500 for the whole period of training and inference 2022-2024.
- 121464 models were trained with news sentiment parameter.
- Combinations of loss functions, activations, optimizers and technical indicators were trained with and without news sentiment parameter + baseline models without news sentiment parameter and without any technical indicators.  
- Loss functions {"mean_squared_error", "huber", "log_cosh"}
- Activations {"relu", "tanh", "leaky_relu", "elu"}
- Optimizers {"adam", "rmsprop", "nadam"}
- Technical indicators {"RSI", "OBV", "ADX", "MACD", "AD", "200_MA", "no_indicators"}
- Up to 9 servers (16 Ampere CPU cores and 32 GB RAM each) were used for training and inference simultaneously.

#### Inference
- 2 600 031 predictions of stock prices were made for each traded Thursday from Nov/1/2023 to Nov/1/2024.
- 50 981 models were used for inference.
- Threshold for loss value of model to make its way to the inference stage was set to 0.005.

#### News
- 1397474 news were used to train models.


#### Modeling
- Vanguard S&P 500 ETF (VOO) was picked as a benchmark.
- Most successful trading simulation outperformed the benchmark by 16%. (1 351 544.8 vs 1 510 182.29).
- Most successful trading simulation was done for companies which have at least 2% weight in S&P 500.
- We don't have any news for companies hban, ce. For jpm and mtb news are partialy missing.
- Excluded tickers ['pltr', 'dell', 'erie', 'kkr', 'crwd', 'gddy', 'vst', 'gev', 'solv', 'smci', 'deck']
- Models for gehc, gev, hsy, hubb, hum, hwm, ice, idxx, iex, iff, incy, intc, intu, invh, ip, ipg, iqv, kvue, solv, sw, vlto were not trained. Training was stopped because of lack of disk space to preserve models.

#### Inference
- 2 600 031 predictions of stock prices were made for each traded Thursday from Nov/1/2023 to Nov/1/2024.
- 50 981 models were used for inference.
- Threshold for loss value of model to make its way to the inference stage was set to 0.005.


#### Conclusion (preliminary)
- Models with news sentiment parameter performed worse than the models without news sentiment by 22.4%. 1 285 958.89 vs 1 510 182.29. 
- Models with min loss without news sentiment segregation performed slightly worse than models without news 1 493 827.71 vs 1 510 182.29.


### Description of what was done

#### Data preparation
- Close prices and volume for each company in S&P 500 was downloaded from Yahoo Finance with yfinance library.
- News for each company for each day starting from 2022-03-01 was downloaded with Alpha Vantage API [https://www.alphavantage.co](https://www.alphavantage.co/documentation/).
- Technical indicators were calculated for each company for each day starting from 2022-03-01 based data from Yahoo Finance.
- Before training models close prices and volumes were normalized with scaler from sklearn from 0 to 1. `scaler = MinMaxScaler(feature_range=(0, 1))`
- Technical indicators were normalized with scaler from sklearn from 0 to 1. `scaler = MinMaxScaler(feature_range=(0, 1))`
- News sentiment per each company was calculated as an average of sentiment scores for each news article for each day.
- Training data was split into train and validation with 80% and 20% of data respectively.
- Splits were not respcted.

#### Training and evaluation
- For each company 504 models were trained. With all combinations of loss functions {"mean_squared_error", "huber", "log_cosh"}, activations {"relu", "tanh", "leaky_relu", "elu"}, optimizers {"adam", "rmsprop", "nadam"}, technical indicators {"RSI", "OBV", "ADX", "MACD", "AD", "200_MA"} and without of technical indicators and news (with or without) sentiment parameter.

Example of model training command:
```
2024/11/23 15:14:30 INFO running modelone.py script cmd="/home/anton/code/modelone/modelone.py --ticker rmd --window_length 35 --epochs 20 --up_front_prediction_days 5 --amount_of_params 4 --amount_of_neurons 128 --optimizer adam --loss_function mean_squared_error --activation relu --tech_indicator OBV --without_news false --without_indicators false"
```

- Model consists of input layer, 3 LSTM layers with 128, 64 and 20 neurons each and output Dense layer. 
- Models were trained for 20 epochs. An epoch is when all training examples have been shown to the model once.
- We tried to train models to predict close price for 5 days ahead based on 35 days of historical data.
- Up to 9 servers (16 Ampere CPU cores and 32 GB RAM each) were used for training simultaneously. 
- Ansible was used to manage servers fleet.
- Training and elaluation were done for about 16 hours.

#### Inference
- 2 600 031 predictions of stock prices were made for each traded Thursday from Nov/1/2023 to Nov/1/2024.
- 50 981 models were used for inference.
- Threshold for loss value of model to make its way to the inference stage was set to 0.005.
- Up to 5 servers (16 Ampere CPU cores and 32 GB RAM each) were used for inference simultaneously.

#### Modeling
- Vanguard S&P 500 ETF (VOO) was picked as a benchmark.
- Most successful trading simulation outperformed the benchmark by 16%. (1 351 544.8 vs 1 510 182.29).
- Most successful trading simulation was done for companies which have at least 2% weight in S&P 500.
- We don't have any news for companies hban, ce. For jpm and mtb news are partialy missing.
- Excluded tickers ['pltr', 'dell', 'erie', 'kkr', 'crwd', 'gddy', 'vst', 'gev', 'solv', 'smci', 'deck'] - they were excluded/included in S&P 500 during the period of training and inference.
- Models for gehc, gev, hsy, hubb, hum, hwm, ice, idxx, iex, iff, incy, intc, intu, invh, ip, ipg, iqv, kvue, solv, sw, vlto were not trained. Training was stopped because of lack of disk space to preserve models.
- Trading simulation was done for each company for each Thursday from Nov/1/2023 to Nov/1/2024.
- At the start we have 1 000 000 USD deposit.
- Deposit was distributed between companies in S&P 500 with weights proportional to their weight in S&P 500.
- We can't use free cash of one company to buy another company.
- Each Thursday we make predictions for each company.
- If prediction is at least 1.5% higher/lower than today's close price we buy/sell some amount of shares of this company or do nothing depending on ADX technical indicator value on that day:
    - if ADX is lower than 25 we do nothing
    - if ADX is higher or equal 25 and lower 50  we buy/sell half of shares of this company
    - if ADX is higher or equal 50 we buy/sell all shares of this company
- We sell all shares of company at last Thursday of the trading period.

#### Conclusion
Answering the central research question: Can a Machine Learning model that integrates both technical analysis and news sentiment analysis outperform the traditional Buy-and-Hold strategy in real investment environments? 
Based on the results of the study the answer is No. On a growing market (about 32% growth) from Nov/1/2023 (S&P 500 index was 4567.81) to Nov/1/2024 (S&P 500 index was 6032.39) buy-and-hold S&P index strategy outperforms the ML-based approach by 23.4%.

We didn't find any significant difference in performance between four types of models (with news and with indicators, with news and without indicators, without news and with indicators, without news and without indicators). I believe that we can reconsider the source of the news or try several in our further research.

During our study we built a framework for building ML-based price prediction models and performed a trading simulation which will be used for further research.

However there are some points where we can improve the results of the ML-based approach:
- We can perform simulation more precisely gradually increasing training data set and performing training before making predictions for each date.
- We can use data for training for a longer period of time.
- We can use more technical indicators.
- We can use some LSTM-alternative models.
- We can try to reconsider the architecture of the model.



