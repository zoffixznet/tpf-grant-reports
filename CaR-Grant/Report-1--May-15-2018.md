# Perl 6 CaR TPF Grant: Monthly Report (May, 2018)

This document is the May, 2018 progress report for The Perl Foundation's [Perl 6 Constant and Rationals Grant](http://news.perlfoundation.org/2018/04/grant-proposal-perl-6-bugfixin.html).

------------------

## Tangibles Produced

This month I was working in docs and roast repos, in separate
`car-grant-midrat` branches. So far, they contain [29 documentation commits](https://github.com/perl6/doc/compare/car-grant-midrat?expand=1#files_bucket) and [11 spec commits](https://github.com/perl6/roast/compare/master...car-grant-midrat?expand=1#files_bucket) adding about 5,824 words of documentation and 152 tests.

The tests specced the `MidRat` type and the documentation, along with
documenting `MidRat`/`MidRatStr` types, centered largely around new
[`Language/Numerics` page](https://github.com/perl6/doc/blob/car-grant-midrat/doc/Language/numerics.pod6) that describes all of the available Perl 6 numeric types—including native
and atomic numerics—their purpose, properties, and hierarchy. This guide was
not on the list of deliverables of the Grant and has been produced as a bonus
item.

## General Info

I started the grant by working on adding the proposed `MidRat`/`MidRatStr` type
pair. This was the most contentious part of the [Rationals Work Proposal](https://github.com/rakudo/rakudo/blob/master/docs/archive/2018-03-04--Polishing-Rationals.md) that before the grant underwent three revisions,
settling on solving the problem by adding the `MidRat` type.

I went full-on with *Test/Documentation Driven Development* and I think the
result is a fantastic example of the power of this approach. After adding
5,824 words of docs and 152 tests, I still haven't written a single line of code
of the implementation in the compiler itself.

The process highlighted several issues with `MidRat` type and I've tentatively
decided not to implement it at this time. Its functionality will be subsumed
with the plain `Rat` type. The plan to change `Rat` type to use `uint64`
native type for the denominator is tentatively canceled and instead an `Int` denominator will accept denominators over 64 bits in size in all the cases where
a `MidRat` type were going to be produced. At the same time, the `Rat` will
degrade to a `Num` in all the cases where a `MidRat` were to do so.

Doing so robs us of potential performance improvements due to using native
denominator, however we still have the option of implementing a `MidRat`-type
solution open in the future. Were we to go with implementing `MidRat` right
now, we'd be locking ourselves into supporting it basically forever, and
exposing ourselves to yet unforeseen design issues.

## Issues/Clarifications of `MidRat`

#### What is a `.Numeric`/`.Real` of a `MidRat`?

Writing documentation first showed an ambiguity with what `.Numeric`/`.Real`
methods should return for a `MidRat`. The [work proposal](https://github.com/rakudo/rakudo/blob/master/docs/archive/2018-03-04--Polishing-Rationals.md) had `MidRat` as an allomorph of `Rat` and `FatRat`,
and while `.Numeric` on `IntStr` allomorph returns its numeric component,
a `MidRat` would have *two* such components.

This issue is complicated by the fact that `prefix:<+>` uses `.Numeric`,
so that means `0 + MidRat.new(1,2)` would produce a `Rat` objet, yet
`+MidRat.new(1,2)` would produce a potentially different object. To maintain
consistency there, at the time I settled on simply having `.Numeric`/`.Real`
coerce the `MidRat` to a `Rat`, which means that had the potential of
degradation of that `Rat` to a `Num`. Bring the `MidRatStr` into play and
you get three different types by chaining `.Numeric` calls.
That was already slightly weird.

#### Convergence to Plain Rat

While documenting infectiousness of numerics, the question of infectiousness
of `MidRat` came about. The original [work proposal](https://github.com/rakudo/rakudo/blob/master/docs/archive/2018-03-04--Polishing-Rationals.md) was vague in this area but after documenting the
`MidRat` type, it became obvious that its infectiousness must be the
*same as if `Rat` type was used instead*. That's the first point of convergence
to `Rat`.

The second became apparent when writing degradation tests. The
[work proposal](https://github.com/rakudo/rakudo/blob/master/docs/archive/2018-03-04--Polishing-Rationals.md) argued for `MidRat` to be an allomorph of `Rat` and
`FatRat` types. A `MidRat` has infectiousness of a `Rat`. But that means this
code…

    my FatRat $a = get-rational;
    my FatRat $b = get-rational;
    say $a + $b;

…has the potential to produce a `Num` of all things (`MidRat`s degrade to
`Rat` and that can degrade to a `Num`). That's certainly unacceptable, so
by this point, I decided to toss `FatRat` from the ancestors of `MidRat`,
leaving it be just a subclass of `Rat`. This largely resolved the issue of
what to return from `.Numeric`/`.Real` methods.

...

By now you can see that the `MidRat` got reduced to just being a `Rat` with
non-native denominator. Adding two extra types (`MidRat`/`MidRatStr`) to
an already populous zoo of Perl 6 numerics for such a tiny detail was hard
for me to justify. So, after [flipping a coin](https://irclog.perlgeek.de/perl6/2018-05-16#i_16169789) to confirm my gut feeling, I decided to abandon the idea of `MidRat` and to repurpose the `Rat` type to resolve all of the issues `MidRat` were meant to resolve.

## Going Forward

I still plan on doing the Bonus Work of the grant, fixing bugs in native
`uint64` attributes, despite no longer planning to use them directly in
the denominator of the `Rat` type. I'll also try to make a perf improvement
by selecting native/non-native NQP ops for dividing `Rat`s, to make up for
not using native attributes in them.

In lieu of implementing the `MidRat` type originally promised, I'll work on
resolving some additional bugs. Likely trying to resolve value drift in
MoarVM `Num`ification of `Rational`s that currently does not select the
closest representable `Num`.

For the next month, I plan to work more on Rationals, fixing the data race
and normalization of zero-denominator Rationals.

## Related Work Done By Others

* AlexDaniel++ fixed object-identity bug in `Rationals`. The fix was done
using the `.REDUCE-ME` method, which has the data race and will go away
in about a month, but in the meantime, object identity will be correct.
* thundergnat++ did some work on improving performance of `Rat.Str` and `FatRat.Str` and fixed an issue with unwanted trailing zeros.
