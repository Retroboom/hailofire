# The Hail of Fire Arsenal: A Deep Feature Review and Competitive Analysis

A narration-friendly, read-aloud version. This document is based on a direct, line-by-line reading of the actual Arsenal source code, the single-page application named index dot html in the Retroboom Arsenal repository, which is just under fourteen thousand lines long, together with its data file, named hof underscore arsenal dot json. Tables, code symbols, and file names have been rewritten as plain spoken prose so that a text-to-speech reader can narrate it smoothly.


## Part One. A deep pass through every feature of the Arsenal.


### What the Arsenal is, and how it is built.

The Arsenal is a single-page web application. It is written in plain HTML, CSS, and JavaScript, with no front-end framework and no build step. Its only external libraries are the Firebase software development kit, for authentication and the real-time database; a markdown renderer called Marked, used for article bodies; and a drag-and-drop library called SortableJS, used for reordering teams and platoons. It is carefully optimized for mobile, with proper handling of the iPhone notch and safe-area insets, a dark theme, a film-grain noise overlay, and per-nation insignia drawn as scalable vector graphics.

The unit data lives in a separate file of about three hundred sixty kilobytes. It contains two hundred eighty-three units across five nations. By unit count, that is roughly eighty-one German entries, seventy-three British, forty-eight American, forty-three Soviet, and thirty-eight French. It is worth pausing on that last one: the printed rulebook ships only four arsenals, the United States, Germany, the Soviet Union, and Britain. The app has already added a fifth nation, France, that the book does not cover. Each unit record is rich. Beyond the game statistics, it carries a paragraph of historical lore, a list of theaters the unit served in, and a list of special-rule notes. So the data is not just a stat block; it is closer to a small illustrated encyclopedia.


### Accounts, sign-in, and the subscriber tier.

Sign-in is handled by real Firebase Authentication, using a Google sign-in popup. There is also a guest mode, so a player can use the tool without an account. This is a genuine, industry-standard authentication system, not a homemade password box.

There is a subscriber tier. Units and some content carry a "free" flag, and anything not marked free is locked unless the viewer is a subscriber. The lock is unlocked through a Ko-fi subscription. So the Arsenal follows a freemium model: a free core, with premium units and premium briefings unlocked by paying supporters. This is an important correction to a common assumption that the tool is entirely free. It is free to start, but not everything is free.

Administrator access is handled properly and defensively. When a user signs in, the application checks their email address against an allow-list stored on the server, in a protected database location. Crucially, the code fails closed: it drops all administrator privileges first, and only restores them after the server confirms the account is an administrator. A comment in the code explains the reasoning, that a stale session or a mid-render timing gap must never be allowed to leave administrator controls exposed. This is mature security thinking.


### The Force Builder.

The Force Builder is the heart of the application. You assemble a company from platoons, and each platoon from teams chosen out of the arsenal. Platoons are categorized into tiers, such as combat, weapons, regimental, and divisional support. The builder enforces an organization chart, much like the force-organization charts in commercial wargames. The number of combat platoons you field unlocks support slots: each combat platoon unlocks one weapons-support slot and one regimental-support slot, while divisional support is limited to one slot plus one more for every three combat platoons. In other words, you cannot field an army that is all heavy support and no infantry; the structure mirrors a real order of battle.

The builder supports unit quality, letting you mark a force as Elite or Poor, exactly as the rulebook allows. It handles support-weapon attachment, where a weapon team is folded into another team. It includes deployment helper actions, such as spreading attached teams across a platoon, sending everything to a headquarters platoon, or assigning teams one at a time. Teams can be reordered by dragging. Quantities can be changed. Platoons can be duplicated, renamed, collapsed, and locked. Each unit is shown as a detailed card with its statistics, and the application even parses armament strings to detect machine guns and display them sensibly.


### The unit detail panel and the historical browsers.

Tapping a unit opens a detail panel that shows its full statistics, its historical lore paragraph, the theaters it served in, its special rules, and a link out to a Wikipedia search for that vehicle or weapon. This turns force-building into something educational, not just mechanical.

Beyond free-form building, there are historical organization browsers. One lets you browse and add whole platoons from historical orders of battle, organized by nation. So a newcomer can start from a real, period-correct organization rather than a blank page.


### The force library and sharing.

Forces can be saved to the cloud, loaded, duplicated, deleted, and reordered. Every saved force records its author and its computed Break Limit. Forces can be shared: the application generates a shareable link, using a force parameter in the web address, and copies it to the clipboard, so another player can open your exact list. Administrators can additionally publish a force as an official Hail of Fire force, and there is a featured strip that highlights selected forces. So the library is both a personal locker and a small community showcase.


### The balancing system. This is the Arsenal's masterpiece, and it has three layers.

