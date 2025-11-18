What's the state of the literature and scholarly opinions on developing solutions in balance theory for balancing a graph? What's the state of this field, I suppose? Do people sit around writing algorithms, balancing things? What is the best solution known? Like the most computationally efficient? Are there any algorithms that are able to solve it in one pass? Is the emphasis on finding the most energy-efficient, like the minimum way to get to a solution? Are Monte Carlo algorithms used? stuff like that

Short version: yes, people absolutely “sit around writing algorithms to balance things” — but the moment you ask for *global* optimality, you’re in NP-hard hell, so the field is mostly about clever relaxations, heuristics, and dynamics that *tend* toward balance rather than magically fixing everything in one cheap pass.

I’ll walk through the landscape and hit your specific questions.

---

## 1. What problem are we actually talking about?

Structural balance theory = signed graphs (edges + / –) and the condition that **every cycle has positive sign** (product of edge signs = +1). Equivalently: the nodes can be partitioned into at most two “camps” such that:

* edges within each camp are positive
* edges across camps are negative

That’s a *balanced* graph (Cartwright–Harary theorem).

Two main algorithmic problems:

1. **Testing**: “Is this graph balanced?”
2. **Balancing / distance-to-balance**:

   * Minimal number of sign flips / edge deletions to *make* it balanced = **frustration index** (also called line index of balance).([ScienceDirect][1])

Your “balancing the graph” is essentially: choose a node partition (and maybe flips) minimizing frustration index / energy.

---

## 2. Complexity: why there’s no magic one-pass exact algorithm

Key fact:

* Computing the **frustration index** is NP-hard, and under standard assumptions it’s even NP-hard to approximate within any constant factor for general graphs.([Wiley Online Library][2])
* It’s equivalent to ground-state energy of an Ising spin glass and closely related to MAX-CUT.([ORSNZ][3])

There *are* special cases:

* **Planar graphs**: frustration index can be computed in polynomial time (via planar MAX-CUT reductions / spin-glass tricks).([ORSNZ][3])
* **Tiny parameter k** (frustration index): fixed-parameter algorithms exist in O(2^k m²), but k has to be extremely small to be useful.([ORSNZ][3])

So:

* Checking *if* a signed graph is balanced is easy.
* Minimally *fixing* it is fundamentally hard; no known globally optimal fast one-shot solution in general.

This is the backdrop for basically all algorithmic work.

---

## 3. What do people actually do? (State of the field)

Broadly three streams:

### A. Static optimization: frustration index & partitions

This is the “solve the NP-hard problem as well (or as fast) as we can” camp.

1. **Exact ILP / binary optimization models**

   * Aref, Mason, Wilson develop several ILP formulations that compute the frustration index exactly and show they can handle real signed networks with ~15,000 edges in under a minute using Gurobi on modest hardware.([arXiv][4])
   * These are essentially “best exact methods” right now for general graphs; they’re still exponential in the worst case, but heavily optimized.

2. **Spin-glass / energy-landscape methods**

   * Facchetti–Iacono–Altafini map structural balance to an Ising spin glass and use ground-state algorithms from statistical physics to compute global balance of large online social networks.([PNAS][5])
   * Follow-up looks at the **low-energy landscape**: number and structure of metastable states, etc.([Physical Review][6])

3. **Heuristics & metaheuristics (memetic, local search, etc.)**

   * Lijia Ma et al. (MLMSB) and Sun et al. (Meme-SB) propose **memetic algorithms** (evolutionary + local search) to reduce a weighted energy function and perform “balance transformation.”([ScienceDirect][7])
   * These scale well to big graphs, but you lose any guarantee of exact optimality.

4. **Correlation clustering & approximation algorithms**

   * The “minimum disagreement” correlation clustering with 2 clusters is equivalent to finding a partition minimizing frustration (for complete graphs).
   * Giotis & Guruswami give a PTAS with runtime n^O(1/ε²) for (1+ε)-approximation. Recent work (Ashvinkumar et al. 2023) improves this to *almost* quadratic time in n for fixed ε.

   So for *complete graphs*, the “best known” in the theoretical sense is a near-quadratic time randomized (1+ε)-approximation for the optimal two-camp partition.

### B. Streaming / “single-pass” algorithms

This is directly on your “one pass” question.

2023 paper **Efficient Streaming Algorithms for Structural Balance** (Ashvinkumar et al.) looks at complete signed graphs where edges arrive as a stream:

* **Testing balance**:

  * They give a **single-pass randomized algorithm using O(log n) space** that decides whether the graph is balanced with high probability. This is essentially optimal in space (you need Θ(log n) bits just to encode one vertex).
* **Approximate frustration-minimizing partition**:

  * They give a **single-pass semi-streaming algorithm** (Õ(n/ε²) space) which outputs a partition whose frustration is within (1+ε) of optimal, with high probability, in polynomial time.
  * Offline (non-streaming), they also give an improved nearly-quadratic-time (1+ε)-approximation, improving on Giotis–Guruswami after ~20 years.

Important: there is a lower bound in the same paper showing that **any single-pass algorithm that finds the exact optimum partition needs Ω(n²) space**, i.e. you basically have to store the whole graph. No nontrivial exact one-pass solution exists in the streaming sense.

So: “one pass” **randomized approximate**? Yes.
“One pass exact”? No, provably impossible in the usual models.

### C. Dynamic models / gradient flows

This is the physics-y / ODE side: given a signed network, evolve it according to some local rule until it “balances” or reaches a metastable state.

Examples:

* **Triangle-flip dynamics**: Antal–Krapivsky–Redner (2005) flip edges in unbalanced triangles and study the time to reach a balanced configuration and number of jammed states.([Physical Review][8])
* **Continuous-time ODE**: Marvel–Strogatz–Kleinberg consider a matrix X of friendliness and the system dX/dt = X², proving generic initial conditions lead either to all-friendliness or two hostile factions = structurally balanced outcome.([PNAS][9])
* **Gradient flows on signed Laplacians**: Cisneros-Velarde et al. formulate balance as minimizing an energy associated with the signed Laplacian and study convergence of gradient flows.([eScholarship][10])
* **“Structural balance in one time step”** (Krawczyk & Kułakowski 2022): for *complete graphs with odd n* they derive an explicit **one-step update formula** that takes any signing to a balanced one.([ScienceDirect][11])

Caveat on that last one: it’s *one synchronous global update* using full knowledge of the graph, not a streaming “read edges once” algorithm. It’s basically a closed-form dynamical map, not a low-information pass.

These dynamics typically follow some **energy function** (counts of unbalanced triads, Ising Hamiltonian, etc.). They converge to local or global minima—but no guarantee the *path* is the minimal number of flips.

---

## 4. “Energy”, cost criteria, and what people optimize

There are several typical objectives:

