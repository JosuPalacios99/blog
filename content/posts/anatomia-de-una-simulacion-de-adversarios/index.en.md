---
title: "Anatomy of an Adversary Simulation: from objective to report"
date: 2026-08-25T10:00:00+02:00
draft: false
author: "j0su"
tags: ["red-team", "adversary-simulation", "threat-intelligence", "mitre-attack", "pre-attack", "c2", "opsec", "purple-team"]
categories: ["fundamentals"]
series: ["Red Team: The art of thinking like the adversary"]
summary: "How an adversary simulation really works, phase by phase: defining objectives, threat modelling, PRE-ATT&CK and ATT&CK, demonstrating impact, and delivering a report that speaks the language of the business."
cover:
  image: "intro-comic-en.png"
  alt: "The heist blueprint"
  relative: true
ShowToc: true
TocOpen: false
---

## Introduction

In the [first article](../red-teaming-pensar-como-el-adversario/) of this series we covered _what_ Red Teaming is; in the [second](../que-servicio-de-seguridad-ofensiva-necesitas/), _when_ each service fits. The question almost nobody gets to see from the inside remains: once the engagement is signed, how is it actually done? What happens between the contract and the final report landing on the table? This article opens that box, and it does so with the same cast of characters: the museum with its most prized painting, the director who believes everything is safe, the guard who does not watch the cameras, and above all the thief, one of the good guys, one who plays on our team. Every phase of a simulation has its mirror in how that thief would plan and execute the heist.

At its core, simulating an adversary means reproducing the behaviour of someone who is genuinely coming after you, in order to measure whether your defence notices and reacts. Its goal is not to hand over a list of vulnerabilities (other services already do that) but to answer three uncomfortable questions: do they detect it?, how quickly?, do they know what to do once they detect it? And it tests all three legs at once, just like in the museum: technology (the cameras, the display case), processes (the key hanging in the utility room), and people (the guard who does not watch the screens). A real attacker will always go for the weakest, and it is rarely the technology.

One nuance worth locking in right now, because it shapes everything that follows: we are talking about _simulation_, not _emulation_. Emulating means copying a specific, real adversary, reproducing their known techniques step by step. Simulating is more open: a _plausible_ adversary is built, one that is credible for your organisation, without the straitjacket of tracing a single actor move by move. And that freedom is precisely the advantage, because it lets the team adapt on the fly, switch techniques when one fails or when the defence reacts, just as a real attacker would. Emulation follows a script; simulation improvises within a profile. Our thief is not a famous thief whose every gesture we mimic: he is a capable thief we let loose to see how the building holds up.

With that in mind, the real work begins. And the first thing, before touching anything at all, is to be crystal clear about what we are looking for.

![The heist blueprint](intro-comic-en.png)

## Defining the objectives: the decision that moves the most value

Here is an idea that surprises a lot of people: the value of a simulation is largely decided before execution even starts. Not at the keyboard, but in the meeting where they decide what will be tested. An exercise is worth exactly as much as the questions it was designed to answer. And those questions are the objectives.

![Choosing the objective that matters: the king on the board is the painting to steal, the pawn is initial access](objetivos-comic.png)

Back to the museum: telling the thief _"get in and grab whatever you can"_ sounds flexible, but it is the worst way to begin. Without a clear target, it is the thief himself who decides on the fly which gallery matters, where to invest time, and what counts as success. He may walk out with a vase from the lobby and call it a day, while the painting that actually keeps you up at night is still hanging untouched. The result is an exercise full of findings that are technically correct yet leave unanswered exactly what you cared about.

### The problem with "get in and see what you find"

An open scope, along the lines of _"try whatever you can think of"_, drags three problems that come up again and again:

- Misaligned expectations. With a fuzzy scope, every stakeholder builds their own picture of what was being tested, and nobody agrees when reading the results. A clear objective ties the exercise to a concrete decision from minute one.

- The environment moves. With permission to explore everything, the team spends days on paths that stop mattering when the environment changes mid-exercise. A bounded objective concentrates effort on what weighs the most and is less likely to shift.

