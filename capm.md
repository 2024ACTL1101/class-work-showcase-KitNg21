
# CAPM Analysis

## Introduction

In this assignment, you will explore the foundational concepts of the Capital Asset Pricing Model (CAPM) using historical data for AMD and the S&P 500 index. This exercise is designed to provide a hands-on approach to understanding how these models are used in financial analysis to assess investment risks and returns.

## Background

The CAPM provides a framework to understand the relationship between systematic risk and expected return, especially for stocks. This model is critical for determining the theoretically appropriate required rate of return of an asset, assisting in decisions about adding assets to a diversified portfolio.

## Objectives

1. **Load and Prepare Data:** Import and prepare historical price data for AMD and the S&P 500 to ensure it is ready for detailed analysis.
2. **CAPM Implementation:** Focus will be placed on applying the CAPM to examine the relationship between AMD's stock performance and the overall market as represented by the S&P 500.
3. **Beta Estimation and Analysis:** Calculate the beta of AMD, which measures its volatility relative to the market, providing insights into its systematic risk.
4. **Results Interpretation:** Analyze the outcomes of the CAPM application, discussing the implications of AMD's beta in terms of investment risk and potential returns.

## Instructions

### Step 1: Data Loading

- We are using the `quantmod` package to directly load financial data from Yahoo Finance without the need to manually download and read from a CSV file.
- `quantmod` stands for "Quantitative Financial Modelling Framework". It was developed to aid the quantitative trader in the development, testing, and deployment of statistically based trading models.
- Make sure to install the `quantmod` package by running `install.packages("quantmod")` in the R console before proceeding.

```r
# Set start and end dates
start_date <- as.Date("2019-05-20")
end_date <- as.Date("2024-05-20")

# Load data for AMD, S&P 500, and the 1-month T-Bill (DTB4WK)
amd_data <- getSymbols("AMD", src = "yahoo", from = start_date, to = end_date, auto.assign = FALSE)
gspc_data <- getSymbols("^GSPC", src = "yahoo", from = start_date, to = end_date, auto.assign = FALSE)
rf_data <- getSymbols("DTB4WK", src = "FRED", from = start_date, to = end_date, auto.assign = FALSE)

# Convert Adjusted Closing Prices and DTB4WK to data frames
amd_df <- data.frame(Date = index(amd_data), AMD = as.numeric(Cl(amd_data)))
gspc_df <- data.frame(Date = index(gspc_data), GSPC = as.numeric(Cl(gspc_data)))
rf_df <- data.frame(Date = index(rf_data), RF = as.numeric(rf_data[,1]))  # Accessing the first column of rf_data

# Merge the AMD, GSPC, and RF data frames on the Date column
df <- merge(amd_df, gspc_df, by = "Date")
df <- merge(df, rf_df, by = "Date")
```

#### Data Processing 
```r
colSums(is.na(df))
# Fill N/A RF data
df <- df %>%
  fill(RF, .direction = "down") 
```

### Step 2: CAPM Analysis

The Capital Asset Pricing Model (CAPM) is a financial model that describes the relationship between systematic risk and expected return for assets, particularly stocks. It is widely used to determine a theoretically appropriate required rate of return of an asset, to make decisions about adding assets to a well-diversified portfolio.

#### The CAPM Formula
The formula for CAPM is given by:

$$
E(R_i) = R_f + \beta_i (E(R_m) - R_f)
$$

Where:

- $E(R_i)$ is the expected return on the capital asset,
- $R_f$ is the risk-free rate,
- $\beta_i$ is the beta of the security, which represents the systematic risk of the security,
- $E(R_m)$ is the expected return of the market.



#### CAPM Model Daily Estimation

- **Calculate Returns**: First, we calculate the daily returns for AMD and the S&P 500 from their adjusted closing prices. This should be done by dividing the difference in prices between two consecutive days by the price at the beginning of the period.
  