1. **Frustration index**: minimize number of “frustrated” edges (those inconsistent with the camp assignment). This is the standard “distance to balance” metric.([SciSpace][12])

2. **Triadic energy**: Marvel et al. define an energy counting unbalanced triads; balanced states are ground states.([Physical Review][13])

3. **Weighted energy with transformation costs**: Ma et al. (MLMSB) and Du et al. allow different edges to carry different costs for being flipped / maintained and define a more general energy H_w to model “expensive to change” relationships.([ScienceDirect][14])

4. **Partial/weak balance and diversity**: more recent work uses frustration indices for *weak* balance and relates them to diversity indices, with the same NP-hardness baggage.([PubMed Central][15])

Is the emphasis on “minimum effort path” (fewest flips along the trajectory)? Not really. People mostly care about:

* final energy minimum (global or local)
* quality of approximation to global minimum
* algorithmic runtime / scalability

The *path length* (how many intermediate changes) is usually not the main objective, except in a few papers about transformation sequences and “reversing” balance.([ScienceDirect][14])

---

## 5. Monte Carlo and stochastic methods

Yes, Monte Carlo is a standard tool, especially in the physics-adjacent work:

* Marvel’s “Energy Landscape of Social Balance” uses concepts of metastable states and energy barriers, naturally analyzed with Monte Carlo / random walks on the state space.([Physical Review][13])
* Facchetti et al. use algorithms for ground-state search in Ising spin glasses, which heavily rely on simulated annealing / Monte Carlo or related heuristics to explore low-energy states of huge graphs.([PNAS][5])
* Heat-bath dynamics + mean-field approximations have been applied to structural balance dynamics in complete graphs to derive critical temperatures and phase behavior.([APS Journals][16])

So yes: Monte Carlo, simulated annealing, heat-bath dynamics, etc. are very much in the toolkit.

---

## 6. Direct answers to your specific questions

**Q: “What’s the state of the literature / field?”**

* Mature but still active.
* We have a clean graph-theoretic framework (Heider, Cartwright–Harary, Zaslavsky, etc.), and a modern wave of work since ~2000 tying it to **spin glasses**, **correlation clustering**, and **large-scale network analysis**.
* Recent directions:

  * exact & ILP methods that scale to mid-size networks
  * near-quadratic (1+ε)-approximation algorithms
  * streaming / sketching algorithms for massive graphs
  * dynamic models (ODEs, cellular automata, gradient flows)
  * odd stuff like **quantum balance theory** beginning to appear.([arXiv][17])

**Q: “Do people sit around writing algorithms, balancing things?”**

Yes. A significant chunk of work is precisely:

> Given a big signed network, can we compute how far it is from balance, and how to minimally modify it?

The Aref & Wilson line, the memetic algorithms, the streaming algorithms, and the physics-based ground-state solvers are all exactly that.

**Q: “What is the best solution known? The most computationally efficient?”**

For general *complete* graphs, in the theoretical sense:

* Best known: randomized (1+ε)-approximation to the frustration-minimizing partition in essentially **Õ(n²/ε²)** time (for fixed ε) using correlation-clustering style methods.
* For exact solutions: ILP / branch-and-cut / spin-glass solvers are the best practical tools, but they’re still exponential in worst case.

No known polynomial-time exact algorithm for general graphs, and none is expected.

**Q: “Are there any algorithms that are able to solve it in one pass?”**

Depends what you mean:

* **Streaming / one-pass over edge list**:

  * Exact frustration-minimizing partition: **no**, provably impossible without Ω(n²) space.
  * Balance testing: **yes**, there is a single-pass randomized algorithm using O(log n) space for complete graphs.
  * (1+ε)-approx frustration partition: **yes**, single pass in semi-streaming space Õ(n/ε²), but only approximate.

* **One-step dynamics**:

  * On complete graphs with odd n, there is a closed-form one-step update (Krawczyk & Kułakowski) that maps any signing to a balanced one — but it’s not a streaming algorithm; it assumes full global access and does a matrix-style transformation.([ScienceDirect][11])

**Q: “Is the emphasis on the most energy-efficient (minimum way to get to a solution)?”**

Mostly the emphasis is on:

* minimum *final* energy / frustration index
* quality of approximation
* and computational cost (runtime, memory)

Exact minimal-length transformation sequences are not the mainstream focus; they’re occasionally studied but the big workhorses are “jump to a low-energy configuration,” not “take the fewest discrete edge flips along the way.”

**Q: “Are Monte Carlo algorithms used?”**

Yes. Explicitly and heavily in:

* spin-glass / energy-landscape work
* heat-bath and Glauber dynamics
* heuristic solvers for large networks where exact ILP is impossible

---

If you want, next step I can sketch a concrete algorithmic stack you could actually implement:

* quick linear-time “is it balanced?” check
* a simple local-search heuristic to decrease frustration
* plus how you’d plug in Monte Carlo / simulated annealing if you wanted to explore multiple low-energy “worlds” of the same signed network.

[1]: https://www.sciencedirect.com/science/article/pii/S0166218X16304073?utm_source=chatgpt.com "Frustration and isoperimetric inequalities for signed graphs"
[2]: https://onlinelibrary.wiley.com/doi/full/10.1002/net.21907?utm_source=chatgpt.com "A modeling and computational study of the frustration index ..."
[3]: https://orsnz.org.nz/Repository/CONF51/ORSNZ17_Aref.pdf?utm_source=chatgpt.com "Computing the frustration index in signed graphs using ..."
[4]: https://arxiv.org/abs/1611.09030?utm_source=chatgpt.com "A modelling and computational study of the frustration index in signed networks"
[5]: https://www.pnas.org/doi/10.1073/pnas.1109521108?utm_source=chatgpt.com "Computing global structural balance in large-scale signed ..."
[6]: https://link.aps.org/doi/10.1103/PhysRevE.86.036116?utm_source=chatgpt.com "Exploring the low-energy landscape of large-scale signed ..."
[7]: https://www.sciencedirect.com/science/article/abs/pii/S0950705115001811?utm_source=chatgpt.com "A memetic algorithm for computing and transforming ..."
[8]: https://link.aps.org/doi/10.1103/PhysRevE.72.036121?utm_source=chatgpt.com "Dynamics of social balance on networks | Phys. Rev. E"
[9]: https://www.pnas.org/doi/10.1073/pnas.1013213108?utm_source=chatgpt.com "Continuous-time model of structural balance"
[10]: https://escholarship.org/content/qt8bm719kp/qt8bm719kp.pdf?utm_source=chatgpt.com "Structural Balance via Gradient Flows Over Signed Graphs"
[11]: https://www.sciencedirect.com/science/article/abs/pii/S0378437122007117?utm_source=chatgpt.com "Structural balance in one time step"
[12]: https://scispace.com/pdf/balance-and-frustration-in-signed-networks-439jmd0ipl.pdf?utm_source=chatgpt.com "Balance and Frustration in Signed Networks under ..."
[13]: https://link.aps.org/doi/10.1103/PhysRevLett.103.198701?utm_source=chatgpt.com "Energy Landscape of Social Balance | Phys. Rev. Lett."
[14]: https://www.sciencedirect.com/science/article/abs/pii/S0378437116305441?utm_source=chatgpt.com "Optimizing transformations of structural balance in signed ..."
[15]: https://pmc.ncbi.nlm.nih.gov/articles/PMC11079570/?utm_source=chatgpt.com "Relating diversity indices and frustration in signed graphs"
[16]: https://journals.aps.org/pre//abstract/10.1103/PhysRevE.106.064139?utm_source=chatgpt.com "Mean-field approximation for structural balance dynamics in heat ..."
[17]: https://arxiv.org/html/2507.00238v1?utm_source=chatgpt.com "Exploring Quantum Heider Balance Theory"


