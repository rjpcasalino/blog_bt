---
title: '<span class="use-latex-logo"></span> adventures with nix'
layout: post
---

<template id="latex-logo">
  <span class="latexlogofont"><span class="latexlogo">L<span class="latexlogosup">a</span>T<span class="latexlogosub">e</span>X</span></span>
</template>

First, [Exploring Nix Flakes: Build <span class="use-latex-logo"></span> Documents Reproducibly](https://flyx.org/nix-flakes-latex/) is an invaluable resource and a must read to get up to speed quickly with:


<h1>
<span class="use-latex-logo"></span>
</h1>

<hr>

It doesn't hurt to have the <span class="use-latex-logo"></span> User's Guide and Reference Manual by Lamport on your desk if you're into dead print/trees. I've got the Second Edition which appears to be from 1994 when I was seven years old. Still, it's useful. 

Next, [<span class="use-latex-logo"></span> at wikibooks](https://en.wikibooks.org/wiki/LaTeX) makes things a bit easier if you're one who likes to sit at the computer and type away. Reading books is old hat, sorry Lamport.

I've had that <span class="use-latex-logo"></span> manual sitting around for awhile and had even dogeared some pages but I set it aside a bit ago because I had no real need. Now with the world going mad and jobs being lost all around us I've wanted to update my resume. My old CV was written using [groff](https://www.gnu.org/software/groff/) and it just wasn't up to snuff (the CV or groff). Handling fonts in groff seemed to be a real pain and it's no cake walk with LaTeX but it's far easier. <span class="use-latex-logo"></span> has a thriving community whereas groff's is smaller and withering. Don't get me wrong I like groff but <span class="use-latex-logo"></span> is the way to go in the waning days of 2023 (revised in 2026 and mood is same...). No ands ifs or buts about it. And seeing as it's soon to be a new year new things are afoot. I'll paste my `flake.nix` here (it's pretty much the one you'd find on flyx.org):

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
              # re: latexmk see: https://latex.us/support/latexmk/INSTALL
              tex = pkgs.texlive.combine {
                inherit (pkgs.texlive) scheme-full latex-bin latexmk lwarp;
              };
            in
            rec {
              packages = {
                document = pkgs.stdenvNoCC.mkDerivation rec {
                  name = "latex-document";
                  src = self;
                  buildInputs = [ pkgs.coreutils pkgs.poppler-utils tex ];
                  phases = [ "unpackPhase" "buildPhase" "installPhase" ];
                  buildPhase = ''
                    export PATH="${pkgs.lib.makeBinPath buildInputs}";
                    mkdir -p .cache/texmf-var
                    env TEXMFHOME=.cache TEXMFVAR=.cache/texmf-var \
                      SOURCE_DATE_EPOCH=$(date +%s) \
                      OSFONTDIR=${pkgs.commit-mono}/share/fonts \
                      latexmk -interaction=nonstopmode -pdf -lualatex \
                      lamport.tex;
                  '';
                  installPhase = ''
                    mkdir -p $out
                    cp lamport.pdf $out
                  '';
                };
              };
              packages.default = packages.document;
              devShell = with pkgs; mkShell { packages = [ packages.document.buildInputs ]; shellHook = ''SOURCE_DATE_EPOCH=$(date +%s); printf "\t%s\n\t%s\n", "Hello LaTeX", "run latexmk -interaction=nonstopmode -pdf -pvc -lualatex <your_tex_doc.tex>"''; };
            });
    }

Running `nix build` will produce a dir called `result` with your document in it (lamport.pdf in this case). Or you are free to run `nix develop` and have (hoepfully) everything you'd need to run the commands manually. `-pvc` flag means generate a continuous preview which `latexmk` will try to open with `acroread` which has been removed from nixpkgs for security reasons. You can probably hack `latexmk` to open another program instead but maybe another time?

<hr>

Soon we'll explore creating HTML from these tex docs and still use nix so everything is sorta contained.

