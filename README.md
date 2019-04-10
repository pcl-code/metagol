Metagol is an inductive logic programming (ILP) system based on meta-interpretive learning. Please contact Andrew Cropper (andrew.cropper@cs.ox.ac.uk) with any questions / bugs. If you use Metagol for research, please use [this citation](https://raw.githubusercontent.com/metagol/metagol/master/metagol.bib).

#### Using Metagol

Metagol is written in Prolog and runs with both Yap and SWI. The following code demonstrates learning the grandparent relation given the mother and father relations as background knowledge:

```prolog
:- use_module('metagol').

%% first-order background knowledge
mother(ann,amy).
mother(ann,andy).
mother(amy,amelia).
mother(linda,gavin).
father(steve,amy).
father(steve,andy).
father(gavin,amelia).
father(andy,spongebob).

%% predicates that can be used in the learning
prim(mother/2).
prim(father/2).

%% metarules
metarule([P,Q],([P,A,B]:-[[Q,A,B]])).
metarule([P,Q,R],([P,A,B]:-[[Q,A,B],[R,A,B]])).
metarule([P,Q,R],([P,A,B]:-[[Q,A,C],[R,C,B]])).

%% learning task
a :-
  %% positive examples
  Pos = [
    grandparent(ann,amelia),
    grandparent(steve,amelia),
    grandparent(ann,spongebob),
    grandparent(steve,spongebob),
    grandparent(linda,amelia)
  ],
  %% negative examples
  Neg = [
    grandparent(amy,amelia)
  ],
  learn(Pos,Neg).

```
Running the above program will print the output:

```prolog
% clauses: 1
% clauses: 2
% clauses: 3
grandparent(A,B):-grandparent_1(A,C),grandparent_1(C,B).
grandparent_1(A,B):-mother(A,B).
grandparent_1(A,B):-father(A,B).
```

where the predicate `grandparent_1/2` is invented and corresponds to the parent relation.

#### Metarules

Metagol requires higher-order metarules to define the form of clauses permitted in a hypothesis. An example metarule is:

```prolog
metarule([P,Q,R],([P,A,B]:-[[Q,A,C],[R,C,B]])).
```

In this metarule, known as the chain metarule, the symbols `P`, `Q`, and `R` denote existentially quantified higher-order variables, and the symbols `A`, `B`, and `C` denote universally quantified first-order variables. The list of symbols in the first argument denote the existentially quantified variables which Metagol will attempt to find substitutions for during the learning.

Users need to supply Metarules. We are working on automatically identifying the necessary metarules, with preliminary work detailed in the paper:

* Cropper, A. and Muggleton S.H. [Logical minimisation of meta-rules within meta-interpretive learning](http://andrewcropper.com/pubs/ilp14-minmeta.pdf). ILP 2014.

* Cropper, A., and Tourret, S. [Derivation reduction of metarules in meta-interpretive learning](http://andrewcropper.com/pubs/ilp18-dreduce.pdf). ILP 2018.

For learning dyadic programs without constant symbols, we recommend using these metarules:

```prolog
metarule([P,Q],([P,A,B]:-[[Q,A,B]])). % identity
metarule([P,Q],([P,A,B]:-[[Q,B,A]])). % inverse
metarule([P,Q,R],([P,A,B]:-[[Q,A],[R,A,B]])). % precon
metarule([P,Q,R],([P,A,B]:-[[Q,A,B],[R,B]])). % postcon
metarule([P,Q,R],([P,A,B]:-[[Q,A,C],[R,C,B]])). % chain
```

Please look at the examples to see how metarules are used.

#### Recursion

The above metarules are all non-recursive. By contrast, this metarule is recursive:

```prolog
metarule([P,Q],([P,A,B]:-[[Q,A,C],[P,C,B]])).
```

The find-duplicate, sorrter, and string examples illustrate learning recursive programs.

#### Metagol settings


Metagol searches for a hypothesis using iterative deepening on the number of clauses in the solution. You can specify a maximum number of clauses:

```prolog
metagol:max_clauses(Integer). % default 10
```

The following flag denotes whether the learned program should be unfolded to remove unnecessary invented predicates:

```prolog
metagol:unfold_program. % default false
```

For instance, with the flag set to false, Metagol would learn this great-grandmother theory:

```prolog
ggmother(A,B):-mother(A,C),ggmother_1(C,B).
ggmother_1(A,B):-mother(A,C),mother(C,B).
```

With the flag set to true, Metagol would learn this great-grandmother theory:

```prolog
ggparent(A,B):-mother(A,C),mother(C,D),mother(D,B).
```

The following flag denotes whether the learned theory should be functional:

```prolog
metagol:functional. % default false
```
If the functional flag is enabled, then the must define a func_test predicate. An example func test is:

```prolog
func_test(Atom,PS,G):-
  Atom = [P,A,B],
  Actual = [P,A,Z],
  \+ (metagol:prove_deduce([Actual],PS,G),Z \= B).
```

This func test is used in the robot examples. Here, the `Atom` variable is formed of a predicate symbol `P` and two states `A` and `B`, which represent initial and final state pairs respectively.  The func_test checks whether the learned hypothesis can be applied to the initial state to reach any state `Z` other that the expected final state `B`. For more examples of functional tests, see the robots.pl, sorter.pl, and strings2.pl files.


### Publications

Here are some publications on MIL and Metagol.


#### Key papers

* Andrew Cropper, Stephen H. Muggleton: Learning efficient logic programs. Machine learning (2018). https://doi.org/10.1007/s10994-018-5712-6.

* Andrew Cropper, Stephen H. Muggleton: Learning Higher-Order Logic Programs through Abstraction and Invention. IJCAI 2016: 1418-1424

* Stephen H. Muggleton, Dianhuan Lin, Alireza Tamaddoni-Nezhad: Meta-interpretive learning of higher-order dyadic datalog: predicate invention revisited. Machine Learning 100(1): 49-73 (2015)

* Stephen H. Muggleton, Dianhuan Lin, Niels Pahlavi, Alireza Tamaddoni-Nezhad: Meta-interpretive learning: application to grammatical inference. Machine Learning 94(1): 25-49 (2014)

#### Theory and Implementation

* Andrew Cropper, Sophie Tourret: Derivation Reduction of Metarules in Meta-interpretive Learning. ILP 2018: 1-21

* Andrew Cropper, Stephen H. Muggleton: Learning Efficient Logical Robot Strategies Involving Composable Objects. IJCAI 2015: 3423-3429

* Andrew Cropper, Stephen Muggleton: Can predicate invention compensate for incomplete background knowledge? SCAI 2015: 27-36

* Andrew Cropper, Stephen H. Muggleton: Logical Minimisation of Meta-Rules Within Meta-Interpretive Learning. ILP 2014: 62-75

* Stephen H. Muggleton, Dianhuan Lin, Jianzhong Chen, Alireza Tamaddoni-Nezhad: MetaBayes: Bayesian Meta-Interpretative Learning Using Higher-Order Stochastic Refinement. ILP 2013: 1-17

#### Applications

* Stephen H. Muggleton, Ute Schmid, Christina Zeller, Alireza Tamaddoni-Nezhad, Tarek R. Besold: Ultra-Strong Machine Learning: comprehensibility of programs learned with ILP. Machine Learning 107(7): 1119-1140 (2018)

* Stephen Muggleton, Wang-Zhou Dai, Claude Sammut, Alireza Tamaddoni-Nezhad, Jing Wen, Zhi-Hua Zhou:
Meta-Interpretive Learning from noisy images. Machine Learning 107(7): 1097-1118 (2018)

* Dianhuan Lin, Eyal Dechter, Kevin Ellis, Joshua B. Tenenbaum, Stephen Muggleton: Bias reformulation for one-shot function induction. ECAI 2014: 525-530

* Colin Farquhar, Gudmund Grov, Andrew Cropper, Stephen Muggleton, Alan Bundy: Typed meta-interpretive learning for proof strategies. ILP (Late Breaking Papers) 2015: 17-32

* Andrew Cropper, Alireza Tamaddoni-Nezhad, Stephen H. Muggleton: Meta-Interpretive Learning of Data Transformation Programs. ILP 2015: 46-59

#### Theses

* Andrew Cropper: Efficiently learning efficient programs. Imperial College London, UK 2017

* Dianhuan Lin: Logic programs as declarative and procedural bias in inductive logic programming. Imperial College London, UK 2013
