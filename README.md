# NAME

Sub::Curried - automatically curried subroutines

# SYNOPSIS

    curry add_n_to ($n, $val) {
       return $n+$val;
    }

    my $add_10_to = add_n_to( 10 );

    say $add_10_to->(4);  # 14

    # but you can also
    say add_n_to(10,4);  # also 14

    # or more traditionally
    say add_n_to(10)->(4);

# DESCRIPTION

Currying and Partial Application come from the heady world of functional
programming, but are actually useful techniques.  Partial Application is used
to progressively specialise a subroutine, by pre-binding some of the arguments.

Partial application is the generic term, that also encompasses the concept of
plugging in "holes" in arguments at arbitrary positions.  Currying is more
specifically the application of arguments progressively from left to right
until you have enough of them.

# USAGE

Define a curried subroutine using the `curry` keyword.  You should list the
arguments to the subroutine in parentheses.  This isn't a sophisticated signature
parser, just a common separated list of scalars (or `@array` or `%hash` arguments,
which will be returned as a _reference_).

    curry greet ($greeting, $greetee) {
        return "$greeting $greetee";
    }

    my $hello = greet("Hello");
    say $hello->("World"); # Hello World

## Currying

Currying applies the arguments from left to right, returning a more specialised function
as it goes until all the arguments are ready, at which point the sub returns its value.

    curry three ($one,$two,$three) {
        return $one + $two * $three
    }

    three(1,2,3)  # normal call - returns 7

    three(1)      # a new subroutine, with $one bound to the number 1
        ->(2,3)   # call the new sub with these arguments

    three(1)->(2)->(3) # You could call the curried sub like this, 
                       # instead of commas (1,2,3)

What about calling with _no_ arguments?  By extension that would return a function exactly
like the original one... but with _no_ arguments prebound (i.e. it's an alias!)

    my $fn = three;   # same as my $fn = \&three;

## Anonymous curries

Just like you can have anonymous subs, you can have anonymous curried subs:

    my $greet = curry ($greeting, $greetee) { ... }

## Composition

Curried subroutines are _composable_.  This means that we can create a new
subroutine that takes the result of the second subroutine as the input of the
first.

Let's say we wanted to expand our greeting to add some punctuation at the end:

    curry append  ($r, $l) { $l . $r }
    curry prepend ($l, $r) { $l . $r }

    my $ciao = append('!') << prepend('Ciao ');
    say $ciao->('Bella'); # Ciao Bella!

How does this work?  Follow the pipeline in the direction of the <<...
First we prepend 'Ciao ' to get 'Ciao Bella', then we pass that to the curry that
appends '!'.  We can also write them in the opposite order, to match evaluation
order, by reversing the operator:

    my $ciao = prepend('Ciao ') >> append('!');
    say $ciao->('Bella'); # Ciao Bella!

Finally, we can create a shell-like pipeline:

    say 'Bella' | prepend('Ciao ') | append('!'); # Ciao Bella!

The overloaded syntax is provided by `Sub::Composable` which is distributed with 
this module as a base class.

## Argument aliasing

When all the arguments are supplied and the function body is executed, the
arguments values are available in both the named parameters and the `@_`
array.  Just as in a normal subroutine call, the elements of `@_` (but
_not_ the named parameters) are aliased to the variables supplied by the
caller, so you can use pass-by-reference semantics.

    curry set ($a, $b) {
      foreach my $arg (@_) { $arg = 1; } # affects the caller
      $a = $b = 2;                       # doesn't affect the caller
    }
    my ($x, $y) = (0, 0);
    set($x)->($y); # $x == 1, $y == 1

## Stack traces

The innermost stack frame has the function name you defined, with all the
accumulated arguments.  Any intermediate stack frames have the same or
similar function names; currently there is a `__curried` suffix, but that
may change in the future.  Currently there is only one intermediate stack
frame, showing just the arguments that were passed in the final call that
reached the required number of arguments, but that may change in the future.
If you supply all the arguments in one call, there are no intermediate stack
frames.

    use Carp 'confess';
    curry func ($a, $b, $c, $d) {
      confess('ERROR MESSAGE');
    }
    sub call {
      func(1)->(2)->(3, 4);
    }
    call();

    ERROR MESSAGE at script.pl line 3
           main::func(1, 2, 3, 4) called at .../Sub/Curried.pm line 202
           main::func__curried(3, 4) called at script.pl line 6
           main::call() called at script.pl line 8

# BUGS

No major bugs currently open.  Please report any bugs via RT or email, or ping
me on IRC (osfameron on irc.perl.org and freenode)

# SEE ALSO

[Devel::Declare](https://metacpan.org/pod/Devel%3A%3ADeclare) provides the magic (yes, there's a teeny bit of code
generation involved, but it's not a global filter, rather a localised
parsing hack).

There are several modules on CPAN that already do currying or partial evaluation:

- [Perl6::Currying](https://metacpan.org/pod/Perl6%3A%3ACurrying) - Filter based module prototyping the Perl 6 system
- [Sub::Curry](https://metacpan.org/pod/Sub%3A%3ACurry) - seems rather complex, with concepts like blackholes and antispices.  Odd.
- [AutoCurry](https://metacpan.org/pod/AutoCurry) - creates a currying variant of all existing subs automatically.  Very odd.
- [Sub::DeferredPartial](https://metacpan.org/pod/Sub%3A%3ADeferredPartial) - partial evaluation with named arguments (as hash keys).  Has some
great debugging hooks (the function is a blessed object which displays what the current
bound keys are).
- [Attribute::Curried](https://metacpan.org/pod/Attribute%3A%3ACurried) - exactly what we want minus the sugar.  (The attribute has
to declare how many arguments it's expecting)

# AUTHOR and LICENSE

    (c)2008-2013 osfameron@cpan.org

## CONTRIBUTORS

- Florian (rafl) Ragwitz
- Paul (prj) Jarc

This module is distributed under the same terms and conditions as Perl itself.

Please submit bugs to RT or shout at me on IRC (osfameron on #london.pm on irc.perl.org)

A git repo is available at [http://github.com/osfameron/Sub--Curried/tree/master](http://github.com/osfameron/Sub--Curried/tree/master)