$$
\text{Daily Return} = \frac{\text{Today's Price} - \text{Previous Trading Day's Price}}{\text{Previous Trading Day's Price}}
$$

```r
#fill the code
daily_estimations_df <- df
daily_estimations_df$daily_return_AMD <- NA
daily_estimations_df$daily_return_gspc <- NA
for (i in 2:nrow(daily_estimations_df)) {
 daily_estimations_df$daily_return_AMD[i] <- (amd_df$AMD[i]-amd_df$AMD[i-1])/(amd_df$AMD[i1])
 daily_estimations_df$daily_return_gspc[i] <- (gspc_df$GSPC[i]-gspc_df$GSPC[i-1])/(gspc_df$G
SPC[i-1])
}
```

- **Calculate Risk-Free Rate**: Calculate the daily risk-free rate by conversion of annual risk-free Rate. This conversion accounts for the compounding effect over the days of the year and is calculated using the formula:
  
$$
\text{Daily Risk-Free Rate} = \left(1 + \frac{\text{Annual Rate}}{100}\right)^{\frac{1}{360}} - 1
$$

```r
#fill the code
daily_estimations_df$risk_free_rate <- NA
for (i in 1:nrow(daily_estimations_df)) {
 daily_estimations_df$risk_free_rate[i] <- (1 + df$RF[i]/100)^(1/360)-1
}
```


- **Calculate Excess Returns**: Compute the excess returns for AMD and the S&P 500 by subtracting the daily risk-free rate from their respective returns.

```r
#fill the code
daily_estimations_df$excess_returns_AMD <- NA
daily_estimations_df$excess_returns_gspc <- NA
for (i in 1: nrow(daily_estimations_df)) {
 daily_estimations_df$excess_returns_AMD <- daily_estimations_df$daily_return_AMD - daily_es
timations_df$risk_free_rate
 daily_estimations_df$excess_returns_gspc <- daily_estimations_df$daily_return_gspc - daily_
estimations_df$risk_free_rate
}
```


- **Perform Regression Analysis**: Using linear regression, we estimate the beta (\(\beta\)) of AMD relative to the S&P 500. Here, the dependent variable is the excess return of AMD, and the independent variable is the excess return of the S&P 500. Beta measures the sensitivity of the stock's returns to fluctuations in the market.

```r
#fill the code
Model_beta <- lm(daily_estimations_df$excess_returns_AMD ~ daily_estimations_df$excess_return
s_gspc, data = daily_estimations_df)
summary(Model_beta)
beta <- coef(Model_beta[2])
cat("The beta coefficient is: ", Model_beta$coefficients[2][1])
```


#### Interpretation

What is your \(\beta\)? Is AMD more volatile or less volatile than the market?

**Answer:**
I found my beta (from the previous part) to be 1.569999. Because the value of beta is greater than 1, it indicates that AMD’s stock is significantly more volatile than the overall market. For instance, if the market moves by 1%, AMD is expected to move by approximately 1.5530%. This higher volatility implies higher risk and potential for higher returns, aligning with the expectations of more aggressive investors.


#### Plotting the CAPM Line
Plot the scatter plot of AMD vs. S&P 500 excess returns and add the CAPM regression line.

```r
#fill the code
plot<-ggplot(daily_estimations_df,aes(x=daily_estimations_df$excess_returns_gspc, y=daily_est
imations_df$excess_returns_AMD))+
 geom_point()+
 geom_smooth(method="lm",col="red")+
 labs(title="AMD Vs S&P 500",
 x="S&P 500",
 y="AMD")
plot
```

### Step 3: Predictions Interval
Suppose the current risk-free rate is 5.0%, and the annual expected return for the S&P 500 is 13.3%. Determine a 90% prediction interval for AMD's annual expected return.



**Answer:**

```r
#fill the code
# Extract beta and standard error of the forecast
sf <- summary(Model_beta)$sigma
beta <- coef(Model_beta)[2]
# Convert daily standard error to annual
annual_sf <- sf * sqrt(252)
# Suppose current risk-free rate (Rf) is 5% and expected return for S&P 500 (Rm) is 13.3%
Rf <- 0.05
Rm <- 0.133
# Calculate expected return for AMD using CAPM
E_Ri <- Rf + beta * (Rm - Rf)
# Calculate prediction interval
alpha <- 0.10
# Degrees of freedom: n-2
t_value <- qt(1 - alpha / 2, df = nrow(daily_estimations_df) - 2)
lower_bound <- E_Ri - t_value * annual_sf
upper_bound <- E_Ri + t_value * annual_sf
cat("The annual expected return of AMD is:",E_Ri,"\nThe 90% prediction interval for AMD's ann
ual expected return is: [", lower_bound, ", ", upper_bound, "]\n")
```
