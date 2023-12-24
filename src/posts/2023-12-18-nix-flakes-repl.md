---
title: 'nix flakes repl'
layout: post
---

First, see: [https://github.com/NixOS/nix/pull/9043](https://github.com/NixOS/nix/pull/9043).

Second, see: [https://nixos.org/manual/nix/stable/command-ref/new-cli/nix3-repl.html](https://nixos.org/manual/nix/stable/command-ref/new-cli/nix3-repl.html).

This post is an attempt for myself to document certain aspects of the repl and nix itself. I hope you find it useful!

If you're running any thing older than `nix (Nix) 2.19.2` you'll have to enable `repl-flake`. On NixOS you can pass in `repl-flake`:

    nix = {
       extraOptions = ''
         experimental-features = nix-command flakes repl-flake
       '';
    };

OK, with that done you can run:

    you ~ % nix repl nixpkgs

And see:

    Welcome to Nix 2.19.2. Type :? for help.

    Loading installable 'flake:nixpkgs#'...
    Added 5 variables.
    nix-repl>

Running `nix repl` without the `nixpkgs` bit will leave you with `builtins` only.

Tab completion is very handy. You can use `:doc` to learn:
    
    nix-repl> :doc __add 
    Synopsis: builtins.add e1 e2

        Return the sum of the numbers e1 and e2.

Sometimes the value will have no documentation:

    nix-repl> :doc htmlDocs.nixpkgsManual
    error: value does not have documentation

Nevertheless, you can scope things out:

    nix-repl> :e htmlDocs.nixpkgsManual
    ...in the $EDITOR
     94 in pkgs.stdenv.mkDerivation {
     95   name = "nixpkgs-manual";
     96
     97   nativeBuildInputs = with pkgs; [
     98     nixos-render-docs
     99   ];

Remember that `:?` prints the commands available to you.

Something I just [learned](https://discourse.nixos.org/t/korora-a-tiny-fast-type-system-for-nix-in-nix/36900/5) regarding the repl and importing:

    nix-repl> korora = import (builtins.fetchGit "https://github.com/adisbladis/korora.git") { inherit lib; }

    nix-repl> t = korora.string

    nix-repl> t.check 1234
    error:
           ... while calling the 'throw' builtin

             at /nix/store/n5jj2abpas1ihg5l26cysyl8rak7pa21-source/default.nix:101:50:

              100|       inherit name verify;
              101|       check = v: if verify v == null then v else throw (verify v);
                 |                                                  ^
              102|     };

           error: Expected type 'string' but value '1234' is of type 'int'

And you can do things that don't really make too much sense:

    nix-repl> pkgs = legacyPackages.x86_64-linux 

    nix-repl> drv = pkgs.runCommand "fortune-kind" { buildInputs = [ pkgs.fortune-kind ]; } "fortune-kind; fortune-kind > $out"

    nix-repl> :b drv

    This derivation produced the following outputs:
      out -> /nix/store/l11i1nlmkcdvsyi56m9fp6hjn1yyvjvq-fortune-kind

    nix-repl> :log drv
    Pete:   Waiter, this meat is bad.
    Waiter: Who told you?
    Pete:   A little swallow.

And even sillier things:

    nix-repl> drv = pkgs.writeShellApplication { name = "show rjpc.net";  runtimeInputs = [pkgs.curl pkgs.w3m]; text = ''curl -s 'https://rjpc.net' | w3m -dump -T text/html''; }

    nix-repl> :b drv

    This derivation produced the following outputs:
      out -> /nix/store/7l4aggxkh6lyxg1axy486rr44kpmvgh1-show-rjpc.net

Jump into a shell and:

    rjpc ~ % bash /nix/store/7l4aggxkh6lyxg1axy486rr44kpmvgh1-show-rjpc.net/bin/show\ rjpc.net

    ~/rjpc

        Hi, I'm just another "Netizen Smith" cruising the metaverse for lovers and
        friends

`WirteShellApplication` isn't too silly actually:

> This can be used to easily produce a shell script that has some dependencies (runtimeInputs). It automatically sets the PATH of the script to contain all of the listed inputs, sets some sanity shellopts (errexit, nounset, pipefail), and checks the resulting script with shellcheck.

There's more to read using `:e` -

    310   /*
    311     Similar to writeShellScriptBin and writeScriptBin.
    312     Writes an executable Shell script to /nix/store/<store path>/bin/<name> and
    313     checks its syntax with shellcheck and the shell's -n option.
    314     Individual checks can be foregone by putting them in the excludeShellChecks
    315     list, e.g. [ "SC2016" ].
    316     Automatically includes sane set of shellopts (errexit, nounset, pipefail)
    317     and handles creation of PATH based on runtimeInputs
    318
    319     Note that the checkPhase uses stdenv.shell for the test run of the script,
    320     while the generated shebang uses runtimeShell. If, for whatever reason,
    321     those were to mismatch you might lose fidelity in the default checks. */

Remember that nix is lazy but `:p` helps there:

    nix-repl> :p lib.lists.toposort (a: b: a < b) [ "Mary" "Sam" "Ryan" "Mike" "Anna" "Vera" "Bill" "Rick" ] 
    { result = [ "Anna" "Bill" "Mary" "Mike" "Rick" "Ryan" "Sam" "Vera" ]; }

Working with nix and the repl is exhausting so it's a good idea to step outside now and then.

Some random things:

    nix-repl> silly = x: {y ? "", ... }@args: z: x + y + args.a + args.b + " " + builtins.toString z

    nix-repl> silly "hello" {y = " world"; a = " uh"; b = " is this thing on?";}  10
    "hello world uh is this thing on? 10"

And some stuff in my cribsheet that probably should be here as well:

    nix hash to-sri --type sha256 $(nix-prefetch-url --unpack https://github.com/r-c-f/waynergy/archive/refs/tags/v0.0.16.tar.gz)

You could read the man page but it might just leave you...unhappy. In "nix classic" you could use:

    nix-hash --to-sri

This is very useful if you ever want to update a package in nixpkgs or package one yourself. There are probably other ways to get the sha256 but I don't know of them.
