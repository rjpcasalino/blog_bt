---
title: 'LaTeX adventures with nix'
layout: post
---
First, [Exploring Nix Flakes: Build LaTeX Documents Reproducibly](https://flyx.org/nix-flakes-latex/) is an invaluable resource and a must read to get up to speed quickly with LaTeX. No, I will not offer the stylized version of the logo in HTML at the moment. I will soon, probably!

You can probably guess why I've gotten interested in LaTeX suddenly and again. 

It doesn't hurt to have the LaTeX User's Guide and Reference Manual by Lamport on your desk if you're into dead print/trees. I've got the Second Edition which appears to be from 1994 when I was seven years old. Still, it's useful. 

Next, [LaTeX at wikibooks](https://en.wikibooks.org/wiki/LaTeX) makes things a bit easier if you're one who likes to sit at the computer and type away. Reading books is old hat, sorry Lamport.

Truth be told I've had that LaTeX manual sitting around for awhile and had even dogeared some pages but I set it aside a bit ago because I had no real need. Now with the world going mad and jobs being lost all around us I've wanted to update my resume. My old CV was written using [groff](https://www.gnu.org/software/groff/) and it just wasn't up to snuff (the CV or groff). Handling fonts in groff seemed to be a real pain and it's no cake walk with LaTeX but it's far easier. LaTeX has a thriving community whereas groff's is smaller and withering. Don't get me wrong I like groff but LaTeX is the way to go in the waning days of 2023. No ands ifs or buts about it. And seeing as it's soon to be a new year new things are afoot. I'll paste my `flake.nix` here (it's pretty much the one you'd find on flyx.org):

    {
      description = "Get Started with LaTeX";
      inputs = {
        nixpkgs.url = github:NixOS/nixpkgs/nixos-unstable;
        flake-utils.url = github:numtide/flake-utils;
      };
      outputs = { self, nixpkgs, flake-utils }:
        with flake-utils.lib; eachSystem allSystems
          (system:
            let
              pkgs = nixpkgs.legacyPackages.${system};
              tex = pkgs.texlive.combine {
                inherit (pkgs.texlive) scheme-minimal latex-bin latexmk;
              };
            in
            rec {
              packages = {
                document = pkgs.stdenvNoCC.mkDerivation rec {
                  name = "latex-document";
                  src = self;
                  buildInputs = [ pkgs.coreutils pkgs.latex2html tex ];
                  phases = [ "unpackPhase" "buildPhase" "installPhase" ];
                  buildPhase = ''
                    export PATH="${pkgs.lib.makeBinPath buildInputs}";
                    mkdir -p .cache/texmf-var
                    env TEXMFHOME=.cache TEXMFVAR=.cache/texmf-var \
                      SOURCE_DATE_EPOCH=$(date +%s) \
                      OSFONTDIR=${pkgs.commit-mono}/share/fonts \
                      latexmk -interaction=nonstopmode -pdf -lualatex \
                      document.tex; \
                      latex2html -noinfo document.tex
                  '';
                  installPhase = ''
                    mkdir -p $out
                    cp document.pdf $out/
                    cp -r document $out/
                  '';
                };
              };
              packages.default = packages.document;
              devShell = with pkgs; mkShell { packages = [ packages.document.buildInputs ]; shellHook = ''echo -n Hello LaTeX''; };
            });
    }


Running `nix build` will produce a dir called `result` with your document in it.