The first layer is the manual Force Parity Procedure from the rulebook. The application walks two players through ranking each other's platoons, pairing them up, declaring which platoon in each pairing looks more threatening, and tallying the result, including the size-advantage bonus when one platoon fields more teams than its opposite number. This is a faithful, guided implementation of the book's pen-and-paper method, including a tally that adds a point for the winner of each pairing and an extra point when that winner also has more combat teams.

The second layer is called Quick Compare, and it is a quantitative shortcut. Under the hood, the Arsenal carries a complete points formula, even though Hail of Fire is famously a point-free game. This formula, the team-score function, is genuinely sophisticated. It scores each unit from its statistics: its range, broken into steps every four inches beyond sixteen; its rate of fire, with an acceleration term so that extra shots matter more on already-deadly weapons; and its anti-tank power, mapped onto tiers with fractional interpolation between them. It then applies a series of carefully reasoned multipliers. Turreted vehicles are worth more than hull-mounted ones. Longer range is worth more. Weapons that can only inflict suppression, rather than kill, are halved. The anti-tank contribution scales with the square root of rate of fire, which the code explains as a compromise between doubling shots doubling hits and the diminishing value of extra shots once a target is already dead. Immobile guns, which cannot reposition to exploit a flank, take a penalty. Engineer teams whose anti-tank value only works in an assault are discounted. Snipers are given a flat score to reflect their ability to target leaders, a qualitative threat the formula cannot otherwise capture. Upgrades like the Panzerfaust are costed per combat squad. Recon platoons receive a forty percent premium for their extended spotting and potential double activation. Quick Compare then totals these scores for each force, grants the defender a ten percent bonus unless it is a meeting engagement, and converts the gap between the two forces into a reduction of the stronger side's Break Limit. The points themselves stay hidden from ordinary players, shown only to administrators, so the point-free feel of the game is preserved at the table.

The third layer is the most remarkable, and I have not seen its equal in any commercial wargame companion app. It is a full Monte Carlo battle simulator. The application can take two forces and actually play them against each other, hundreds of times, to estimate a win probability. The simulator models the real rules closely. Its penetration math is exactly the rulebook's: when anti-tank exceeds armor, the chance to destroy in one hit is one minus five-sixths raised to the power of the difference, which is precisely the book's rule of rolling the difference and destroying on any six. It blends front and flank armor by assuming about a third of shots catch the flank as maneuver happens. It models infantry kills with a hit probability and a worst-of-dice resolution, with about forty percent of targets assumed to be in hard cover. It models artillery batteries, including their per-game fire-mission budgets, their barrage radius growing with the number of guns, and the reduction of incoming fire by a vehicle's flank armor. In each simulated turn, every living shooter intelligently picks the enemy it can most reliably kill, breaking ties toward higher-value targets; both sides fire simultaneously; casualties are tallied against each side's Break Limit; and the game ends when someone breaks. It runs six hundred of these games for each candidate setting.

Then it does something genuinely clever. It sweeps adjustments to the stronger side's Break Limit, searching for the setting that brings the win probability closest to fifty-fifty, and recommends that as the balanced handicap. It also reports a coverage figure: the fraction of the enemy force that a given side simply cannot harm by any means, direct fire or artillery. That immunity gap, for example your anti-tank weapons being unable to scratch the enemy's heaviest tanks, is something a points system can never express, because no number of points can fix the fact that one of your units literally cannot hurt one of theirs. The simulator surfaces it directly.

So the Arsenal offers three tiers of balancing in one tool: the subjective, social method from the book; a fast, rigorous heuristic points engine; and a simulation that plays out the actual battle hundreds of times and tunes the handicap to a coin-flip. That is a spectrum from casual to scientific that no other companion app I am aware of attempts.


### The Break Point Tracker.

Built into the app is a live, in-game Break Point tracker. During play, each side accumulates Break Points toward its Break Limit. The tracker rolls the dice for you, using the rulebook's exact table: a roll of one through three generates one Break Point, four or five generates two, and a six generates three. It tracks each player's running total against their limit, detects the moment a force breaks, supports undoing the last roll, can show or hide the scores for hidden-information play, can be minimized during a game, and even plays small sound effects. Game state is saved, so a game can be resumed. This is the in-app tracker that belongs to the Arsenal, and it is wired directly to the same Break Limit the Force Builder computes.


### Briefings, the field manual, and the rules reference.

The Arsenal includes a content system. There are briefings, which are articles written in markdown and rendered in the app, with support for drafts, published posts, pinned posts, teaser previews, a reader view, an in-app editor for administrators, share links, and a what's-new prompt. Some briefings are subscriber-only. There is also a searchable rules reference, with text search, highlighting of matched terms, and jump-to-section navigation. So the application is not only a builder; it is also a living rules companion and a small magazine.


