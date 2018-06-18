# Perl 6 CaR TPF Grant: Monthly Report (June, 2018)

This document is the June, 2018 progress report for The Perl Foundation's [Perl 6 Constant and Rationals Grant](http://news.perlfoundation.org/2018/04/grant-proposal-perl-6-bugfixin.html).

------------------

## Tangibles

The bonus deliverable "Perl 6 Numerics" Language documentation page was
[merged to master](https://github.com/perl6/doc/compare/9e1b78a73063...ef8c8911d5e0). [It describes](https://docs.perl6.org/language/numerics) all of the available Perl 6 numerics, their
interactions, suitability, and hierarchy.

The bulk of work on constants also [has been merged](https://github.com/rakudo/rakudo/pull/1935/files) to `post-release-2018.06` branch, which will be merged to `master` after this month's release. I wrote 200 spec tests, available in [`S04-declarations/constant-6.d.t` spec file](https://github.com/perl6/roast/blob/post-release-2018.06/S04-declarations/constant-6.d.t) and about [500 words of documentation](https://github.com/perl6/doc/commit/086d7c11bda89b8505196316f3b5d11a2c27d660) to cover this work.

## Rationals

Since my last report, I first continued working on Rationals, focusing on three
pieces of work that currently reside in
[`car-grant-unreduce` branch](https://github.com/rakudo/rakudo/compare/car-grant-unreduce)

1. Fixing the rare data race and doing some optimizations
2. Fixing bad math in some ops with Zero-Denominator Rationals (ZDRs)
3. Attempting a trial implementation where ZDRs are
    marked with a role, allowing us to improve performance of some operators.

The (1) was successful and I already was able to optimized some
ops due to Rationals being always reduced now: `==` was made 28% faster and `===` was made 52% faster. I also made
creation of `Rational`s 19% faster and argless `Rational.round` 4.7x faster
(used by `.Str` and `.base`).

The plan for (2) was to try normalization to `<1/0>`, `<0/0>`, `<-1/0>` and
the hope was that alone would fix all math problems. However, after that change some issues still remained.
Also, doing this normalization created a new problem where code like
`say 42 / 0` would throw an Exception with message `"division of 1 by 0 attempted"` which is quite confusing for users who did not internalize that
`/` op with `Int` objects is really just a `Rat` constructor. The 6.c spec
actually covers this exact scenario and expects the thrown `Exception` to
report value `42` for numerator, thus blocking this change.

The first attempt at (3) ended with a dead-end.
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

The bulk of the work on constants is now completed and has been merged to
`post-release-2018.06` branch, which will be merged to `master` after this
month's release.

I wrote 200 spec tests, available in [`S04-declarations/constant-6.d.t` spec file](https://github.com/perl6/roast/blob/post-release-2018.06/S04-declarations/constant-6.d.t), about [500 words of documentation](https://github.com/perl6/doc/commit/086d7c11bda89b8505196316f3b5d11a2c27d660), and [8 commits of the implementation](https://github.com/rakudo/rakudo/pull/1935/files).

Except for native types, the type constraints are now enforced on constants.
The auto-coercive behaviour for `%-` sigilled constants was blocked by
one tests in 6.c specification and so that behavior has been added to 6.d
language and currently requires the use of `use v6.d.PREVIEW` pragma to enable
(6.c behavior is to simply throw without any attempts to coerce, making
`constant %foo = :42foo, :70bar` fail, because it's a `List`).

One of the remaining things for constants is improving error reporting for unsupported constructs. Most likely I will implement support for coercers—even though they're currently not available on variables, there should be no problem with executing them during constant creation.

The other remaining item is natively-typed constants. Currently, `my int const foo = 42` actually does **not** create a natively-typed constant at all. I hope the knowledge I'll gain while implementing the Grant's bonus work for support of native-typed unsigned attributes will help me in implementing the natively-typed constants as well.

## No Report for July / Further Work

As has been discussed with and approved by my grant manager, I plan to now
take a month off working on this grant, so there will be no report in mid-July,
and the next report will be in mid-August. I plan to take this time to work on resolving Rakudo's outstanding spectest issues on Windows and possibly resolving
some of the open Issues with our community websites.

After that period, I will resume working on the grant, finishing the
remaining work on constants, and then going back to Rationals, and finally
finishing the bonus work with the natively-typed entities.
