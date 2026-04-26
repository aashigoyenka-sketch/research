# README — `round3_trader_inverted_execution_only.py`
This README explains the uploaded IMC Prosperity Round 3 trader in beginner-friendly language.
The code is an **inverted execution** strategy: it keeps the fair-value models, but flips the aggressive buy/sell execution logic.
## Quick summary
- The bot estimates fair value for Hydrogel, Extract, and VEV vouchers.
- It prices VEVs like call options using Black-Scholes.
- It tracks moving averages, volatility, inventory, and approximate PnL.
- It uses risk controls to reduce size, close only, or flatten positions.
- The special part is that `aggressive_buy()` and `aggressive_sell()` are intentionally inverted.
## High-level flow
```text
run()
  -> load saved state
  -> read order books
  -> calculate market features
  -> calculate fair values
  -> calculate risk controls
  -> trade Hydrogel
  -> trade Extract
  -> trade VEV options
  -> save state
  -> return orders
```
## Main concepts
**Bid** = best price someone will buy from you.

**Ask** = best price someone will sell to you.

**Mid** = average of bid and ask.

**Fair value** = the bot's estimate of true product value.

**Edge** = minimum safety margin before trading.

**Position** = how many units the bot currently holds. Positive means long, negative means short.

**Delta** = option exposure to the underlying Extract price.

## Most important warning
Although the file says the original signals are correct and execution is backwards, that is a strategy hypothesis. It should be tested product-by-product. Inverting all aggressive execution can still lose if only one part of the model was wrong.

## Line-by-line beginner explanation

Format: `line number: code` followed by a beginner explanation.