### Audit findings on the real Arsenal.

Now the problems, stated honestly and ordered by severity.

First, a medium-severity issue. The markdown in briefings is rendered to the page without sanitization. The modern version of the Marked library does not strip dangerous HTML by default, and the code comment even describes the output as, quote, sanitized-ish, end quote. That means malicious markup placed in a briefing body could execute script in a reader's browser. Because only administrators can author briefings, the practical risk is limited to an administrator harming themselves or to a compromised administrator account, but it should be closed by passing the rendered output through a sanitizer such as DOMPurify.

Second, a low-to-medium issue, inconsistent output encoding. The application has an escaping helper and uses it in many places, around thirty-eight of them. But there are also many places where names are inserted into the page without escaping. Most of those are trusted, static arsenal data, like unit and weapon names, so the real risk is low, but the inconsistency is a latent hazard. Every free-text field a user can type, such as a force name or a display name, should be routed through the escaping helper before it reaches the page.

Third, a verification item rather than a confirmed flaw. The client-side security is well designed, but the real enforcement lives in the Firebase database security rules, which cannot be inspected from the downloaded code. If those rules are permissive, the administrator list, the force library, and the briefings could be written to directly, bypassing the carefully written interface. Confirming and locking those rules is the single most important thing to verify.

Fourth, a medium issue. There is no offline or installable-app support. There is no service worker and no web-app manifest. Given that the Break Point Tracker is meant for use at the table, where venue wireless internet is often poor, the inability to run fully offline or install the tool as an app is a real limitation.

Fifth, a low-to-medium issue, accessibility. For an application of nearly fourteen thousand lines, there are very few accessibility attributes, almost no explicit roles, and only a handful of form labels. The interface leans heavily on color and on custom controls, which is a barrier for users of assistive technology and for colorblind users.

Sixth, a sustainability note rather than a bug. This is a large, ambitious, single-developer application with bespoke data and no visible community contribution pipeline. The freemium subscription does provide some revenue to support maintenance, which helps, but the long-term resilience of the project still rests largely on one person.

To be clear about the positives, because they are substantial. This is a genuinely impressive piece of software for a hobby project. Real authentication with fail-closed administrator gating. An escaping helper that is actually used. A three-tier balancing system culminating in a faithful Monte Carlo simulator. Organization-chart validation. Shareable forces. A built-in live tracker. A markdown content system. A searchable rules reference. Rich, lore-filled data across five nations. Careful mobile rendering. The defects are a sanitization gap, some encoding inconsistency, unverifiable database rules, and the absence of offline support and accessibility polish.


## Part Two. A deeper comparison with companion apps for other games.

To compare fairly, it helps to first separate the kinds of companion tools that exist, because most products only do one or two of these jobs, while the Arsenal attempts nearly all of them.

There are six distinct jobs a wargame companion can do. One, list building and validation, meaning assembling a legal army. Two, combat math, meaning calculating the odds of a single attack. Three, force balancing, meaning telling two unequal forces how to even the odds. Four, in-game tracking, meaning keeping score during play. Five, rules and reference, meaning searchable rules and unit data. And six, content and community, meaning articles, sharing, and a library. Let me walk through the landscape job by job.


### Job one. List building and validation.

This is the most crowded category. For Flames of War, Battlefront runs an official web builder called Forces of War, which validates against the force-organization charts and prints unit cards, on a freemium basis where many lists and cards must be purchased. Across many game systems, the long-standing universal builder was BattleScribe, whose developer has effectively stepped away, leaving the community data to be maintained by volunteers; its rising successor is New Recruit, a free, ad-supported web builder that imports BattleScribe files and supports dozens of systems. For Battlegroup, the very game Hail of Fire most resembles, there is no official app, but there is a capable community-made, open-source web builder, the Battlegroup Builder, maintained on GitHub and updated as recently as early twenty twenty-six, which lets you pick from many army lists across theaters. For Bolt Action there are apps such as Second in Command and Codex Commander.

On this job, the Arsenal is competitive but not dominant. Its builder is polished, enforces an organization chart, and carries beautiful lore-rich data, but it covers one game and five nations, whereas BattleScribe and New Recruit cover hundreds of systems through crowd-sourced data, and Forces of War carries the authority of the official publisher. The Arsenal's advantage here is integration and presentation, not breadth.


### Job two. Combat math.

Some ecosystems, especially Warhammer forty thousand, have excellent standalone combat calculators, such as UnitCrunch and Tactical Cogitator, which compute the probability distribution of damage for a single attack sequence. These are sharp, popular tools. But note what they do: they answer the question, if this unit shoots that unit, what happens. They do not build forces, they do not balance armies, and they do not play out whole games.

