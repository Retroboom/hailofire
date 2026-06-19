# The Arsenal Companion Ecosystem: Audit and Comparison

A narration-friendly review of the Hail of Fire Arsenal companion website and its associated apps, followed by a comparison with contemporary companion sites and apps for other wargames. This document was written to be read aloud, so tables, code references, and line numbers have been rewritten as plain spoken prose.


## A note on scope and method.

The Hail of Fire project repository contains the source code for two of the three companion apps: the Break Point Tracker, and the Operation Overlord Campaign Tracker, which exists in both a newer version of roughly three thousand six hundred lines and an older version of roughly two thousand six hundred lines. I audited those directly at the code level. The main Arsenal force builder and Force Parity calculator live on the separate live website, which blocked automated access during this review, returning a Forbidden error through its content delivery network. So that portion is assessed from the rulebook's description plus reasonable inference, and I have flagged it as such rather than claiming firsthand inspection.


## Part One. Audit and Review of the Arsenal Ecosystem.


### What the ecosystem actually is.

The Arsenal is broader than a single page. It is a small constellation of free, web-based companion tools for Hail of Fire. There are four parts.

First, a Force Builder, on the live site, used to build platoons and companies from the national arsenals.

Second, a Force Parity, or Quick Compare, calculator, also on the live site, which automates the point-free balancing procedure from the rulebook.

Third, the Break Point Tracker, which is an in-play scorekeeper for the Break Point and Break Limit system.

Fourth, the Operation Overlord Campaign Tracker, which is a full persistent campaign system. It includes a sector and city map with territorial control, a player roster, a leaderboard, a coins and rerolls economy, a submit-for-approval game reporting workflow, and an admin panel. It is all backed by a Firebase real-time database with live synchronization between users.

That is an unusually wide footprint. Most companion tools for other games stop at list building. The Arsenal also provides in-play state tracking and campaign management that integrate with Hail of Fire's actual mechanics, such as the Break Limit and Force Parity. That integration is its single biggest strength, and it is the throughline of this audit.


### Technical profile, from the source code.

On the technology stack. These are hand-written single-file web pages, using plain HTML, CSS, and JavaScript, with no framework and no build step, and no dependencies beyond Google Fonts and the Firebase software development kit. The result is lightweight, fast to load, and trivially easy to host.

On data persistence. The Break Point Tracker uses browser local storage only, so it works without an account and survives a page refresh, though it is tied to one device. The Campaign Tracker uses the Firebase real-time database with live listeners for multi-user synchronization, which is a genuinely good fit for a live community campaign.

On theming. There is a consistent military aesthetic, using the Oswald and Black Ops One fonts plus a custom grunge font, a gunmetal color palette, Allied red and Axis charcoal, toast notifications, and tabbed navigation. It is visually polished for a hobby project.

On mobile support. The viewport is configured correctly on all pages, so the apps are usable on a phone at the table. However, there is no progressive-web-app or offline support. There is no service worker and no web app manifest.


### Findings.

Now the findings, ordered by severity.

Finding one, high severity. A stored cross-site-scripting vulnerability through player names. User-supplied player names are inserted directly into the page's HTML with no escaping. A player who registered with a name containing a malicious image tag and an error handler could execute arbitrary script in every visitor's browser, including the administrator's browser. This appears in the leaderboard rendering and in several other places where game and player data are rendered. There are over fifty places in the file where HTML is set directly, which is the pattern that enables this.

Finding two, high severity, though dependent on server configuration. Administrator access is based on client-side trust, and the password hash is readable by the public. The admin password hash is pulled down to every visitor's browser and the comparison happens in JavaScript on the client. Admin actions are gated only by a simple client-side check of an is-admin flag. This means the real protection depends entirely on the Firebase security rules on the server. The client-trust pattern strongly suggests those rules may be permissive, in which case anyone could write directly to the campaign database, bypassing the user interface entirely. There is also a trust-on-first-use weakness: if no password has been set yet, the first person to type one becomes the administrator.

Finding three, medium severity. The exposed hash enables offline brute-force attacks. Because the hash is readable on the client, a weak or unsalted administrator password could be cracked offline at leisure. There is no rate limiting and no lockout. My recommendation is to use Firebase Authentication, even anonymous authentication with custom claims, instead of a shared hashed password.