- Time spent deciding, not testing. Without a defined target, half the operation goes into deciding where to invest effort rather than actually testing. You end up with a report full of correct observations and little certainty about what truly mattered.

To be clear: a good team will find unexpected things along any path, and that is great. But those discoveries should complement the objective-driven work, not replace it.

### A single primary objective

From here comes a very concrete recommendation: structure the exercise around a single _primary objective_. When a client arrives with two or three, the same problem reappears: the team has to decide where to focus, and that decision ends up being made in the field instead of in the scoping conversation. A single primary objective guarantees clear, actionable evidence on what matters most, rather than half-answers spread across several areas.

That primary objective is not reached in one jump: it is reached by chaining small milestones that open the way toward it. On top of that, _secondary objectives_ can be set, carrying less weight, pursued if the primary is already achieved or if time allows. The hierarchy sits at three levels: the milestones that lead to the objective, the primary objective that rules, and the secondaries that add value if there is room.

The chess analogy works well here. The _king_ is the primary objective, the one that decides the game: in the museum, stealing the Mona Lisa. You do not reach it in one move; you get there by moving pieces: sneaking through the ventilation duct, dodging the guard, reaching the gallery. Each of those moves is a milestone, a small objective that by itself wins nothing but without which you never reach the king. And if along the way you can also grab a vase from the lobby without straying from your game, that is a secondary objective: nice to have, but never at the expense of the king.

### Two levels: strategic and tactical

Objectives live at two different heights, and it is worth keeping them separate:

- _Strategic objectives._ The _what_ the exercise must achieve, the business outcome being pursued. They are documented in the contract or _statement of work_. Example: _"determine whether an attacker can reach the customers' card data"_.
- _Tactical objectives._ The _how_, specifically, the team will pursue that outcome: what is allowed, what is not, what rules of engagement apply. They are documented in the _rules of engagement_.

The strategic objective says which gallery to reach; the tactical one sets the boundaries, the hours, and which display cases must not actually be broken.

### What good objectives have in common

Objectives that work share four qualities, and all four must be present at the same time:

- _Specific_: it points to a concrete asset or outcome and focuses on _impact_, not on the method. The typical mistake is pinning down a technique ("obtain domain admin credentials") when the real question is whether the attacker could reach the confidential data. Locking in the _how_ narrows the options and loses paths that mattered just as much.
- _Answerable_: it poses a question the exercise can close within the agreed constraints. If it is too broad, there is no way to know when it has been achieved.
- _Actionable_: its answer connects to a decision the organisation is willing to make.
- _Relevant_: it reflects a real business risk, not a legacy system that nobody loses sleep over.

The difference is best seen by comparing weak and solid examples:

| Weak objective | Solid objective |
|---|---|
| "Obtain domain admin credentials" | "Determine whether an attacker with access to a corporate workstation could reach confidential customer data" |
| "Assess our security posture" | "Access the databases holding card data within the cardholder data environment" |
| "Test the AI chatbot" | "Verify whether a user with standard application access could read other users' conversations or execute code on the underlying infrastructure" |

Notice the pattern: the weak one describes a technique or a vague wish; the solid one describes a concrete, measurable outcome tied to something that hurts.

### Where to start: four families of objectives

In practice, most objectives that deliver value fall into one of these four categories:

- _Critical asset reachability._ Can an attacker reach a high-value target? Production systems, confidential data, tier-zero administrative control, the infrastructure behind an AI system.
- _Detection validation._ Would certain attacker behaviours trigger an alert? At what point in the chain would the SOC notice?
- _Response capability testing._ If detected, can the team contain the activity? What does the response reveal about escalation, containment, eviction, and recovery? An objective like this might, for example, ask to exfiltrate a large volume of data from a sensitive system, to measure not only whether it is detected but whether the team responds well under realistic conditions.
- _Assumption testing._ Does a specific security belief hold up against a real attacker? Does the segmentation, privileged access, system isolation, or sensitive data flow actually work the way it is supposed to?

