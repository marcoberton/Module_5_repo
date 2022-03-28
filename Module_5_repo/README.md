# Module_5_repo
Challenge for Module 5
## Creating Two Financial Planners, One for Emergencies and One for Retirement
# Import libraries and dependencies
# Import the required libraries and dependencies
import os
import requests
import json
import pandas as pd
from dotenv import load_dotenv
import alpaca_trade_api as tradeapi
from alpaca_trade_api.rest import REST, TimeFrame
from MCForecastTools import MCSimulation

%matplotlib inline

# Set your Alpaca API Keys
alpaca_api_key = os.getenv("ALPACA_API_KEY")
alpaca_secret_key = os.getenv("ALPACA_SECRET_KEY")

alpaca = tradeapi.REST(
    alpaca_api_key,
    alpaca_secret_key,
    api_version="v2")
    
# Set tickers and start + end dates along with timeframe, need to update to "get_bars" format
tickers = ["SPY", "AGG"]
timeframe = "1Day"
start_date = pd.Timestamp("2020-08-07", tz="America/New_York").isoformat()
end_date = pd.Timestamp("2020-08-07", tz="America/New_York").isoformat()
limit_rows=1000
# This is how I had to format the new tables, if there is a better way update this ReadMe
prices_df_spy = alpaca.get_bars("SPY", timeframe, start=start_date, end=end_date, limit=limit_rows).df
column_names = [("SPY", x) for x in prices_df_spy.columns]
prices_df_spy.columns = pd.MultiIndex.from_tuples(column_names)

prices_df_agg = alpaca.get_bars("AGG", timeframe, start=start_date, end=end_date, limit=limit_rows).df
column_names = [("AGG", x) for x in prices_df_agg.columns]
prices_df_agg.columns = pd.MultiIndex.from_tuples(column_names)
prices_df_agg.head()

prices_merged = pd.merge(prices_df_spy,
                         prices_df_agg,
                         how="inner",
                         left_index=True,
                         right_index=True)
                         
# Store closing prices for later df
agg_close_price = float(prices_merged["AGG"]["close"])
spy_close_price = float(prices_merged["SPY"]["close"])

# Calculate the total value of the member's entire savings portfolio

# Make a savings df to use for MonteCarlo calculations later
savings_df = pd.DataFrame(
    {"amount": [total_crypto_wallet, total_stocks_bonds]},
     index=['crypto', 'stock/bond']
    )
    
# Make a new df to use for MonteCarlo calculations (don't forget to make new Timeframe variables)
prices_mc_spy = alpaca.get_bars("SPY", timeframe, start=start_date_mc, end=end_date_mc, limit=limit_rows).df
column_names_mc = [("SPY", x) for x in prices_mc_spy.columns]
prices_mc_spy.columns = pd.MultiIndex.from_tuples(column_names)

prices_mc_agg = alpaca.get_bars("AGG", timeframe, start=start_date_mc, end=end_date_mc, limit=limit_rows).df
column_names_mc = [("AGG", x) for x in prices_mc_agg.columns]
prices_mc_agg.columns = pd.MultiIndex.from_tuples(column_names)

prices_merged_mc = pd.merge(prices_mc_spy,
                         prices_mc_agg,
                         how="inner",
                         left_index=True,
                         right_index=True)
                         
# Run your MC calculations & calculate cumulative return
MC_thirty_year = MCSimulation(
    portfolio_data = prices_merged_mc,
    weights = [0.6, 0.4],
    num_simulation = 500,
    num_trading_days = 252*30
)
MC_thirty_year.portfolio_data.head()

MC_thirty_year.calc_cumulative_return()

# Visualize the data using line plots & histograms

# Generate summary statistics from the 30-year Monte Carlo simulation results
thirty_year_table = MC_thirty_year.summarize_cumulative_return()

# Use the lower and upper 95% confidence intervals to calculate the range of the possible outcomes for the current stock/bond portfolio
ci_lower_thirty_cumulative_return = round(thirty_year_table[8]*total_stocks_bonds,2)
ci_upper_thirty_cumulative_return = round(thirty_year_table[9]*total_stocks_bonds,2)
Print the result of your calculations

# Configure a Monte Carlo simulation to forecast 10 years cumulative returns with an 80-20% split instead

# Plot the results

# Use the lower and upper 95% confidence intervals to calculate the range of the possible outcomes for the current stock/bond portfolio