# Nifty Correlation

### Correlation:
Correlation among index components. For instance, an index may have zero change for a day either because none of the components moved, or because half of the components rose while the other half fell. In the first case, the correlation would be very high, while in the second case the correlation would be very low. In other words, an index can have very low volatility in and of itself, while its components may actually be quite volatile independently.
The goal of this strategy is to identify when implied correlation is low, indicating that index option premiums are undervalued relative to that of stock options.

## Objective: 
To exploit the correlation between Nifty and it's constituents. This can be achieved by buying index option straddles and simultaneously selling straddles in the options of the index components on a weighted basis, for this version of the backtest.  

Ideally, with perfect correlation, the stocks should run up or fall equally with the index, in which case both our longs and shorts will make money. 
It is a delta neutral strategy. We trade the Implied correlation between the index and it's constituents.

Implied Correlation is the correlation between the implied volatilities of index options and the weighted implied volatilities of options on the index components.
When the ratio is low there is low correlation between the index and it's constituents and when the ratio is high there is high correlation between the index and it's constituents.
(Ideally, we want the ratio to be low when we are opening positions.)

## Backtest:

### Data used:
- EoD options data for Nifty and it's constituents from 2016 to 2021.
- Expiry dates
- Stock weights
- Strike Selection: We have used strikes closest to the delta (due to inavailablity of data/low liquidity in some stocks).

Data holes have been filled with the last available values for that month.
On Expiry day, intrinsic value has been calculated and used for in the money options and for out of the money options, option price is set to 0.05 .

### Leverage
10x 

### Position Sizing:
- In this backtest, we have assigned quantities based on the market caps of the stocks, using minimum share logic.
- Index exposure is calculated by dividing MinShareExposure by MinShareValue.
- Index exp= (minShareExp) / (MinShareValue)
- Individual stock exposure= (Index exp) / (stock weight)
- Individual stock qty= (Individual stock exposure)/ (Equity close of that stock)
- Index exposure and total stock exposure will be the same
- The margin required for 1 unit will be Index exposure + total stock exposure
- No of units = (equity available)/ (margin required for 1 unit)

MinShareExposure: Exposure required to open a position in the stock with the lowest index weightage.
MinShareValue: Equity price of the stock with the lowest index weightage.
### Entry: 
First day of the month 
### Exit: 
Monthly expiry 
### Adjustments: 
None 
### Optimisations: 
The backtest was run for all deltas in the range of 5 to 50.

[DeltaComparison.xlsx](https://github.com/qodeinvestments/Swan-Documentation/files/9683261/DeltaComparison.xlsx)

Database Path: \\iMAC2\F\All Databases\Extra\Nifty 50 Correlation EoD database
