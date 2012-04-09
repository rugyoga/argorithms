---
layout: post
title: "Revisiting Tournament sort"
tagline: "A sort that's easy on the compares"
category:
tags: []
---
{% include JB/setup %}

## Sorting fundamentals

In undergraduate Computer Science, probably the most basic topic that we're taught in an Algorithms and Data Structures is sorting. If you ask any CS undergrad worth their salt, they'll tell you that a good sort is O( n log n ). The other nugget they'll take away is that quicksort is the fastest sorting algorithm and that's that. Let's take a closer look at that first assertion.

Where do we get the n log n from? Given an array of n distinct elements, they can be arranged in n! different ways. If we have a binary comparison operator, it will take log2( n! ) operations to correctly identify which permutation we have. The n log n comes from Stirling's approximation with says if we replace n! with n^n, we can get a crude approximation log2( n^n ) which is n log2( n ). While n log2 n is fine for big O notation, our (ambitious) goal for a practical sort should be to get close to log2( n! ) comparisons. Let's see what we can do...

## Let's have a tournament!

We live in a competitive society. You can hardly surf the channels on the TV without coming across some sporting competition or other. If I asked you to find the best (tennis/pool/chess/scrabble/tiddlywinks) player amongst a group of people, you would likely suggest a knockout tournament. Basically, we pair everyone up with one other competitor and have them play to determine the winner. Then we do the same with the winners. We rinse and repeat until we're left with one competitor who is undefeated and we declare them the champion. Here's how we might implement that in ruby:

    class Tournament
      def initialize( v )
        @tourney = knockout( v )
      end
  
      def pop
        winner, @tourney = Tournament.extract_winner( @tourney )
        winner
      end
  
      def knockout( v, i=0, k=v.size-1 )
        return Tournament.player( v[i] ) if i == k
        j = (i+k)/2
        Tournament.match( knockout( v, i, j ), knockout( v, j+1, k ) )
      end
  
      def self.sort!( v )
        tourney = Tournament.new( v )
        v.size.times{ |i| v[i] = tourney.pop }
      end

      def self.sort( v )
        tourney = Tournament.new( v )
        Array.new( v.size ){ |i| tourney.pop }
      end
    end

Here our tournament is represented as a perfectly balanced tree of n-1 matches except at the leaf nodes where it's the actual item.
	
## Finding the second best

So we've found our champion and showered them with champagne, wreaths of laurels and gold sovereigns. But who is the 2nd best player? Do we need to have another tournament to find that out? I sure hope not! If you listened to most sports commentators, they'd assert that it was the player that lost in the final. Alas, that's not necessarily the case - it could be any of the players that the champion beat on the way to winning. Most sports tournaments use seeding to try and ensure that the top players meet in the final so that it presents the best sporting spectacle. However that only works if we have some information ahead of time regarding the seeding. In a practical sorting scenario, we likely don't - that's why we're sorting! So we need to determine the strongest of the champion's victims by having them play enough matches to determine a winner. If we recurse down the tree and replace the champ with the strongest loser at each point, this will have the desired effect:

    class Match
      attr_reader :winner, :losers, :winners

      def initialize( top, bot )
        top_w = top.is_a?( Match ) ? top.winner : top
        bot_w = bot.is_a?( Match ) ? bot.winner : bot
        @winner, @winners, @losers =
          top_w <= bot_w ?
          [top_w, top, bot] :
          [bot_w, bot, top]
      end
  
      def remove_winner
        winners.is_a?( Match ) ? Match.new( losers, winners.remove_winner ) : losers;
      end
    end

    class Tournament
      def Tournament.player( p )
        p
      end
  
      def Tournament.match( a, b )
        Match.new( a, b )
      end
  
      def Tournament.extract_winner( t )
        t.is_a?( Match ) ? [t.winner, t.remove_winner] : [t, nil]
      end
    end

So how expensive is it to rebuild the tree of Matches after we've extracted the champion? Well we started with a perfectly balanced tree of Matches, so it will take at most log2( n ) operations to find the runner-up. Sounds like a n log2( n ) algorithm? Well it's slightly better than that - each time we extract a runner-up, the tree gets smaller - n/2 of the operations will be log2( n ), then n/4 will be log2( n ) - 1 etc etc.

## Performance in ruby

If we instrument our code and run some benchmarks again the built-in, we're in for some slightly good news and some very bad news. The very bad news is that our shiny new sort is at least two orders of magnitude slower than Array#sort. The good news is that our sort is within a couple of percent of the optimal sort, log2( n! ), in terms of the number of comparisons used. We shouldn't beat ourselves up too badly over the wall clock performance - we're not competing with ruby code, we're competing with an algorithm coding in C and baked into the ruby interpreter. The other issue is that ruby isn't an ideal testing ground for benchmarking data structures as their space performance of the individual nodes of the match tree is very poor as ruby stores them as a dictionary. If we switch to a language where we have an standard sort function implemented in the language itself and has finer grain control of space allocation of user defined types, we should see a big improvement. Sure enough, we translated our code into Java and the results were much much better.

## Converting to Java


