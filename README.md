
# Polymarket Arbitrage Bot (Dutching Scanner)

## Overview
This bot scans Polymarket events and detects **guaranteed arbitrage opportunities** using **dutching**.
It supports:
- **2-outcome markets** (classic YES/YES hedge)
- **3-outcome markets** (e.g. Team A / Team B / Draw)
- Sports, politics, and other categories
- Automatic rescan every 30 seconds
- Telegram notifications when new opportunities appear

The goal is simple:  
**allocate capital across all outcomes so that no matter which outcome resolves TRUE, the payout is higher than the total cost.**

---

## Core Arbitrage Logic

### Key Condition (General)
For a market with **K outcomes**:

Let:
- `p₁, p₂, …, pₖ` = YES prices of each outcome
- `fee_buffer` = safety margin for fees & spread

Then **guaranteed arbitrage exists if**:

```
p₁ + p₂ + … + pₖ + fee_buffer < 1
```

If this condition is NOT met, arbitrage is impossible.

---

## Dutching Formula (Position Sizing)

Let:
- `C` = total capital (e.g. $100)
- `S = p₁ + p₂ + … + pₖ`

### Shares per outcome
All outcomes are sized to have **equal payout**:

```
shares = C / S
```

### Dollar allocation per outcome
```
betᵢ = C × (pᵢ / S)
```

So the total cost is exactly `C`.

---

## Guaranteed Profit

### Profit (percentage)
```
profit % = (1 / S − 1) × 100
```

### Profit (dollars)
```
profit $ = C × (1 / S − 1)
```

This profit is identical regardless of which outcome resolves TRUE.

---

## Maximum Allowed Prices (Execution Safety)

To remain arbitrage-safe **after slippage**, each outcome has a maximum price:

For outcome `i`:
```
max_priceᵢ = (1 − fee_buffer) − (sum of all other prices)
```

If the live market price rises above this level, the arbitrage breaks.

---

## Flow of the Bot

1. **Fetch active Polymarket events**
   - By default: all categories (sports, politics, crypto, etc.)

2. **Extract candidate outcomes**
   - Events with exactly **2 or 3 outcomes** that have valid buy prices
   - Ignore large multi-candidate markets (4+ outcomes)

3. **Check arbitrage condition**
   - Apply `sum(prices) + fee_buffer < 1`

4. **Filter by profit range**
   - Only keep opportunities with profit between **1% and 10%**
   - This avoids unrealistic or illiquid markets

5. **Compute sizing**
   - Dollar amount per outcome
   - Profit in % and $

6. **Display results**
   - Clearly shows prices, bet sizes, profit, and safety limits

7. **Telegram notification**
   - Sends alert when a **new arbitrage opportunity** appears
   - Prevents repeated spam for the same market

8. **Repeat**
   - Automatically rescans every **30 seconds**

---

## Important Notes

- This bot detects **mathematical arbitrage**, not execution guarantees.
- Liquidity, order book depth, and slippage still matter.
- Very high profit percentages usually indicate **fake arb** due to thin liquidity.
- For testing purposes, the bot allows 3-outcome markets; real deployment should be more selective.

---

## Intended Use
- Education & testing of Polymarket pricing inefficiencies
- Monitoring mispriced multi-outcome events
- Building automated alert systems for low-risk trading strategies