Finding four, medium severity, a maintainability concern. There are two near-duplicate copies of the campaign app that are drifting apart. The newer version is about a thousand lines longer than the older one. The risk is that a bug gets fixed in one copy and not the other. The project should pick a single source of truth.

Finding five, medium severity. There is no offline or progressive-web-app support for the in-play trackers. Tabletop venues often have poor wireless internet, so the Break Point Tracker especially should run fully offline and be installable as an app. Local storage provides only partial resilience.

Finding six, low to medium severity. There are accessibility gaps. There are no accessibility attributes anywhere in the markup. The Break Point Tracker has no proper form labels. And the two sides are distinguished largely by color, red and charcoal, which is a concern for colorblind users.

Finding seven, low severity. There are no automated tests, no shared input-validation or escaping utility, and no error handling boundaries. This is standard for a project at this stage, but the cross-site-scripting issue in finding one is a direct consequence of the missing escaping utility.

Finding eight. I could not verify the live Force Builder or Force Parity calculator, because automated access was blocked. I cannot confirm its data model, its validation, or its save and share behavior firsthand, so that piece is assessed from the rulebook description only.

Now, the positives, stated plainly. The real-time synchronization, the submit-for-approval moderation workflow, the coins and rerolls meta-economy, and the live map control are thoughtfully designed features that most single-game companion tools never even attempt. The code is clean and readable. The aesthetic is cohesive. For a free, single-developer project, the ambition and the execution are both high. The issues above are the predictable security and maturity gaps of a hobby web application, not signs of a bad design.

My top three fixes, in order. First, escape all user-supplied text before inserting it into the page, or use text-content assignment instead of raw HTML. Second, move administrator access to Firebase Authentication and lock down the database rules so that writes are validated on the server, not merely gated in the user interface. Third, add a service worker and a manifest so that the Break Point Tracker becomes an installable, offline application.


## Part Two. Comparison with Contemporary Companion Sites and Apps.

The relevant peer group splits into two camps. First, the official and community army builders for mainstream game systems. And second, the near-total absence of digital tooling in the historical-skirmish space that Hail of Fire actually competes in.


### The landscape, as of twenty twenty-six.

Forces of War is Battlefront's official web-based force builder for Flames of War, fourth edition. It is account based, it validates armies against the force organization charts, and it prints unit cards. It uses a freemium model: the core builder is usable, but many lists and cards are locked behind payment, so you need the books or digital purchases. The data is authoritative, curated, and closed.

War Room is Battlefront's official mobile app, covering Team Yankee and Flames of War, providing digital cards and play assistance, with paid content.

New Recruit is the rising universal web and progressive-web-app builder. It is free but ad-supported, with community-maintained data across dozens of systems. It can import BattleScribe files and supports link-based sharing and validation.

BattleScribe is the long-standing universal builder. Its developer has effectively abandoned it, with roughly three years without app support, though the underlying data is kept alive by a community data project. It is now being superseded by New Recruit. Its remaining strength is mature, offline-capable native apps.

There are also fan-made Flames of War builders, such as one called FOW List, while the older Easy Army tool is effectively defunct.

Chain of Command, from Too Fat Lardies, has essentially no official digital companion. Force and support selection comes from PDFs and printed army handbooks, in keeping with the publisher's analog, narrative-focused ethos. A handful of fan-made list pickers exist, and that is all.

Battlegroup, from Iron Fist Publishing, keeps its points-based lists in the printed army books. There is no polished official app, only community spreadsheets and fan calculators.


### Section A. Scope of features.

This is where the Arsenal is most distinctive. Nearly all of the competitors are list builders. Their job ends when your roster validates and prints. The Arsenal instead spans the full arc from building, to balancing, to playing, to running a campaign.

It has a force builder, comparable to the others. It has a Force Parity balancer, which has no peer, for reasons I will explain shortly. It has a Break Point Tracker for live game state, which has no equivalent in Forces of War, New Recruit, or BattleScribe; War Room is the only competitor doing any in-play assistance, and even it does not track a force-break economy. And it has a persistent campaign system with map control, leaderboards, and moderated reporting, which no mainstream army builder offers at all. Campaigns in other ecosystems are run on separate platforms, like Discord, league websites, or virtual tabletops.

