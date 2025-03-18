---
title:
- A formalization of Borel determinacy in Lean
author:
- Paweł Balawender
date:
- March 19, 2025
# pandoc -t beamer prezentacja.md -o prezentacja.pdf --bibliography references.bib --citeproc  -M link-citations=true -V colortheme:crane -V theme:CambridgeUS --csl apa.csl
# pdftk A=inria-curryhoward.pdf B=prezentacja.pdf cat B1-7 A8-15 output out.pdf
---

# Choice of proof assistant
>- Suppose you have just proved a theorem and would like to formalize it
>- In which system should you do it? In Rocq? In Isabelle? Maybe Mizar?
>- Good criteria: user experience, existing standard libraries, etc.
>- But the proof assistant is only as strong as the logic it uses internally!

# The power of different provers
>- Crucial criterion: if you took the logic of a specific prover, could you prove your theorem in this logic?
>- If your theorem prover is based on intuitionistic logic, you might not prove a classical theorem
>- Specifically: in Lean, you will not prove that for every predicate $P$, $P \lor \lnot P$ holds.
>- (Lean is expressive enough to understand the second-order sentence $\forall P, P \lor \lnot P$ and take is as an axiom, so you can use it without proving)
>- Beware: these logics are not simple!

# Let's consider a game
>- Consider the set of all infinite sequences of natural numbers, $\mathbb{N}^\mathbb{N}$, and a subset $A \subseteq \mathbb{N}^\mathbb{N}$, which we call the winning set.
>- Alice and Bob alternately pick natural numbers $n_0, n_1, n_2, ...$. Alice is the first one to pick.
>- That generates a sequence $\langle n_i \rangle_{i \in \mathbb{N}}$. Alice wins iff the generated sequence belongs to $A$, the set of winning sequences.

# When Alice can win?
>- If the winning set is finite, then the set of elements $B_1$ that Bob should choose in his first turn is also finite. So Bob can choose any number from $\mathbb{N} - B_1$ and the rest of the game doesn't matter - Alice loses.
>- In this proof, we need to be able to:
>- Define natural numbers $\mathbb{N}$
>- Define an infinite, ordered sequence of numbers
>- Define a set of sequences, the winning set
>- Define taking the second element of a sequence
>- Define a set $B_1$ of second elements
>- Prove that $\mathbb{N} - B_1$ is not empty
>- Prove that for any suffix, Bob's sequence won't be winning

# ZFC: Zermelo-Fraenkel axioms of set theory + choice (1922)
>- Extensionality
>- Unordered Pair
>- Specification (Comprehension)
>- Sum Set
>- Power Set
>- Infinity
>- Replacement (*)
>- Regularity
>- Choice (**)

# Axiom of Extensionality
Two sets are equal if and only if they contain exactly the same members.
$\forall u(u \in X \iff u \in Y) \implies X = Y$

# Axiom of the Unordered Pair
Given any two sets, we can form a new set containing precisely those two sets.
$\forall a \forall b \exists c \forall x(x \in c \iff (x = a \lor x = b))$

# Axiom of Specification (Separation / Comprehension)
We can form a subset of an existing set consisting of all elements that satisfy a given property.
$\forall X \forall p \exists Y \forall u(u \in Y \iff (u \in X \land \phi(u,p)))$

# Axiom of the Sum Set (Union)
Given a collection of sets (which is itself a set), we can form a new set containing all the elements that belong to at least one set in the collection.
$\forall X \exists Y \forall u(u \in Y \iff \exists z(z \in X \land u \in z))$

# Axiom of the Power Set
For any set, we can form a new set that contains all possible subsets of the original set.
$\forall X \exists Y \forall u(u \in Y \iff u \subseteq X)$

# Axiom of Infinity
This axiom postulates the existence of at least one set with infinitely many elements. A common example is the set of natural numbers (or a set that can be put into one-to-one correspondence with it).
$\exists S [\emptyset \in S \land (\forall x \in S) [x \cup \{x\} \in S]]$

# Axiom of Replacement
If F is a function, then for any X there exists a set $\{F(x):x \in X\}$
$(\forall x \forall y \forall z [\phi(x,y,p) \land \phi(x,z,p) \implies y = z]) \implies (\forall X \exists Y \forall y [y \in Y \iff (\exists x \in X) \phi(x,y,p)])$