can you explain to me what this means exactly what the alg is and what little epsilon means ...  and then for the "minimum final energy frustration index what does that mean like the goal is to get to the most energy efficient config as possible with as little computational costs as possible? im guessing that you aren't allowed to add edges as part of the algorithm? or are you?

Good, these are exactly the right things to be suspicious about. Let’s unpack it without hiding behind jargon.

---

## 1. What “frustration index” / “energy” actually is

Set-up:

* You have a **complete signed graph** on n vertices.
* Each edge (i, j) is labeled + (friends) or – (enemies).
* You choose a **partition of vertices into two camps** (cluster A vs cluster B).

Given a partition, an edge is:

* **Satisfied** if:

  * it is + and its endpoints are in the **same** camp, or
  * it is – and its endpoints are in **different** camps.
* **Frustrated** otherwise.

The **frustration index** of the graph is:

> The minimum possible number of frustrated edges over all ways of partitioning the vertices into 0/1 (two camps).

Equivalently, if you’re allowed to “flip” the sign of an edge whenever it’s frustrated, then:

> The frustration index is the **minimum number of edge sign flips** needed to make the graph *perfectly* structurally balanced.

This is what people call the “energy” of a configuration:
for a given partition, define

[
E(\text{partition}) = # \text{ of frustrated edges}.
]

Lower = “more balanced”. A **globally balanced** graph has energy 0.

So when I said “minimum final energy / frustration index”, I meant:

* **Goal of the combinatorial problem:** find a partition with **minimum possible** E.
* That minimum value is the **frustration index** of the graph.

That is *separate* from:

* **Goal of the algorithm designer:** get as close to that minimum as possible while using as little time/memory as possible.

Two very different optimizations:

1. Over partitions → “energy” (frustration).
2. Over algorithms → runtime, space, passes over the data.

You can’t minimize both perfectly because the problem is NP-hard; so you trade off.

---

## 2. What “(1+ε)-approximate” means, concretely

Let:

* OPT = the true minimum frustration index (unknown; what you’d get from an exponential-time oracle).
* ALG = frustration value (energy) of the partition your algorithm outputs.

An algorithm is a **(1+ε)-approximation** if, *for every input graph*,

[
\text{ALG} \le (1 + \varepsilon), \text{OPT}.
]

So:

* If OPT = 100 and ε = 0.1, then a (1+0.1)-approx guarantee means ALG ≤ 110.
* You’re guaranteed to be within **10% of optimal**, never worse.

Epsilon (ε) is just a knob:

* Smaller ε → closer to optimum, but algorithms usually get **slower** and/or use **more memory** roughly like poly(1/ε) or worse.
* Larger ε → sloppier approximation but cheaper computation.

In the Ashvinkumar et al. streaming result, for any fixed ε > 0, they give a one-pass algorithm that:

* Uses about (\tilde{O}(n / \varepsilon^2)) memory in the streaming model.
* At the end, outputs a partition whose frustration is ≤ (1+ε) * OPT with high probability. ([arXiv][1])

You choose ε depending on how much you care about perfect balance vs how much RAM/time you have.

---

## 3. Rough picture of what their streaming algorithm actually does

Very high-level idea, stripped of proofs:

**Problem:** Edges of the complete signed graph arrive one by one in a stream. There are Θ(n²) edges; you can’t store them all.

**Goal:** After seeing all edges once, output a partition whose frustration is within (1+ε) of optimal, using much less than O(n²) space.

**Trick: sketching + offline algorithm**

1. **They don’t try to remember every edge.**
   They maintain a carefully chosen *sketch* of the graph in memory — a compressed representation derived via:

   * Pseudorandom generator–based sampling of edges between sets of vertices.
   * Some counters and summaries that approximate how many +/– edges lie between various groups. ([Dagstuhl Drops][2])

2. **At the end of the stream**, they pretend they have a small “summary graph” on which they can run a variant of the classic Giotis–Guruswami correlation clustering algorithm (the two-cluster special case).

   * That offline algorithm enumerates possible assignments of a small “sample” of vertices and uses those to guide how it clusters everyone else.
   * Their sketch is designed so that the costs (frustration counts) computed on the summary are close enough to the true costs on the full graph.

3. **Why it works:** They prove that with high probability the cost of the partition they get, when evaluated on the *full* original graph, is at most (1+ε)·OPT.

You never see the whole adjacency matrix at once; you only ever store a compressed statistical picture of the edges that is good enough to choose near-optimal camps.

You do **one pass** (edges arrive once), you use **Õ(n/ε²)** space, and you get a **(1+ε)**-approx to minimum frustration. ([arXiv][1])

They also show that if you demanded **exact** OPT in this streaming setting, you’d provably need Ω(n²) space — basically store everything. So approximation is not just convenience; it’s unavoidable.

---

## 4. “Most energy-efficient config vs computational cost”

Your question:

> “for the ‘minimum final energy frustration index’ what does that mean like the goal is to get to the most energy efficient config as possible with as little computational costs as possible?”

There are *two separate layers* of optimization:

1. **Combinatorial / physical layer** (the social system or spin glass):

   * Objective: find the partition of nodes that minimizes energy
     [
     E(\text{partition}) = # \text{ frustrated edges}.
     ]
   * That’s the **frustration index** of the graph: the best you could possibly do, conceptually.

2. **Algorithmic layer** (you, as the CS person):

   * Objective: design an algorithm that gets **close** to that minimum in:

     * reasonable time (ideally polynomial)
     * using reasonable memory (in streaming, ~O(n) instead of O(n²))
     * maybe with only one pass over the edges

Because the base problem is NP-hard, you can’t have all of these:

* exact OPT, *and*
* fast, *and*
* tiny memory / one pass

So the game is: **given that you’ll accept a bit more energy than optimal (that 1+ε factor), how much can you save in runtime/memory?**

