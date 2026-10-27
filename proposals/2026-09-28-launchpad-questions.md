# Token launchpad: what we need you to decide

**For MAXLONG. From SuitVentures, 28 September 2026.**

The design and the build plan are finished. Three of the questions below hold up the contracts. The rest are
settings we can change at any time, or calls that are yours rather than ours. Every question has our
recommendation, so the shortest useful answer is "all as recommended".

## How it will work

A creator names a token, picks an asset to raise in, sets a target and a deadline, and signs one transaction.
People contribute, all at the same price. Then one of two things happens.

**The target is reached.** In a single transaction the token is created, a Uniswap pool opens at exactly the
price everyone paid, the whole raise goes into that pool as its liquidity, and the liquidity is locked
forever. Contributors claim their tokens. The token is on the LongMarkets table within a minute with a chart,
a holder list and working Buy and Sell buttons.

**The target is missed.** Every contributor takes back everything they put in. No token is ever created and
no fee is taken.

### No bonding curve

There is another way to run a launchpad, and Pons and NTRPY both use it: a bonding curve. The price starts
low and climbs with every purchase, so the first buyer pays least and the hundredth pays more. There is no
target and no deadline. The token trades against a formula until enough has been raised to open a pool.

We are not doing that, for four reasons.

1. **Your own rules do not fit it.** You settled in September that a failed raise refunds everyone in full
   and that the fee is earned only on success. A curve never fails and never refunds. People leave by selling
   back into it, usually at a loss.
2. **It is a second market.** A curve is a place where the token trades before the pool exists, so it needs
   its own price feed, its own chart and its own trading path through the site. That is a second version of
   everything already built.
3. **It holds more of other people's money for longer**, and the contract doing it is about three times the
   size. That is the part an audit prices.
4. **It rewards speed over interest.** Because the first buyer pays least, a launch becomes a race between
   bots. NTRPY taxes the first sixty seconds at up to 99 percent to blunt this. One price for everyone has no
   race to blunt.

What you give up: nobody gets in cheaper than anybody else, nobody can sell before the pool opens, and the
opening price is one the creator chose rather than one a crowd discovered.

## The three that hold up the build

### 1. The supply split

**We recommend:** 1 billion tokens. 45 percent sold in the raise, 45 percent paired into the pool, 10 percent
to the creator.

The original proposal said 60/30/10. That cannot work. The tokens going into the pool have to match the
tokens sold, or the pool opens at a different price from the one contributors just paid. At 60/30 the pool
would open at double the raise price, and every contributor would have an instant reason to sell into a pool
holding their own money. 45/45 opens the pool at exactly the price people paid.

### 2. The creator's ten percent

**We recommend:** released a little each day across 90 days, starting at launch.

Handing a creator a tenth of the supply on day one is the most common way a launch dies in its first hour. A
90 day release costs an honest creator nothing and removes the thing contributors fear most.

### 3. Can a creator set a floor?

**We recommend:** yes, between 50 and 100 percent of the target, defaulting to all or nothing.

A raise that reaches 90 percent of its target and refunds everyone is a bad outcome for everybody. A floor
lets the creator say in advance that 80 percent is enough to launch. Tokens left unsold are burned, so the
pool still opens at the price people paid.

## Settings, changeable at any time

### 4. Who may launch

**We recommend:** anyone with a wallet, with the switch built in so you can restrict it to Suits NFT holders
later without redeploying anything.

Gating it gives the Suits NFT a real job beyond the trading perk. Leaving it open gets you more launches. You
can start open and close it later, or the reverse.

### 5. The launch fee

**We recommend:** 2 percent of a successful raise, taken in the asset raised, earned only on success. The
contract will refuse to ever set it above 5 percent, so contributors can check that ceiling themselves.

The old 1 percent went away when the launch layer was dropped in September, so this is a fresh decision.

### 6. Trading fees from launched pools

**We recommend:** 70 percent to the creator, 30 percent to MAXLONG.

This is revenue you may not have counted on. The liquidity locked in a launched pool still earns a fee on
every trade in that pool, forever, at 1 percent of each trade. Somebody has to be able to collect it or it
sits there untouched. It is separate from, and on top of, the 0.5 percent the site already earns on trades
routed through it.

A trade on a launched token pays:

| Fee | Rate | Who receives it |
|---|---|---|
| Site routing fee | 0.5 percent | MAXLONG |
| Pool trading fee | 1 percent | 70 percent creator, 30 percent MAXLONG |

### 7. Which assets a raise may be denominated in

**We recommend:** USDG plus the ten stock tokens with the deepest markets today, with any of the other 54
addable in a minute.

Every asset on the list is an asset a new pool gets paired against. A thin one makes a thin market.

### 8. Raise size limits

**We recommend:** USD 1,000 minimum and USD 250,000 maximum, set per asset.

The minimum keeps out dust. The maximum caps how much money one contract holds while this is new.

## Your calls, not ours

### 9. Other venues, and GigaDex

Every launch opens a Uniswap v4 pool. The other venue on Robinhood Chain is GigaDex, which is native to this
chain, and NTRPY sends every token it graduates there.

If you want a relationship with GIGA, that is worth having on its own terms and needs nothing from us. If you
want it inside the product, it splits into three separate pieces of work, and they are not equal.

| What it could mean | Our view |
|---|---|
| Launch into GigaDex pools instead of Uniswap | Not advisable. LongMarkets lists Uniswap v4 pools only, so a token you launched would not appear on your own site |
| Show GigaDex pools on the markets table | Real work. GigaDex is a different design from Uniswap v4 and needs its own indexing and its own price maths. A separate change order |
| Let people trade GigaDex pools through the site | A second path in the router, and its own change order |

**We recommend** launching on Uniswap v4, because it is the only venue your own site can show, and treating
anything GigaDex as its own conversation with its own price.

### 10. Who may contribute, and from where

For your counsel, not for us. A launchpad makes MAXLONG the party creating and selling new tokens, and the
raises are denominated in tokenised equities. That is a different legal position from listing pools other
people made, and it deserves an opinion before the first real launch rather than after it.

### 11. An audit

**USD 3,000 to 15,000, one to three weeks.**

The launch contract holds contributors' money between the contribution and the launch. That is the one place
in this system where a mistake costs users directly. You can launch without an audit and carry that risk, or
budget for one. Either way we will rehearse a full launch and a full refund on the real chain with our own
money first.

## Already settled, no action needed

From your decisions in early September, unchanged.

| Rule | Where it stands |
|---|---|
| A failed raise | Everyone refunded in full, no fee of any kind |
| The fee | Earned only on success |
| Per wallet limit | At most 10 percent of the raise target, no minimum |
| Liquidity after launch | Locked forever, nobody can withdraw it, including us |
| Our own launches on the table | Ranked by the same rules as every other token, no favours |

## What happens next

Answers to 1, 2 and 3 let the contracts be finished. Everything else can be answered while we build, and 4 to
8 can be changed after launch without touching the code. The running cost of the site does not change: the
launchpad adds nothing to the monthly bill.