# Axiom of Foundation (Regularity)
This axiom prevents the existence of infinite descending chains of set membership; every nonempty set has an $\in$-minimal element.
$\forall S [S \neq \emptyset \implies (\exists x \in S) [S \cap x = \emptyset]]$

# Axiom of Choice
Given any collection of non-empty sets, it is possible to select one element from each set, even if the collection is infinite and there is no specific rule for making the selection.
$\forall x \in a \exists y A(x,y) \implies \exists f \forall x \in a A(x, f(x))$

# {N, P(N), P(P(N)), ...}
>- This set is not definable in Zermelo set theory, i.e. ZF without the axiom schema of replacement!
>- Will we not run into troubles while proving statements about more complicted winning sets in our game?

# Gale-Stewart
>- A Gale-Stewart game is a pair $G = (T, P)$, where T is a nonempty pruned tree (without leaves) and $P \subseteq [T]$ is the winning set.
>- Discrete topology on $\mathbb{N}$: the topology where every subset of $\mathbb{N}$ is open.
>- When we say 'open game', 'closed game', 'Borel game' we only consider e.g. openness of [T] in $\mathbb{N}^\mathbb{N}$ with the product topology

# Box topology
- The topology on $\prod_{\alpha \in J} X_\alpha$ with basis $\prod_{\alpha \in J} U_\alpha$, where each $U_\alpha$ is open in $X_\alpha$.

# Product topology
>- The topology on $\prod_{\alpha \in J} X_\alpha$ with basis $\prod_{\alpha \in J} U_\alpha$, where each $U_\alpha$ is open in $X_\alpha$ and all but finitely many $U_\alpha = X_\alpha$.
>- The key difference is that the box topology allows infinitely many factors in the basis element to be proper open subsets, while the product topology requires all but finitely many to be the entire space.

# Intuition
>- In an open game, if Player I is going to win, they will win after a finite number of moves. There will be a point in the game where, no matter what Player II does afterwards, the resulting infinite play will be in Player I's winning set. Player I's victory becomes guaranteed at some finite stage.
>- In a closed game, if Player II is going to win (meaning the play will *not* be in Player I's winning set $P$), they will win after a finite number of moves. There will be a point in the game where, no matter what Player I does afterwards, the resulting infinite play will *not* be in Player I's winning set. Player II's victory (Player I's loss) becomes guaranteed at some finite stage.

# Is the game determined when the winning set is closed? (Kechris)
>- Consider a game $G(T, P)$ where Player I wins if the play is in $P$, for $P$ closed.
>- If Player II has a winning strategy, the game is determined.
>- Assume Player II does not have a winning strategy. We will show Player I has one.
>- A position $p = (a_0, a_1, ..., a_{2n-1}) \in T$ (with I to play next) is "not losing for I" if Player II does not have a winning strategy from that position.
>- I.e. II has no winning strategy in the game $G(T|_p, P|_p)$, where $T|_p = \{s \mid ps \in T\}$ , and $P|_p = \{y \mid py \in P\}$. So $\varphi$ is not losing for I.

# Crucial observation
>- Suppose position $p = (a_0, ..., a_{2n-1})$ is not losing for Player I
>- It means that Player I has at least one move $a_{2n}$ such that for any possible response $a_{2n+1}$ by Player II (resulting in the position $pa_{2n}a_{2n+1}$), the new position $pa_{2n}a_{2n+1}$ is also not losing for Player I.
>- If for every possible move $a_{2n}$ by Player I, there existed a response $a_{2n+1}$ by Player II leading to a position from which Player I loses, then the initial position $p$ would be losing for Player I, which contradicts our assumption.

# Player I's Strategy Construction
>- Player I starts by choosing $a_0$ such that for all $a_1$, the position $(a_0, a_1)$ is not losing for Player I.
>- II then plays some $a_1$. I responds with $a_2$ such that for all responses $a_3$, $(a_0, a_1, a_2, a_3)$ is not losing for I, **etc.**

# Remark: neighbourhoods
>- Suppose $(a_n) = (a_0, a_1, ...) \in W$, for $W$ open.
>- Then there exists a neighbourhood $N$ around $(a_n)$, contained in $W$.
>- There is a $k$ such that $N_{(a_0, ... ,a_{2k-1})} \cap [T] \subseteq W$

