### What is perpetual contract
A perpetual contract is a financial contract without an expiration date. It allows traders to speculate on an asset’s price movements without the need to hold the asset itself. The perpetual contract has become the most-traded derivative in the crypto space due to its flexibility, liquidity, and ease of understanding. Perpetual contracts use margin trading. **The maximum leverage is 10x at ParaPara, with the initial margin ratio of 10%, and the maintenance margin ratio of 7.5%.** A trader can withdraw from the margin account as long as the remaining balance stays over the initial margin (10%). When the balance of a margin account falls below the maintenance margin (7.5%), a keeper will liquidate the margin account at the mark price. 


### No funding payment
In other perpetual contract exchanges, perpetual contracts rely on the [funding payment](https://help.ftx.com/hc/en-us/articles/360027946571-Funding) to anchor its trading price to the underlying asset's spot price. Traders has to pay attention to the funding rate all the time, which is very annoying. Parapara is different. The purpose of funding payment is to provide an incentive for traders to align prices. On Parapara, we subscribe to oracle price feeds and take the initiative to align prices constantly, thus no need to have funding payment. This will probably create a new trading paradigm in perpetual contracts, such as HODL strategy in derivatives.


### Oracles
The purpose of the oracle is to provide index price ($P_0$ in the pricing formula) for any asset. There are some options currently available, including [JustLink](justlink.io) and [Band Protocol](https://bandprotocol.com/). We may also build our own oracle services to provide more price feeds of different underlying assets.

### Prices
There are three price types on ParaPara. 

- Market Price - The marginal price($P_t$ in the pricing formula);
- Index Price - The oracle price ($P_0$ in the pricing formula);
- Mark Price - the price used to calculate unrealized P&L, and this affects the initial and maintenance margin of a position;
  $$Mark Price = Index Price + EMA(Market Price - Index Price)$$
### Fees
As mentioned earlier, there is no funding payment or leverage interest on Parapara, traders are only charged one-time transaction fee for each trade. The rate is 0.3% or 0.1% depending on the trading direction, which encourages arbitrageurs to restore internal price to the oracle price.


### Liquidation
When the margin balance is less than the maintenance margin, anyone can trigger the liquidation process to get liquidation rewards. Currently the liquidation reward (or liquidation penalty for the account holder) is 2.5% of position value at entry. The rest will go to trader's account.  So the trader also has incentives to close such positions to get his liquidation fee back.
