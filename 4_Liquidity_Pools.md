### What is liquidity pool

The concept of the liquidity pool is at the core of AMM functionality. It serves as the counterparty to any trades. **For Parapara, the liquidity pool actually serves as a margin pool, which is solely composed of USD stable coins used to open long and short positions.** When a trader goes long, the liquidity pool will consume margin and go short with the same amount and price. When a trader goes short, the liquidity pool will consume margin and go long with the same amount and price. 

### Model Setup
The following model is set up for margin trade with liquidity pools.
- $M$ - the amount of USD stable coins in the liquidity pool
- $P_0$ - the oracle price/ index price
- $P_t$ - the marginal price/market price
- $X_0$ - the initial virtual amount of base token in the pool, and it is calculated with the following equation:
$$X_0 = M/P_0 \tag{1}$$
- $X$ - the current virtual amount of base token in the pool after some trades.
- $\Delta Y$ - the virtual paid amount of quote token to the pool.
- $c$ - the liquidity coefficient of the pool.

The purpose of virtual base token $X_0$ and $X$ is to facilitate the application of the pricing formula
$$P_t=P_0 \cdot (1-c+(\frac{X_0}{X})^2\cdot c) \tag{2}$$

For instance, suppose the liquidity coefficient $c$ is 0.1. BTC-USD pool has 100000 USD stable coins, and current oracle price is 10000 USD. The initial amount of base tokens in the pool is $100000/10000=10$. 

Please note that Parapara’s AMM does not conduct token exchanges, but long/short with traders. So the base token or quote token here are all virtual in order to facilitate calculation.

#### Step 1

If Jack opens a long position of BTCUSD of 1BTC at 2x leverage, the remaining base tokens in the pool is $X=10-1=9$, and the paid amount of quote tokens is
$$ \Delta Y = \int^{X_2}_{X_1}{P_0 \cdot (1-c+(\frac{X_0}{X})^2\cdot c)dx} =P_0 \cdot (X_2-X_1)\cdot(1-c+c\cdot \frac{X_0^2}{X_1\cdot X_2}) \tag{3}$$ 
which is 10111.11USD (see detailed calculation in pricing formulas section), thus the entry price for this long position is 10111.11USD ($10111.11/1.0$). On the liquidity pool side, the liquidity pool opens a short position of 1 BTC at 10111.11USD.



| Actions | Traders' position | LP's position | Marginal price after action |
|:--- |:--- |:--- |:--- |
| Jack longs 1 BTC | Jack: long position of 1 BTC\@10111.11USD with collateral 5055.55USD | short position of 1 BTC\@10111.11USD | 10234.56 USD |



#### Step 2

Then Jill comes and sells a short position of BTCUSD of 1.5BTC at 2x leverage. We can calculate the amount of quote tokens Jill can get by selling this position with (3), which is 15087.30USD in this case. The average entry price for this short position is 10058.20 USD ($15087.30/1.5$). Meanwhile, the liquidity pool closes its short position of 1BTC at 10111.11USD ($10000*(10-9)*(1-0.1+0.1*10/9)$),  and then opens a long position of 0.5BTC at 9952.38 USD ($10000*(10.5-10)*(1-0.1+0.1*10/10.5)$). 

| Actions | Traders' position | LP's position | Marginal price after action |
|:--- |:--- |:--- |:--- |
| Jack longs 1 BTC | Jack: long position of 1 BTC\@10111.11USD with collateral 5055.55USD | short position of 1 BTC\@10111.11USD | 10234.56 USD |
| Jill shorts 1.5 BTC | Jill: short position of 1.5BTC\@10058.20USD with collateral 7543.65USD | long position of 0.5 BTC\@9952.38USD | 9907.03 USD |

Now the amount of base tokens in the pool $X = 10-1+1.5=10.5$, and paid amount of quote token $\Delta Y  = 10111.11 - 15087.30=-4976.19 USD$, the minus sign means liquidity pool pays the virtual quote tokens to the trader in exchange for base tokens.

#### Step 3
After a while,  the oracle price rises to 11000USD. 
Please note that the liquidity pool is not risk neutral at the moment. It has exposure to a long position of 0.5BTC with an entry price at 9952.38USD. If the liquidity provider wants to withdraw his liquidity share, he will get a proportional margin balance refund and long position, which can be closed at his disposal. 

