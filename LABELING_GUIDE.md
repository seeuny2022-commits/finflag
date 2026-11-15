# FinFlag Labeling Guide

## General rules
1. **Judge only the text in the row.** Don't look up the video or the stock.
2. **Mark 1 only when it's clearly there.** If you're unsure, mark 0 and write a note.
3. **The creator's own trades count as advice.** "I'm buying X" from a finfluencer is effectively a recommendation, and so are plans to buy ("I'll probably start buying under $50").
4. **Quotes and warnings don't count.** If the creator quotes someone else's hype in order to criticise it, or warns *against* hype, mark 0.
5. **Broken or too-short text:** if a row is garbled or just one sentence, mark all 0 and note "unclear."
6. **Judge each flag separately.** A video can be cautious overall and still contain one red flag.

## GUARANTEE: promises that you'll make money
**1 if** the creator says or strongly implies the profit is certain, easy or huge ("guaranteed," "can't lose," "free money," "money in the bag").
**0 if** it's past performance, an analyst target, or hedged ("could," "I think").

## URGENCY: pressure to act now
**1 if** the viewer is pushed to act quickly or told they'll miss out ("buy before Monday," "don't miss this," "last chance"). This includes pressure to buy the creator's own product ("sign up now").
**0 if** it's patient or neutral ("I'm watching it," "add it to your watchlist," "wait for confirmation").

## NO_RISK: a recommendation with no downside mentioned
**1 if** the creator recommends or endorses buying or selling **and** never mentions any risk, downside or uncertainty.
**0 if** *any* real downside is mentioned, including business problems such as rising costs, competition, slowing growth, debt or a high valuation; **or** if there's no recommendation at all.
A boilerplate "not financial advice," or generic "do your own research," does **not** count as mentioning risk.

## CERTAINTY: confident positive predictions stated as fact
**1 if** a future price or result is stated as a sure thing **in a way that encourages the viewer to invest** ("this WILL hit $500," "earnings will absolutely explode").
**0 if** it's hedged, attributed to analysts, a fair-value estimate with reasoning, a warning, or a prediction of a fall.

## PROMOTION: selling you something they benefit from
**1 if** the creator **invites the viewer** to join or buy a paid membership, Discord, course, software, sponsor, affiliate link or coupon code.
**0 if** it's only "like and subscribe," a free resource, or thanking existing members.

## ALL_IN: encourages big or high-risk bets
**1 if** the creator encourages high-risk investing *as something to copy*: **any** options strategy (buying **or selling** calls or puts), margin or borrowing, leverage, short-squeeze or meme plays, or putting a big share of your money into one stock.
**0 if** it's normal investing (buying shares, dollar-cost averaging, diversifying), or options are explained neutrally with clear warnings.

## RISKY: decided by a rule, not a feeling
> **RISKY = 1 if GUARANTEE, CERTAINTY or ALL_IN is 1, or if URGENCY and NO_RISK are both 1.**

PROMOTION alone does **not** make a video risky, because many honest creators also sell memberships.

## Change history
- **After label review:** CERTAINTY limited to confident *positive* predictions that encourage investing; 3 labels updated (seg 555, 600, 501), logged in `labels.csv`.
- **After prompt v1 on dev:** added clarifications for business risks, "do your own research," thanking members, judging flags separately, selling options, and creators' buying plans. These clarify the existing rules and didn't change any labels.
