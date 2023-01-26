# black_scholes
Project to learn about Black Scholes Equation to price European non-dividend paying options

# Project Summary

#### Obtain risk free rate from German 10 year bond
```python
rf_ticker = yf.Ticker("DE0001135275:BUND")  # use 10 year German bond to emulate European risk free rate "DE0001135275:BUND" # 10 Year Treasury US "^TNX"
info = rf_ticker.info
r = (info['regularMarketPrice']) / 100  # obtaining risk free rate
```


#### Black Scholes Python Implementation
```python
def black_scholes(self, option_type, S0, K, T, r, sigma):
    # flag, S0, K, T, r, vol_old
    # step 1: define d1 and d2
    d1 = 1 / (sigma * np.sqrt(T)) * (np.log(S0 / K) + (r + sigma ** 2 / 2) * T)

    d2 = d1 - sigma * np.sqrt(T)

    nd1 = norm.cdf(d1)

    nd2 = norm.cdf(d2)

    n_d1 = norm.cdf(-d1)

    n_d2 = norm.cdf(-d2)

    try:
        if option_type == 'c':
            price = nd1 * S0 - nd2 * K * np.exp(-r * T)
        else:
            price = K * np.exp(-r * T) * n_d2 - S0 * n_d1
        return price
    except:
        print('Please confirm all parameters are correct')
 ```
 
#### Using Newton Raphson root finding method to find implied volatility where the partial derivative of the formula in terms of impled volatility (vega) is f'(x) 
 ```python
 def implied_vol(self, S0, K, T, r, market_price, flag='c', precision=0.00001):
    max_iter = 200  # max no. of iterations, no convergence = no defined zero with function specifed
    vol_old = 0.3  # initial guess

    for k in range(max_iter):
        bs_price = self.black_scholes(flag, S0, K, T, r, vol_old)
        Cprime = vega(flag, S0, K, T, r, vol_old) * 100  # one percent step change in volatility
        C = bs_price - market_price

        vol_new = vol_old - C / Cprime  # formula
        new_bs_price = self.black_scholes(flag, S0, K, T, r, vol_new)
        if abs(vol_old - vol_new) < precision or abs(new_bs_price - market_price) < precision:
            break
        vol_old = vol_new
    implied_vol = vol_new

    return implied_vol
 
 ```
 #### Plotting Black Scholes vs Strike Price -> Graph shows as strike increases call option price decreases
 ```python
 def graph_calc(ticker, option_type = 'c'):
    vol = Vol(ticker)
    tick = yf.Ticker(ticker)
    option_dates = tick.options
    expiration_date = option_dates[0]
    if option_type == 'c':
        df = tick.option_chain(expiration_date).calls
    elif option_type == 'p':
        df = tick.option_chain(expiration_date).puts

    df['Black_Scholes'] = (vol.black_scholes(option_type, get_market_price(ticker), df['strike'],
                                             calculate_T(str(expiration_date), str(current_date)), r,
                                             df['impliedVolatility']))

    plt.scatter(df['strike'], df['lastPrice'], s=25, c='blue', marker='o', label='Actual Price')
    plt.scatter(df['strike'], df['Black_Scholes'], s=25, c='red', marker='o', label='Black Scholes Price')
    plt.xlabel('Strike Price')
    plt.ylabel('Option Price')
    plt.title('Option Price Vs Strike Price')
    plt.legend()
    plt.show()
    
 graph_calc("RDS-A",'p') # Royal Dutch Shell plc is a European Non Dividend paying stock
 ```
Libraries Use: `scipy`, `numpy`, `matplotlib`, `py_vollib`, `yfinance`
 
 # Future Improvements
 * Learn to implement dividends to price dividend paying options
 * Account for fact that risk free rate and underlying asset price follows stochastic processes
 
