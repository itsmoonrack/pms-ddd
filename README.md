## Performance and Transactions Domain

### Anatomy of a portfolio
Every portfolio is composed of transactions each of which reference _securities_,of which are represented by a _Ticker Symbol_. This is what you see in the **Overview**,**Fundamentals**, and **Performance** views. The symbol for a security may represent acompany's _Common Stock_, such as GOOG for Google. Or it may represent different classes of shares of a company's common stock, such as BRK.A and BRK.B for Berkshire Hathaway, or an _Exchange-Traded Fund_, _Mutual fund_, or anything else that you can own shares of. Technically, a portfolio can contain a stock _Index_ like the Dow (.DJI) or NASDAQ (.IXIC), but that is just a convenience so you can compare your securities' performance against the broader market; we will not discuss stock indexes further. We start with the fundamental unit of portfolios, the transaction.

#### Transactions
**A transaction is assumed to have the following values that you can set:**

_Share count_: the number of shares referenced by the transaction. This can be zero for a "watchlist" item, that is, a stock that you added to your portfolio just to keep an eye on its performance, not because you own any shares of it.

_Cost per share_: the cost to purchase each share, in the currency of the exchange on which the share is traded. In the case of cash deposits or withdrawals, this is just the amount of the transaction.

_Commission_: the cost to execute the transaction with a broker

_Type_: One of "Buy", "Sell", "Sell Short", "Buy to Cover", "Deposit cash", "Withdraw cash", "Dividend", or "Split". Dividends and splits are computed automatically based on the traded company's history; you cannot set these yourself.

_Share count_, cost per share, and commission are all optional. If you leave any blank, we treat it as zero.

**A transaction also has certain values that are set automatically based on its traded company:**

_Share price_: the trading price of the share at the time computations are performed, in the currency of the company's stock exchange

_Price change_: the percentage change in the trading price of the share since the market open

_Dividend value_: For dividend transactions, this is the amount of the dividend per share of the stock

**Finally, there are values that are derived from those we have looked at already:**

_Transaction-adjusted share count_: this is the number of shares in the transaction, as of the time of the computation, based on the company's split history. For example, if you purchased 100 shares of a stock that then split 2:1 (meaning that you receive two shares in exchange for every individual share you own), this value would be 200.

_Event-adjusted share count_: this is like the transaction-adjusted share count, but instead of being split-adjusted to the present, it is adjusted to the time of a relevant event, such as a split or dividend issue.

_Cash value_: this value depends on the type of the transaction you decide to do.

See Transaction types:

+ _"Buy" or "Buy to Cover"_: The amount it cost to make the transaction. This is negative, since purchasing depletes your bank account. `cash value = -(share count * cost per share + commission)`
+ _"Sell" or "Sell Short"_: The amount you made on the sale. `cash value = share count * cost per share - commission`
+ _"Deposit cash"_: `cash value = cost per share`
+ _"Withdraw cash"_: `cash value = -cost per share`
+ _Dividend_: `cash value = event-adjusted share count * dividend value`

#### Short Lots
Short lots work pretty much the same as long lots, but with buys and sells reversed. In short lots, a position is opened by selling shares you do not technically own yet (which makes you money), and closed by buying shares at a later date to cover the transaction (which costs you money). This way, you make money if the security's price goes down, and lose money if it goes up.The most important difference with short lots is the definition of cost basis. Since the gains from a short sale transaction are not realized until a covering buy is made, Google Finance computes the value of an uncovered short as if a covering buy were made at the current stock price. For this reason, both the basis and the return are dependent upon the price of the stock.

