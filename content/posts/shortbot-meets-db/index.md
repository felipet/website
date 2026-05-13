+++
date = '2025-02-19T09:10:00+01:00'
title = 'Shortbot Meets DBs'
author = "Felipe Torres González"
cover = ""
description = "The IbexShortBot included a long desired feature: access to a local cache with the short positions to avoid collecting those straight from the source. A huge reduction of the latency is the main outcome of the changes, but there is much more."
categories = ['shortbot']
tags = ['shortbot', 'programming', 'finance', 'ibex35']

+++

# Introduction

The [IbexShortBot](/projects/shortbot) was a stateless bot. A stateless bot doesn't keep any sort of information of the users. It just replies to the user's requests, one at a time. Despite that design decision made the bot very restricted on its features, it helped me to focus on delivering a solution as quickly as I could, to attract investor's interest on using the bot.

The bot has been used for six months and the user's feedback is positive. That's enough for me to decide to take it a step further. 

**It's time for the bot to meet databases.**

# New features

I ran a survey between the users of the bot, and by far, the most requested feature is related to following stocks and receiving updates when any short position changes.

Compared to the current flow, in which an user has to select the stock every time she/he aims to check whether the position changed or not, the subscription-based flow would heavily simplify the user's experience, and it would avoid missing any update in a position.

It seems a simple change: add an SQLite database, keep user's subscriptions and run a timed-service to inform users when a change occurred.

But this time, I want to think big. That flow was in my mind since the beginning, but I considered that it was more interesting to deliver a simple solution quickly, than a full solution that would have required more development time.

# A Fully Integrated Stock Market Data Information System

One year ago, I started to work on the idea of having a Rust-based ecosystem for investment analysis and trading tools. The overall idea is having all the data centralised in a local database system that would feed my analysis tools.

Information about investment and trading is everywhere these days. One just needs to choose an investment strategy and follow it. However, what if I'd like to explore and implement my own financial models, or design custom trading oscillators? What if I rather make my own way than follow someone else's? Most would say, that's stupid, as only a few can beat the market. I'll say that mostly true when you look where most of people do. But I'm convinced that it is possible to beat inefficient markets. 

I won't enter in what is an efficient market, and what is not. It would be a pretty long explanation (it would well deserve it's own blog entry). Let's say, very very briefly, that an efficient market would find a fair price for a stock pretty quickly, whilst an inefficient market wouldn't do it. It means that inefficient markets include both under valuated and over valuated stocks in higher proportions than efficient markets. Here's where one can beat the market.

My market of choice is the IBEX35, and I do believe it is an inefficient market. Why do you tell me all this nonsense, you'd ask. The answer is related to what I said in the second paragraph of this section: inefficient markets don't offer the market information as easily as efficient markets do, and that's why it is so important for me to develop a **fully integrated stock market data information system**.

And a (small) part of this information system is the **IbexShortBot**.

## IbexShortBot and The Finance Data Harvesting Library

A few days ago, I posted an entry in the blog talking about a new library that I wrote in Rust for collecting financial data from several sources and centralise it in a local database. You can read more about it in this entry: [A Finance Data Harvesting Library](./posts/finance_data_harvester).

The library along a CLI tool that makes use of it are in production now, and the bot simply needs to connect to the DB, and retrieve the active positions for the chosen ticker by the user. That meant I had to remove all the code related to collecting and processing data from the external source (CNMV's page) and write a simple handler for the database. The changes are included in the [PR-11](https://github.com/felipet/shortbot/pull/11) of the repository.

# Improvements Achieved

The most obvious improvement is the huge reduction in the latency of an user request: **from tens of seconds to nearly zero!** Short positions are cached now in my local database, so the bot just needs to issue a query rather than a complex procedure to collect and parse data from an external website (which is pretty slow, by the way).

Only that would have been worth the changes and the effort. It's a change that heavily improves user's experience.

Other improvements are not so obvious to end users, but equally important as they leverage more important changes to come: positions are kept in a database, so the bot will not only access current short positions (as it happened before) but an entire registry with the evolution of the positions. Further analyses will be possible such as tendency of the overall short position against a company, track the activity of a particular hedge fund, or get notified when a company starts getting short positions. There are endless possibilities!

Aside from that, the project is better structured now because the logic related to the data harvesting is taken to a different library, which means the bot repository only tracks code related to the bot itself.

Finally, the usage of the external website with the data (CNMV's) is fairer now as the requests are centralised from a single place which scans the web a few times per day. Before, the website was requested every time an user of the bot asked for a list of positions. Also, if multiple users requested the same data, it meant that multiple requests were made rather than sharing the results as it happens now. I find it important as well, as I prefer not to abuse of external free services, specially when it is obvious that they suffer from performance issues.

# Future Plans

The new feature means a considerable step forward for the bot. It is no longer a stateless bot: short positions are kept and an historical record is available. However, the only current benefit of having such record is the latency reduction for the access to the current positions. There is no logic that makes use of past positions.

Including further features that make use of the historical record of short positions scores high in my _todo_ list. Many possible analyses are available now. Also, it will be possible to detect changes: new positions, positions that got removed, ... And notify users, which means a huge change: from a request-based flow to a subscription-based flow.

While including new features related to analysis is sort of easy, going from a request-based flow to a subscription-based flow is not. The architecture of Teloxide bots (the framework in which is based the **IbexShortBot**) makes it rather difficult if possible. Thus, it will require a lot of analysis and effort, anyway I am keen to see this feature working, so I will take the challenge!



