Overview
--------

I have a problem. I have a couple million nucleic acids sequences, 
each 18 bases long. Each base is 'A', 'C', 'G', or 'T'. 18 bases in a row 
makes for 69 billion possible sequences. The actual data (2M) is very 
sparse compared to the potential list (69B).

I want to eliminate duplicates out of my 2M sequences. But by "duplicate" I 
don't mean exact match only. I mean any sequence that is one mutation (a 
single base changes from one letter to another letter) or two mutations distant.

This is quite the problem. When you have 18 bases and mutate 2 of them 
that's 2,754 possible mutations (18 \* 3 \* 17 \* 3 ... right?) for each sequence. 
So to de-dupe my 2M sequences I actually have to do 5.5B searches through 2M 
sequences (11 quadrillion comparisons). That's a lot, even with good indexes.

And this problem is exponential as 18 bases grows to 19 or 20 or 30...

I want a parallel processing solution so the work doesn't take so long. 
Luckily there's a 2000 node super-computer I can use. That should help.

The idea is to break my 2M sequences into an arbitrary number of shards.  
To talk through a small example, let's pretend my sequences are 4 bases 
long instead of 18.

    AAAA..TTTT is 256 sequences

We split the search space into 4 nodes:

    AAAA..ATTT   64 sequences
    CAAA..CTTT   64 sequences
    GAAA..GTTT   64 sequences
    TAAA..TTTT   64 sequences

Now each node is responsible for doing lookups and remembering statistics
regarding a maximum of 64 sequences each. And if our data is very sparse
then the actual number is far less.


Getting started
---------------

Here's an Erlang / MutationGrid "Hello World". First, install Erlang (http://erlang.org). 
Under Ubuntu, you can simply:

    apt-get install erlang

Then, download, compile and run some of our source code:

    $ git clone git://github.com/stesla/mg.git
    $ cd mg/src
    $ erl
    1> c(mg_util).
    {ok,mg_util}
    2> mg_util:combinations("ACGT", 1).
    ["$CGT","A$GT","AC$T","ACG$"]
    3> mg_util:combinations("ACGT", 2).
    ["A$$T","$C$T","A$G$","$CG$","AC$$","$$GT"]
    4> q().

Did that whet your appetite?



jhannah's hypothetical API
--------------------------

This is what I thought we were going to write. stesla's function names are different.
So this might be conceptually useful.

hmm... how do we bootstrap each node? text file ssh'd to the node?

**mutate(Sequence, NumberOfMutations)**

  Starting with Sequence (e.g. 'AAAA'), iterate every possible Mutant caused by introducting
  NumberOfMutations (e.g. '1' or '2'). For each local Mutant, if Mutant is a known Sequence
  then increment that Sequence's counter. For each remote Mutant, increment_sequence()
  to the node responsible.

**increment_sequence(Sequence)**

  Another node has discovered that one of our Sequences is a Mutant of one of their
  sequences. Increment our counter.

**sequence_is_local(Sequence)**

  Returns true if Sequence is in the local shard. False otherwise.

**node_for_sequence(Sequence)**

  Returns the node responsible for Sequence.

**report()**

  Generate a report of all statistics that have been gathered for all of our Sequences.


Random notes, links
-------------------

* http://erlang.org/download/getting_started-5.4.pdf
* You need matching ~.erlang.cookie files.
* http://mad.printf.net/MSCC_matching_instructions/matching.html