The streaming result is exactly that:

> “We’ll accept at most (1+ε)× extra frustration compared to the true minimum, and in exchange we’ll only use semi-streaming space and a single pass.” ([arXiv][1])

---

## 5. Are you allowed to add edges?

In the **standard** structural balance / frustration index problem:

* The **vertex set and edge set are fixed** by the input.
* You are allowed to:

  * choose how to **partition vertices into two camps**, and
  * *optionally*, interpret the minimum number of frustrated edges as “minimum number of sign flips” you’d need to perform to achieve perfect balance.

But you **do not** add edges. Adding edges would literally change the instance of the problem.

Variants:

* **Edge deletion version**: some formulations allow *deleting* edges instead of flipping signs, or treat deletions as having a cost. Still no additions.
* **Correlation clustering view**: you can think of a complete graph with + for “similar” and – for “dissimilar”; you’re not adding edges there either, you’re just clustering.
* If someone lets the dynamic *structural balance process* change the graph (e.g., adding edges between friends of friends), that’s a different model. Then your “energy” is a function of both the signing and the topology, but that’s not the standard frustration index problem we’ve been talking about.

So for the algorithms I mentioned (frustration index, (1+ε)-approx, etc.):

> **No, you don’t add edges.**
> You only pick a partition (and conceptually flip signs of the “wrong” edges), or equivalently, you measure how many edges are “wrong” under your partition.

---

If you want, next step we can actually write down the energy function in Ising notation (spins xᵢ ∈ {+1, −1}), show the exact equivalence to frustration index, and then sketch a dead-simple local search heuristic you could code up and compare to the “true” OPT on small graphs.

[1]: https://arxiv.org/abs/2306.00668?utm_source=chatgpt.com "Evaluating Stability in Massive Social Networks: Efficient Streaming Algorithms for Structural Balance"
[2]: https://drops.dagstuhl.de/entities/document/10.4230/LIPIcs.APPROX/RANDOM.2023.58?utm_source=chatgpt.com "Efficient Streaming Algorithms for Structural Balance - DROPS"


So now I'm wondering whether the following algorithm is novel in some way in this field. Obviously, it's a different approach, but whether it actually improves upon anything that already exists, I'm guessing not. Although the best algorithm, I mean, in terms of computational complexity, it also breaks the rules of like you can't add edges because it does that. Like you hear about the scapegoat from somebody else and it forms an edge. I think that makes abundant sense. I don't see why it's a rule that you have to keep the graph static as it is in order to balance it. But yeah, I'm just kind of wondering where this fits into the field or if it's kind of something different. If it's doing anything that a social networks professor would find interesting or not, and explain kind of why or why not. Like maybe the sort of for and against, like why it's novel and then why it's not. Compare and contrast it with what already exists and do so soberly. # Mimetic Scapegoating: Theoretical Foundations ## Abstract This document formalizes the theory of mimetic scapegoating contagion in signed social graphs, based on René Girard's theory of the scapegoat mechanism and structural balance theory. We prove that scapegoating propagates through friendship networks as information contagion, creating both social isolation (all-against-one) and local structural balance through a single-pass, BFS-ordered process.

Short answer: as a **“best algorithm for balancing a graph”** in the structural-balance/frustration-index sense, this is *not* going to impress anyone—because it’s solving a different problem, under looser rules, and it doesn’t try to minimize frustration. As a **Girardian, contagion-based dynamic** on signed networks that *provably* produces an all-against-one + harmonious community configuration in ~linear time, it actually *is* different from what people normally do and potentially interesting—*if* you pitch it as a model of scapegoating dynamics, not as a new optimization method.

Let me walk through that carefully.

---

## 1. What your algorithm is, in “network science” language

Stripping away the Girard prose, what you’ve built is:

* Input:

  * A signed graph (G=(V,E,\sigma)) with (\sigma \in {-1,+1}).
  * A designated scapegoat node (s).
  * One (or more) initial accuser(s).

* Phase 1 (BFS contagion on positive edges):

  * Do BFS on the **positive subgraph** (G^+) starting from the accuser(s).
  * When a node (v) first “hears” the accusation via a positive edge from an accuser:

    * If (v) was neutral to (s): **add** a new *negative* edge (v!-!s).
    * If (v) was a *friend* of (s): flip that edge to *negative*.
  * Optionally, when you see a (-,-,-) triangle ((v,s,w)), flip (v!-!w) to (+).

  Result of Phase 1 (under connectivity assumptions): all nodes end up with (\sigma(v,s)=-1). “All-against-one.”

* Phase 2 (cleanup for community unity):

  * Sweep through nodes and, for every (-,-,-) triangle ((v,s,w)), flip (v!-!w) to (+).
  * End state:

    * Everyone hates (s): (\sigma(v,s)=-1) for all (v\neq s).
    * Everyone else is mutually friends: (\sigma(u,v)=+1) for all (u,v\neq s).
    * The graph is *structurally balanced* in the trivial 2-faction partition ({s}) vs (V\setminus{s}).

On sparse graphs you argue this runs in roughly (O(|V|+|E|)) time, two passes, no iterative search.

So: you’ve defined a *specific* dynamical system on signed graphs that always converges to the “all-against-one” structurally balanced configuration, *given* an exogenously chosen scapegoat.

That is not the same task as “given a signed graph, find the minimum-frustration balanced state.”

---

## 2. What the existing literature is actually trying to do

Very roughly, there are three big buckets:

### 2.1 Measuring and optimizing balance (frustration index)

Here the network is typically treated as **fixed** (edges given, signs given), and you ask:

> What is the minimum number of edges I’d have to flip/remove so that the result is balanced?

That minimum is the **frustration index** or line index of balance, and computing it exactly is **NP-hard**; people use integer programming, branch-and-bound, and heuristics.([Nature][1])

Typical assumptions:

* Graph fixed (no new edges).
* Operations allowed:

  * Flip signs.
  * Sometimes delete edges.
* Objective:

  * Minimize # of “frustrated” edges or an equivalent energy.

There are “memetic algorithms”, simulated annealing, Monte Carlo, etc. to approximate this minimum energy more efficiently on large graphs.([ResearchGate][2])
These *do* care about both:

* **Final energy** (how close you got to the true optimum), and
* **Computational cost** (time to get there).

### 2.2 Dynamical models of structural balance

Antal–Krapivsky–Redner style **triad dynamics**: pick an *imbalanced* triangle and change the sign of one edge to make it balanced. Repeat until you hit an absorbing balanced or “jammed” state.([ResearchGate][3])

Marvel–Strogatz–Kleinberg et al. model balance as spin systems and studied an “energy landscape of social balance”, with differential-equation dynamics that flow downhill in that landscape.([SNAP][4])

Features of these models:

