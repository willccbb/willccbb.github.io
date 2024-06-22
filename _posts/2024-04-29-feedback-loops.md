---
layout: post
title: "Five years of feedback loops"
date: 2024-04-29
description: "A summary of my PhD + thoughts on what comes next."
---

Before diving into individual projects and results, I want to start with some high-level context for the kinds of questions I’ve been thinking about over the past few years. This also serves a PhD debrief of sorts, with some thoughts at the end on what comes next.

Coming into grad school in 2019, I was obsessed with multi-agent learning. Deep RL had a lot of momentum with recent breakthrough results for Go and Poker, and self-driving cars seemed to be right around the corner. I’d just spent a summer playing around with multi-agent RL in trading market simulators. It felt, and perhaps still is, inevitable that the world would be increasingly powered by continuously learning algorithmic systems interacting with each other, giving rise to potentially chaotic feedback-loop dynamics. This was exciting, and also concerning.

During undergrad I’d fallen in love with theory, and had been studying the interconnected web of recent results in fairness, privacy, and game theory, mentored by the illustrious dynamic duo of Aaron Roth and Michael Kearns. It was clear that this flavor of theory offered a powerful toolkit for studying the social impact of machine learning systems, and I was on the hunt for research questions to work on in grad school. However:
- The basic primitives for differential privacy were well-established at this point, and the biggest open questions were in moving beyond low-sensitivity one-time queries for tabular data, and in tightening various bounds. Problems like unbounded losses for regression, node-level privacy for graphs, interactive data reuse, and synthetic data generation. Important work for bringing DP mainstream, certainly, but not quite what I wanted to write a thesis on (though I did return to it for a while — more on that another time, perhaps).
- Fairness was quickly becoming more of a political challenge than an algorithmic one, with new mutually-incompatible target constraints popping up each month, and it seemed unlikely that a “fairness constraint to rule them all” would be established any time soon, in contrast to differential privacy. Yet, like privacy, a standard recipe was emerging: pick your fairness constraint, then optimize your error objective subject to that constraint. I wanted to find questions where the goal was fuzzier, where there was more room for creativity in modeling the problem.

Fortunately, (algorithmic) game theory was the topic I was most personally inspired by, and there were still plenty of unknowns. It also felt fairly universal, in the sense that any multi-agent system with strategic actors can be viewed as a game, and it had some nice synergies with machine learning theory. So, I set off to Columbia to work with Tim Roughgarden (@tim_roughgarden) and Christos Papadimitriou, who’d been shaping the field from its early days.

Some useful facts to know about algorithmic game theory:
- No-regret algorithms (like Multiplicative Weights or Online Gradient Descent) converge to Nash equilibria in two-player zero-sum games, and some in other restricted classes of games (notably, “potential games”), when used by all players.
- Finding Nash equilibria in general non-zero-sum games is PPAD-hard, and almost certainly computationally intractable.
- No-regret algorithms converge to coarse correlated equilibria in general-sum games, and no-swap-regret algorithms converge to correlated equilibria.
- (Coarse) correlated equilibria can generally be computed efficiently in offline settings, and can be optimized over in some cases (e.g. if via linear programming there are only a constant number of players).
- Global objectives in non-zero-sum games (like “social welfare”) can vary drastically between best-case and worst-case (Nash, correlated, coarse correlated) equilibria (as well as non-equilibria) — quantifying this is the focus of studies on the “Price of Anarchy” and related metrics for utility gaps in games.

With this context, it seemed clear that equilibrium notions were a useful language for studying outcomes of multi-agent learning — practical machine learning algorithms, as well as algorithms with formal no-regret guarantees, often resemble gradient descent — but convergence to some equilibrium was no guarantee that you’d reach a good equilibrium.  And so to me, the million-dollar question was:

> How do you design multi-agent systems which give rise to desirable behavior at equilibrium?

As a system designer, modeled either as a welfare-aligned player in the game, or as an outside party with direct-but-constrained control over the utility functions for other agents, the hope would be to steer the dynamics of self-interested adaptive behavior in directions which benefit everyone. The problem seemed big, open, and challenging, but maybe possible. And it wasn’t all-or-nothing — there could be many intermediate worthwhile results along the route to a generalizable approach.
The first real theory paper I wrote in grad school (“Targeted Intervention in Random Graphs”, SAGT 2020) took this angle. We looked at a network game setting from a lovely recent paper (“Targeting Interventions in Networks” by Galeotti, Golub, and Goyal) which had a closed-form Nash equilibrium as well as a potential structure. The goal here was to optimally allocate an “intervention budget” across strategic agents in a graph to yield a best-possible equilibrium. We added a learning spin — there was already a closed-form optimal intervention when the whole graph is known, and we gave a handful of results for approximating the optimal intervention when the graph is sampled randomly from a distribution but our knowledge of its exact topology is limited.

