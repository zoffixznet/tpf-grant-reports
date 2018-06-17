# Perl 6 CaR TPF Grant: Monthly Report (June, 2018)

This document is the June, 2018 progress report for The Perl Foundation's [Perl 6 Constant and Rationals Grant](http://news.perlfoundation.org/2018/04/grant-proposal-perl-6-bugfixin.html).

------------------

## Rationals

The bonus deliverable "Perl 6 Numerics" Language documentation page was
[merged to master](https://github.com/perl6/doc/compare/9e1b78a73063...ef8c8911d5e0). It describes all of the available Perl 6 numerics, their
interactions, suitability, and hierarchy.

Since my last report, I first continued working on Rationals, focusing on three
pieces of work that currently reside in
[`car-grant-unreduce` branch](https://github.com/rakudo/rakudo/compare/car-grant-unreduce)

1. Fixing the rare data race and doing some optimizations
2. Fixing bad math in some ops with Zero-Denominator Rationals
3. Attempting a trial implementation where Zero-Denominator Rationals (ZDRs) are
    marked with a role, allowing us to improve performance of some operators.

The (1) was successful and even resulted in ability to optimize some ops
that weren't foreseen in original work proposal: `==` was made 28% faster
and `===` was made 52% faster. I also made creation of
`Rational`s 19% faster and argless `Rational.round` 4.7x faster
(used by `.Str` and `.base`).

The plan for (2) was to try normalization to `<1/0>`, `<0/0>`, `<-1/0>` and
the hope was that alone would fix all math problems, but some still remained.
Also, doing this normalization created a new problem where code like
`say 42 / 0` would throw an Exception with message `"division of 1 by 0 attempted"` which is quite confusing for users who did not internalize that
`/` op with `Int` objects is really just a `Rat` constructor. The 6.c spec
actually covers this exact scenario and expects the thrown `Exception` to
report value `42` for numerator, thus blocking this change.

The first attempt at (3) ended with dead-end.
Marking ZDRs with a role created an extra dispatch ambiguity with some
operators like `cmp`. We already had to disambiguate that op
with `is default` marker to disambiguate between `Rational` and `Real`
candidates, so now we'd need an `is default of defaults` trait :). I gave up
on this for now, but will likely revisit and try again.

As you can see, the `Rational`s work was a mixed bag, so to experiment with the
problem space a bit I decided to implement my own rationals outside of core
from scratch. The work is available in [`zoffixznet/perl6-Rashnl` repo](https://github.com/zoffixznet/perl6-Rashnl), which might become
an ecosystem module if it offers significantly better rationals.

I doubt the entirety of `Rashnl` could be made core, as my current approach
does not involve separate roles at all and just has a single class with a flag
for fattiness that marks what in core is a `FatRat` type. The primary purpose
of that work is to experiment with code and learn some lessons that could
be applicable for core `Rational`s and achieve the goals of this Grant.

Due to that detour for the `Rational`s work and more time to think required, I
switched to the work on constants for the time being…

## Constants

The work on constants is now completed and has been merged to
`post-release-2018.06` branch, which will be merged to `master` after this
month's release.

I wrote XXXX spec tests, XXXX words of documents, and XXXX commits of the
implementation.




------------------

# Documentation

The bonus deliverable "Perl 6 Numerics" Language documentation page was
[merged to master](https://github.com/perl6/doc/compare/9e1b78a73063...ef8c8911d5e0). It describes all of the available Perl 6 numerics, their
interactions, suitability, and hierarchy.






========= REVIEW STUFF BELOW ===========


## Bugs Fixed

Fixed [RT#130774 `Rational.REDUCE-ME has a data race`](https://rt.perl.org/Ticket/Display.html?id=130774)

## Commits

### Rakudo
https://github.com/rakudo/rakudo/commit/6dd20588b6dfb75
https://github.com/rakudo/rakudo/commit/22724c0eee

### Roast
https://github.com/perl6/roast/commit/1d10e9dc12
