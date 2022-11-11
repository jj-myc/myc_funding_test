# myc_funding_test
For testing funding rate trades using the Mycelium perpetual swap exchange

### Tasks:
- Get Binance price data for tokens in the Myc perp swap pool
- Get Binance funding rate data from the same time period as the above price data\
- Get Mycelium price data
- Get Mycelium funding rate data

### Plan
1. Define the universe based on assets in the MLP pool: https://swaps.mycelium.xyz/dashboard
2. Get price data for MYC perp swaps
3. Get price data for that universe from binance for the last 2 months
4. Get funding rate data for same period and universe
5.  Put data through formula $(1)$ from here: https://www.notion.so/mycelium-eth/Funding-Rate-Arb-on-Myc-5c104b6ad0cd4a36858533617f9091de#4ae5517e3c004f77b7e386a4d500d814

### Task Notes:
1. Mycelium Universe:
	- `WETH, WBTC, LINK, UNI, USDC, USDT, DAI, FRAX, FXS, BAL, CRV`

2. Get data from Myc Perp swaps
	1. API docs: https://dev.api.tracer.finance/docs/#/Perpetual%20Swaps/getSwapsFundinRates
		1. For Funding rates request URL: https://dev.api.tracer.finance/trs/fundingRates
		2. For for Historical Prices: https://api.tracer.finance/trs/priceUpdates
	
MYC Funding Rate Notes
- Data for funding rate has endFundingRate and startFundingRate
- Raw data columns:
`     `| `token` | `symbol` | `endFundingRate` | `startFundingRate` | `utilization` | `timestamp`
	- The funding rate paid at the hour is the difference between `endFundingRate` and `startFundingRate` 
	- `endFundingRate` and `startFundingRate` units are 1000000 
  
$$ F^H_t = \frac{endFundingRate - startFundingRate}{1\ 000 \ 000} \ $$

- Funding rate is only updated if there is a trade that changes the value, therefore, if there is a value missing it is simply the previous value, price data is the same
- Funding rate data is only record from the 1st of October on the API, this will be the starting time for the rest of the data pulled

Funding data frames have the following headers:
- `index | timestamp | symbol | fundingRate`
	- index is a list of integers because: 
		- data for all tickers comes in the one df
		- easier to get df

MYC  Price Notes:
- raw data columns:
`index` | `token` | `symbol` | `price` | `timestamp` | `blockNumber` | `txnHash` 
	- `token` = token address
	- `timestamp` = unix timestamp string

- perp price data comes in pages of 100
- token query appear not to work
- price data comes with all tokens through it
- price data is only updated if there is a trade that changes the value, therefore, if there is a value missing it is simply the previous value as the price data is the same
- no way to get periodic data
	- all data needs to be pulled
	- threading required
- headers are:
	-  `timestamp | symbol | timestamp`

3. Get data for universe on Binance
	- perps on Binance can be denominated in BUSD and USDT so both will be used.
	- perps aren't available for stable tokens, i.e. there is no USDC-BUSD perp
	- therefore the binance universe is:
  
			- `BALUSDT, BTCBUSD, BTCUSDT, CRVUSDT, ETHBUSD, ETHUSDT, LINKBUSD, LINKUSDT, UNIBUSD, UNIUSDT`
note: 
		- there is no `CRVBUSD` perp
		- `ETHBUSD` and `ETHUSDT` are perps for `WETH`
		- same for BTC 
- for binance price data:
	- using `autotrader.AutoData` to pull price data at 8 hr interval
	- columns for raw data:
		`time` | `Open` | `High` | `Low` | `Close`
- for binance funding rate data:
	- using `autotrader.AutoData` and `ccxt`
	- columns for raw data:
		- `time` | `rate`
	