The unrealized $P\& L$ of pool is determined by the entry price, mark price and position size:
$$
\begin{aligned}
P\& L &= (Mark Price - Entry Price) * Size \tag{4}\\
&= (11000-9952.38)*0.5 = 523.81USD \\
\end{aligned}
$$
The Equity to Net Position Ratio ($ENP$) is quotient of current equity of the liquidity pool, to the contract value of the net position of all opening positions in the pool.
$$
\begin{aligned}
ENP &= (M + P\& L) / (CurrentNetPosition * MarkPrice) \tag{5}\\
&= (100000+523.81)/(0.5*11000)=1827.71 \% \\ 
\end{aligned}
$$
$ENP$ characterizes the risk exposure of the liquidity pool. When $ENP$ drops to 50%, the liquidity pool will not be able to open new position which increases its net position. When $ENP$ drops to 20%, the liquidation process will be triggered for the liquidity pool.
The liquidity pool's marginal price is also updated:
$$
\begin{aligned}
P_t=&P_0 \cdot (1-c+(\frac{X_0}{X})^2\cdot c) \\
=&11000*(1-0.1+0.1*(\frac{10}{10.5})^2)=10897.73 USD \\
\end{aligned}
$$

| Actions | Traders' position | LP's position | Marginal price after action |
|:--- |:--- |:--- |:--- |
| Jack longs 1 BTC | Jack: long position of 1 BTC\@10111.11USD with collateral 5055.55USD | short position of 1 BTC\@10111.11USD | 10234.56 USD |
| Jill shorts 1.5 BTC | Jill: short position of 1.5BTC\@10058.20USD with collateral 7543.65USD | long position of 0.5 BTC\@9952.38USD | 9907.03 USD |
| Oracle price rises to 11000 USD | - | - | 10897.73 USD |

#### Step 4
The marginal price is less than the oracle price which will encourage arbitrageurs come in and help to restore $X$ to $X_0$. Jack sees this opportunity and opens another long position of 10.5-10.0=0.5 BTC to restore $X$ with $\Delta Y$:
$$
\begin{aligned}
\Delta Y &= P_0 \cdot (X_2-X_1)\cdot(1-c+c\cdot \frac{X_0^2}{X_1\cdot X_2}) \\
&= 11000*(10.5-10)*(1-0.1+0.1*\frac{10*10}{10.5*10})=5473.81 USD \\
\end{aligned}
$$
$\Delta Y$ is updated while keeping $X_0$ invariant. Thus the entry price of this position is 10947.62 USD ($5473.81/0.5$)

| Actions | Traders' position | LP's position | Marginal price after action |
|:--- |:--- |:--- |:--- |
| Jack longs 1 BTC | Jack: long position of 1 BTC\@10111.11USD with collateral 5055.55USD | short position of 1 BTC\@10111.11USD | 10234.56 USD |
| Jill shorts 1.5 BTC | Jill: short position of 1.5BTC\@10058.20USD with collateral 7543.65USD | long position of 0.5 BTC\@9952.38USD | 9907.03 USD |
| Oracle price rises to 11000 USD | - | - | 10897.73 USD |
| Jack longs 0.5 BTC | Jack: long position of 1 BTC\@10111.11USD and 0.5 BTC\@10947.62USD | close long position of 0.5 BTC\@10947.62USD | 11000 USD |

Now the liquidity pool has closed all its position, and resulted to a realized P&L
$$
\begin{aligned}
P\& L &= (Exit Price - Entry Price) * Size \\
&= (10947.62-9952.38)*0.5=497.62USD \\
\end{aligned}
$$


### LP's rewards 
Traders are charged transaction fee for each trade. 70% of transaction fees are kept in the liquidity pool and distributed to LPs. The rest 30% will be used to purchase and redeem PARA token, thereby improve the value for all PARA token holders.
Meanwhile, most PARA tokens (75%) are reserved for incentive purpose and will be distributed to the community.  Liquidity mining will be definitely a great start to attract more users to the platform.

### LP's risk
As illustrated in the above example, providing liquidity is not always risk free. The LP's fund could be liquidated in the worst case, though the possibility is extremely small. In general, providing liquidity is profitable in most market environments, but may lose some value when market changes volatilely. 