##### Short Lot Computations
+ _Initial investment_: This is the same as for long lots: it is the negative of the cash value of the transaction that opens the lot. However, because that translation is a sale, the cash value is positive, and the initial investment is negative.
+ _Purchase cost_: This is equivalent to the cost basis of a long lot. `purchase cost = initial investment * (remaining quantity / initial quantity)` But remember that because the initial investment is negative, so is the purchase cost.
+ _Cost basis_: `cost basis = remaining quantity * share price`
+ _Market value_: `market value = -(remaining quantity) * share price = -cost basis`
+ _Gain_: Recall that for a long lot, gain is the difference between how much you would make by selling your current holdings of an asset at the current market price, and how much you originally paid for that asset. Short sales can be a little counterintuitive, because your income is realized at the time you open the position, and the cost is paid at the time of buying the shares to cover that sale. The gain is still the difference between your income and outlay. When you make a short sale, you incur an obligation to later purchase shares to cover the sale. Therefore, the amount of the outlay is dependent on the current market price and your number of currently held shares. Your income, on the other hand, is fixed based on the price of the stock at the time you made the short sale -- this is your "cash in". `Gain = cash in + market value`. (Remember that market value is negative)
+ _Today's gain_: `today's gain = -(remaining quantity) * price change`
+ _Gain percentage_: `gain percentage = gain / cost basis`
+ _Returns gain_: If your lot includes any covered buy transactions, these are also treated as outlays, but instead of being based on the current market price, they are computed with the price associated with the buy transaction -- this is your "cash out".  `Returns gain = cash in - (-market value + cash out)`.
+ _Overall return_: As with long lots, this is your returns gain as a percentage of your overall outlay on all shares in the lot, not solely those you still hold. `Overall return = cash in - (-market value + cash out) / (-market value + cash out)` 

**Since short lots can be counter-intuitive, an example is in order:**

Transaction: 4/1/2008 SHORT SELL XYZZ 100 @ $471.09 ($15 commission)

At this point you have an obligation to deliver 100 shares at some point in the future. Suppose the stock is now trading at $450, down $10 since the market open. Then:

+ Initial investment = -$47,094+ Purchase cost = -$47,094 -> This is a negative cost, because you made money on this part of the deal.
+ Cost basis is 100 * $450 = $45,000.+ Market value is -100 * $450 = -$45,000 -> Since this is an obligation to provide a security, not a security you own, the value is negative.
+ Gain is $47,094 + (-$45,000) = $2,094 -> Since the stock went down since purchase, you would make money if you bought shares today to cover the sale.
+ Today's gain is -100 * (-$10) = $1,000 -> Since the stock went down today, the amount of money you stand to make went up.+ Gain percentage is $2,094 / $45,000 = 4.65% -> This is the percentage you would make if you bought shares today to cover the sale, since you spend $45,000 to make a $2,094 profit.
+ Cash in = $47,094+ Returns gain = $47,094 - (-(-$45,000)) = $2,094 -> Short lots mean lots of negative numbers!
+ Overall return = $2,094 / $45,000 = 4.65%It gets a little trickier if your short lot includes some covered buys. Consider this example:

Transaction 1: 4/1/2008 SHORT SELL XYZZ 100 @ $471.09 ($15 commission)
Transaction 2: 5/5/2008 BUY TO COVER XYZZ 50 @ $573.20 ($15 commission)

We calculate the same numbers, bearing in mind that you only hold obligations to sell 50 shares at the time of calculation. Suppose the stock is now trading at $450, down $10 since the market open.Then:

+ Initial investment = -$47,094+ Purchase cost = -$47,094 * (50 / 100) = -$23,547
+ Cost basis is 50 * $450 = $22,500+ Market value is -50 * $450 = -$22,500
+ Gain is $23,547 + (-$22,500) = $1,047
+ Today's gain is -50 * (-$10) = $500
+ Gain percentage is $1,047 / $22,500 = 4.65%
+ Cash in (transaction 1) is $47,094
+ Cash out (transaction 2) is $28,675+ Returns gain = $47,094 - (-(-$22,500) + $28,675) = -$4,081
+ Overall return = -$4,081 / (-(-$22,500) + $28,675) = -7.97%

Transaction: 5/5/2008 BUY TO COVER XYZZ 50 @ $573.20 ($15 commission) -> Now you own 50 shares. 

The cost basis is 50 * 471.09 + 7.50 = $23,562. (Remember that commission costs are apportioned across all the shares you bought originally.)

#### Cost Basis
We should linger on _Cost basis_ for a moment. When Google Finance presents summary statistics for your portfolio, it computes those numbers based on the number of shares you still own. So, if you bought 100 shares of XYZZ, then sold them all, your cost basis will be reported as 0, as will your market value, gain, etc. If you only sold 25 shares, then all of your statistics will be based on the 75 shares that you still own. Notice that since this ratio is applied to your entire initial investment, which includes commission costs, only the portion of your commission costs that applied to the stocks you still own will be considered.

Some examples:
_Suppose you buy 100 shares of XYZZ on April 1, 2008:_

Transaction: 4/1/2008 BUY XYZZ 100 @ $471.09 ($15 commission) -> the cost basis is 100 * 471.09 + 15 = $47.124.00

_Now Suppose you buy 100 shares of XYZZ on April 1, 2008, but sell 50 on 5/5/2008:_

Transaction: 4/1/2008 BUY XYZZ 100 @ $471.09 ($15 commission) -> At this point you own 100 shares.
Transaction: 5/5/2008 SELL XYZZ 50 @ $573.20 ($15 commission) -> Now you own 50 shares. 

The cost basis is 50 * 471.09 + 7.50 = $23,562.

_(Remember that commission costs are apportioned across all the shares you bought originally.)_

#### Overall Return
The overall return will consider all the shares in your transaction history, whether you still own them or not. This is useful for evaluating your overall investment strategy, rather than simply tracking the stocks you currently own. We start by looking at all the transactions in the lot, and adding up how much money each of these transactions either cost you, or made you.

**For example:**

Transaction: 4/1/2008 BUY XYZZ 100 @ $471.09 ($15 commission) -> costs $47,124.00. We call this cash out.
Transaction: 5/5/2008 SELL XYZZ 50 @ $573.20 ($15 commission) -> makes $28,645.00. We call this cash in.

**Now we can compute the returns gain?** This is similar to the gain computed earlier, except that it takes into account the money you made on all transactions.

`returns gain = market_value + cash in - cash out.`

In our example, suppose XYZZ is trading at $484.77 today. That means that market value (based on the 50 shares still owned) is $24,238.50. Plugging this in to our equation gives us a returns gain of $5,759.50.
The overall return rate is just the returns gain divided by the amount you paid to establish the lot:

`Overall return = returns gain /cash out`

**the computation of return for short sales is somewhat different: see our page 'Short Lots'

#### Long Lots
Each security is divided into Lots. These do not appear in the user interface, but they are important for calculating gains and returns. Lots, in turn, are composed of transactions. Every time you purchase shares of a stock, a new lot is opened. When you sell shares, they are deducted from the oldest lot that still has shares. (Why the oldest first? We take our cue here from the U.S. tax code) If the oldest lot has shares, but not enough to complete the sale, then the sale may be split into two (or more) transactions, in order to allocate it across multiple lots. If all the lots together do not have enough shares to complete the sale, then the sale is invalid, and an error message is displayed.

**This is easiest to explain with examples:**

Suppose you buy 100 shares of XYZZ on April 1, 2008, then 100 more shares on April 1, 2009.

Lot 1: Transaction: 4/1/2008 BUY XYZZ 100
Lot 2: Transaction: 4/1/2009 BUY XYZZ 100

Each BUY transaction starts its own lot.

Now suppose you sold 50 shares of XYZZ on May 5, 2008. That would be divided up like this:

Lot 1: Transaction: 4/1/2008 BUY XYZZ 100 Transaction: 5/5/2008 SELL XYZZ 50
Lot 2: Transaction: 4/1/2009 BUY XYZZ 100

Suppose you also sold 80 shares of XYZZ on September 19, 2009. Since Lot 1 only has 50 shares left, the sale has to be split. In the Transactions tab, you will see your sale of 80 shares, but internally, it is treated like this:

Lot 1: Transaction: 4/1/2008 BUY XYZZ 100 Transaction: 5/5/2008 SELL XYZZ 50 Transaction: 9/19/2009 SELL XYZZ 50
Lot 2: Transaction: 4/1/2009 BUY XYZZ 100 Transaction: 9/19/2009 SELL XYZZ 30 

At this point, Lot 1 is considered a closed lot, that is, it has no more shares available for sale. Lot 2 still has 70 shares. 

Everything we have talked about so far applies to long lots, lots that are created by buying and holding stock. But lots can also be opened by short sale. These short lots behave analogously to long lots, except that every short sale starts a new short lot, and any covering buys will be deducted from the oldest short lot. Short lots will be discussed in more detail later on, so don't worry if you are unfamiliar with the term.

##### Long Lot Computations
A simple lot (sometimes called a long lot) consists of a stock purchase, possibly followed by stock sales. Each lot maintains three fundamental values that affect all the remaining calculations:+ initial quantity: This is the share count of the transaction that opened the lot.+ remaining quantity: This is the number of shares that have not been matched with subsequent sell transactions.+ initial investment: This is the negative of the cash value of the transaction that opened the lot, because the cash value is the effect on your bank account, but the initial investment is the opposite: it is the value that has been "put into" the stock. So, for example, the cash value of a purchase of 10 shares of XYZZ at $350 is -$3500, and the initial investment is $3500.

**From these values, the following investment statistics can be computed for the lot:**

`cost basis: cost basis = initial investment * (remaining quantity / initial quantity)`

`market value: market value = remaining quantity * share price`

`gain: gain = market value - cost basis [Note that this might be negative!]`

`todays's gain: today's gain = remaining quantity * price change`

`gain percentage: gain percentage = gain / cost basis`

#### Portfolio summary calculations
The lot calculations are by far the trickiest part of entire process. Once that step is done, the summary values for each security are calculated. These are the values that appear in each row under the Performance tab. First, cost basis, market value, gain, and todays gain are all computed as the sum of the corresponding values of all the lots for a security.

Then the gain percentage is calculated by: `gain percentage = gain / cost basis`

The total return for each security is calculated similarly: Returns gain and cash out are summed over all the lots for the security, then the total return is calculated by:`total return = returns gain / cash out`

Next, the summary values for the portfolio are calculated. These are the values that appear along the final row in the Performance tab. First, for each of the securities in the portfolio, cost basis, market value, gain, and today's gain are converted from the security's currency to the portfolio currency (which you can set in the Edit Portfolio page). Then the converted values are summed over all the currencies to give the portfolio values. Market value is adjusted by adding any cash deposits and subtracting any cash withdrawals. Then gain percentage is computed in the by-now familiar way: `gain percentage = gain / cost basis`

Finally, the overall return is computed by converting the returns gain and cash out from each of the securities from the security currency to the portfolio currency, then summing them to get portfolio values. The total return is the calculated by: `total return = returns gain / cash out`

**Currency: an extra wrinkle for global portfolios**

Currency is an issue worth revisiting, because it can get a little complicated. Every security on Google Finance has a currency in which it is valued. For example, GOOG is priced in dollars, whereas Vodafone (LON:VOD) is priced in British pence. The currency we use for a security is the currency used by the stock exchange on which the security is traded. Since all the securities need to be combined in order to show portfolio currencies, you have the option to specify a portfolio currency by clicking the "Edit Portfolio" link on the main portfolio page. When the security values, such as gain or market value, are rolled up for all the companies, each is first converted from its own currency to the portfolio currency, using today's exchange rate. Then all the summary values will be displayed in the portfolio currency.

**End Notes**

[1] U.S. Dept. of the Treasury, Internal Revenue Service Publication 550, "Investment Income and Expenses (Including Capital Gains and Losses)", 2008. Chapter 4, Section "Basis of Investment Property", Subsection "Identifying stocks or bonds sold", Paragraph "Identification not possible": "If you buy and sell securities at various times in varying quantities and you cannot adequately identify the shares you sell, the basis of the securities you sell is the basis of the securities you acquired first." This rule is sometimes called FIFO, for "First in, First Out". Since Google Finance does not let you manually match sales with lots, this is a reasonable rule to use.

9/30/09, written by Patrick Coskren, Software Engineer
