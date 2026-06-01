---
layout: post
title: "What One Match Tells Us — and What It Doesn't"
date: 2026-06-01
categories: polaris
---

*POLARIS build log · Post 1*

This is the first technical post in a series documenting POLARIS, a research project on player-conditional decision policies in football. Before building any model, I wanted to understand exactly what a single match looks like in the data I'm working with — the J1 League 2024 season, combining Hudl StatsBomb event data with Hudl Pro GPS physical data. This post walks through one match, Yokohama F. Marinos vs Sagan Tosu (2024-07-03, 0–1), five figures at a time, to establish what these data can and cannot tell us. The conclusion sets up why the project takes the shape it does.

---

## 1. The shape of a match

Before modeling anything, it helps to see what a single match *is* in event-stream form. The top panel shows the per-minute event density for each side (3-minute rolling average); the bottom panel marks goals and substitutions.

![The shape of a match]({{ site.baseurl }}/assets/blog1/fig1_match_shape.png)

No score is needed to read the story. Yokohama F. Marinos (blue) controlled the opening half, but Sagan Tosu (pink) surged at the start of the second half and scored in the 52nd minute — the pink density spike and the goal are visibly linked. The trailing home side then dominated possession through the closing stages, chasing an equaliser that never came. A football match, reduced to data, is roughly 3,300 timestamped events; their rhythm alone encodes the narrative.

## 2. Where things happen

Each event carries a pitch coordinate, so we can ask where each side spent its time. Both teams are shown attacking left-to-right.

![Where things happen]({{ site.baseurl }}/assets/blog1/fig2_event_locations.png)

The contrast is tactical. Yokohama F. Marinos (blue) concentrated play in central areas of the attacking half — the signature of a possession-dominant side working the ball through the middle. Sagan Tosu (pink) shows a heavy cluster deep in their own left-back zone and a thinner, wing-oriented attacking presence — the footprint of a counter-attacking team that defends, builds from the back, and strikes down the flanks. The away side's lower event count (1,283 vs 1,924) and deeper positioning are exactly what we would expect from the team that scored against the run of play and then defended a lead.

## 3. The only off-ball picture we get

Figures 1 and 2 tracked the ball. But football is a game of twenty-two players, and for almost every event we only know where the ball was — not where the other players stood. The single exception is the shot freeze frame.

![The only off-ball picture we get]({{ site.baseurl }}/assets/blog1/fig3_freeze_frame.png)

Here is Sagan Tosu's 52nd-minute goal: scorer Ayumu Yokoyama (star) has slipped in behind to roughly [102, 33], while Yokohama's back line (blue) is still holding a flat line further out — present, but beaten by the run behind them. This is the richest spatial snapshot the dataset offers, and it is sobering: only eight players are recorded, and only at the instant of a shot. For the other ~3,200 events in this match, the off-ball picture is simply unavailable. Any model built on these data must work without it.

## 4. What the body was doing

The dataset's third source is GPS tracking: every player's physical output, binned into 15-minute windows. The left panel shows high-intensity running per bin — the rhythm of effort. The right panel re-expresses the same data as cumulative load.

![What the body was doing]({{ site.baseurl }}/assets/blog1/fig4_physical_load.png)

Togashi (pink) spikes to 352m of high-intensity running late in the first half; Yan Matheus (green) collapses to 42m in the 61–75 window before rebounding. The cumulative panel on the right is the quantity POLARIS treats as fatigue: a monotonically rising burden, not a momentary effort. Note Amano's blue line — it simply stops after the 61–75 bin, because he was substituted off in the 65th minute. The body's record ends exactly when his match did. This is the data layer that no event stream captures, and the one that almost no decision-modeling work in football has used.

## 5. One player, one match

Finally, we bring the layers together for a single player. Jun Amano's 92 actions are plotted by type across his 65 minutes on the pitch, with under-pressure decisions outlined in black; the dashed curve behind them is his accumulating high-intensity load from Figure 4.

![One player, one match]({{ site.baseurl }}/assets/blog1/fig5_player_match.png)

The natural question for POLARIS is whether decision-making shifts as that fatigue curve climbs: does a tiring player pass more and carry less, take fewer risks under pressure?

In this single match, the honest answer is that we cannot tell. Amano's defensive share rises only marginally — from 18% of his actions in the first third of his shift to 21% in the last — and his under-pressure rate barely moves (0.33 to 0.25). Both differences sit well inside the noise of 92 events.

**That is precisely the point.** A fatigue–decision relationship, if it exists, is invisible at the scale of one player in one match. Recovering it requires pooling hundreds of players across hundreds of matches, and borrowing statistical strength across players who share a position, a role, a fatigue regime. That is what POLARIS is built to do — and it is where this walkthrough ends and the modeling begins.

---

## What we learned about the data

A few things this walkthrough confirmed, which shape the modeling to come:

- **Event timestamps and locations are clean and near-complete** (~98% of events carry a pitch coordinate), so ball location and match time are reliable state features.
- **Off-ball spatial information is scarce**: the only multi-player snapshots are shot freeze frames, averaging ~8 players and occurring only at shots. Continuous 22-player tracking is not available in this dataset. The model cannot depend on it.
- **GPS physical data exists at 15-minute resolution**, which sets the time granularity for any fatigue-conditioned analysis. Cumulative high-intensity distance is a clean, monotonic fatigue proxy, and a substituted player's record terminates naturally — encoding time-on-pitch for free.
- **Player identity, position, and under-pressure flags are present on events**, which are the raw materials for modeling decisions at the level of the individual player.

*Next post: turning these raw layers into a state representation — what a "decision moment" looks like, and how fatigue, scoreline, and context combine into the state a player acts from.*

---

*POLARIS is an independent research project. Data: Hudl StatsBomb J1 League 2024 free release + Hudl Pro physical data.*