I think it’s a neat paper, but perhaps one that was more educational for me to write than it’d be for someone else to read. The project taught me a lot about spectral graph theory and random graphs, but if you’re already an expert in those areas, you could probably fill in a lot of the results quickly by just reading the GGG paper. But it also wasn’t clear where to go next from here. Examples of potential games with closed-form Nash equilibria were limited, and most games don’t have unique Nash equilibria which will be reached by no-regret dynamics. A generalizable approach would likely need to target (coarse) correlated rather than Nash equlibria, ideally in settings beyond tabular normal-form games. So, I temporarily set aside the goal of “steering towards optimality” and shifted gears to thinking about regret minimization for multi-agent reinforcement learning.

The dominant paradigms for studying multi-agent RL were extensive-form games (think “game trees”) and stochastic (or Markov) games (think “multi-agent MDPs”). A number of people (many at CMU) were already hard at work cleaning up the big open questions surrounding extensive-form games (analogues of the results for tabular games, largely successfully), and I opted to focus instead on stochastic games, where the landscape was more ambiguous.

Regret minimization in reinforcement learning can be tricky business — this is doable in extensive-form games, and can be done in finite-horizon MDPs if rewards shift adversarially while transition kernels are held static, but seemed tricky if transition kernels can shift as well. That latter case is what you’d need to fully replicate “convergence to correlated equilibrium” in the same style as the tabular results, but it also seemed possible that alternate methods could work. I hammered on this for a while, found a few things, wrote them up, and published (“Learning in Multi-Player Stochastic Games”, UAI 2021). The results suggested that “no-regret convergence to equilibrium” might not be the end-all-be-all approach; there was a concrete setting (constant-horizon general-sum stochastic games) where:
- Polynomial-time convergence to equilibrium via decentralized learning was possible, but
- Polynomial-time convergence to equilibrium via decentralized no-regret learning is NP-hard.

But beyond this observation, this was another case where the paper was likely most useful for my own educational process, and where it wasn’t clear what steps to take next. Shortly after, other results came out which enabled decentralized convergence with longer horizons via shared randomness (if, as I assumed, non-stationary policies are allowed), but showed that the stationary policy equilibrium problem is PPAD-hard to compute even offline (see “The Complexity of Markov Games”, Daskalakis et al.). This seemed like a “nail in the coffin” of sorts for stochastic games, and I was still far from obtaining satisfying results about “steering towards optimality”.

A major burst of inspiration came from “Strategizing against No-Regret Learners” by Deng, Schneider, and Subramanian. The paper is only 11 pages, with a handful of simple-but-elegant proofs, and is likely the single most influential paper on my research trajectory to date. It motivated a focus on Stackelberg games, where one player has “first-mover advantage”, and offered a set of useful tools for analyzing the behavior of many no-regret algorithms (notably by leveraging the “mean-based” property, introduced in earlier work about learners in auctions). Crucially, it suggested an approach towards the kinds of problems I was hoping to work on. I’d also been reading papers about strategic classification and performative prediction, both of which involved studying multi-agent feedback loops through a Stackelberg lens. It seemed like there was work to be done here.

With research, there are always tradeoffs between studying narrower settings, where domain-specific assumptions can make problems more tractable, and general settings which are more broadly applicable but also more challenging. I did a mix of both. In particular, under the backdrop of the 2020 election cycle and the COVID-19 pandemic, I spent a lot of time thinking about feedback loops on social media. It was becoming clear that polarization and misinformation problems were being magnified by recommendation algorithms, and that classical recommender systems theory didn’t have immediate answers for a world where preferences are adaptively shaped by algorithms. I also thought about no-regret trajectories in normal-form games, algorithms for learning Stackelberg equilibria, and structural properties which enable steering the adaptive behavior dynamics of multi-agent systems; the recommendations project ended up opening some new angles here as well.

