### Lets start from Uniswap
As the killer app in DEX world, Uniswap makes constant product AMM so famous, and makes it the defacto standard in many DEXs. So let's start from here.

The bedrock for Uniswap is constant product formula.
$$ X_0 \cdot Y_0 = K \tag{1}$$ 
Where 

- $X_0$ - the balance of basic token (such as BTC) in the pool
- $Y_0$ - the balance of quote token (such as USDT) in the pool
- $K$ - the constant product value

The spot price of basic token is:
$$P_0 = Y_0/X_0 \tag{2}$$

Traders can buy/sell basic token or quote token from/to the pool. Let's suppose a trader buys $\Delta x$ basic tokens with $\Delta y$ quote tokens. Now the pool has $X_0-\Delta x$ basic tokens and $Y_0+\Delta y$ quote tokens, and $K$ is invariant.
$$(X_0-\Delta x)(Y_0+\Delta y)=K \tag{3}$$
the spot price is changed: 
$$P_t = (Y_0+\Delta y)/(X_0-\Delta x) \tag{4}$$

Combining (1)(3), we can get the expression of $\Delta y$:
$$\Delta y=\frac{\Delta x \cdot Y} {(X-\Delta x)} \tag{5}$$

Therefore
$$
\begin{aligned}
P_t =& (Y_0+\Delta y)/(X_0-\Delta x)\\
=& \frac{Y_0+\frac{\Delta x \cdot Y} {(X-\Delta x)}}{(X_0-\Delta x)}\\
=& \frac{Y_0\cdot X_0 - Y_0\cdot \Delta x + \Delta x\cdot Y_0}{(X_0-\Delta x)^2}\\
=& \frac{Y_0\cdot X_0}{(X_0-\Delta x)^2}\\
=& (\frac{X_0}{X_0-\Delta x})^2\cdot \frac{Y_0}{X_0}
\end{aligned} \tag{6}
$$

Finally, the relationship between $P_t$ and $P_0$ is obtained:
$$ P_t=P_0\cdot (\frac{X_0}{X_0-\Delta x})^2 \tag{7}$$

This equation shows that the updated spot price is only related to the shortage of base token and the original price. When $(X_0- \Delta x)$ is half of $X_0$, the price is 4 times the original price.

let $X=X_0-\Delta x$ , (7) is rearranged as:
$$ P_t=P_0\cdot (\frac{X_0}{X})^2 \tag{8}$$

### Optimized pricing formula
We introduces a custom pricing formula derived from (8):
$$ P_t=P_0\cdot (1-c+c\cdot (\frac{X_0}{X})^2) \tag{9}$$
where $c$ is called liquidity parameter, and is in range $[0, \infty]$. Note that

1. if $c=1$, it is Uniswap pricing formula
2. if $c<1$, the price is less sensitive to $\Delta x$, thus provide more liquidity
3. if $c>1$, the price is more sensitive to $\Delta x$, thus make more slippage

on Parapara, $c$ is initially set to 0.1 to provide 10x liquidity compared to Uniswap. Also $c$ could be set to a varible to adjust supply curve in different price ranges, which can be discussed through a decentralized governance approach.

### Why no impermanent loss
The root cause of impermanent loss is that AMM relies on arbitrage to align its price to the index price. On Parapara, the inclusion of index price $P_0$ in the pricing formula eliminates the arbitrage opportunities and LP's value are thus protected.
But Parapara provides a different arbitrage opportunity. When a trader sells $\Delta x$ tokens, the marginal price $P_t$ is less than the index price $P_0$ according to (9) . This will attract arbitrageurs to buy the same amount $\Delta x$ tokens to restore the marginal price $P_t$ to $P_0$, and vice versa. **As long as $P_t$ equals $P_0$ when $P_0$ is updated, it should incur no change in LP's value.**

### Examples
In this part, we'll provide an example of how the trading formula works. Notice that trading fees are ignored for simplification.
For a BTC-USDT pool, suppose the liquidity pool has the initial state as

- $X_0=10$ 
- $P_0 = 10000$
- $c=0.1$

When Jack buys 1 BTC from the pool, the pool becomes in shortage of base token, thus the paid amount of USDT should be the integral of the $P_t$:
$$
\begin{aligned}
\Delta Y = \int^{X_2}_{X_1}{P_0 \cdot (1-c+(\frac{X_0}{X})^2\cdot c)dx} =& P_0 \cdot (X_2-X_1)\cdot(1-c+c\cdot \frac{X_0^2}{X_1\cdot X_2})\\
\end{aligned} \tag{10}
$$

$X_2$ and $X_1$ are 9 and 10 respectively. The amount of USDT Jack needs to pay in (10) is 10111.11 USDT.
$$
\begin{aligned}
\Delta Y =& P_0 \cdot (X_2-X_1)\cdot(1-c+c\cdot \frac{X_0^2}{X_1\cdot X_2})\\
=& 10000*(10-9)*(1-0.1+0.1*\frac{10*10}{10*9})\\
=& 10111.11 USDT \\
\end{aligned}
$$

The pool's marginal price $X_t$ is updated
$$
\begin{aligned}
P_t=& P_0\cdot (1-c+c\cdot (\frac{X_0}{X})^2) \\
=& 10000*(1-0.1+0.1*(10/9)^2) \\
=& 10234.56 USDT \\
\end{aligned}
$$
Since pool's price is now greater than index price $P_0$, it creates an arbitrage opportunity that arbitrageurs should sell the same amount of BTC to the pool to restore its price. 
Suppose an arbitrageur Jill sells 1 BTC to the pool. The amount of USDT she earn is
$$
\begin{aligned}
\Delta Y &=P_0 \cdot (X_2-X_1)\cdot(1-c+c\cdot \frac{X_0^2}{X_1\cdot X_2}) \\
&= 10000*(10-9)*(1-0.1+0.1*\frac{10*10}{9*10}) \\
&=10111.11USD \\
\end{aligned}
$$
After Jill, the pool has restored the base token balance and returned to the initial state.