# Winning Argument for Player I
>- We claim this strategy is winning for Player I.
>- Consider a run $(a_0, a_1, ...)$ of the game in which Player I followed the strategy.
>- Then, forall $n$, $(a_0, ... a_{2n-1})$ is not losing for Player I.
>- Since $P$ is closed in $[T]$, $P^c is open.
>- Suppose that $(a_0, ...) \notin P$.
>- There exists $k$ such that the neighborhood $N(a_0, ..., a_{2k-1}) \cap [T] \subseteq P^c$.

# Winning Argument for Player I
>- But this means that any suffix after $(a_0, ..., a_{2k-1})$ leads to winning, i.e. Player II has a trivial winning strategy!
>- So Player I loses from this position.
>- This contradicts the fact that $p$ was chosen by Player I such that for any next move by Player II, the resulting position is "not losing for Player I".
>- Therefore, the assumption that $(a_0, ...) \notin P$ must be false
>- So Player I wins.

# Remarks on Axiom of Choice (from Kechris)

>- Theorem 20.1 (Determinacy of Open/Closed Games) generally requires the **Axiom of Choice** due to the single-valuedness condition in the definition of a strategy. A strategy specifies a unique move for each possible history.

# Is the game determined when the winning set is Borel?
>- This is much more difficult!
>- Harvey Friedman showed that determinacy for Gale-Stewart games where the winning set is only Borel, is not provable in ZF without the axiom schema of replacement!
>- But will we be able to prove it in Lean 4?

# Lean 4: hierarchy of universes
>- In Lean, every object lives at one of (more than) three levels
>- There are the two universes Type and Prop at the top
>- There are the types and the theorem statements one level below them
>- Then there are the terms and the theorem proofs at the bottom. 
>- The type of all real numbers $\mathbb{R}$ is a type, so $\mathbb{R}$ lives at the middle level, and real numbers like 7 are terms; we write $7 : \mathbb{R}$ to indicate that 7 is a real number.

# ZFC version used in Lean 4: infinite hierarchy of universes

>- `Type` is the type of (small) types.
>- it leads to paradoxes to set the type of `Type` to be also `Type`
>- the type of `Type` is `Type 1`, etc.

# ZFC version used in Lean 4: pre-sets
Set some specific universe `u`. Define a notion of **pre-set** inductively:
```lean4
inductive PSet : Type (u + 1)
  | mk (a : Type u) (A : a → PSet) : PSet
```

# ZFC version used in Lean 4: sets as pre-set quotients
Define extensional equivalence on pre-sets. Define ZFC sets by quotienting pre-sets by extensional equivalence.

# ZFC version used in Lean 4: classes
Define classes as sets of ZFC sets. Then the rest is usual set theory.
```lean4
def Class :=
  Set ZFSet
```


# Modeling ZFC in Coq
>- There are a few projects, but no uniform way to work with ZFC in Coq
>- I honestly don't know the details. You define that a ZFC-set is a Type, then define some properties. It is very very subtle
>- In the HoTT book, there is a whole section on ZFC. Requires in-depth HoTT knowledge, so also category theory and algebraic topology.


# Theory of Mizar: Tarski-Grothendieck set theory
>- ZFC + Tarski's axiom, which implies existence of inaccessible cardinals
>- enough to define category theory, in contrast to ZFC
>- Mizar: a Polish theorem prover. In 2009 its mathlib was the biggest body of formalized maths in the world!
>- the underlying theory of Mizar is precisely first-order logic with Tarski-Grothendieck set theory
>- if you were to formalize that ALL games are determined, Mizar would certainly not be your best assistant: determinacy of all games implies Axiom of Determinacy, which is known to contradict the Axiom of Choice.

# Alexander Grothendieck (1928-2014)
![alt text](image.png)
>- algebraic geometry
>- synthesis between geoalg and number theory
>- synthesis between geoalg and topology
>- Grothendieck universes
>- introduced topoi to category theory
>- worked with Teichmuller, whose idea I showed in my last presentation; Grothendieck-Teichmuller group

# Throwback: How can you expect tax-payers to believe in this? (Inter-universal Teichmüller theory)
![Alt text](intergalactic-theory.png)

# Grothendieck, 1970
![alt text](image-1.png)

# Grothendieck, Lasserre, France, 2013
![alt text](image-2.png)


<!-- # References {.allowframebreaks} -->