### The "what if we pull it off?" test

Before signing off on an objective, one question filters almost everything: what would your organisation do if the exercise achieved it?

- _"We would prioritise remediation"_ or _"we would invest in detection"_ → good objective.
- _"It would be interesting to know"_ → needs refining.
- _"We probably wouldn't change anything"_ → probably not worth pursuing.

This same logic helps later: clear objectives make sure findings turn into remediation, detection engineering, or practice scenarios, instead of ending up as a report that gets filed and forgotten.

And there is an extra benefit, far from trivial: clear objectives are much easier to explain to leadership. _"We tested whether an attacker could reach the payments database"_ tells a CFO far more than _"we tested lateral movement techniques"_. One speaks about money; the other, about a technicality they cannot interpret.

At the end of the day, defining objectives well means committing the organisation to decide _what it needs to learn_ before the exercise begins. Not everything fits in a single simulation. The right question is not _"what could we test?"_ but _"what matters the most right now?"_. That is the painting worth stealing. With that locked in, we can talk about where the thief starts.

## Where it starts: Zero Knowledge vs Assumed Breach

With the objective clear, it is time to decide the starting point. We covered this in depth in the [first article](../red-teaming-pensar-como-el-adversario/#aproximaciones-zero-knowledge-y-assumed-breach), so here just the reminder and what changes in execution.

- _Zero Knowledge_: the thief starts on the pavement, with no blueprint and no keys, and has to do everything, from studying the building to getting the first foothold inside.
- _Assumed Breach_: it is taken for granted that the attacker is already inside (a compromised workstation, a set of credentials) and the exercise starts from there.

![Zero Knowledge versus Assumed Breach: starting from the pavement or starting already inside with a trusted insider](aproximaciones-comic.png)

For Assumed Breach, a real-world example helps. In the documentary _Sapo, S.A. Memorias de un ladrón_ (Prime Video), Jon Imanol Sapieha, alias _El Sapo_, tells how he planned in 2001 what would become the largest art theft from a private collection in Spain: he stole from the armoured penthouse vault of Esther Koplowitz paintings by Goya, Brueghel, and Juan Gris worth over 30 million euros. The interesting part is _how_ he set it up. He did not force a single lock at first: he spent three years winning over the security guard. He rented a flat next door, became his friend, and gradually drew him into a world of luxury (cars, yachts, parties in Ibiza) until the guard was on his side. When the day came, the most fortified point of the building had already been compromised from the inside: a trusted person.

Translated to the cyber world, that guard is the employee or trusted credential the attacker has already compromised: someone who clicked where they should not have, credentials leaked in a breach, or even a disgruntled employee who decides to open the door. That human factor of winning the insider cannot be simulated in an exercise, and that is precisely why Assumed Breach exists: instead of reproducing the "three years" it took to win the guard over, it starts from the position where the attacker already has him in his pocket and asks what truly matters: from that position of trust, how far can they get before someone notices?

The difference, in terms of the path we are about to walk, is straightforward: Zero Knowledge requires covering the entire pre-compromise phase; Assumed Breach compresses it and drops you straight into post-exploitation, which is where most of the value lies. That is why we will walk the full path, the Zero Knowledge one, because it includes every phase; in an Assumed Breach we would skip initial access and land in post-exploitation. With that clear, preparation begins.

## Threat Modelling: what threat are we up against

We know which painting to steal and where the thief starts. One piece is missing before lifting a finger: what kind of threat are we testing the building against? A museum does not defend the same way against an opportunistic pickpocket as against a gang that has been studying it for months. Deciding what we are up against is the work of _threat modelling_, and it is carried out by the threat intelligence team (CTI).

![The intelligence team profiles a plausible adversary: methods, tools, and modus operandi in a dossier](threat-modelling-comic.png)

Their job is to answer one question: if a museum like this were to suffer a real robbery, who would go after it and how? CTI looks at the organisation's sector, geography, exposure, and what it holds of value, and profiles which adversaries would have real motives to attack it. A financial entity attracts groups with economic motivation; a strategic company, state-sponsored actors. The defence that needs testing changes with each one.

And here the nuance we established at the beginning comes back: we are running a _simulation_, not an emulation. We do not clone a single actor gesture by gesture; we build a _plausible_ adversary, a credible profile inspired by how the groups that would attack an organisation like this actually behave, with the freedom to choose the path. With that profile, the team gathers the _how_: what techniques those groups actually use, where they get in, how they move, how they hide. It comes from intelligence reports and the track record of several actors, not a single one. The result is a catalogue of tactics, techniques, and procedures (TTPs) that are believable: the behavioural playbook the offensive team will reproduce. In the museum it would be studying how art-theft gangs operate (entering during the day as maintenance, bribing someone on the inside, exploiting the shift change) without copying a specific heist.

To keep this from being a jumble of loose notes, the industry uses a common framework: _MITRE ATT&CK_, a knowledge base that catalogues and names the tactics (the _why_: initial access, escalation, exfiltration, etc.) and the techniques (the _how_) of real adversaries. Its value is putting everyone on the same language: CTI, the offensive team, and the defensive team all reference each technique with the same identifier. On top of that framework, tools like _ATT&CK Navigator_ allow painting the TTP map of the chosen profile, which serves as a compass for what follows and, later on, to compare what the attack did with what the defence saw.

The output of this phase is a _threat profile_: who plays the adversary for us, what they pursue, with what techniques, and within what limits. That document guides everything that comes next, from what infrastructure to build to how the first entry vector is planned. And that is what follows.

## The map of the path: PRE-ATT&CK and ATT&CK

We now have the objective, the starting point, and the threat profile. From here the fieldwork begins, and it helps to have a map. Everything the offensive team does can be split into two halves, and the line between them is a very specific moment: when they begin _executing the attack vectors_, that is, when the actual attack is launched against the victim.

- _PRE-ATT&CK_, or the pre-compromise phase: all the preparation _before_ launching the attack. The thief studying the museum, getting the work overalls, and forging the fake badge, everything before setting foot in the building.
- _ATT&CK_, or the post-compromise phase: what happens _from_ the moment the vectors are executed. Getting in, orienting, moving, escalating, and reaching the painting. The thief crossing the door and advancing toward gallery 3.

![The two halves of the path separated by the door: preparation before, execution after](mapa-fases-comic.png)

Within the PRE-ATT&CK phase, work advances along two parallel tracks:

- The _analysis track_, which narrows the focus in three chained steps: from the _exposure surface_ (everything the organisation shows the world) to the _attack surface_ (what is actually attackable) and from there to the _attack vectors_ (the specific paths that will be used to try to get in).
- The _build track_, which runs in parallel: setting up the _infrastructure_ from which to operate and developing the _artefacts_ and delivery methods that will be used.

When both tracks converge, analysis and armament ready, the attack phase begins and the vectors are executed. That is the structure we will follow: first the PRE-ATT&CK phase with its two tracks, then the ATT&CK phase with what happens once the attack is launched.

> A note on precision. You may have heard of PRE-ATT&CK as a standalone MITRE matrix. It was: it existed as an independent matrix cataloguing reconnaissance and preparation techniques prior to the attack. In October 2020, with ATT&CK version 8, MITRE retired it and moved those techniques into the main matrix (_Enterprise_), where they now live as two tactics: _Reconnaissance_ (TA0043) and _Resource Development_ (TA0042). In other words, "PRE-ATT&CK" is no longer an official matrix, but the term remains very convenient for naming everything that happens before the compromise, and that is why we use it here with that meaning.

With the map in mind, we enter the first half: preparation.

## PRE-ATT&CK phase: preparing the heist

Everything that follows happens without the victim noticing a thing, because we have not touched them yet. It is the most invisible phase and, at the same time, the one that decides a large part of the outcome. A thief who shows up at the museum without having studied the building lasts five minutes. Let us walk through the two tracks, starting with analysis.

![The two parallel tracks of preparation: analysing the target and building the armament](pre-attack-comic.png)

### Analysis track (1): the exposure surface

The first step is to look at the organisation through the eyes of an attacker who knows nothing about it yet. What does it show the world without realising? This is _reconnaissance_ (in ATT&CK, the _Reconnaissance_ tactic), and in a Zero Knowledge exercise it is where everything starts from scratch.

Here, everything the organisation lets the outside see is collected, almost always without touching it directly, using open sources (OSINT): domains and subdomains, IP ranges, internet-facing services and portals, technologies in use, employee names and emails that appear on LinkedIn or in breaches, documents with metadata, forgotten code repositories, credentials already floating around from previous breaches. Everything adds up. The sum of all that is the _exposure surface_: the shadow the company casts outward.

In the museum this is the phase of studying the building from the pavement: how many entrances it has, what time the cleaning staff come in, who provides the air conditioning, what can be seen through the window. Nobody has gone inside yet; they are only observing.

### Analysis track (2): the attack surface

Not everything that is exposed is attackable. The next step is to filter that exposure and keep only what truly offers a way in. Of the hundred subdomains found, which ones run something vulnerable or misconfigured? Of the three hundred emails, which are still active and good candidates for a phishing campaign? Of the exposed services, which ones accept authentication, run an old version, or have an exploitable misconfiguration?

That subset, what goes from "it is out there" to "this could be a way in", is the _attack surface_. It is the vocabulary we already used in the [second article](../que-servicio-de-seguridad-ofensiva-necesitas/): the exposure surface is everything you have; the attack surface is what gives when you push. In the museum, it is crossing off the fortified items from the list and keeping the maintenance door whose lock has been broken for months.

### Analysis track (3): the attack vectors

With the attack surface narrowed down, the _attack vectors_ are laid out: the specific paths that will be used to try to gain initial access. It is not a "let's see what sticks" approach, but a handful of well-thought-out options prioritised by how plausible each one is for the threat profile that intelligence defined, and by how little noise it makes.

A vector might be a _spearphishing_ email to a specific group of employees with a crafted attachment; another, exploiting an exposed service with a known vulnerability; another, a web application with a misconfiguration; another, dropping a USB stick in the car park. One or several candidates are chosen, ranked by probability of success and stealth, and saved for the attack phase. In the museum these are the ways of sneaking in that the thief considers: entering as maintenance, slipping in during the shift change, bribing a supplier. He picks the best and discards the rest.

### Build track (1): the infrastructure

In parallel with all that analysis, the workshop from which to operate must be set up. A serious attacker does not launch the attack from their home laptop: they build an _infrastructure_ designed to work well and, above all, not to give them away. A few concepts are worth mentioning here, even briefly.

At the heart sits the _C2_ (Command and Control), the server from which the operator controls the compromised machines and sends them orders. There are frameworks built for this: commercial ones like _Cobalt Strike_ or open-source ones like _Mythic_ or _Sliver_. But exposing the C2 directly to the internet is painting a target on yourself, so _redirectors_ are placed in front: intermediate servers that receive traffic from the victims and forward it to the C2, so that what the defence sees is the redirector, not the real server. It is also common to separate long-term infrastructure (the one that maintains stealthy access, "on a slow burn") from short-term infrastructure (the noisier one, for one-off tasks), so that if one falls it does not take the other down with it.

On top of that come _OPSEC_ (operations security) details that make the difference between going unnoticed and getting caught: registering domains with good reputation or _categorising_ them so they look legitimate (an HR website, an online store), using valid TLS certificates to encrypt traffic and give it a normal appearance, and in general making everything the victim touches look as ordinary as possible. In the museum it is preparing the van with the maintenance company's logo, the convincing uniform, and the paperwork that holds up to a glance. The better the disguise, the later the alarms go off.

### Build track (2): artefacts and delivery methods

With the infrastructure standing, the weapons are still missing: the _artefacts_ that will run on the victim's machine and the way to get them there.

The typical artefact is a _payload_ or a _loader_: a file that, when executed, opens the communication channel with the C2 and gives the operator control of that machine. The problem is that victims are not defenceless: they have antivirus and _EDR_ watching. That is why a large part of the work here is _evasion_: making the artefact not trigger. This includes techniques to bypass Windows inspection mechanisms such as _AMSI_ (the one that inspects scripts in memory) or _ETW_ (the one that logs telemetry for the EDR), obfuscating code, or signing binaries so they look trustworthy. There is no need to go into detail; the key takeaway is that an artefact that works in the lab but gets flagged instantly in the real environment is useless.

And then there is the _delivery method_: how that artefact reaches the victim. It depends on the chosen vector. If the vector is phishing, it will be an attachment or a link in a credible email; if it is the car-park USB, it will be on the drive; if it is an exposed service, it will be delivered by exploiting that service. In the museum these are the thief's tools and the way to smuggle them in: the gloves, the lock pick, and the cleaning cart where everything is hidden on the way in.

When both tracks are ready, analysis with its prioritised vectors and build with the infrastructure and artefacts in place, preparation ends. Everything up to this point was done without the organisation noticing a thing. That is about to change.

## ATT&CK phase: executing the heist

This is where the simulation touches the victim for the first time and, with it, where the defence truly begins to be measured. Everything before was silent preparation; from now on, every action leaves a trace the Blue Team could see. This is the half that delivers most of the exercise's value.

![Already inside: the loop of orienting and moving toward the objective while dodging the cameras](attack-comic.png)

### Initial access: crossing the door

The vectors laid out in the previous phase are executed in order of priority until one works and yields the first foothold inside the organisation. This is _initial access_ (in ATT&CK, _Initial Access_): the employee who opens the email attachment and runs the artefact, the exposed service that gives way, the USB someone plugs in. At that moment, the C2 receives the first connection from inside. The thief has crossed the maintenance door dressed as a technician.

And something is already being measured here, even though the exercise has barely started: did any alarm fire when the artefact ran? Did the EDR flag anything? Did anyone in the SOC notice that odd email or that unusual outbound connection? Initial access is not just an attacker's achievement; it is the first reading of the defensive sensor.

### Post-exploitation: the loop inside the building

Gaining initial access is not reaching the painting; it is stepping through the service door into a huge, dark building. What comes next, _post-exploitation_, is the longest and most delicate part, and it is worth understanding well: it is not a list of steps walked in a straight line; it is a _loop_ that repeats over and over until the objective is reached.

The loop goes roughly like this:

- _Situational awareness_. The moment you land on a machine, the first thing is to orient yourself: where am I?, what machine is this?, who am I on it, a regular user or something more?, what domain is it in?, what is around me, what other machines and services can I see?, is there an EDR watching me? Just like the thief, freshly inside, stands still for a moment and looks: which room he is in, where the cameras are, which way to go next.
- _Act on what you see._ With that mental map, the next move is decided, and it is almost always one of these:
  - _Persist_ (Persistence): the first thing upon landing on a machine worth keeping is to secure a way back. Leave a method to re-enter if you are kicked out or if it reboots, so you do not risk losing access and having to start over.
  - _Escalate privileges_ (Privilege Escalation): go from a regular user to administrator of the machine, because many internal doors only open with higher permissions.
  - _Steal credentials_ (Credential Access): grab usernames and passwords, tokens, or tickets that allow moving as if you were someone legitimate.
  - _Move laterally_ (Lateral Movement): jump from that machine to a more interesting one, getting closer to the objective. From the maintenance room to the hallway, from the hallway to gallery 3.
  - All of this while _evading the defence_ (Defence Evasion) at every step, minding OPSEC to make as little noise as possible.
- _Re-orient._ Each action changes the situation: now you are an administrator, or you are on a new machine, or you hold another user's credentials. So the environment is assessed again and the loop repeats. Recognise, act, recognise, act, one step closer each turn.

That cycle repeats as many times as needed, always with a compass: the objective defined at the start. Nothing that does not bring you closer to the painting gets touched, because every unnecessary action is noise and an opportunity for the defence to catch you.

### Objective completion: reaching the painting

The loop ends when what you came for is reached: the primary objective set in the contract. In our case, reaching the system that authorises transfers. The thief is finally standing before the display case in gallery 3, with the painting within arm's reach.

But, and this is key, reaching the painting is not the end of the exercise. Having access to the target system means having demonstrated that it _can_ be reached; what remains is demonstrating what that means for the organisation. And that is precisely the next step.

## Demonstrating impact: from "we can" to "this is what it costs"

There is an enormous distance between saying _"we gained access to the transfers system"_ and making people understand what that implies. The first is a technical statement that sounds like noise to an executive; the second is what keeps them up at night. The work of this phase is to bridge that distance: translate the access achieved into a tangible _business consequence_.

It is the idea of _IS_ versus _SHOULD-BE_ that we saw in the [first article](../red-teaming-pensar-como-el-adversario/). The organisation believes its critical system is safe (the _should-be_); the exercise reveals the reality (the _is_). And that reality is not proven with a screenshot of an access granted, but by showing what could be done with it. Depending on the objective, that takes different forms:

- If the objective was _data_, it is demonstrated that it can be exfiltrated: not by actually taking the customer database, but by proving in a controlled way that the exit path exists and works, for example by extracting a decoy file or a limited, pre-agreed sample.
- If the objective was a _business system_, such as our bank's transfers system, the extent of the control achieved is demonstrated without actually causing harm: showing that a transfer _could_ be ordered, not ordering it; that a record _could_ be tampered with, not tampering with it.
- If the scenario is about _resilience_, such as a ransomware simulation, the encryption capability is demonstrated on a limited, agreed-upon set, to measure the real impact without bringing down operations.

The word that governs all of this is _controlled_. The goal is to _prove_ the impact, not to _cause_ it. This is where the rules of engagement agreed at the start are worth their weight in gold: they define how far you can go so that the demonstration is credible without putting the real business at risk. The good thief in the exercise takes the painting out of the display case, shows it to the camera to prove he could, and hangs it back. He does not take it home.



Demonstrating impact is what turns a technical finding into a business argument. _"An attacker with a well-crafted email could, in X days, reach the point of ordering transfers from your core system"_ moves budgets and decisions in a way that _"we found a lateral movement path to the SAP server"_ never will. And that argument is exactly what needs to be put in writing.

## The report: where the exercise becomes value

Everything above is worthless if it ends up in a report nobody knows how to read or that gets filed and forgotten. The deliverable is the real product of the exercise: it is the only thing that remains when the offensive team leaves. A report that only lists commands, CVEs, and screenshots has failed, no matter how brilliant the technical side was. A good report tells a story and ends in decisions.

A report that delivers value usually rests on four pieces:

- _The incident narrative, step by step._ The story of how the team went from the first email to the transfers system: what was done, in what order, exploiting what. Not to show off, but because that narrative is a map. It shows the complete chain and, above all, reveals at which points it could have been cut. It is the equivalent of reconstructing the museum heist minute by minute: where the thief got in, which door was open, who did not look.
- _Findings, prioritised and by root cause._ Not a flat list of flaws, but the problems ordered by severity and, when possible, pointing to the _root cause_ rather than the symptom. Ten alerts that did not fire may have a single configuration flaw behind them; attacking the root fixes all ten at once. A good report distinguishes the underlying problem from the surface scratch.
- _The translation into business risk._ Every important finding accompanied by what it means in terms that leadership understands: not _"Kerberos misconfigured"_, but _"this is what allowed reaching the payments system, and this is what it would cost if it happened for real"_.
- _Ordered recommendations._ What to change, in what order, and why. If, when the document is closed, the organisation does not know exactly what to do on Monday morning, the report has not finished its job.

And there is something this report has that a pentest report does not: it measures the _defence_. Because the exercise was carried out without alerting the Blue Team, the report can tell the other side of the story. Was anything detected? What exactly, and what went unnoticed? How long did it take to see it? How long to react? Those times, how long you take to detect and how long to respond, are worth more than any list of vulnerabilities, because those are the times that truly count on the day of a real incident.

That is why the report usually speaks two languages at once, for two different audiences. For the _technical team and the Blue Team_, the detail: the chain, the techniques, which detection rule was missing, which configuration to change. For _leadership_, the executive summary: what was demonstrated, what real risk it poses to the business, and where it is worth investing. The same exercise, told at two heights.

Comparing what the offensive team did with what the defence was able to see, that TTP list mapped with ATT&CK at the beginning, is exactly the terrain of the _Purple Team_ we introduced in the [first article](../red-teaming-pensar-como-el-adversario/): placing the executed actions and the detected ones side by side, and using the gaps between them as a map of the blind spots. That is where the report stops being a verdict and becomes an improvement plan.



## Conclusions

We have opened the box and walked through an adversary simulation from start to finish: from setting the objective to profiling the threat, from preparing the heist in the shadows to executing it, from reaching the painting to demonstrating what it would truly cost and putting it all in writing. If anything should be clear after the entire journey, it is three things.

- The value is decided before touching a keyboard. The part that most determines whether an exercise is useful is not how flashy the intrusion is, but how well-defined the objective is. A clear target turns the simulation into an answer to a question that matters to the business; a _"get in and see what you find"_ turns it into a pile of loose findings that nobody knows how to prioritise. Decide first what you need to learn.

- Most of the work is invisible. What people picture about a Red Team, someone typing until something breaks, is barely the tip. Beneath it lies threat profiling, carefully built infrastructure, artefacts crafted not to trigger, vectors thought through and prioritised. Like the thief who spends three years winning the guard over, the bulk of the heist happens before the heist. That silent preparation is what separates an exercise that teaches something from a group of people trying things.

- The exercise is the means, not the end. Reaching the objective is not winning, and the report is not a trophy. All of this is worth what the organisation does with it on Monday morning: what it fixes, what detection it builds, what it decides to invest in. A simulation is not a snapshot of your security posture; it is the push that makes it mature. And like every snapshot, it expires: security is not a state you arrive at, but something you maintain exercise after exercise.

We close with the thief standing before the display case, showing the painting to the camera before hanging it back. He does not take it. In its place, he leaves a map of how he got in, which doors he found open, and who was not watching. That map is worth far more than the painting, because it is what ensures that, next time, whoever comes in through the air duct does not walk out waving.

There is one loose end that deserves its own article: what happens when the exercise stops being an isolated snapshot and becomes a continuous programme, with Purple Team as the engine and specialised services fine-tuning every corner. But that will be another day.

## References

1. SpecterOps, _How to Set Red Team Objectives That Produce Value_ (7 Jul 2026): [specterops.io](https://specterops.io/blog/2026/07/07/how-to-set-red-team-objectives-that-produce-value/). (Foundation for the objectives section: qualities of a good objective, single primary objective, strategic and tactical levels, and the "what if we pull it off?" test.)

2. _Sapo, S.A. Memorias de un ladrón_, documentary series (Prime Video, 2022), about Jon Imanol Sapieha, alias _El Sapo_, and the theft of Esther Koplowitz's art collection (Madrid, 2001). Real case used as an analogy for the Assumed Breach approach: spending years winning the trust of an insider to exploit that position.

3. MITRE, _ATT&CK_: [attack.mitre.org](https://attack.mitre.org/). Knowledge base of tactics, techniques, and procedures (TTPs) from real adversaries, used as a common language for threat modelling and mapping the exercise. Mapping tool: _ATT&CK Navigator_.



5. Previous articles in this series: [Red Teaming: think like the adversary](../red-teaming-pensar-como-el-adversario/) (what it is, emulation vs simulation, Zero Knowledge vs Assumed Breach) and [Which offensive security service do you need?](../que-servicio-de-seguridad-ofensiva-necesitas/) (maturity ladder).