The Arsenal contains this capability, but as an internal engine rather than a user-facing calculator. Its kill-probability functions are the same kind of math the forty-thousand calculators expose, but they are wired into something larger.


### Job three. Force balancing. This is where the comparison becomes lopsided.

Almost every game in the hobby balances forces through a static points system written by the publisher. You add up points, and equal points are assumed to be a fair fight. The army builders above are, at heart, points calculators that enforce a number. None of them tells you, for two specific and unequal forces, how to adjust the game so the match becomes a coin flip. None of them simulates the actual battle.

The Arsenal does exactly that, and it does it three ways: the social ranking method, a rigorous hidden points heuristic, and a six-hundred-game Monte Carlo simulation that tunes the Break Limit until the win probability is as close to fifty-fifty as it can get, and that also reports which enemy units you are simply incapable of harming. In the entire landscape of tabletop companion apps, simulation-based force balancing is essentially unheard of. Monte Carlo methods appear in military-analysis and sports-betting tools, and single-attack probability appears in the forty-thousand calculators, but a companion app that plays a specific tabletop army against another specific army hundreds of times to set a fair handicap is, as far as I can determine, unique to the Arsenal. This is its single strongest claim to originality, and it is a direct and elegant answer to the one weakness in Hail of Fire's point-free design, namely the subjectivity of judging a fair fight.


### Job four. In-game tracking.

Some apps do this. There are battle-tracker apps in the forty-thousand world that track command points, objectives, and score during a game. But for historical company-level games like Flames of War, Battlegroup, and Chain of Command, in-app live tracking that is wired to the game's own attrition rules is rare to nonexistent. The Arsenal's Break Point Tracker is built into the same tool that builds and balances the force, and it rolls the game's actual Break Point dice and counts against the very Break Limit the builder computed. That end-to-end continuity, from building a force to tracking its collapse, is something the standalone builders cannot offer because they stop at the list.


### Job five. Rules and reference.

Battlefront's separate War Room app provides digital cards and play assistance for its games, and some publishers, such as the makers of Warmachine, ship official rules apps. The Arsenal bundles a searchable rules reference and, through its unit detail panels, a genuinely educational layer of historical lore with outbound encyclopedia links. It compares well here, though the big publishers' official apps carry the authority and the production budget.


### Job six. Content and community.

Most builders are purely utilitarian. The Arsenal adds a markdown-based briefings system, a featured-forces showcase, shareable force links, and a what's-new feed, blurring the line between a tool and a small online magazine. Few single-game companions attempt this.


### The historical-wargaming context, specifically.

It is worth narrowing to the games Hail of Fire actually sits beside, because the contrast is sharpest there. Chain of Command, from Too Fat Lardies, has essentially no digital companion at all; its force selection lives in PDFs and printed handbooks, by design and temperament. Battlegroup has a good community-built list builder, but it is a list builder only: no balancer, no simulator, no in-game tracker, no integrated rules or lore. Flames of War has the most developed official digital presence of the three, through Forces of War and War Room, but those are a points-validating list builder and a card-and-rules aid respectively, and they are monetized more aggressively. Against this specific field, the Arsenal is not merely competitive; in scope it is in a category of its own, because it is the only one that spans building, balancing by simulation, live tracking, rules, lore, and community in a single free-to-start tool.


### Where the Arsenal still trails.

Honesty requires naming the gaps. On the breadth and longevity of data, the crowd-sourced ecosystems behind BattleScribe and New Recruit, spanning hundreds of games and sustained by many volunteers, are far more resilient than one developer maintaining one game's data. On official authority, Forces of War and the Warmachine app carry the publisher's stamp and budget. On platform maturity, the established tools offer offline use, app-store installs, and years of testing at scale, while the Arsenal has no offline mode and some unresolved hardening and accessibility work. And on monetization clarity, the freemium model, while sensible, means the Arsenal is no longer the unambiguously free option it is sometimes assumed to be.


### The final verdict.

Measured as a list builder alone, the Arsenal is good but not exceptional, beaten on breadth by the universal tools and on authority by the official ones. Measured as a complete companion ecosystem for a single game, it is extraordinary, and its Monte Carlo balancing engine is, as far as I can find, genuinely without peer in the tabletop hobby. It turns the central philosophical bet of Hail of Fire, that a fair fight should be judged by the situation rather than by a points list, into working software that can actually simulate the fight and hand you a balanced handicap. In the historical-wargaming niche it inhabits, alongside Chain of Command and Battlegroup, it is not just ahead; it is playing a different game entirely. The work that remains, sanitizing the markdown, finishing the output encoding, verifying the database rules, adding offline support, and improving accessibility, is the difference between an exceptional hobby project and a reference-grade product. The foundation is already there.

End of document.
