# Title
# Subtitle

In our previous post we discussed the concept of balance theory and how people tend to take sides in relation to a polarizing figure like Trump, the result being the sort of partisanship we see today. In that post I showed this diagram from E = 9 to the end state of E = 0.

![Comparison of Energy States](../balance-theory/case_studies/6person_polarization/outputs/comparison.png)

Now getting this result computationally, for the size of graph that it is was surprisingly difficult. The normal way to solve a problem like this is to use a greedy algorithm, that is an algorithm that makes the most beneficial move at each step. But at the initial state there are no beneficial moves, and the only path to get to the E = 0 state is to make things worse first.

This is something that has been encountered before, the solution is called hill-climbing, where the algorithm makes a random move (or in this case more than one) in order to reach a state where it's possible it could descend at last to a lower energy state.

![energy trajectory](../balance-theory/case_studies/6person_polarization/outputs/energy_trajectory.png)

Here we see the energy trajectory of one of the successful runs, but that masks the fact that there were MANY more runs that failed utterly and succeded only in making things worse. In this case the success rate was around 1 in 3. If we were to scale this up to a larger graph, that success rate would plummet and the computational time required to reach a solution would explode.

The [algorithm](https://github.com/ajax-sheep/balance-theory/blob/main/balance_theory/solvers/local.py) (which uses a monte carlo approach) I began to find both ineffective and theoretically unsatisfying. We humans do not make our choice of relations rationally. We do not calculate the energy of our social networks and then make decisions of who we will and will not be friends with in an effort to lower the temperature of our entire network. We are no more network optimizers than we are homo economicus. So, I went back to the drawing board and back to Girard.

# The Scapegoat Effect

Girard's cultural theory can be broken down into five steps:

1. Mimetic desire We've covered this already. Humans imitate each other's desires, leading to rivalry.
2. Mimetic crisis: Undifferentiated rivalries create social instability, what we might call the peak of the hill-climbing algorithm.
3. Scapegoat mechanism: Groups resolve crisis by uniting against a single victim. (this is what we will cover now.)
4. Sacred violence: The collective murder creates temporary peace and social order (stay tuned). 
5. Return to step 1. 

Girard explain the scapegoat mechanism below better than I ever could:

> By a scapegoat effect I mean that strange process through which two or more people are reconciled at the expense of a third party who appears guilty or responsible for whatever ails, disturbs, or frightens the scapegoaters. They feel relieved of their tensions and they coalesce into a more harmonious group. They now have a single purpose, which is to prevent the scapegoat from harming them, by expelling and destroying him. ...  Mimetic attraction is bound to increase with the
number of those who converge on one and the same antagonist. Sooner or later a snowball effect must occur that involves the entire group minus, of course, the one individual, or the few against whom all hostility focuses and who become the "scapegoats," in a sense analogous to but more extreme than our everyday sense of the word "scapegoat." Whereas mimetic appropriation is inevitably divisive, causing the contestants to fight over an object they cannot all appropriate together, mimetic antagonism is ultimately unitive, or rather reunitive since it provides the antagonists with an object they can really share, in the sense that they can all rush against that victim in order to destroy it or drive it away. (THE GIRARD READER, p. 11-12)

So, thought I, how might we represent this mechanism graphically? How does this scapegoating actually spread through a network? And how does it create social harmony in the aftermath? 

Girard implicitly gives all the steps neccessary to make this into a an algorithm, in his discussion, but let's make it more explicit:

We start again with our graph of nodes and +/- edges and triangular relationships. A triangle is "balanced" in the case of +++ or +--, and unbalanced in the case of +-+ or ---. 

We start with an accusation: one random node accuses another, of what it does not matter. But the sign between them flips to negative if it wasn't already. 

Then the accusor spreads the accusation to all of their friends (neighbors with positive edges), who then if they know the accused, flip the sign between them to negative, or if they don't know the accused form a negative opinion of them (creates a negative edge between them). 

As this hatred of the accused spreads through the network, eventually the scapegoat becomes infamous—everyone hates them, and is therefore in a triangular relationship with everyone else in relation to the scapegoat. So between you and anyone else you know in the community you have a negative edge between you and the scapegoat, a negative edge between the other person and the scapegoat, and then some other edge between you and the other person. So that's --?. Now we know from last time the only way to balance such a triangle is for that edge between you and the other non-accused member of the community is for that last edge to be positive.

This means through the hatred of the scapegoat the entire community becomes unified, +++ triangles across the board.

![5node_partial](../mimetic-contagion/output/basic/5node_partial.gif)

For the technically minded among you here is the more mathy version of the algorithm:

[ insert more technical description here ]

Now the power of this algrithm is actually its simplicty. It requires no global optimization, no serious calculations, no machine learning, no probability or randomness. With few exceptions (which we will dive into next) it achieves a balanced state in a single pass through the network. And it's relatively computationally efficient even for larger networks.[^1]

![20node_wave](../mimetic-contagion/output/basic/20node_wave.gif)

Now, this process does not always end in all-against-one, but this is entirely dependent on who the initial accuser and scapegoat are.

For instance in the simplest case of a chain of three nodes (A - B - C) if the initial accuser is A and the scapegoat is B, then the accusation will never reach C. 

![3node_chain](../mimetic-contagion/output/bridge/3node_chain.gif)

Similarly if you have two communities connected through a couple of bridge nodes, if you accuse one of the bridge nodes, the accusation will never reach the other community.

![comparison](../mimetic-contagion/output/bridge/comparison.png)

This also applies to hubs of networks. If you are on the periphery of a network and you accuse the hub, your accusation will never reach your fellow periperhals. Whereas, if you are the hub and you accuse one of the peripherals, your accusation will quickly reach the entire network and form a death-star-looking coalescence around the scapegoat.

![comparison](../mimetic-contagion/output/centrality/comparison.png)

This all lines up with Girard's observation that scapegoats are typically on the periphery of society: 

> I do not think we could say that
the victim is randomly chosen. After all, randomness means pure
chance. If we look at myths, we will see that the victims are too
often chosen among physically challenged people or foreigners,
to be a purely random event: these 'preferential signs' increase
the possibilities of being selected as scapegoat. It is very clear
in Isaiah, in the 'Servant of Jahweh'. People have what could be
called a natural dislike for exceptions, physical deformities, which
become signs of victimizations. In the 'Servant of Jahweh', there
is a passage that reads: 'He had no beauty or majesty to attract us
to him, nothing in his appearance that we should desire him. He
was despised and rejected by men, a man of sorrows, and familiar
with suffering. Like one from whom men hide their faces he was
despised, and we esteemed him not' (Isaiah 53.2-3). Preferential
signs of victimization are given as reasons for victimizing this
person, reasons that are insufficient, scandalous, but do not allow
us to always speak of pure randomness. Infirmities, or unpleasant
traits, are mistaken for guilt. That is the reason why in medieval
illustrations witches very often are represented a little bit like the
Jews in anti-Semitic caricatures, with distorted features, hunch-
backed, limping. If you look at the Greek gods, far from being
beautiful, they are very often like that: short, one-eyed, mutilated, 
stuttering, deformed (there is a parodical text by Lucian of
Samosata, Tragodopodagra, which is all about that).19 There are also
exceptions, which show out-of-the-ordinary beauty, like that of
Apollo or Venus, but we have to remember that both extremes
are usually more scapegoated than average people. The king is
a preferred target for victimization. After all, the institution itself
originates in scapegoating. Therefore, the king tends to go back to
his original status.20 So we shouldn't say randomness stricto sensu,
and it would be better to say arbitrariness.

*Therefore, it is a combination of arbitrariness and necessity.*

Very often, but not necessarily, because even if there isn't a prefer-
ential sign of victimization, the scapegoat will be chosen anyway.
At that crucial moment something will often be interpreted as a
sign. Anything. And everybody thinks that they have found the
solution, the culprit. In a way, the scapegoat mechanism functions
like false science, like a great discovery that is made, or something
that is suddenly revealed, and then one reads in the eyes of other
people the same insight, that, therefore, the conviction of the
crowd becomes increasingly reinforced. (EVOLUTIONand
CONVERSION
Dialogues on the Origins of Culture, p. 69)

Now we can add on top of this *why* it is that the scapegoat is often on the periphery: because if they are not, if they have defenders that are gatekeepers of the social network, then they could prevent the spread of the accusation throughout the whole community. If the accusation does not spread through the network, then the accused fails to be a scapegoat at all. 

Now the other scenario Girard touches on here is the possibility of the scapegoat being someone at the top of the social hierarchy, in this case a king, but he says the same also for those who are rich and powerful:

> In normal times the rich and powerful enjoy all sorts of protection and privileges
which the disinherited lack. We are concerned here not with normal circumstances but with
periods of crisis. A mere glance at world history will reveal that the odds of a violent death at
the hands of a frenzied crowd are statistically greater for the privileged than for any other
category. Extreme characteristics ultimately attract collective destruction at some time or
other, extremes not just of wealth or poverty, but also of success and failure, beauty and
ugliness, vice and virtue, the ability to please and to displease. The weakness of women,
children, and old people, as well as the strength of the most powerful, becomes weakness in
the face of the crowd. Crowds commonly turn on those who originally held exceptional
power over them. (Girard Reader, p. 112)

We can also now explain this phenomenon in more depth: if we think about a graph where there is one node above all others that everyone knows (a king, a billionaire, a celebrity, etc.), someone who is already famous, if that person is accused, no new edges need be created in the spreading of the contagion, only some surface level + must flip to - and suddenly this poor nexus node has gone from fame to infamy.

![comparison](../mimetic-contagion/output/effort/comparison.png)

Whether peripheral or famous, the effort of the community in scapegoating you is the same.

Now Girard also touches on who does not make a good scapegoat (and the answer may surprise you):

> It is clearly legitimate to define the difference between sacrificeable and
nonsacrificeable individuals in terms of their degree of integration, but such a definition
is not yet sufficient. In many cultures women are not considered full-fledged members
of their society; yet women are never, or rarely, selected as sacrificial victims. There
may be a simple explanation for this fact. The married woman retains her ties with her
parents' clan even after she has become in some respects the property of her husband
and his family. To kill her would be to run the risk of one of the two groups' interpreting
her sacrifice as an act of murder, committing it to a reciprocal act of revenge.

This brings us back to the bridge nodes, you can think of a woman given from one family to another as a kind of bridge between the two communities. Scapegoating her would likely not lead to unanimous agreement among the two clans but bifurcation:

![two_communities](../mimetic-contagion/output/bridge/two_communities.gif)

Such an affront though would likely not go unheeded, instead the one family would escalate in vengence against the other for the scapegoating of their member: 

![escalation](../mimetic-contagion/output/escalation/escalation_animation.gif)

The end state is two mutually antagonistic communities, although the two are mostly disconnected they are each unified against the individual member of the opposite community who has been designated as the scapegoat. 

This is not a version of the scapegoat phenomenon that I have seen Girard discuss in any depth, but it makes sense, in order for a Montigue to hate a Capulat, he need not know and hate every Capulat, he might only hate ___ who first ____. perhaps after so many genrations of infighting the scapegoat is no longer an individual but the concept of the Montigues themselves. And just by virtue of this hatred, were a Capulat to meet any Montigue on the steet, they would be enemies automatically by association.

This mechanism not only establishes a hatred of the victim but also a unity among the persecutors—taking each respective community that was previously divided and making them unified against the common enemy. 

![fragmented_escalation](../mimetic-contagion/output/escalation/fragmented_escalation_animation.gif)

Thus we've arrived at something actually quite [Schmittian](https://en.wikipedia.org/wiki/Carl_Schmitt) in nature: defining the political in terms of your enemy. 

Thus if we return back to our Trump graph, we can see it in a new light: here Trump is the scapegoat of one community (the Democrats) while being defended by the other (the Republicans). 

You see the same mechanism in foreign affairs: take the case of WWII, there is the scapegoating of a specific individual, Hitler, in propaganda and ridiculous cartoons, who stands in for the whole German nation. The allies thus are able to unify themselves against the common enemy of Hitler and the concept of the German nation and were they to meet a German on the battlefield, they are obviously auotmatic enemies to the point where they immediately start trying to kill each other.

Now what is important here is not the guilt or innocence of the scapegoat but the fact that the mechanism of coalescnce around them is the same and has the same unifying effect—it is that this is the same mechanism that unified montigue against capulat, libs against trump, allies against hitler, germans against the jews, the proletariat against the bourgeoisie, the french people against marie antoinette, the KKK against blacks, the people of Salem against witches, the *mobile vulgus* against Jesus. 

None of these are morally equivalent instances—and when we say that we mean that the appointed victim is in some cases "innocent" and in other cases "guilty"—but just looking at the list in each case the persecutors had their very thoroughly defined reasons for their persecution, and though we may look back at one or the other and say "oh they were unjust in doing that" and most of the work of history these days is to do just that—morally grandstanding from the lofty heights of modernity back on the evil past—the useful thing in analyzing these events *wie sie eigentlich gewesen sind* is to see the mechanism of scapegoating underlying all. 

Perhaps you were wondering earlier why you should care about all this graph and math stuff. It is because it allows us to better understand and unmask the social madnesses that we use to organize ourselves. And doing so allows us to illuminate the darker aspects of our past, present, and humanity in general in a new revealing light. 















[^1]: A few caveats here: although should be single pass theoretically, in practice we use two phases one for the spread of the accusation and a second to clean up any --- that involve the scapegoat to make them +--. Additionally, the algorithm gets much slower with graphs of 1000 nodes or more, mostly due to implementationnaly inefficiencies on my part as opposed to any fundamental limitation of the algorithm.
