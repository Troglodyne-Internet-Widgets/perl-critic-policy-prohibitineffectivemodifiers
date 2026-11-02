# NAME

Perl::Critic::Policy::RegularExpressions::ProhibitIneffectiveModifiers - Write a regex modifier only where it changes what the regex does.

# VERSION

version 0.001

# Perl::Critic::Policy::RegularExpressions::ProhibitIneffectiveModifiers

A reader takes each modifier on a regex as a statement about what it does.
`/g` says it can match more than once, `/i` that case matters to it, `/m`
that it anchors lines, and `/o` that it interpolates something that is
compiled once.  Where the pattern cannot do that, the modifier is a statement
that is not true, and the reader has to work out that it is not:

```
$name =~ s/^\s+//g;                   # reported: ^ matches once
$name =~ s/^\s+//;

next unless $line =~ m/\A\d+\z/i;     # reported: digits have no case
next unless $line =~ m/\A\d+\z/;
```

## PROHIBITED

```
s/^\s+//g             # /g, anchored at the start: it matches once
s/\A\s+//g            # \A as well as ^
m/\A(\w+)/g           # in a match as well
s/\s+\z//g            # anchored at the end, and it cannot match empty
s/x$//g               # $ without /m, and it cannot match a newline alone
s/,\Z//g              # \Z is the same as $ without /m
s/\A\s+//gm           # /m does not change \A, and has no ^ or $ to change
m/\d+/i               # /i, and nothing has a case
m/[!-@]/i             # nor does a range without a letter in it
qr/\d/i               # a compiled regex keeps its own modifiers
m/foo/m               # /m, and no ^ or $
m/\Afoo\z/m           # \A, \z and \Z are not line anchors
m/foo/o               # /o, and nothing to interpolate
s/foo/$bar/o          # /o compiles the pattern, not the replacement
```

## ALLOWED

```
s/^\s+//gm            # ^ under /m is the start of every line
m/^key: .*$/mg        # several lines of one form
s/\s+$//g             # \s can match the final newline alone, see CAVEATS
s/b*\z//g             # can match empty, so twice at the end
s/$/!/g               # "a\n" has two places for $ without /m
s/.$//sg              # . under /s can match the final newline alone
m/\A\s*/gc            # /c keeps pos() for the next match
m/\Ga/g               # \G is where the last match stopped
s/\Aa|b//g            # alternation: b is not anchored
s/\A$prefix//g        # interpolation may bring an alternation
m/[!-~]/i             # a range with letters in it
m/\x{df}/i            # sharp s matches SS under /i
m/\p{Lu}/i            # a property matches the other case under /i
m/(\d)\1/i            # a backreference compares without case
m/$word/i             # interpolation may bring letters
m/a(?=$)/m            # a line anchor anywhere counts
m/$record/m           # interpolation may bring ^ and $
m/$r/g                # where $r = qr/^key: .*$/m keeps its own /m
m/$x/o                # /o with something to compile once
m/a/s                 # /s, /x and /a are never reported, see CAVEATS
```

## CAVEATS

`/s`, `/x`, `/xx`, `/a` and `/aa` are never reported.  The policies
`RegularExpressions::RequireDotMatchAnything`,
`RegularExpressions::RequireExtendedFormatting` and
`RegularExpressions::RequireDefault` ask for them on every regex, so they are
written where they change nothing.
`RegularExpressions::RequireLineBoundaryMatching` asks for `/m` in the same
way; with that policy on, turn this one off, or the two disagree
about every regex without `^` or `$`.

Only a modifier written on the regex is reported.  A modifier from `use re`
in scope is a default for the file, not a choice made for this regex, so it is
not reported.  It still counts toward what the pattern means, so under
`use re '/m'`, `s/^\s+//g` is not reported.

`/g` at the end of the string is reported only where the policy can tell that
the pattern cannot match twice.  Before `\z` that means it cannot match the
empty string, so something in it has to match at least one character.  Before
`$`, or `\Z`, without `/m`, there are two places to end, at the final newline
and after it, so something in it has to match a character that is not a
newline.  The policy does not reason about greediness, so `s/\s+$//g`, which
matches once, is not reported.

`m//g` in scalar context sets `pos()` where it stops, and the next `\G`
starts there.  When a match anchored at the start is there for that, write
`/gc`, which says so, or say `## no critic (ProhibitIneffectiveModifiers)`.

A regex that interpolates a variable is not checked for `/g`, `/i` or `/m`.
The variable can bring an alternation, a letter or a line anchor, and the
policy cannot see what it holds.

## METHODS

### supported\_parameters

### default\_severity

### default\_themes

### applies\_to

### violates

# AUTHORS

Current Maintainers:

- George S. Baugh <george@troglodyne.net>

# COPYRIGHT AND LICENSE

Copyright (c) 2026 Troglodyne LLC

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:
The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.
THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
