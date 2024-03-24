# MTGO-Buy-Signals-Analysis
Bootcamp Project 4

## Overview and Purpose
The goal of this project is to apply a Machine Learning Model to a dataset and use it to forecast. Here I use Python, JSON, and SQL to build a custom-created dataset from over 100 Magic the Gathering Online tournaments, encompassing thousands of cards. I then use a Python script to initialize, train, and evaluate an ARIMA model to predict pricing trends for specific cards, finally yielding a suggested buylist analysis to profit from the model. 

This project builds upon my previous Decklist Analysis Tool from project 3, which can be found here: https://github.com/Koskadelli/Magic-Decklist-Analysis-Tool. Pricing visualizations that triggered this investigation are found on my Public Tableau: https://public.tableau.com/app/profile/davin.frankosky1360/viz/MTGMetagameAnalysis/MTGO5-0LeagueAnalysis?publish=yes.

## Dataset and Process Summary
### Dataset Building: Previous Work from Project 3
1) Use Python with BeautifulSoup to scrape card data from https://www.mtgo.com/decklists, (specifically format leagues) for all of January 2024. These are all decks that went 5-0.
2) Incorporate online card pricing information from https://www.goatbots.com.
3) Store, organize, and clean the data using Pandas Dataframes. Export as CSVs and build a Postgres SQL database, named MTG_Metagame. 
![ERD for my SQL Database](https://github.com/Koskadelli/MTGO-Buy-Signals-Analysis/blob/main/Images/MTG%20DB%20ERD.png?raw=true)

### Hypothesis for Project 4
For any specific card, multiple events in a row where the usage is higher than the previous week's average number of copies triggers a buy signal if the price does not show a significant fluctuation. 
![Buy Indicator Example Chart](https://github.com/Koskadelli/MTGO-Buy-Signals-Analysis/blob/main/Images/Buy_indicator_example.png?raw=true) 

### Data Modeling: New Work for Project 4
1) Import a subset of data from my Postgres database using psycopg2 and an SQL query. I specifically wanted the EventDate, CardID,	Name, TotalCopies (across Maindeck and Sideboard), and the PriceOnEventDay. The SQL query also specifically lists cards for each day even if they did not appear in a tournament that day, because I needed to capture the price on that day and the fact that 0 copies were played.
![SQL Query](https://github.com/Koskadelli/MTGO-Buy-Signals-Analysis/blob/main/Images/SQL_query.png?raw=true) 
2) Clean the Dataframe:
    * Drop any rows with NaN data
    * Update dtypes
    * Drop unncessary columns after using them to QA that my SQL query was correct
    * Limit the dataset to only use cards that are somewhat regularly played (at least 100 copies across all tournaments in January)
    * Limit the dataset to only cards worth more than $1 on average
3) Research and select a Model to apply. For this project, I used an ARIMA model as it worked well for my data for the following reasons:
    * Perfect for Short term forecasting
    * My dataseries is consistent and stationary
    * My data is not impacted (much) by seasonal data
    * The model has a history of being used, with highly accurate success, in financial markets and sales forecasts
4) Split the data into training and testing subsets, and applied the ARIMA model. An example of it being applied for a particular card:
    * Exogeneous Variable Selection
        + A variable that originates outside the model and is not influenced by the variables within the model.
        + I infer that a card’s usage in tournaments is independent of its price. This is generally true in high-level Magic play.
        + Use this variable to explain/predict the dependent variable (price).
    * Choosing the order for the model. Defined as order = (p, d, q).
        + p: Autoregressive (AR) order. p = 1 means the model includes one lagged value. 
        + d: Differencing order. This indicates how many times the data have been differenced to achieve stationarity. (Differencing is a method to remove trends or seasonal structures).
        + q: Moving Average (MA) order. This represents the number of lagged forecast errors included in the model. (This component helps capture the shock or surprise elements in the past forecast errors)        
![ARIMA Example](https://github.com/Koskadelli/MTGO-Buy-Signals-Analysis/blob/main/Images/ARIMA_example.png?raw=true)  
5) Results were highly accurate across many examples. For the above:
    * Accuracy: Calculated via Mean Absolute Percentage Error, or mape.
    * Buy Target: if the max predicted price is higher than the final day price of the training data, return True.
![ARIMA Example Prediction](https://github.com/Koskadelli/MTGO-Buy-Signals-Analysis/blob/main/Images/Example_Prediction.png?raw=true) 
![ARIMA Example Prediction Chart](https://github.com/Koskadelli/MTGO-Buy-Signals-Analysis/blob/main/Images/Train_Test_Comparison_Chart.png?raw=true) 
6) Optimization: In the Python code, various optimization notes are commented. This includes limiting the tournament format for the dataset, the cutoffs for played and price thresholds, as well as maniuplating the order of the ARIMA model. I alsoused the pmdarima Python package to utilize the auto_arima function. This was used for educational purposes, but was not necessary to apply to every card as the model was already more than accurate enough, and it generally recommended close to what I was already using. 
7) Applied the model to every card in the dataset. Model summary across all cards: 
![ARIMA Summary](https://github.com/Koskadelli/MTGO-Buy-Signals-Analysis/blob/main/Images/Accuracy_Summary.png?raw=true)
8) Projected the model 3-days into the future past where the training dataset ended, and then researched if its recommendations would have yielded a profit
![Buylist Projection Outcomes](https://github.com/Koskadelli/MTGO-Buy-Signals-Analysis/blob/main/Images/Buylist_Outcomes.png?raw=true)
![Buylist Outcomes Summary](https://github.com/Koskadelli/MTGO-Buy-Signals-Analysis/blob/main/Images/Buylist_Outcomes_Summary.png?raw=true)


## Datasource References
1) MTGO Decklists: https://www.mtgo.com/decklists
2) MTGO Card Pricing Information: https://www.goatbots.com
3) Followed Wizards of the Coast Fan Content policy: https://company.wizards.com/en/legal/fancontentpolicy
4) What Is ARIMA Modeling resource article: https://www.mastersindatascience.org/learning/statistics-data-science/what-is-arima-modeling/#:~:text=An%20ARIMA%20model%20order%20is,of%20the%20data%20over%20time.
5) Accuracy Measures resource article: https://intuendi.com/resource-center/forecast-accuracy/#:~:text=Mean%20Absolute%20Error%20(MAE)%3A,the%20direction%20of%20the%20errors.
6) ARIMA Model application for stocks resource article and picture for Powerpoint: https://blog.quantinsti.com/forecasting-stock-returns-using-arima-model/ 

## Coding References
Various coding assistance throughout my Python via ChatGPT: https://chat.openai.com/. 