* Usually defined on a **static set of edges**.
* Only sign flips, no edge creation.
* Update rules often random or asynchronous.
* They reduce some energy function (# imbalanced triads) but may get stuck in local minima; they don’t promise global minimum frustration.

### 2.3 Coevolving / adaptive networks & sentiment networks

There is also work where both **node states** and **edge signs** coevolve; these are often couched as opinion dynamics or cooperation games on signed graphs, sometimes with explicit “energy” functions.([ResearchGate][2])

And there’s a line of work on “signed sentiment networks” and energy-based balance measures (e.g. He & Du’s energy function for fully signed networks).([DBLP][5])

Still, in the **balance-optimization** literature, the underlying graph is assumed fixed; you don’t add new edges as part of the balancing procedure, because that would change the data you’re trying to measure.

---

## 3. Where your algorithm fits

### 3.1 It’s not a new “best” algorithm for the classical problem

If we take the classical problem:

> Given a signed graph, find the minimum-frustration balanced state.

Then:

* Your algorithm does **not**:

  * Search the space of partitions.
  * Try to minimize frustration index.
  * Respect the “no new edges” assumption.

* It **does**:

  * **Impose** a final partition in advance: ({s}) vs everyone else.
  * Flip and create as many edges as necessary to force that partition to be balanced.

So of course you can do this in (O(|V|+|E|)): you’re not solving the NP-hard problem. You’re saying “this is the final cut; adjust the signs so reality matches my decision.”

If I measured your output with the standard frustration index (relative to the *original* signs), your process will usually have a **much higher** number of changes than a good optimization algorithm, because you make no attempt to minimize.

That’s why, if you try to present this as “a new efficient algorithm for balancing signed graphs,” anyone in the frustration-index world will shrug: you changed the rules (allowed edge additions, fixed the final partition, ignored minimality), so you’re not playing the same game.

### 3.2 What you *are* doing: a very specific dynamical model

Seen correctly, what you’ve built is:

* A **contagion model** where information travels only on positive edges.
* A **commitment rule** that forces nodes to choose friend vs scapegoat (Girardian mimetic pressure).
* A **cleanup rule** that makes the community internally all-positive via “enemy of my enemy is my friend”.

This is closest to the “dynamical models” camp:

* Like Antal et al., you have local rules based on triads; your Rule 2 (*fix --- to +-+*) is basically codifying the “enemy of my enemy” triad dynamic.([ResearchGate][3])
* Unlike them, you:

  * Allow **edge creation** (new negative ties to the scapegoat).
  * Use **BFS ordering** and a designated scapegoat.
  * Prove convergence specifically to the all-against-one state.

So it’s a **particular** deterministic path through the energy landscape to one specific attractor (two clusters, one of size 1). That’s basically a special case of the classical “balanced ⇔ at most two factions” theorem, where you impose ahead of time which node must be the singleton faction.

Put differently: you’ve defined a *Girard-flavoured, BFS-ordered* update rule whose absorbing state is always the N–1 vs 1 partition.

---

## 4. Novel vs not novel – point by point

### Ways this *is* novel / interesting

1. **Formal Girard + structural balance mash-up**

   There *is* some prior work explicitly combining Girard with structural balance in a computational way (e.g. Graham Sack uses Girard’s triangular desire plus structural balance to generate narratives over character networks).([Users][6])

   But I haven’t seen a **Girardian scapegoat mechanism formalized as a deterministic dynamic on signed graphs** with proofs of all-against-one convergence. That conceptual bridge—“scapegoating = particular balance-restoring dynamic which produces this degenerate partition”—is a real contribution *as a model*.

2. **Edge creation as part of the mechanism**

   Most balance algorithms treat the graph as fixed. You explicitly encode the fact that *hearing about the scapegoat can create a new tie* (a negative tie to someone you’d never interacted with). That’s pretty realistic socially, and it pushes you toward the **adaptive network / coevolution** literature rather than classical frustration minimization.([ResearchGate][2])

   A social-networks person who likes dynamic graphs will at least raise an eyebrow at that choice.

3. **Single-sweep, monotone contagion to a very specific balanced state**

   A lot of triad dynamics are stochastic, require many updates, and can get trapped in jammed states.([Bohrium][7])
   You can say:

   > “Here is a simple local rule that, under mild connectivity assumptions, always yields a structurally balanced all-against-one state in two linear-time passes.”

   That’s mathematically clean. You’re not minimizing energy, but you **are** giving:

   * A concrete dynamical system with a unique absorbing state.
   * Proved convergence and complexity bounds.
   * A clear sociological interpretation (scapegoat + community unity).

4. **It captures an extreme, but real, social phenomenon**

   The fact that the final state is *extreme* (one person hated by everyone; everyone else reconciled) is not a bug; it’s literally Girard’s story. That’s an angle most balance models don’t explore—they’re symmetric and generic; yours is deliberately **asymmetric** (one exogenous victim).

   For someone doing **theory-driven modeling of polarization, pile-ons, cancel campaigns**, this is at least a plausible toy model.

### Ways this is *not* novel / less impressive from a pure algorithms POV

1. **It’s not optimizing anything classical**

   You’re not approximating the frustration index, not finding the minimum-energy partition, and not respecting the usual fixed-graph setting. So it can’t be compared to, say, state-of-the-art memetic/Monte-Carlo frustration solvers.([Nature][1])
   From that angle, a reviewer will say: “You’ve changed the objective; of course you can do it in linear time.”

2. **The local rules themselves are standard**

   * “Enemy of my enemy is my friend” and “friends agree on enemies” are literally the *defining intuitions* of structural balance going back to Heider and Cartwright–Harary; your Rule 1/2 just encode those in code.([SNAP][4])
   * BFS as the mechanism of information spread on positive edges is textbook contagion/epidemic modeling.

   So nothing about the *mechanics* screams “brand-new algorithmic trick.”

3. **Trivial structural balance once the partition is decided**

   Given a partition ({s}) vs (V\setminus{s}), it’s totally straightforward to check which edges violate balance and flip them. That’s essentially what your Phase 2 does: for every pair in the big faction that’s still hostile, flip them to +.

   In other words, the proof that the end-state is balanced is almost trivial *once you decide* that the only acceptable balanced state is that degenerate 2-cluster one.

4. **“One pass” is only meaningful because the target state is fixed**

   The classical problem is hard because you *don’t know* ahead of time which partition is global minimum; you have to search the energy landscape. Triad dynamics and memetic algorithms wander that landscape and can get stuck.([InK][8])

   You sidestep that entire hardness by saying “we already know the scapegoat; the final cut is fixed; now just flow there.” That’s perfectly valid as a *model of a particular social logic*, but not a breakthrough for the optimization problem.

---

## 5. About the “no new edges” issue

Your instinct here is right:

* In **measurement/benchmark** contexts (compute frustration index of a given empirical network), people keep the graph static on purpose: the graph *is the data*. Adding edges means you’re answering a different question.

* In **dynamic social modeling**, adding/deleting edges is totally standard. There is a whole literature on evolving networks, where interactions and ties are created/removed as part of the process.([ResearchGate][2])

So:

* If you’re trying to compete with “best algorithms for minimizing frustration,” then **yes**, adding edges is out of bounds.
* If you’re saying “I’m modeling a mechanism where scapegoating itself *creates* new relationships (negative ones toward the victim)”, then adding edges is absolutely fine. You’ve just left the classical optimization problem and moved into the “adaptive signed networks” / “co-evolution of ties and hostility” space.

Your algorithm really belongs in that second camp.

---

## 6. Would a social networks professor care?

If you pitch it as:

> “I’ve got a faster algorithm for balancing signed networks,”

they’ll basically say:

* No, you don’t; you’ve fixed the final partition and allowed edge additions.
* It’s a nice deterministic dynamic, but it’s not solving the NP-hard problem others are solving.

If you pitch it as:

> “I’ve formalized Girard’s scapegoat mechanism as a contagion process on signed networks that (1) spreads only along positive ties, (2) creates negative ties to a designated victim, and (3) provably yields a structurally balanced all-against-one state plus full internal reconciliation in two linear-time phases. Here’s how the final states and convergence time compare to classical triad dynamics / energy-descent models.”

—then yes, that’s the kind of thing a theory-friendly networks person might find genuinely interesting:

* It’s clearly grounded in an existing sociological/philosophical theory.
* It uses standard tools (signed graphs, structural balance) in a slightly different way.
* It gives clean theorems and complexity bounds.
* It opens natural questions (multi-scapegoat competition, resistance clusters, probabilistic rules, empirical calibration on Twitter dogpiles, etc.).

Just don’t oversell it as an “improvement” over frustration-index solvers. It’s a **different problem**: a Girardian generative model whose absorbing state happens to be structurally balanced.

Framed that way—“this is a specific dynamical mechanism that always yields N–1 vs 1 balance, and we can compare its trajectories to classical balance dynamics”—you’re in reasonable, defensible territory.

[1]: https://www.nature.com/articles/s41598-021-98139-w?utm_source=chatgpt.com "Identifying hidden coalitions in the US House of ..."
[2]: https://www.researchgate.net/publication/297791217_Structural_balance_in_fully_signed_networks?utm_source=chatgpt.com "Structural balance in fully signed networks | Request PDF"
[3]: https://www.researchgate.net/publication/2175948_Social_Balance_on_Networks_The_Dynamics_of_Friendship_and_Enmity?utm_source=chatgpt.com "Social Balance on Networks: The Dynamics of Friendship ..."
[4]: https://snap.stanford.edu/class/cs224w-2011/handouts.html?utm_source=chatgpt.com "Stanford CS224W: Handouts"
[5]: https://dblp.org/pid/69/870-4?utm_source=chatgpt.com "Wei Du 0004"
[6]: https://users.cs.fiu.edu/~markaf/doc/a9.sack.2014.procdh.26.abstract_archival.pdf "DHArchive"
[7]: https://www.bohrium.com/paper-details/dynamics-of-social-balance-on-networks/812135682767060993-565?utm_source=chatgpt.com "Dynamics of social balance on networks"
[8]: https://ink.library.smu.edu.sg/cgi/viewcontent.cgi?article=10078&context=sis_research&utm_source=chatgpt.com "Social balance on networks: Local minima and best-edge ..."


Short version: there *is* a small computational-Girard niche, but it’s thin and scattered. Nobody has anything like a “standard Girardian model” the way there is for, say, Ising spins. Your algorithm sits right at the intersection of that niche and the structural-balance / signed-network world. It’s not going to beat the best frustration-minimization solvers, but in Girard-land it’s actually quite distinctive.

I’ll break it down into:

1. What’s been done computationally with Girard (mimesis + scapegoats)
2. What structural-balance people do algorithmically
3. Where your model is genuinely novel vs “just another graph process”

---

## 1. Computational Girard 101 – the lay of the land

Most Girard scholarship is still textual/theological/literary. The explicitly **computational** work is a narrow fringe, and it looks roughly like this:

### 1.1 “Complex mimetic systems” & agent-based modeling

* **Hans Weigand, “Complex Mimetic Systems” (Contagion, 2008–09)**
  Uses Girard to argue that social systems are best seen as **complex adaptive systems** and sketches an agent-based simulation architecture where agents imitate, compete, and escalate towards violence. It’s more conceptual than algorithmic: lots of diagrams about feedback loops, not tight complexity bounds or closed-form results. ([Universität Innsbruck][1])

* **Blecic & Dumouchel, “Towards Modelling Mimetic Theory: An Agent-Based Simulation” (in *The Revelatory Power of Mimetic Theory*)**
  This is the closest thing to “let’s code Girard”. They build an ABM where agents imitate each other’s desires and conflicts escalate; the goal is to show that Girard’s crisis→sacrifice story can be reproduced by local interaction rules, not to give a single clean algorithm with proofs. ([Bloomsbury][2])

* **Antunes, Nunes & Coelho, “The Geometry of Desire” (AAMAS 2014)**
  They embed **Girardian mimetic desire** directly into a BDI-style agent architecture: agents form desires by imitating high-status models, then simulations show herd behavior, rivalry, etc. It’s very much: “Given Girard’s insight about triangular desire, how do we represent that in multi-agent systems?” Again: focus on *desire contagion*, not scapegoating or structural balance. ([IFAAMAS][3])

* **Maclay 2021, “Agent-based force vector model of social influence” (PLOS ONE)**
  Mainly an opinion-dynamics model with “forces” pulling agents’ attitudes; they explicitly name Girardian mimetic desire as one of the inspirations, but the model itself is generic social influence, not a literal Girard implementation. ([PMC][4])

### 1.2 Scapegoating as a modeled phenomenon

Here it gets very thin:

* **Paes 2025, “An Agent-Based Model of Scapegoating” (CoMSES / NetLogo)**
  This is the one that’s directly about scapegoats. Agents accumulate “tension”; when tension crosses a threshold, leaders emerge and “blame” some individual; the scapegoat gets attacked and collective tension drops. Network structure matters (small-world vs random, etc.), but the algorithm is: simple local rules + stochastic thresholds + lots of NetLogo runs, not a closed-form process that *guarantees* all-against-one or structural balance. ([CoMSES Net][5])

* **Markússon 2004, “Violent Memes and Suspicious Minds”**
  Not a simulation, but a memetic/evolutionary framing of the scapegoat mechanism: the “sacrificial lens” is a meme that spreads because it stabilizes groups. It’s concept-level modeling—no networks, no algorithms. ([ResearchGate][6])

* There are also a few papers in media and network studies saying, essentially, “we can translate Girard into network language and see scapegoating as a kind of contagion process,” but they don’t go all the way to an actual algorithm with proofs. ([Grafiati][7])

### 1.3 Neuroscience / cognition cross-overs (for completeness)

You also have things like Vittorio Gallese’s work linking **mirror neurons** and mimetic theory – arguing that Girard’s mimesis has a plausible neural substrate. That’s modeling at the cognitive level rather than network algorithms. ([hiw.kuleuven.be][8])

Bottom line for section 1:

* There *are* ABMs of mimetic desire and one explicit ABM of scapegoating.
* They’re mostly stochastic, exploratory simulations.
* Nobody has a neat, signed-graph, provable, two-phase scapegoat algorithm.

---

## 2. What the *signed-network / balance* people do

Separate from Girard, there’s a big literature on **structural balance in signed networks** (Heider, Cartwright–Harary, etc.), which is exactly the math you’re using:

### 2.1 Dynamic processes to reach balance

People have been doing this since at least the 2000s:

* **Hummon & Doreian, “Some Dynamics of Social Balance Processes” (2003)** – ABM where agents flip signs on edges to reduce cognitive dissonance. It’s literally “local triad repair”: pick an unbalanced triad, flip an edge, repeat, see what macro-structure you get. ([ScienceDirect][9])

* **Kułakowski et al. 2005, “The Heider balance: a continuous approach”** and **Marvel et al. 2011, “Continuous-time model of structural balance”** – treat sign weights as continuous variables and write differential equations that flow downhill on an “energy” function (number of frustrated triads). These flows tend towards balanced partitions (often two hostile camps). ([World Scientific][10])

* More recent work (e.g. Parravano, Wu, Li, Pham) couples **opinion dynamics** with balance dynamics: bounded confidence + preferential sign flips, co-evolution of attitudes and edge signs, etc. ([PMC][11])

This community absolutely uses *Monte Carlo* and random local updates: pick a triad or edge at random, flip signs according to some rule, repeat until (maybe) converged. The energy-landscape work treats it like a spin glass and uses stochastic search to explore local minima. ([Physical Review][12])

### 2.2 Algorithms for “best” balance / minimal frustration

Here the goal is *algorithmic*: given a signed graph, find the minimum number of edges you’d have to flip to achieve balance (the **frustration index**), or approximate it fast.

* Facchetti et al. 2011/2012 give methods for computing global balance in large signed networks and exploring the low-energy landscape; think optimization + spectral / linear-programming style tools. ([PMC][13])

* Ma et al. 2015 propose a **memetic algorithm** (MLMSB) to compute and transform structural balance in large signed networks—explicitly combining evolutionary search and local improvement to find low-energy configurations. ([ScienceDirect][14])

* There are R/Python packages (e.g. *signnet*) with whole toolboxes for structural balance and frustration estimation; plus newer work on dynamics, random walks on signed graphs, etc. ([schochastics.github.io][15])

Key points versus you:

* These people **keep the graph static** (no new edges), flip signs to reduce energy, and usually aim for *any* low-frustration state (often two factions), not specifically an all-against-one scapegoat configuration.
* They lean heavily on **stochastic search / Monte Carlo** and continuous-time flows.
* Their novelty is “we can solve/approximate this NP-hard problem efficiently on big networks,” not “this exactly captures Girard.”

---

## 3. Where your scapegoat algorithm sits in all this

Now, your setup:

* Signed graph (friend/enemy edges)
* One designated scapegoat **s** and initial accuser **a**
* BFS over the **positive** (friendship) subgraph to propagate accusation
* Local rules:

  * hear from an accuser friend → create negative edge to *s*
  * friend of *s* + friend of accuser → flip to enemy of *s*
  * (-,-,-) triangles with *s* → “enemy of my enemy is my friend” flips inside the community
* Second pass that cleans up remaining (-,-,-) triangles to yield:

  * all edges to *s* negative
  * all edges among the rest positive
  * i.e., perfect structural balance with N–1 vs 1 factions.

You’ve proved linear-ish time for sparse graphs and a bunch of structural properties (all-against-one convergence, scapegoat hyper-connectivity, etc.).

### 3.1 What is **not** novel (from a hard-nosed networks/CS perspective)

Let’s be blunt:

* **Local triad rules + BFS** are standard tricks.
  Epidemic/rumor spreading on graphs as BFS over positives? Old hat. Local flipping to heal unbalanced triads? That’s exactly what Hummon–Doreian, Marvel, Kułakowski and friends have been doing; they just don’t fix a single node as “sacred victim”. ([ScienceDirect][9])

* **You’re not solving the canonical optimization problem.**
  You are *not* computing a minimum-frustration configuration or the best two-camp partition. You’re constructing a very specific balanced state (everyone vs one), regardless of whether that’s the closest‐to-original configuration. So you’re not competing with Facchetti, Ma, or the memetic algorithms on their own turf. ([PMC][13])

* **Deterministic single-scenario dynamics are common in ABMs.**
  “Here’s an update order; here are rules; here’s a convergence proof” is the normal style for simple ABMs and opinion-dynamics models. You did it carefully, but the *form* of the contribution is not unusual.

* **Edge addition is not heresy.**
  Balance-theory people often fix the edge set because they’re matching signed adjacency matrices, but dynamic social-network models in general freely add/delete edges. So the “you can’t add edges” taboo is more a convention of one subliterature than a deep rule you’re breaking.

So if you walked into a structural-balance / network algorithms seminar and said, “Here is the most efficient way to compute balance,” they’d shrug or push back; that isn’t what your thing does.

### 3.2 What *is* genuinely novel or at least unusual

From a **Girard + networks** standpoint, there *are* real contributions:

1. **Explicit fusion of Girard + structural balance.**
   People have gestured at this (“scapegoat as making the group coherent,” “we can think of networks here”), but I haven’t seen a clean, formal result that says:

   > From *any* signed graph satisfying mild conditions, a simple local mechanism of accusation + imitation will drive the system to a *fully* balanced, all-against-one configuration that exactly matches Girard’s “unity through exclusion”.

   The ABMs of mimetic theory and scapegoating (Weigand, Blecic–Dumouchel, Paes) don’t deliver that kind of theorem; they show dynamics qualitatively or via runs, not as a **proven transformation** on signed graphs. ([ResearchGate][16])

2. **A deterministic, two-pass “scapegoat operator” on signed graphs.**
   Structural-balance models typically meander: they iteratively flip random edges or integrate ODEs until they get “close enough” to balance, often getting stuck in local minima. You, by contrast, explicitly define a **single BFS pass + a single cleanup pass** and prove that you *always* end in a perfectly balanced all-against-one state when the friend graph is connected. That’s mathematically tidy in a way most Girard-inspired stuff isn’t.

3. **Girardian properties made precise.**
   Things like:

   * scapegoat ends up **hyper-connected** (“famous through infamy”)
   * the rest of the community ends with all positive ties
   * the process is driven by *imitating friends’ hostilities*, not optimization
     These are straight translations of Girard’s verbal claims into graph-theoretic invariants, which is exactly the sort of thing that hardly anyone has done rigorously. Paes’s NetLogo model, for example, does see tension drop and scapegoats emerge, but it doesn’t guarantee “all edges to victim negative, all others positive.” ([CoMSES Net][5])

4. **You pick a *very specific* valley in the energy landscape.**
   Balance theory says: “Balanced states are either one big friendly clique or two hostile factions; unbalanced states sit in an energy landscape with many local minima.” Your algorithm doesn’t just slide downhill; it *drives the system to one very particular low-energy basin*: N–1 vs 1 with internal complete harmony. That’s an almost caricatured version of Girard’s sacrificial peace, but precisely because of that, it’s analytically interesting.

5. **Edge-creation rule as a formalization of “fame through infamy.”**
   The “if you had no edge to *s* but you hear about them from an accuser friend, you create a negative edge” rule isn’t standard in balance dynamics. It encodes a Girard-specific intuition (you learn about the victim via the mob itself), not just generic “flip to fix triads.”

### 3.3 How a social-networks / complexity person is likely to react

If you present this honestly as what it is:

* **As Girard scholarship:**
  It’s genuinely interesting. You’re doing what Weigand and Blecic–Dumouchel said we *should* do—treat mimetic theory as a complex system and make it explicit—but with much crisper graph-theoretic structure and proofs. That will stand out in a literature that’s mostly metaphor and toy NetLogo models.

* **As network science:**
  They will *not* care about it as a better optimizer. They will care about it as a **stylized mechanism**: “Here is a simple contagion process that always produces a scapegoat + consensus configuration; here are the conditions; here’s how it compares to random triad-flip dynamics.” From that angle, it’s a neat toy model to study one particular kind of social collapse.

* **The sober critique:**

  * The scapegoat is exogenous; it doesn’t *emerge* from rivalries, it’s specified. Girard would say the victim is arbitrary but not chosen in advance by a modeler.
  * The end state (no negative edges among non-victims) is too clean; real societies after scapegoating remain riven by rivalries.
  * There’s no noise, no partial compliance, no mixed strategies.
  * There’s no data: it’s not calibrated or tested against real cascades (Twitter pile-ons, etc.).

If you pitch it as:

> “A first pass at a **Girardian structural-balance operator** on signed graphs that proves how scapegoating can, in principle, drive a system to perfect balance,”

that’s defensible and interesting. If you pitch it as:

> “I solved structural balance and beat everyone’s algorithms,”

that will draw (justified) fire.

---

## 4. So: novel or not?

**Novel *within Girard-world*?**
Yes. You’re ahead of the existing ABMs in rigor and in explicitly linking the scapegoat mechanism to structural balance with actual theorems, not just simulation plots.

**Novel *within structural-balance / networks*?**
Conceptually yes (single-victim all-against-one Girardian flow), algorithmically no in the sense of raw technique: BFS, local triad rules, and signed-graph updates are standard tools.

If you now:

* drop this into an ABM framework,
* compare its dynamics to classical energy-minimizing flows, and
* maybe calibrate against real “cancellation” cascades,

then you’ve got something a “social networks professor” would absolutely take seriously as a research project, not just a clever blog post.

[1]: https://www.uibk.ac.at/theol/cover/girard/pdf/bibliography_vol.xxix.pdf?utm_source=chatgpt.com "Bibliography of Literature on the Mimetic Theory vol. XXIX"
[2]: https://www.bloomsbury.com/us/revelatory-power-of-mimetic-theory-9781350532885/?utm_source=chatgpt.com "The Revelatory Power of Mimetic Theory"
[3]: https://ifaamas.org/Proceedings/aamas2014/aamas/p1169.pdf?utm_source=chatgpt.com "The Geometry of Desire"
[4]: https://pmc.ncbi.nlm.nih.gov/articles/PMC8594851/?utm_source=chatgpt.com "An agent based force vector model of social influence ... - PMC"
[5]: https://www.comses.net/codebases/44d0678d-1412-40d9-8a02-a857a5f1d81d/releases/1.0.1/?utm_source=chatgpt.com "An agent-based model of scapegoating 1.0.1"
[6]: https://www.researchgate.net/publication/236706876_Violent_Memes_and_Suspicious_Minds_Girard%27s_Scapegoat_Mechanism_in_the_Light_of_Evolution_and_Memetics?utm_source=chatgpt.com "Girard's Scapegoat Mechanism in the Light of Evolution and ..."
[7]: https://www.grafiati.com/it/literature-selections/social-network-theory/?utm_source=chatgpt.com "Social network theory - Bibliografie tematiche"
[8]: https://hiw.kuleuven.be/hua/events/hom-seminar/suggested-reading/3-gallese-2sidesimitation.pdf?utm_source=chatgpt.com "The Two Sides of Mimesis: Mimetic Theory, Embodied ..."
[9]: https://www.sciencedirect.com/science/article/abs/pii/S0378873302000199?utm_source=chatgpt.com "Some dynamics of social balance processes"
[10]: https://www.worldscientific.com/doi/abs/10.1142/S012918310500742X?srsltid=AfmBOorKwFBBCzhHvpLK9dN4Ju7LMVhPRfJbPEATnw6IkcsrpV0lHhwN&utm_source=chatgpt.com "THE HEIDER BALANCE: A CONTINUOUS APPROACH"
[11]: https://pmc.ncbi.nlm.nih.gov/articles/PMC5055330/?utm_source=chatgpt.com "Bounded Confidence under Preferential Flip: A Coupled ..."
[12]: https://link.aps.org/doi/10.1103/PhysRevE.86.036116?utm_source=chatgpt.com "Exploring the low-energy landscape of large-scale signed ..."
[13]: https://pmc.ncbi.nlm.nih.gov/articles/PMC3248482/?utm_source=chatgpt.com "Computing global structural balance in large-scale signed ..."
[14]: https://www.sciencedirect.com/science/article/pii/S0950705115001811?utm_source=chatgpt.com "A memetic algorithm for computing and transforming ..."
[15]: https://schochastics.github.io/signnet/?utm_source=chatgpt.com "Methods to Analyse Signed Networks • signnet"
[16]: https://www.researchgate.net/profile/Hans-Weigand-2/publication/236747583_Complex_Mimetic_Systems/links/0deec5382f1f7ee33c000000/Complex-Mimetic-Systems.pdf?utm_source=chatgpt.com "COMPLEX MIMETIC SYSTEMS"