My verdict for this section: the Arsenal offers the broadest ecosystem of any single-game companion in this comparison, by a wide margin. The competitors are deeper in the single dimension of list building, but far narrower overall.


### Section B. Cost and openness.

The Arsenal is free, with no ads and no paywall. Forces of War and War Room are freemium or paywalled, charging for lists, cards, and digital content. New Recruit is free but ad-supported. BattleScribe is free.

My verdict: the Arsenal is the most generous of the group. It is fully free, with no ads and no upsell, which fits Hail of Fire's "still free" positioning.


### Section C. Game-mechanic integration. This is the real differentiator.

Every mainstream builder is fundamentally a points-validation engine. You enter units, the tool sums the points, and it checks the total against an army list. The Arsenal cannot do that, and deliberately does not, because Hail of Fire has no points system. And that is precisely the point. Its Force Parity calculator automates a comparative, point-free balancing procedure that no other tool even needs to exist, because no other game is built this way. Likewise, the Break Point Tracker and the campaign map are wired directly to Hail of Fire's actual mechanics, such as the Break Limit formula of five plus two per unit, and sector control. A generic builder like New Recruit structurally cannot replicate this. It is not a data-entry problem; it is a different design paradigm.

My verdict: the Arsenal's integration with its parent game is tighter than any competitor's, precisely because it is bespoke. This is its strongest reason to exist.


### Section D. Data breadth, maintenance, and sustainability.

Here the Arsenal is at a structural disadvantage. BattleScribe and New Recruit ride a crowd-sourced data ecosystem spanning hundreds of game systems, with a contributor pipeline that outlives any single maintainer. Forces of War has official, authoritative, professionally curated data. The Arsenal, by contrast, is single-game, single-developer, with bespoke data and no visible community data format or contribution pipeline. Its bus factor is one. If the maintainer steps away, it stops, in the way that BattleScribe stalled, except that BattleScribe at least had community data that others could fork and carry on.

My verdict: the Arsenal is the weakest in the group on breadth and long-term sustainability. A documented, open data format, for example the arsenals expressed as structured data files that the community could extend, would materially reduce this risk.


### Section E. Technical maturity, security, and user experience.

Forces of War, New Recruit, and BattleScribe are mature platforms, with accounts, validation engines, offline support in BattleScribe's native apps, sharing and export, and years of quality assurance at scale. The Arsenal is lighter and faster to load, with a more cohesive visual identity and more ambitious live features, but it carries the maturity gaps from Part One: the stored cross-site-scripting issue, the client-trust administrator model, the lack of offline support, the accessibility gaps, and the duplicated source. None of the mainstream tools would ship with a stored cross-site-scripting hole in a public multiplayer feature.

My verdict: the Arsenal wins on aesthetics, on load weight, and on feature ambition, but loses on hardening, on offline capability, on accessibility, and on scale-tested robustness.


### The final verdict.

The Arsenal is best understood not as just another army builder, but as a tightly game-integrated companion ecosystem, covering building, balancing, play-tracking, and campaigns, for a single free game. Against the mainstream tools, that is Forces of War, New Recruit, BattleScribe, and War Room, it wins decisively on scope, on cost and openness, and on mechanical integration. Its Force Parity calculator and its Break Point and campaign trackers do things that a points-validation builder structurally cannot. It loses, just as decisively, on data breadth, on sustainability given its bus-factor of one, on security hardening, on offline support, and on accessibility. That is exactly the profile you would expect from a passionate single-developer project measured against commercial and community-scale platforms.

In the historical-wargaming niche it actually inhabits, the comparison is even more favorable. Its direct philosophical peers, Chain of Command and Battlegroup, ship essentially no digital companion tooling at all. So the Arsenal is not merely competitive there; it is in a class by itself.

The path from impressive hobby project to reference-grade companion runs through three things. Fix the security issues. Make the in-play trackers installable and offline-capable. And open the arsenal data so that the community can help carry it.

End of document.
