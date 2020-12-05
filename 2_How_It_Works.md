## How it works

Parapara introduces automated market maker(AMM) into derivative trading. AMM is actually a pool of funds deposited by liquidity providers(LP).  Each time a trader opens a position, AMM will use the funds as margin to open the opposite position as the counterparty, and after the trader closes his position, AMM will also close the position and settle the profit. So unlike other DEXs, **Parapara’s AMM does not conduct token exchanges, but long/short with traders.** Parapara also uses a deterministic pricing formula to set the price for each trade, rather than fixing it at the certain value. It helps us to retain the core function of price discovery, and prevent whale trading by introducing a larger slippage when the trade volume is large.

<div align=center>
<img width="720" src=".\system_design.png"/>
</div>

For each underlying asset, Parapara creates a corresponding perpetual contract to track its price. Perpetual contract accepts liquidity providers' fund as margin, and concentrates fund around the oracle price to provide sufficient liquidity to traders. In return LPs are rewarded with transaction fee paid by traders. 