Then a couple months ago, after five years of thinking about feedback loops, I handed in my thesis. It was primarily comprised of the following four papers:
- “Is Learning in Games Good for the Learners?”  (NeurIPS ‘23)
- “Diversified Recommendations for Agents with Adaptive Preferences”  (NeurIPS ‘22)
- “Online Recommendations for Agents with Discounted Adaptive Preferences”  (ALT ‘24)
- “Online Stackelberg Optimization via Nonlinear Control”  (in submission) [edit: COLT '24]

I’ll talk a little about each of these here, with longer discussions to come in the future. On the whole, I’m really happy with the thesis and the results I was able to obtain. I think there’s a nice story here, and hopefully some ideas which could be useful for future research.

The first, a collaboration with Jon Schneider and Kiran Vodrahalli (@kiranvodrahalli), is somewhat intended as a spiritual successor to the paper of Jon’s I mentioned earlier. We ask a lot of questions about learning in games, and we find answers for some of them. I’ll highlight a couple here:
- “All players use no-regret algorithms in a game” tells you nothing about which CCE will be reached — for every CCE, there exists a pair of no-regret algorithms which reach it. The same is true for no-swap algorithms and correlated equilibria, as well as pretty much any other regret-equilibrium definition you’d consider. We look at general families of equilibrium notions where constraints need not be symmetric, and we analyze between tradeoffs between regret constraints and other assumptions on learning algorithms. A key takeaway: if you don’t know your opponent’s learning algorithm, committing to the Stackelberg strategy is often dominant (despite being a “positive-regret” algorithm).
- The Stackelberg strategy can be learned online against any no-regret learner via best-response query simulation. The runtime depends on their choice of algorithm; for some classes of no-regret algorithms this can be done in polynomial time, yet others necessitate exponential time.

The recommendations papers (2. and 3.) are both joint with Arpit Agarwal, and are in many ways the “heart and soul” of the thesis. These are the results I’m most proud of from grad school. We started with a set of pretty ambiguous questions:
- How can we model online recommendations in a way which gives rise to the kinds of feedback loops observed in reality while remaining analytically tractable?
- Can we gain any insight into why algorithmic feedback loops occur, and design recommendations algorithms which avoid their potentially harmful consequences?
- How do we characterize the space of possible outcomes and algorithmic benchmarks when  agent preferences are adaptive as a function of our recommendations?

And we found satisfying answers to all of these. Inspired by bandit problems, we look at sequential recommendations to an agent over T rounds, with n items in total, and where we must show a menu of k > 1 items to the agent in each round. The agent will choose a single item probabilistically (as a function of their “preferences”, which are in turn a function of their past choices), and we aim to optimize the long-run choice distribution of the agent under adversarial linear bandit losses. Of course, we can’t force the agent to make a particular choice from the menu, and the mapping between menus and choice distributions is shifting under our feet. The regret rates and feasible benchmarks depend on the details of agent’s preference model, and there were plenty of negative results along the way, but we were ultimately able to find no-regret algorithms for nontrivial benchmarks under fairly minimal assumptions on preference dynamics (basically just that the agent’s “preference scoring functions” are Lipschitz and bounded away from 0).

The final and most recent paper (4.) is with my advisors Tim and Christos, and generalizes some of the techniques developed for the recommendations problem into other settings, and also finds some connections to the earlier work on learning in games. This one isn’t online yet, but I can share a working version if you message me. Fundamentally, “steering the behavior of a system” is a control problem, typically with non-linear dynamics. These kinds of problems are hard in general. However, if the dynamics are “locally controllable”, i.e. you can always nudge the system a little bit in each direction (or stay put), then non-linearity isn’t so bad, especially if we focus on regret minimization rather than computational efficiency. The key “trick” — which we used in the recommendations problem, and extend here to applications for performative prediction, commodities pricing, and a learning-in-games problem — is that local controllability enables nested gradient-based optimization over the state space rather than a policy or action space, provided that you can track this mapping appropriately. I’m brushing over many details here, and this of course won’t capture all multi-agent settings, but we found a nice class of them where where the “behavioral steering” problem is cleanly solvable.

So, what now? After five years, I think it might be time to set aside “feedback loops” for now and try some new things. I’ve pretty much said what I wanted to say on the issue in the thesis, and my focus has been shifting elsewhere lately. Following grad school, I’ll be continuing as a member of Morgan Stanley’s Machine Learning Research group, where I’ve been spending some time over the past year, and I’ve had a lot of fun exploring new research directions there. Like many others, I’ve gotten pretty excited about LLMs; while finishing my thesis, I’ve also spent a lot of this past year reading up on the latest-and-greatest developments in the world of neural sequence modeling. I’ve been enjoying writing code more than ever before, and I've been experimenting with fine-tuning and inference tools for open-weights models. Regardless of any speculation about future model capabilities, we’re in an exciting time with many unknowns, and it seems like we still don’t quite know the limits of the current models. My list of research project ideas is growing far faster than I can keep up with, not to mention the speed at which new tools and models have been released and the backlog of papers I’d like to read. But, it does seem like “multi-agent deep RL” probably won’t be the future of AI that I fantasized about in 2019; the term “agent” already means something else entirely. The gears are turning.