```text
0001: from datamodel import OrderDepth, TradingState, Order
      -> Imports IMC framework classes: order book depth, full trading state, and order object.
0002: from typing import Dict, List, Optional, Tuple
      -> Imports type hints so variables like dictionaries, lists, and tuples are easier to understand.
0003: import json
      -> Imports JSON tools to save and load the bot's memory between ticks.
0004: import math
      -> Imports math functions used for option pricing and rounding.
0005: 
      -> Blank line used to separate sections and improve readability.
0006: 
      -> Blank line used to separate sections and improve readability.
0007: class Trader:
      -> Defines the main Trader class. IMC will instantiate this class and call its run method.
0008:     """
      -> Starts or ends a multi-line documentation comment.
0009:     Inverted Execution IMC Prosperity Round 3 trader.
      -> Documentation line explaining that this strategy is an inverted-execution trader.
0010:     
      -> Blank line used to separate sections and improve readability.
0011:     KEY INSIGHT: The original code's SIGNALS are correct, but the EXECUTION is backwards.
      -> Documentation line stating the strategy hypothesis.
0012:     Solution: Keep fair values, but invert ALL buy/sell decisions.
      -> Documentation line saying fair values are kept but buy/sell decisions are inverted.
0013:     
      -> Blank line used to separate sections and improve readability.
0014:     - When fair > mid (should buy), we SELL instead
      -> Documentation line describing the intended reversal of trade direction.
0015:     - When fair < mid (should sell), we BUY instead
      -> Documentation line describing the intended reversal of trade direction.
0016:     - This flips the strategy from losing to winning
      -> Documentation line describing the intended reversal of trade direction.
0017:     """
      -> Starts or ends a multi-line documentation comment.
0018: 
      -> Blank line used to separate sections and improve readability.
0019:     POSITION_LIMITS = {
      -> Starts the dictionary that stores max allowed position for every product.
0020:         "HYDROGEL_PACK": 100,
      -> A dictionary entry mapping a product or strike to a numeric value.
0021:         "VELVETFRUIT_EXTRACT": 350,
      -> A dictionary entry mapping a product or strike to a numeric value.
0022:         "VEV_4000": 200,
      -> A dictionary entry mapping a product or strike to a numeric value.
0023:         "VEV_4500": 200,
      -> A dictionary entry mapping a product or strike to a numeric value.
0024:         "VEV_5000": 200,
      -> A dictionary entry mapping a product or strike to a numeric value.
0025:         "VEV_5100": 200,
      -> A dictionary entry mapping a product or strike to a numeric value.
0026:         "VEV_5200": 200,
      -> A dictionary entry mapping a product or strike to a numeric value.
0027:         "VEV_5300": 200,
      -> A dictionary entry mapping a product or strike to a numeric value.
0028:         "VEV_5400": 200,
      -> A dictionary entry mapping a product or strike to a numeric value.
0029:         "VEV_5500": 200,
      -> A dictionary entry mapping a product or strike to a numeric value.
0030:         "VEV_6000": 200,
      -> A dictionary entry mapping a product or strike to a numeric value.
0031:         "VEV_6500": 200,
      -> A dictionary entry mapping a product or strike to a numeric value.
0032:     }
      -> Ends the current dictionary.
0033: 
      -> Blank line used to separate sections and improve readability.
0034:     STRIKES = {
      -> Starts the dictionary mapping voucher products to their strike prices.
0035:         "VEV_4000": 4000,
      -> A dictionary entry mapping a product or strike to a numeric value.
0036:         "VEV_4500": 4500,
      -> A dictionary entry mapping a product or strike to a numeric value.
0037:         "VEV_5000": 5000,
      -> A dictionary entry mapping a product or strike to a numeric value.
0038:         "VEV_5100": 5100,
      -> A dictionary entry mapping a product or strike to a numeric value.
0039:         "VEV_5200": 5200,
      -> A dictionary entry mapping a product or strike to a numeric value.
0040:         "VEV_5300": 5300,
      -> A dictionary entry mapping a product or strike to a numeric value.
0041:         "VEV_5400": 5400,
      -> A dictionary entry mapping a product or strike to a numeric value.
0042:         "VEV_5500": 5500,
      -> A dictionary entry mapping a product or strike to a numeric value.
0043:         "VEV_6000": 6000,
      -> A dictionary entry mapping a product or strike to a numeric value.
0044:         "VEV_6500": 6500,
      -> A dictionary entry mapping a product or strike to a numeric value.
0045:     }
      -> Ends the current dictionary.
0046: 
      -> Blank line used to separate sections and improve readability.
0047:     IMPLIED_VOL = {
      -> Starts the dictionary of fixed implied-volatility assumptions for Black-Scholes.
0048:         4000: 0.5404,
      -> Code line used as part of the surrounding block.
0049:         4500: 0.3163,
      -> Code line used as part of the surrounding block.
0050:         5000: 0.2353,
      -> Code line used as part of the surrounding block.
0051:         5100: 0.2325,
      -> Code line used as part of the surrounding block.
0052:         5200: 0.2341,
      -> Code line used as part of the surrounding block.
0053:         5300: 0.2364,
      -> Code line used as part of the surrounding block.
0054:         5400: 0.2218,
      -> Code line used as part of the surrounding block.
0055:         5500: 0.2408,
      -> Code line used as part of the surrounding block.
0056:         6000: 0.2330,
      -> Code line used as part of the surrounding block.
0057:         6500: 0.2330,
      -> Code line used as part of the surrounding block.
0058:     }
      -> Ends the current dictionary.
0059: 
      -> Blank line used to separate sections and improve readability.
0060:     OPTION_RESID_SCALE = {
      -> Starts scaling constants used to normalize option mispricing signals.
0061:         5000: 1.55,
      -> Code line used as part of the surrounding block.
0062:         5100: 1.05,
      -> Code line used as part of the surrounding block.
0063:         5200: 1.40,
      -> Code line used as part of the surrounding block.
0064:         5300: 1.05,
      -> Code line used as part of the surrounding block.
0065:         5400: 0.85,
      -> Code line used as part of the surrounding block.
0066:         5500: 1.35,
      -> Code line used as part of the surrounding block.
0067:     }
      -> Ends the current dictionary.
0068: 
      -> Blank line used to separate sections and improve readability.
0069:     TRADED_OPTION_PRODUCTS = (
      -> Starts the tuple of VEV products that the bot actively trades.
0070:         "VEV_5000",
      -> Code line used as part of the surrounding block.
0071:         "VEV_5100",
      -> Code line used as part of the surrounding block.
0072:         "VEV_5200",
      -> Code line used as part of the surrounding block.
0073:         "VEV_5300",
      -> Code line used as part of the surrounding block.
0074:         "VEV_5400",
      -> Code line used as part of the surrounding block.
0075:         "VEV_5500",
      -> Code line used as part of the surrounding block.
0076:     )
      -> Ends a multi-line tuple or function call.
0077: 
      -> Blank line used to separate sections and improve readability.
0078:     UNDERLYING = "VELVETFRUIT_EXTRACT"
      -> Stores the name of the underlying asset for the vouchers.
0079:     HYDRO = "HYDROGEL_PACK"
      -> Stores a short alias for HYDROGEL_PACK.
0080: 
      -> Blank line used to separate sections and improve readability.
0081:     def __init__(self):
      -> Constructor method, called once when the Trader object is created.
0082:         self.T = 7.0 / 365.0
      -> Sets option expiry to 7 days, expressed as a fraction of a year.
0083: 
      -> Blank line used to separate sections and improve readability.
0084:     # ------------------------------------------------------------------
      -> Comment separator to organize the code into sections.
0085:     # Basic math / order-book helpers
      -> Comment separator to organize the code into sections.
0086:     # ------------------------------------------------------------------
      -> Comment separator to organize the code into sections.
0087: 
      -> Blank line used to separate sections and improve readability.
0088:     @staticmethod
      -> Marks the next method as not needing access to object state.
0089:     def norm_cdf(x: float) -> float:
      -> Defines a helper for the normal distribution CDF used in Black-Scholes.
0090:         return 0.5 * (1.0 + math.erf(x / math.sqrt(2.0)))
      -> Returns the normal CDF approximation using the error function.
0091: 
      -> Blank line used to separate sections and improve readability.
0092:     def bs_call(self, s: float, k: float, sigma: float) -> float:
      -> Defines the Black-Scholes call option pricing function.
0093:         if s <= 0 or k <= 0:
      -> Checks invalid underlying or strike inputs.
0094:             return 0.0
      -> Returns zero when the price calculation is invalid or impossible.
0095:         if sigma <= 0:
      -> Handles the special case where volatility is zero or negative.
0096:             return max(s - k, 0.0)
      -> Returns intrinsic call value when volatility is zero.
0097:         st = sigma * math.sqrt(self.T)
      -> Calculates sigma times square-root of time, used in Black-Scholes.
0098:         d1 = (math.log(s / k) + 0.5 * sigma * sigma * self.T) / st
      -> Calculates Black-Scholes d1.
0099:         d2 = d1 - st
      -> Calculates Black-Scholes d2.
0100:         return s * self.norm_cdf(d1) - k * self.norm_cdf(d2)
      -> Returns theoretical call option price.
0101: 
      -> Blank line used to separate sections and improve readability.
0102:     def bs_delta(self, s: float, k: float, sigma: float) -> float:
      -> Defines a function to calculate option delta.
0103:         if s <= 0 or k <= 0:
      -> Checks invalid underlying or strike inputs.
0104:             return 0.0
      -> Returns zero when the price calculation is invalid or impossible.
0105:         if k <= 4500:
      -> Treats deep in-the-money low-strike vouchers as delta near 1.
0106:             return 1.0
      -> Code line used as part of the surrounding block.
0107:         if sigma <= 0:
      -> Handles the special case where volatility is zero or negative.
0108:             return 1.0 if s > k else 0.0
      -> If volatility is zero, delta is 1 when in-the-money, else 0.
0109:         st = sigma * math.sqrt(self.T)
      -> Calculates sigma times square-root of time, used in Black-Scholes.
0110:         d1 = (math.log(s / k) + 0.5 * sigma * sigma * self.T) / st
      -> Calculates Black-Scholes d1.
0111:         return self.norm_cdf(d1)
      -> Returns call option delta.
0112: 
      -> Blank line used to separate sections and improve readability.
0113:     @staticmethod
      -> Marks the next method as not needing access to object state.
0114:     def best_bid_ask(depth: OrderDepth) -> Tuple[Optional[int], int, Optional[int], int]:
      -> Defines a helper to extract the best bid and ask from an order book.
0115:         best_bid, bid_vol, best_ask, ask_vol = None, 0, None, 0
      -> Initializes best bid, bid volume, best ask, and ask volume.
0116:         if depth.buy_orders:
      -> Checks whether there are buy orders in the book.
0117:             best_bid = max(depth.buy_orders.keys())
      -> Finds the highest buy price.
0118:             bid_vol = depth.buy_orders[best_bid]
      -> Gets volume available at the best bid.
0119:         if depth.sell_orders:
      -> Checks whether there are sell orders in the book.
0120:             best_ask = min(depth.sell_orders.keys())
      -> Finds the lowest sell price.
0121:             ask_vol = -depth.sell_orders[best_ask]
      -> Converts sell volume to a positive number.
0122:         return best_bid, bid_vol, best_ask, ask_vol
      -> Initializes best bid, bid volume, best ask, and ask volume.
0123: 
      -> Blank line used to separate sections and improve readability.
0124:     @staticmethod
      -> Marks the next method as not needing access to object state.
0125:     def round_down(x: float) -> int:
      -> Defines helper to round a number down.
0126:         return int(math.floor(x))
      -> Uses floor to round down to an integer.
0127: 
      -> Blank line used to separate sections and improve readability.
0128:     @staticmethod
      -> Marks the next method as not needing access to object state.
0129:     def round_up(x: float) -> int:
      -> Defines helper to round a number up.
0130:         return int(math.ceil(x))
      -> Uses ceil to round up to an integer.
0131: 
      -> Blank line used to separate sections and improve readability.
0132:     # ------------------------------------------------------------------
      -> Comment separator to organize the code into sections.
0133:     # Persistent state
      -> Comment separator to organize the code into sections.
0134:     # ------------------------------------------------------------------
      -> Comment separator to organize the code into sections.
0135: 
      -> Blank line used to separate sections and improve readability.
0136:     def empty_state(self) -> Dict:
      -> Defines the default persistent memory structure.
0137:         return {
      -> Begins returning a dictionary.
0138:             "last_mid": {},
      -> A dictionary entry mapping a product or strike to a numeric value.
0139:             "ewma": {},
      -> A dictionary entry mapping a product or strike to a numeric value.
0140:             "fast": {},
      -> A dictionary entry mapping a product or strike to a numeric value.
0141:             "var": {},
      -> A dictionary entry mapping a product or strike to a numeric value.
0142:             "prev_pos": {},
      -> A dictionary entry mapping a product or strike to a numeric value.
0143:             "pnl_proxy": 0.0,
      -> A dictionary entry mapping a product or strike to a numeric value.
0144:             "peak_pnl_proxy": 0.0,
      -> A dictionary entry mapping a product or strike to a numeric value.
0145:             "surface_shift": 0.0,
      -> A dictionary entry mapping a product or strike to a numeric value.
0146:             "last_timestamp": -1,
      -> A dictionary entry mapping a product or strike to a numeric value.
0147:         }
      -> Ends the current dictionary.
0148: 
      -> Blank line used to separate sections and improve readability.
0149:     def load_state(self, trader_data: str) -> Dict:
      -> Defines function that loads saved traderData memory.
0150:         if not trader_data:
      -> If no saved state exists, use empty state.
0151:             return self.empty_state()
      -> Code line used as part of the surrounding block.
0152:         try:
      -> Starts a block where errors will be caught.
0153:             data = json.loads(trader_data)
      -> Converts saved JSON string back into a Python dictionary.
0154:         except Exception:
      -> If the try block fails, handle the error safely.
0155:             return self.empty_state()
      -> Code line used as part of the surrounding block.
0156:         base = self.empty_state()
      -> Creates a default state to ensure all keys exist.
0157:         for k, v in base.items():
      -> Loops through every default state key.
0158:             data.setdefault(k, v)
      -> Adds missing keys without overwriting existing saved values.
0159:         return data
      -> Returns the loaded and repaired state dictionary.
0160: 
      -> Blank line used to separate sections and improve readability.
0161:     def maybe_reset(self, state: TradingState, data: Dict) -> None:
      -> Defines a function that resets memory when a new day starts.
0162:         last_ts = data.get("last_timestamp", -1)
      -> Stores last timestamp to detect a new day.
0163:         if last_ts != -1 and state.timestamp < last_ts:
      -> Detects timestamp going backward, which means a new day.
0164:             fresh = self.empty_state()
      -> Creates fresh empty memory.
0165:             data.clear()
      -> Deletes old memory values.
0166:             data.update(fresh)
      -> Replaces old memory with fresh memory.
0167: 
      -> Blank line used to separate sections and improve readability.
0168:     def update_state_and_snapshot(self, state: TradingState, data: Dict) -> Dict[str, Dict]:
      -> Defines function that builds all market features for the current tick.
0169:         snapshot: Dict[str, Dict] = {}
      -> Creates dictionary that will store computed features.
0170: 
      -> Blank line used to separate sections and improve readability.
0171:         for product, depth in state.order_depths.items():
      -> Loops through every product's order book.
0172:             bid, bid_vol, ask, ask_vol = self.best_bid_ask(depth)
      -> Extracts best bid/ask and volumes.
0173:             if bid is None or ask is None:
      -> Skips products without both sides of the book.
0174:                 continue
      -> Code line used as part of the surrounding block.
0175: 
      -> Blank line used to separate sections and improve readability.
0176:             mid = (bid + ask) / 2.0
      -> Calculates mid price.
0177:             prev_mid = data["last_mid"].get(product)
      -> Stores last seen mid prices.
0178:             prev_pos = data["prev_pos"].get(product, 0)
      -> Stores previous positions for PnL proxy calculation.
0179:             if prev_mid is not None:
      -> Only updates PnL proxy if a previous mid exists.
0180:                 data["pnl_proxy"] += prev_pos * (mid - prev_mid)
      -> Stores approximate mark-to-market PnL.
0181: 
      -> Blank line used to separate sections and improve readability.
0182:             micro = (ask * bid_vol + bid * ask_vol) / max(1, bid_vol + ask_vol)
      -> Calculates microprice using bid/ask volume imbalance.
0183:             imb = (bid_vol - ask_vol) / max(1, bid_vol + ask_vol)
      -> Calculates order-book imbalance.
0184:             spread = ask - bid
      -> Calculates bid-ask spread.
0185: 
      -> Blank line used to separate sections and improve readability.
0186:             old_ewma = data["ewma"].get(product, mid)
      -> Stores slow moving averages.
0187:             ewma = 0.05 * mid + 0.95 * old_ewma
      -> Updates slow moving average.
0188: 
      -> Blank line used to separate sections and improve readability.
0189:             old_fast = data["fast"].get(product, mid)
      -> Stores fast moving averages.
0190:             fast = 0.20 * mid + 0.80 * old_fast
      -> Updates fast moving average.
0191: 
      -> Blank line used to separate sections and improve readability.
0192:             ret = mid - data["last_mid"].get(product, mid)
      -> Stores last seen mid prices.
0193:             old_var = data["var"].get(product, ret * ret)
      -> Stores variance estimates.
0194:             var = 0.05 * ret * ret + 0.95 * old_var
      -> Updates variance estimate.
0195: 
      -> Blank line used to separate sections and improve readability.
0196:             data["last_mid"][product] = mid
      -> Stores last seen mid prices.
0197:             data["ewma"][product] = ewma
      -> Stores slow moving averages.
0198:             data["fast"][product] = fast
      -> Stores fast moving averages.
0199:             data["var"][product] = var
      -> Stores variance estimates.
0200: 
      -> Blank line used to separate sections and improve readability.
0201:             snapshot[product] = {
      -> Starts storing computed features for this product.
0202:                 "bid": bid,
      -> A dictionary entry mapping a product or strike to a numeric value.
0203:                 "ask": ask,
      -> A dictionary entry mapping a product or strike to a numeric value.
0204:                 "bid_vol": bid_vol,
      -> A dictionary entry mapping a product or strike to a numeric value.
0205:                 "ask_vol": ask_vol,
      -> A dictionary entry mapping a product or strike to a numeric value.
0206:                 "mid": mid,
      -> A dictionary entry mapping a product or strike to a numeric value.
0207:                 "micro": micro,
      -> A dictionary entry mapping a product or strike to a numeric value.
0208:                 "imb": imb,
      -> A dictionary entry mapping a product or strike to a numeric value.
0209:                 "spread": spread,
      -> A dictionary entry mapping a product or strike to a numeric value.
0210:                 "ret": ret,
      -> A dictionary entry mapping a product or strike to a numeric value.
0211:                 "ewma": ewma,
      -> A dictionary entry mapping a product or strike to a numeric value.
0212:                 "fast": fast,
      -> A dictionary entry mapping a product or strike to a numeric value.
0213:                 "vol": math.sqrt(max(var, 0.0)),
      -> A dictionary entry mapping a product or strike to a numeric value.
0214:             }
      -> Ends the current dictionary.
0215: 
      -> Blank line used to separate sections and improve readability.
0216:         data["peak_pnl_proxy"] = max(data.get("peak_pnl_proxy", 0.0), data.get("pnl_proxy", 0.0))
      -> Stores approximate mark-to-market PnL.
0217:         return snapshot
      -> Returns computed market snapshot.
0218: 
      -> Blank line used to separate sections and improve readability.
0219:     # ------------------------------------------------------------------
      -> Comment separator to organize the code into sections.
0220:     # Inventory and order-placement helpers
      -> Comment separator to organize the code into sections.
0221:     # ------------------------------------------------------------------
      -> Comment separator to organize the code into sections.
0222: 
      -> Blank line used to separate sections and improve readability.
0223:     def buy_capacity(self, product: str, local_pos: Dict[str, int]) -> int:
      -> Defines helper for remaining buy capacity.
0224:         return self.POSITION_LIMITS[product] - local_pos.get(product, 0)
      -> Buy capacity equals max limit minus current position.
0225: 
      -> Blank line used to separate sections and improve readability.
0226:     def sell_capacity(self, product: str, local_pos: Dict[str, int]) -> int:
      -> Defines helper for remaining sell capacity.
0227:         return self.POSITION_LIMITS[product] + local_pos.get(product, 0)
      -> Sell capacity equals max limit plus current position.
0228: 
      -> Blank line used to separate sections and improve readability.
0229:     @staticmethod
      -> Marks the next method as not needing access to object state.
0230:     def buy_reduces_position(pos: int) -> bool:
      -> Defines helper to check if a buy reduces short inventory.
0231:         return pos < 0
      -> Buying reduces risk only if current position is short.
0232: 
      -> Blank line used to separate sections and improve readability.
0233:     @staticmethod
      -> Marks the next method as not needing access to object state.
0234:     def sell_reduces_position(pos: int) -> bool:
      -> Defines helper to check if a sell reduces long inventory.
0235:         return pos > 0
      -> Selling reduces risk only if current position is long.
0236: 
      -> Blank line used to separate sections and improve readability.
0237:     def aggressive_buy(
      -> Defines inverted aggressive buy logic.
0238:         self,
      -> Code line used as part of the surrounding block.
0239:         product: str,
      -> Code line used as part of the surrounding block.
0240:         depth: OrderDepth,
      -> Code line used as part of the surrounding block.
0241:         fair: float,
      -> Code line used as part of the surrounding block.
0242:         edge: float,
      -> Code line used as part of the surrounding block.
0243:         max_qty: int,
      -> Code line used as part of the surrounding block.
0244:         orders: List[Order],
      -> Code line used as part of the surrounding block.
0245:         local_pos: Dict[str, int],
      -> Code line used as part of the surrounding block.
0246:         close_only: bool = False,
      -> Code line used as part of the surrounding block.
0247:     ) -> None:
      -> Ends a multi-line tuple or function call.
0248:         """Buy from asks - INVERTED: Now triggered when fair < mid (opposite of original)."""
      -> Starts or ends a multi-line documentation comment.
0249:         cap = min(max_qty, self.buy_capacity(product, local_pos))
      -> Limits buy size by both max order size and remaining buy capacity.
0250:         if cap <= 0:
      -> If no capacity is available, do nothing.
0251:             return
      -> Exits the function.
0252: 
      -> Blank line used to separate sections and improve readability.
0253:         for ask in sorted(depth.sell_orders.keys()):
      -> Loops through ask prices from cheapest upward.
0254:             if ask > fair + edge or cap <= 0:  # INVERTED: condition flipped
      -> Inverted buy cutoff: stops buying once ask is too high relative to fair plus edge.
0255:                 break
      -> Code line used as part of the surrounding block.
0256:             avail = -depth.sell_orders[ask]
      -> Reads available sell quantity and makes it positive.
0257:             qty = min(cap, avail)
      -> Chooses trade size without exceeding capacity or available volume.
0258:             if qty > 0:
      -> Only places an order if quantity is positive.
0259:                 orders.append(Order(product, ask, qty))
      -> Adds a buy order at the ask price.
0260:                 local_pos[product] = local_pos.get(product, 0) + qty
      -> Updates simulated local position after buying.
0261:                 cap -= qty
      -> Reduces remaining capacity after order.
0262: 
      -> Blank line used to separate sections and improve readability.
0263:     def aggressive_sell(
      -> Defines inverted aggressive sell logic.
0264:         self,
      -> Code line used as part of the surrounding block.
0265:         product: str,
      -> Code line used as part of the surrounding block.
0266:         depth: OrderDepth,
      -> Code line used as part of the surrounding block.
0267:         fair: float,
      -> Code line used as part of the surrounding block.
0268:         edge: float,
      -> Code line used as part of the surrounding block.
0269:         max_qty: int,
      -> Code line used as part of the surrounding block.
0270:         orders: List[Order],
      -> Code line used as part of the surrounding block.
0271:         local_pos: Dict[str, int],
      -> Code line used as part of the surrounding block.
0272:         close_only: bool = False,
      -> Code line used as part of the surrounding block.
0273:     ) -> None:
      -> Ends a multi-line tuple or function call.
0274:         """Sell into bids - INVERTED: Now triggered when fair > mid (opposite of original)."""
      -> Starts or ends a multi-line documentation comment.
0275:         cap = min(max_qty, self.sell_capacity(product, local_pos))
      -> Limits sell size by both max order size and remaining sell capacity.
0276:         if cap <= 0:
      -> If no capacity is available, do nothing.
0277:             return
      -> Exits the function.
0278: 
      -> Blank line used to separate sections and improve readability.
0279:         for bid in sorted(depth.buy_orders.keys(), reverse=True):
      -> Loops through bid prices from highest downward.
0280:             if bid < fair - edge or cap <= 0:  # INVERTED: condition flipped
      -> Inverted sell cutoff: stops selling once bid is too low relative to fair minus edge.
0281:                 break
      -> Code line used as part of the surrounding block.
0282:             avail = depth.buy_orders[bid]
      -> Reads available buy quantity at the bid.
0283:             qty = min(cap, avail)
      -> Chooses trade size without exceeding capacity or available volume.
0284:             if qty > 0:
      -> Only places an order if quantity is positive.
0285:                 orders.append(Order(product, bid, -qty))
      -> Adds a sell order at the bid price; negative quantity means sell.
0286:                 local_pos[product] = local_pos.get(product, 0) - qty
      -> Updates simulated local position after selling.
0287:                 cap -= qty
      -> Reduces remaining capacity after order.
0288: 
      -> Blank line used to separate sections and improve readability.
0289:     def passive_quote(
      -> Defines passive quoting logic around fair value.
0290:         self,
      -> Code line used as part of the surrounding block.
0291:         product: str,
      -> Code line used as part of the surrounding block.
0292:         depth: OrderDepth,
      -> Code line used as part of the surrounding block.
0293:         fair: float,
      -> Code line used as part of the surrounding block.
0294:         edge: float,
      -> Code line used as part of the surrounding block.
0295:         base_size: int,
      -> Code line used as part of the surrounding block.
0296:         orders: List[Order],
      -> Code line used as part of the surrounding block.
0297:         local_pos: Dict[str, int],
      -> Code line used as part of the surrounding block.
0298:         inventory_skew: float = 0.0,
      -> Code line used as part of the surrounding block.
0299:         close_only: bool = False,
      -> Code line used as part of the surrounding block.
0300:     ) -> None:
      -> Ends a multi-line tuple or function call.
0301:         bid, _, ask, _ = self.best_bid_ask(depth)
      -> Gets best bid and best ask.
0302:         if bid is None or ask is None:
      -> Skips products without both sides of the book.
0303:             return
      -> Exits the function.
0304: 
      -> Blank line used to separate sections and improve readability.
0305:         pos = local_pos.get(product, 0)
      -> Reads current local position.
0306:         limit = self.POSITION_LIMITS[product]
      -> Reads position limit.
0307:         skew = inventory_skew * pos / max(1, limit)
      -> Calculates inventory skew to discourage adding to large positions.
0308:         qfair = fair - skew
      -> Adjusts fair value by inventory skew.
0309: 
      -> Blank line used to separate sections and improve readability.
0310:         buy_px = min(bid + 1, self.round_down(qfair - edge))
      -> Calculates passive buy price.
0311:         sell_px = max(ask - 1, self.round_up(qfair + edge))
      -> Calculates passive sell price.
0312: 
      -> Blank line used to separate sections and improve readability.
0313:         if close_only:
      -> If risk mode is close-only, only reduce existing positions.
0314:             if pos < 0 and buy_px < ask and self.buy_capacity(product, local_pos) > 0:
      -> If short, a buy can reduce the position.
0315:                 qty = min(base_size, -pos, self.buy_capacity(product, local_pos))
      -> Buy only enough to reduce short position, respecting capacity.
0316:                 if qty > 0:
      -> Only places an order if quantity is positive.
0317:                     orders.append(Order(product, buy_px, qty))
      -> Places a passive buy order.
0318:                     local_pos[product] = local_pos.get(product, 0) + qty
      -> Updates simulated local position after buying.
0319:             elif pos > 0 and sell_px > bid and self.sell_capacity(product, local_pos) > 0:
      -> If long, a sell can reduce the position.
0320:                 qty = min(base_size, pos, self.sell_capacity(product, local_pos))
      -> Sell only enough to reduce long position, respecting capacity.
0321:                 if qty > 0:
      -> Only places an order if quantity is positive.
0322:                     orders.append(Order(product, sell_px, -qty))
      -> Places a passive sell order.
0323:                     local_pos[product] = local_pos.get(product, 0) - qty
      -> Updates simulated local position after selling.
0324:             return
      -> Exits the function.
0325: 
      -> Blank line used to separate sections and improve readability.
0326:         if buy_px < ask and self.buy_capacity(product, local_pos) > 0:
      -> Ensures passive buy does not cross the ask.
0327:             qty = min(base_size, self.buy_capacity(product, local_pos))
      -> Code line used as part of the surrounding block.
0328:             if qty > 0:
      -> Only places an order if quantity is positive.
0329:                 orders.append(Order(product, buy_px, qty))
      -> Places a passive buy order.
0330:                 local_pos[product] = local_pos.get(product, 0) + qty
      -> Updates simulated local position after buying.
0331: 
      -> Blank line used to separate sections and improve readability.
0332:         if sell_px > bid and self.sell_capacity(product, local_pos) > 0:
      -> Ensures passive sell does not cross the bid.
0333:             qty = min(base_size, self.sell_capacity(product, local_pos))
      -> Code line used as part of the surrounding block.
0334:             if qty > 0:
      -> Only places an order if quantity is positive.
0335:                 orders.append(Order(product, sell_px, -qty))
      -> Places a passive sell order.
0336:                 local_pos[product] = local_pos.get(product, 0) - qty
      -> Updates simulated local position after selling.
0337: 
      -> Blank line used to separate sections and improve readability.
0338:     def force_flatten(self, product: str, depth: OrderDepth, local_pos: Dict[str, int], orders: List[Order], max_qty: int = 9999) -> None:
      -> Defines emergency position-reduction function.
0339:         pos = local_pos.get(product, 0)
      -> Reads current local position.
0340:         bid, bid_vol, ask, ask_vol = self.best_bid_ask(depth)
      -> Extracts best bid/ask and volumes.
0341:         if pos > 0 and bid is not None:
      -> If long, reduce by selling.
0342:             qty = min(pos, bid_vol, max_qty)
      -> Sell size is limited by position, available bid volume, and max quantity.
0343:             if qty > 0:
      -> Only places an order if quantity is positive.
0344:                 orders.append(Order(product, bid, -qty))
      -> Adds a sell order at the bid price; negative quantity means sell.
0345:                 local_pos[product] -= qty
      -> Code line used as part of the surrounding block.
0346:         elif pos < 0 and ask is not None:
      -> If short, reduce by buying.
0347:             qty = min(-pos, ask_vol, max_qty)
      -> Buy size is limited by short position, available ask volume, and max quantity.
0348:             if qty > 0:
      -> Only places an order if quantity is positive.
0349:                 orders.append(Order(product, ask, qty))
      -> Adds a buy order at the ask price.
0350:                 local_pos[product] += qty
      -> Code line used as part of the surrounding block.
0351: 
      -> Blank line used to separate sections and improve readability.
0352:     # ------------------------------------------------------------------
      -> Comment separator to organize the code into sections.
0353:     # Fair values / alpha models - SAME AS ORIGINAL
      -> Comment separator to organize the code into sections.
0354:     # ------------------------------------------------------------------
      -> Comment separator to organize the code into sections.
0355: 
      -> Blank line used to separate sections and improve readability.
0356:     def hydrogel_fair(self, s: Dict) -> float:
      -> Defines fair-value model for Hydrogel.
0357:         anchor = 0.50 * 9991.0 + 0.30 * s["ewma"] + 0.20 * s["fast"]
      -> Stores slow moving averages.
0358:         z = (s["mid"] - s["ewma"]) / max(2.0, s["vol"])
      -> Stores slow moving averages.
0359:         trendy = abs(z) > 1.6 and (z * s["ret"] > 0)
      -> Detects whether price is moving strongly with the trend.
0360:         mr_w = 0.45 if not trendy else 0.18
      -> Sets mean-reversion weight based on trendiness.
0361:         return (
      -> Code line used as part of the surrounding block.
0362:             anchor
      -> Code line used as part of the surrounding block.
0363:             + 0.55 * (s["micro"] - s["mid"])
      -> Code line used as part of the surrounding block.
0364:             + 1.60 * s["imb"]
      -> Code line used as part of the surrounding block.
0365:             - mr_w * s["ret"]
      -> Code line used as part of the surrounding block.
0366:             - 1.35 * z
      -> Code line used as part of the surrounding block.
0367:         )
      -> Ends a multi-line tuple or function call.
0368: 
      -> Blank line used to separate sections and improve readability.
0369:     def extract_fair(self, s: Dict) -> float:
      -> Defines fair-value model for Velvetfruit Extract.
0370:         anchor = 0.55 * 5250.0 + 0.25 * s["ewma"] + 0.20 * s["fast"]
      -> Stores slow moving averages.
0371:         z = (s["mid"] - s["ewma"]) / max(1.5, s["vol"])
      -> Stores slow moving averages.
0372:         trendy = abs(z) > 1.8 and (z * s["ret"] > 0)
      -> Detects whether price is moving strongly with the trend.
0373:         mr_w = 0.28 if not trendy else 0.12
      -> Sets mean-reversion weight based on trendiness.
0374:         return (
      -> Code line used as part of the surrounding block.
0375:             anchor
      -> Code line used as part of the surrounding block.
0376:             + 0.42 * (s["micro"] - s["mid"])
      -> Code line used as part of the surrounding block.
0377:             + 1.10 * s["imb"]
      -> Code line used as part of the surrounding block.
0378:             - mr_w * s["ret"]
      -> Code line used as part of the surrounding block.
0379:             - 0.95 * z
      -> Code line used as part of the surrounding block.
0380:         )
      -> Ends a multi-line tuple or function call.
0381: 
      -> Blank line used to separate sections and improve readability.
0382:     def compute_surface_shift(self, extract_fair: float, snap: Dict[str, Dict], data: Dict) -> float:
      -> Defines function to estimate common VEV surface price shift.
0383:         vals = []
      -> Creates list for option residual values.
0384:         weights = []
      -> Creates list for residual weights.
0385:         for product in self.TRADED_OPTION_PRODUCTS:
      -> Loops through all actively traded option products.
0386:             if product not in snap:
      -> Skips product if no market snapshot exists.
0387:                 continue
      -> Code line used as part of the surrounding block.
0388:             k = self.STRIKES[product]
      -> Gets strike price for this voucher.
0389:             theo = self.bs_call(extract_fair, k, self.IMPLIED_VOL[k])
      -> Computes theoretical Black-Scholes option price.
0390:             resid = snap[product]["mid"] - theo
      -> Computes difference between market mid and theoretical price.
0391:             w = 1.0 / max(1.0, snap[product]["spread"])
      -> Uses inverse spread as weight; tighter markets get more weight.
0392:             vals.append(resid)
      -> Adds residual to list.
0393:             weights.append(w)
      -> Adds weight to list.
0394: 
      -> Blank line used to separate sections and improve readability.
0395:         prev = data.get("surface_shift", 0.0)
      -> Stores common adjustment for the option surface.
0396:         if not vals:
      -> If no option data exists, keep previous shift.
0397:             shift = prev
      -> Code line used as part of the surrounding block.
0398:         else:
      -> Alternative branch if the previous condition is false.
0399:             raw = sum(v * w for v, w in zip(vals, weights)) / max(1e-9, sum(weights))
      -> Computes weighted average residual.
0400:             shift = 0.15 * raw + 0.85 * prev
      -> Smooths new surface shift with old shift.
0401:             shift = max(-5.0, min(5.0, shift))
      -> Caps surface shift between -5 and +5.
0402: 
      -> Blank line used to separate sections and improve readability.
0403:         data["surface_shift"] = shift
      -> Stores common adjustment for the option surface.
0404:         return shift
      -> Returns surface shift.
0405: 
      -> Blank line used to separate sections and improve readability.
0406:     def voucher_fair(self, product: str, underlying_fair: float, surface_shift: float) -> float:
      -> Defines fair-value model for a voucher.
0407:         k = self.STRIKES[product]
      -> Gets strike price for this voucher.
0408:         if k <= 4500:
      -> Treats deep in-the-money low-strike vouchers as delta near 1.
0409:             return max(underlying_fair - k, 0.0)
      -> Code line used as part of the surrounding block.
0410:         if k >= 6000:
      -> For far OTM vouchers, uses fixed tiny value.
0411:             return 0.5
      -> Code line used as part of the surrounding block.
0412:         return self.bs_call(underlying_fair, k, self.IMPLIED_VOL[k]) + surface_shift
      -> Returns Black-Scholes fair plus surface shift.
0413: 
      -> Blank line used to separate sections and improve readability.
0414:     # ------------------------------------------------------------------
      -> Comment separator to organize the code into sections.
0415:     # Risk logic
      -> Comment separator to organize the code into sections.
0416:     # ------------------------------------------------------------------
      -> Comment separator to organize the code into sections.
0417: 
      -> Blank line used to separate sections and improve readability.
0418:     def compute_portfolio_delta(self, local_pos: Dict[str, int], extract_fair: Optional[float]) -> float:
      -> Defines function for total underlying-equivalent exposure.
0419:         delta = local_pos.get(self.UNDERLYING, 0)
      -> Starts delta with actual Extract position.
0420:         if extract_fair is None:
      -> If Extract fair is unavailable, return only actual Extract delta.
0421:             return float(delta)
      -> Returns total delta as a float.
0422:         for product in self.TRADED_OPTION_PRODUCTS:
      -> Loops through all actively traded option products.
0423:             pos = local_pos.get(product, 0)
      -> Reads current local position.
0424:             if pos == 0:
      -> Skips products with zero position.
0425:                 continue
      -> Code line used as part of the surrounding block.
0426:             k = self.STRIKES[product]
      -> Gets strike price for this voucher.
0427:             delta += pos * self.bs_delta(extract_fair, k, self.IMPLIED_VOL[k])
      -> Adds option position times option delta.
0428:         return float(delta)
      -> Returns total delta as a float.
0429: 
      -> Blank line used to separate sections and improve readability.
0430:     def risk_controls(self, state: TradingState, data: Dict, local_pos: Dict[str, int], extract_fair: Optional[float]) -> Tuple[float, bool, bool]:
      -> Defines risk-control function.
0431:         drawdown = data.get("peak_pnl_proxy", 0.0) - data.get("pnl_proxy", 0.0)
      -> Stores approximate mark-to-market PnL.
0432:         gross_opts = sum(abs(local_pos.get(p, 0)) for p in self.TRADED_OPTION_PRODUCTS)
      -> Calculates total absolute option inventory.
0433:         port_delta = abs(self.compute_portfolio_delta(local_pos, extract_fair))
      -> Calculates absolute portfolio delta exposure.
0434: 
      -> Blank line used to separate sections and improve readability.
0435:         risk_mult = 1.0
      -> Starts with full order size.
0436:         close_only = False
      -> Starts with normal trading mode.
0437:         urgent_flatten = False
      -> Starts with no emergency flattening.
0438: 
      -> Blank line used to separate sections and improve readability.
0439:         if drawdown > 2200:
      -> If drawdown is moderate, reduce size.
0440:             risk_mult = min(risk_mult, 0.75)
      -> Code line used as part of the surrounding block.
0441:         if drawdown > 3500:
      -> If drawdown is larger, switch to close-only mode.
0442:             risk_mult = min(risk_mult, 0.50)
      -> Code line used as part of the surrounding block.
0443:             close_only = True
      -> Code line used as part of the surrounding block.
0444:         if drawdown > 5000:
      -> If drawdown is severe, use urgent flattening.
0445:             risk_mult = min(risk_mult, 0.30)
      -> Code line used as part of the surrounding block.
0446:             close_only = True
      -> Code line used as part of the surrounding block.
0447:             urgent_flatten = True
      -> Code line used as part of the surrounding block.
0448: 
      -> Blank line used to separate sections and improve readability.
0449:         if gross_opts > 420:
      -> Reduces risk when option inventory is large.
0450:             risk_mult = min(risk_mult, 0.70)
      -> Code line used as part of the surrounding block.
0451:         if gross_opts > 560:
      -> Close-only mode when option inventory is very large.
0452:             risk_mult = min(risk_mult, 0.45)
      -> Code line used as part of the surrounding block.
0453:             close_only = True
      -> Code line used as part of the surrounding block.
0454: 
      -> Blank line used to separate sections and improve readability.
0455:         if port_delta > 230:
      -> Reduces risk when portfolio delta is large.
0456:             risk_mult = min(risk_mult, 0.70)
      -> Code line used as part of the surrounding block.
0457:         if port_delta > 310:
      -> Close-only mode when portfolio delta is very large.
0458:             risk_mult = min(risk_mult, 0.45)
      -> Code line used as part of the surrounding block.
0459:             close_only = True
      -> Code line used as part of the surrounding block.
0460: 
      -> Blank line used to separate sections and improve readability.
0461:         if state.timestamp >= 96500:
      -> Late-session risk reduction begins.
0462:             risk_mult = min(risk_mult, 0.55)
      -> Code line used as part of the surrounding block.
0463:             close_only = True
      -> Code line used as part of the surrounding block.
0464:         if state.timestamp >= 98500:
      -> Very late-session urgent flatten begins.
0465:             risk_mult = min(risk_mult, 0.25)
      -> Code line used as part of the surrounding block.
0466:             close_only = True
      -> Code line used as part of the surrounding block.
0467:             urgent_flatten = True
      -> Code line used as part of the surrounding block.
0468: 
      -> Blank line used to separate sections and improve readability.
0469:         return risk_mult, close_only, urgent_flatten
      -> Returns risk settings.
0470: 
      -> Blank line used to separate sections and improve readability.
0471:     # ------------------------------------------------------------------
      -> Comment separator to organize the code into sections.
0472:     # Main
      -> Comment separator to organize the code into sections.
0473:     # ------------------------------------------------------------------
      -> Comment separator to organize the code into sections.
0474: 
      -> Blank line used to separate sections and improve readability.
0475:     def run(self, state: TradingState):
      -> Main method called by IMC simulator every tick.
0476:         result: Dict[str, List[Order]] = {}
      -> Creates dictionary that will hold all orders.
0477:         data = self.load_state(state.traderData)
      -> Loads saved memory.
0478:         self.maybe_reset(state, data)
      -> Resets memory if a new day begins.
0479:         snap = self.update_state_and_snapshot(state, data)
      -> Builds market snapshot and updates state.
0480:         local_pos = dict(state.position)
      -> Copies current positions so orders can update local simulated position.
0481: 
      -> Blank line used to separate sections and improve readability.
0482:         extract_fair: Optional[float] = None
      -> Initializes Extract fair value variable.
0483:         surface_shift = data.get("surface_shift", 0.0)
      -> Stores common adjustment for the option surface.
0484:         if self.UNDERLYING in snap:
      -> Checks if Extract market data is available.
0485:             extract_fair = self.extract_fair(snap[self.UNDERLYING])
      -> Calculates Extract fair value.
0486:             surface_shift = self.compute_surface_shift(extract_fair, snap, data)
      -> Updates VEV option surface shift.
0487: 
      -> Blank line used to separate sections and improve readability.
0488:         risk_mult, close_only, urgent_flatten = self.risk_controls(state, data, local_pos, extract_fair)
      -> Computes risk settings for this tick.
0489: 
      -> Blank line used to separate sections and improve readability.
0490:         # 1) Hydrogel Packs
      -> Comment separator to organize the code into sections.
0491:         if self.HYDRO in state.order_depths and self.HYDRO in snap:
      -> Starts Hydrogel trading block.
0492:             orders: List[Order] = []
      -> Creates an empty list of orders for this product.
0493:             depth = state.order_depths[self.HYDRO]
      -> Gets current order book for this product.
0494:             s = snap[self.HYDRO]
      -> Gets precomputed market features for this product.
0495:             fair = self.hydrogel_fair(s)
      -> Calculates Hydrogel fair value.
0496:             spread = s["spread"]
      -> Reads current bid-ask spread.
0497: 
      -> Blank line used to separate sections and improve readability.
0498:             take_edge = max(7.0, 0.65 * spread + 1.2 * (1.0 - risk_mult))
      -> Calculates required edge for aggressive orders.
0499:             make_edge = max(4.0, 0.35 * spread + 1.0 * (1.0 - risk_mult))
      -> Calculates required edge for passive quotes.
0500:             take_qty = max(6, int(24 * risk_mult))
      -> Calculates aggressive order size.
0501:             quote_qty = max(4, int(12 * risk_mult))
      -> Calculates passive quote size.
0502: 
      -> Blank line used to separate sections and improve readability.
0503:             # INVERTED: Swap the order of aggressive trading
      -> Comment separator to organize the code into sections.
0504:             self.aggressive_sell(self.HYDRO, depth, fair, take_edge, take_qty, orders, local_pos, close_only=close_only)
      -> Places inverted aggressive sell logic.
0505:             self.aggressive_buy(self.HYDRO, depth, fair, take_edge, take_qty, orders, local_pos, close_only=close_only)
      -> Places inverted aggressive buy logic.
0506:             self.passive_quote(
      -> Places passive bid/ask quotes.
0507:                 self.HYDRO,
      -> Code line used as part of the surrounding block.
0508:                 depth,
      -> Code line used as part of the surrounding block.
0509:                 fair,
      -> Code line used as part of the surrounding block.
0510:                 make_edge,
      -> Code line used as part of the surrounding block.
0511:                 quote_qty,
      -> Code line used as part of the surrounding block.
0512:                 orders,
      -> Code line used as part of the surrounding block.
0513:                 local_pos,
      -> Code line used as part of the surrounding block.
0514:                 inventory_skew=14.0,
      -> Code line used as part of the surrounding block.
0515:                 close_only=close_only,
      -> Code line used as part of the surrounding block.
0516:             )
      -> Ends a multi-line tuple or function call.
0517:             if urgent_flatten:
      -> Checks whether emergency flattening is active.
0518:                 self.force_flatten(self.HYDRO, depth, local_pos, orders, max_qty=30)
      -> Adds emergency inventory-reduction order.
0519:             if orders:
      -> If any orders were created, add them to result.
0520:                 result[self.HYDRO] = orders
      -> Stores product orders in the result dictionary.
0521: 
      -> Blank line used to separate sections and improve readability.
0522:         # 2) Velvetfruit Extract
      -> Comment separator to organize the code into sections.
0523:         if self.UNDERLYING in state.order_depths and self.UNDERLYING in snap and extract_fair is not None:
      -> Code line used as part of the surrounding block.
0524:             orders = result.get(self.UNDERLYING, [])
      -> Code line used as part of the surrounding block.
0525:             depth = state.order_depths[self.UNDERLYING]
      -> Gets current order book for this product.
0526:             s = snap[self.UNDERLYING]
      -> Gets precomputed market features for this product.
0527:             spread = s["spread"]
      -> Reads current bid-ask spread.
0528:             mispricing = extract_fair - s["mid"]
      -> Calculates Extract model mispricing.
0529: 
      -> Blank line used to separate sections and improve readability.
0530:             alpha_target = int(max(-90, min(90, 18.0 * mispricing)))
      -> Creates desired Extract position from mispricing.
0531:             opt_delta = 0.0
      -> Initializes option delta exposure.
0532:             for product in self.TRADED_OPTION_PRODUCTS:
      -> Loops through all actively traded option products.
0533:                 pos = local_pos.get(product, 0)
      -> Reads current local position.
0534:                 if pos == 0:
      -> Skips products with zero position.
0535:                     continue
      -> Code line used as part of the surrounding block.
0536:                 k = self.STRIKES[product]
      -> Gets strike price for this voucher.
0537:                 opt_delta += pos * self.bs_delta(extract_fair, k, self.IMPLIED_VOL[k])
      -> Adds option position times option delta.
0538: 
      -> Blank line used to separate sections and improve readability.
0539:             target_extract = int(max(-250, min(250, alpha_target - 0.75 * opt_delta)))
      -> Calculates desired Extract position after option hedge.
0540:             current_extract = local_pos.get(self.UNDERLYING, 0)
      -> Reads current Extract position.
0541:             diff = target_extract - current_extract
      -> Calculates how far current position is from target.
0542: 
      -> Blank line used to separate sections and improve readability.
0543:             take_edge = max(2.0, 0.45 * spread + 0.8 * (1.0 - risk_mult))
      -> Calculates required edge for aggressive orders.
0544:             make_edge = max(1.0, 0.22 * spread + 0.5 * (1.0 - risk_mult))
      -> Calculates required edge for passive quotes.
0545:             take_qty = max(12, int(40 * risk_mult))
      -> Calculates aggressive order size.
0546:             quote_qty = max(6, int(18 * risk_mult))
      -> Calculates passive quote size.
0547: 
      -> Blank line used to separate sections and improve readability.
0548:             # INVERTED: Swap the order of aggressive trading
      -> Comment separator to organize the code into sections.
0549:             self.aggressive_sell(self.UNDERLYING, depth, extract_fair, take_edge, take_qty, orders, local_pos, close_only=close_only)
      -> Places inverted aggressive sell logic.
0550:             self.aggressive_buy(self.UNDERLYING, depth, extract_fair, take_edge, take_qty, orders, local_pos, close_only=close_only)
      -> Places inverted aggressive buy logic.
0551:             self.passive_quote(
      -> Places passive bid/ask quotes.
0552:                 self.UNDERLYING,
      -> Code line used as part of the surrounding block.
0553:                 depth,
      -> Code line used as part of the surrounding block.
0554:                 extract_fair,
      -> Code line used as part of the surrounding block.
0555:                 make_edge,
      -> Code line used as part of the surrounding block.
0556:                 quote_qty,
      -> Code line used as part of the surrounding block.
0557:                 orders,
      -> Code line used as part of the surrounding block.
0558:                 local_pos,
      -> Code line used as part of the surrounding block.
0559:                 inventory_skew=6.0,
      -> Code line used as part of the surrounding block.
0560:                 close_only=close_only,
      -> Code line used as part of the surrounding block.
0561:             )
      -> Ends a multi-line tuple or function call.
0562: 
      -> Blank line used to separate sections and improve readability.
0563:             if abs(diff) >= 18:
      -> Hedges only if required adjustment is large enough.
0564:                 hedge_qty = min(abs(diff), max(20, int(50 * risk_mult)))
      -> Calculates hedge order size.
0565:                 hedge_edge = -0.10 if (close_only or urgent_flatten) else -0.35
      -> Sets hedge execution edge.
0566:                 if diff > 0:
      -> If target is higher than current, adjust in one direction.
0567:                     self.aggressive_sell(self.UNDERLYING, depth, extract_fair, hedge_edge, hedge_qty, orders, local_pos, close_only=False)
      -> Places inverted aggressive sell logic.
0568:                 else:
      -> Alternative branch if the previous condition is false.
0569:                     self.aggressive_buy(self.UNDERLYING, depth, extract_fair, hedge_edge, hedge_qty, orders, local_pos, close_only=False)
      -> Places inverted aggressive buy logic.
0570: 
      -> Blank line used to separate sections and improve readability.
0571:             if urgent_flatten:
      -> Checks whether emergency flattening is active.
0572:                 self.force_flatten(self.UNDERLYING, depth, local_pos, orders, max_qty=60)
      -> Adds emergency inventory-reduction order.
0573:             if orders:
      -> If any orders were created, add them to result.
0574:                 result[self.UNDERLYING] = orders
      -> Stores product orders in the result dictionary.
0575: 
      -> Blank line used to separate sections and improve readability.
0576:         # 3) Options: INVERTED execution.
      -> Comment separator to organize the code into sections.
0577:         if extract_fair is not None:
      -> Code line used as part of the surrounding block.
0578:             candidates: List[Tuple[float, str, float]] = []
      -> Creates list of candidate option opportunities.
0579:             for product in self.TRADED_OPTION_PRODUCTS:
      -> Loops through all actively traded option products.
0580:                 if product not in state.order_depths or product not in snap:
      -> Code line used as part of the surrounding block.
0581:                     continue
      -> Code line used as part of the surrounding block.
0582:                 k = self.STRIKES[product]
      -> Gets strike price for this voucher.
0583:                 fair = self.voucher_fair(product, extract_fair, surface_shift)
      -> Calculates voucher fair value.
0584:                 resid = snap[product]["mid"] - fair
      -> Computes difference between market mid and theoretical price.
0585:                 score = abs(resid) / max(0.75, self.OPTION_RESID_SCALE[k])
      -> Converts residual into comparable opportunity score.
0586:                 if score >= 0.80 or local_pos.get(product, 0) != 0:
      -> Code line used as part of the surrounding block.
0587:                     candidates.append((score, product, fair))
      -> Adds this option to candidate list.
0588: 
      -> Blank line used to separate sections and improve readability.
0589:             candidates.sort(reverse=True)
      -> Sorts opportunities from strongest to weakest.
0590:             active = {product for _, product, _ in candidates[:3]}
      -> Keeps only top option opportunities.
0591: 
      -> Blank line used to separate sections and improve readability.
0592:             for product in self.TRADED_OPTION_PRODUCTS:
      -> Loops through all actively traded option products.
0593:                 if product not in state.order_depths or product not in snap:
      -> Code line used as part of the surrounding block.
0594:                     continue
      -> Code line used as part of the surrounding block.
0595:                 if product not in active and local_pos.get(product, 0) == 0:
      -> Skips inactive options unless existing inventory needs management.
0596:                     continue
      -> Code line used as part of the surrounding block.
0597: 
      -> Blank line used to separate sections and improve readability.
0598:                 orders: List[Order] = []
      -> Creates an empty list of orders for this product.
0599:                 depth = state.order_depths[product]
      -> Gets current order book for this product.
0600:                 k = self.STRIKES[product]
      -> Gets strike price for this voucher.
0601:                 fair = self.voucher_fair(product, extract_fair, surface_shift)
      -> Calculates voucher fair value.
0602:                 spread = snap[product]["spread"]
      -> Reads current bid-ask spread.
0603:                 scale = self.OPTION_RESID_SCALE[k]
      -> Reads residual scale for this option strike.
0604: 
      -> Blank line used to separate sections and improve readability.
0605:                 take_edge = max(1.0, 0.20 * spread + 0.55 * scale + 0.75 * (1.0 - risk_mult))
      -> Calculates required edge for aggressive orders.
0606:                 make_edge = max(0.9, 0.18 * spread + 0.35 * scale + 0.40 * (1.0 - risk_mult))
      -> Calculates required edge for passive quotes.
0607: 
      -> Blank line used to separate sections and improve readability.
0608:                 base_take = 34 if k <= 5200 else 26
      -> Sets base aggressive size by strike.
0609:                 base_make = 14 if k <= 5200 else 10
      -> Sets base passive quote size by strike.
0610:                 take_qty = max(6, int(base_take * risk_mult))
      -> Calculates aggressive order size.
0611:                 quote_qty = max(4, int(base_make * risk_mult))
      -> Calculates passive quote size.
0612: 
      -> Blank line used to separate sections and improve readability.
0613:                 # INVERTED: Swap order
      -> Comment separator to organize the code into sections.
0614:                 self.aggressive_sell(product, depth, fair, take_edge, take_qty, orders, local_pos, close_only=close_only)
      -> Places inverted aggressive sell logic.
0615:                 self.aggressive_buy(product, depth, fair, take_edge, take_qty, orders, local_pos, close_only=close_only)
      -> Places inverted aggressive buy logic.
0616: 
      -> Blank line used to separate sections and improve readability.
0617:                 resid = snap[product]["mid"] - fair
      -> Computes difference between market mid and theoretical price.
0618:                 if abs(resid) > 0.40 * scale or local_pos.get(product, 0) != 0:
      -> Code line used as part of the surrounding block.
0619:                     self.passive_quote(
      -> Places passive bid/ask quotes.
0620:                         product,
      -> Code line used as part of the surrounding block.
0621:                         depth,
      -> Code line used as part of the surrounding block.
0622:                         fair,
      -> Code line used as part of the surrounding block.
0623:                         make_edge,
      -> Code line used as part of the surrounding block.
0624:                         quote_qty,
      -> Code line used as part of the surrounding block.
0625:                         orders,
      -> Code line used as part of the surrounding block.
0626:                         local_pos,
      -> Code line used as part of the surrounding block.
0627:                         inventory_skew=3.0,
      -> Code line used as part of the surrounding block.
0628:                         close_only=close_only,
      -> Code line used as part of the surrounding block.
0629:                     )
      -> Ends a multi-line tuple or function call.
0630: 
      -> Blank line used to separate sections and improve readability.
0631:                 if urgent_flatten:
      -> Checks whether emergency flattening is active.
0632:                     self.force_flatten(product, depth, local_pos, orders, max_qty=28)
      -> Adds emergency inventory-reduction order.
0633:                 if orders:
      -> If any orders were created, add them to result.
0634:                     result[product] = orders
      -> Stores product orders in the result dictionary.
0635: 
      -> Blank line used to separate sections and improve readability.
0636:         data["prev_pos"] = dict(state.position)
      -> Stores previous positions for PnL proxy calculation.
0637:         data["last_timestamp"] = state.timestamp
      -> Stores last timestamp to detect a new day.
0638: 
      -> Blank line used to separate sections and improve readability.
0639:         traderData = json.dumps(data, separators=(",", ":"))
      -> Converts memory dictionary to compact JSON string.
0640:         conversions = 0
      -> No conversions are used in this round.
0641:         return result, conversions, traderData
      -> Returns orders, conversion count, and saved memory to simulator.
```

## Practical debugging advice
To understand performance, test these separately:

1. Hydrogel only
2. Extract only
3. VEV options only
4. VEV options with hedge
5. Passive quotes disabled
6. Inverted aggressive execution disabled

If one component is strongly negative, isolate it before combining everything.
