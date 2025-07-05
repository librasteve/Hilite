# Hilite

Hilite is an HTML code highlighter.

Hilite is provided as a plugin supporting both the Air::Plugin:: web authoring and Rakudoc::Plugin::HTML:: (v2) documentation models.

Hilite employs the Rainbow raku highlighter (auth<patrickbr>) for raku and rakudoc and highlighter.js for other languages.

## SYNOPSIS


If you just want to get a web page that highlights raku, the easiest way to consume the Hilite module is with Air::Plugin::Hilite:
```raku
use Air::Functional :BASE;
use Air::Base;
use Air::Plugin::Hilite;

sub SITE is export {
    site :register[Air::Plugin::Hilite.new],
        page
            main
                hilite q:to/END/;
                    use Air::Functional :BASE;
                    use Air::Base;
                    use Air::Plugin::Hilite;

                    sub SITE is export {
                        site :register[Hilite.new],
                            index
                                main
                                    hilite 'say "yo, baby!"';
                    }
                END
}
```

## DESCRIPTION

If you want to consume Hilite in your own code, the following code stubs a Template and a Receptacle (a fancy word for Socket) into which Hilite plugs.

The Template / Receptacle model originated as a Rakudoc Process and is now being adopted by the raku Air module as a wider usage of the Rakudoc::Plugin::HTML:: approach. The medium term aim is to make it possible for any raku web library to reuse Air::Plugin:: / Rakudoc::Plugin::HTML:: modules interchageably. And for Air::Plugin:: modules to work with Rakudoc v2.

```raku
role Air::Plugin::Hilite does Tag {
    use Hilite;   #ie Hilite.rakumod

    #| code to be highlited
    has Str $.code;
    #| language (from highlight.js + haskell + raku + rakudoc)
    has $.lang = 'raku';

    #! make a stub to consume
    my class Template {
        my class Globals {
            has %.helper;

            method escape {
                use HTML::Escape;
                &escape-html;
            }
        }

        has $.globals = Globals.new;

        method warnings {
            $!globals.helper<add-to-warnings>;
        }
    }
    my class Receptacle {
        has %.data;

        method add-templates(*@a, *%h) {}
        method add-data($ns, %config) {
            %!data{$ns} = %config;
        }
    }

    has $!tmpl = Template.new;
    has $!rctl = Receptacle.new;
    has $!hltr = Hilite.new: :css-lib<pico>;

    #| script, styles from Hilite.rakumod
    has @!js-links;     #list of script src urls
    has $!script;
    has @!css-links;    #list of link href urls
    has $.scss;

    submethod TWEAK {
        $!hltr.enable: $!rctl;

        @!js-links   = $!rctl.data<hilite><js-link>.map: *[0];
        @!js-links  .= map: *.split('=')[1];     #pick the url
        @!js-links  .= map: *.substr(1,*-1);     #rm quote marks
        $!script     = $!rctl.data<hilite><js>[0][0];

        @!css-links  = $!rctl.data<hilite><css-link-dark>.map: *[0];
        @!css-links .= map: *.split('=')[1];     #pick the url
        @!css-links .= map: *.substr(1,*-1);     #rm quote marks
        $!scss       = $!rctl.data<hilite><scss>[0][0];
    }

    #| .new positional takes Str $code
    multi method new(Str $code, *%h) {
        self.bless:  :$code, |%h;
    }

    method warnings { note $!tmpl.warnings }

    multi method HTML {
        my %prm = :contents($!code), :$!lang :label;
        $!hltr.templates<code>(%prm, $!tmpl);
    }

    method JS-LINKS  { @!js-links }
    method SCRIPT    { $!script }

    method CSS-LINKS { @!css-links }
    method SCSS      { $!scss }
}
```

Hilite is a direct descendant from, and built as a drop-in replacement for, [Rakudoc::Plugin::HTML::Hilite](https://github.com/finanalyst/rakuast-rakudoc-render/blob/177abccc3215518bb16d689edbdd4854f8eb3d9a/lib/RakuDoc/Plugin/HTML/Hilite.rakumod)

Therefore it starts at v0.2.0.

With the following changes:

- does not use Rakudoc::Render
    - ie. drops $rdp param type check from method enable
- does not use experimental :rakuast;
- rm fontawesome dependency
- new attr :css-lib = bulma (default) | pico
- https://picocss.com support (light / dark tuned)
- rm Deparse (ie. Rainbow only)
- add HTMX update trigger
- add css-link-dark

AUTHOR
======

author<<Richard Hainsworth, aka finanalyst\nElizabeth Mattijsen, aka lizmat\nSteve Roe, aka librasteve\n>>,

COPYRIGHT AND LICENSE
=====================

Copyright 2025 the authors

This library is free software; you can redistribute it and/or modify it under the Artistic License 2.0.

