---
title: 'LaTeX adventures with nix'
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

I've had that <span class="use-latex-logo"></span> manual sitting around for awhile and had even dogeared some pages but I set it aside a bit ago because I had no real need. Now with the world going mad and jobs being lost all around us I've wanted to update my resume. My old CV was written using [groff](https://www.gnu.org/software/groff/) and it just wasn't up to snuff (the CV or groff). Handling fonts in groff seemed to be a real pain and it's no cake walk with <span class="use-latex-logo"></span> but it's far easier. <span class="use-latex-logo"></span> has a thriving community whereas groff's is smaller and withering. Don't get me wrong I like groff but <span class="use-latex-logo"></span> is the way to go in the waning days of 2023 (revised in 2026 and mood is same...). No ands ifs or buts about it. And seeing as it's soon to be a new year new things are afoot.

    {
      description = "Get Started with LaTeX";

      inputs = {
        nixpkgs.url = "github:NixOS/nixpkgs/nixos-unstable";
      };

      outputs = { self, nixpkgs }:
        let
          # Define only Linux and macOS systems natively
          systems = [ "x86_64-linux" "aarch64-linux" "x86_64-darwin" "aarch64-darwin" ];
          forAllSystems = nixpkgs.lib.genAttrs systems;
        in
        {
          packages = forAllSystems (system:
            let
              pkgs = nixpkgs.legacyPackages.${system};
              
              # Size Optimization: Removed `scheme-full`. `texliveSmall` already provides a base environment.
              tex = pkgs.texliveSmall.withPackages (ps: with ps; [
                latex-bin latexmk lwarp enumitem titlesec fontawesome5
              ]);

              # Speed Optimization: Replaced the manual copy loop with `symlinkJoin`
              fonts = pkgs.symlinkJoin {
                name = "resume-fonts";
                paths = with pkgs; [
                  inter montserrat commit-mono geist-font plus-jakarta-sans
                  ibm-plex jetbrains-mono fira-code roboto lato
                ];
              };
            in
            {
              default = pkgs.stdenvNoCC.mkDerivation {
                name = "resume-document";
                src = self;
                buildInputs = [ pkgs.coreutils pkgs.poppler-utils tex fonts ];
                
                # Expose fonts so the devShell can access the same derivation
                passthru = { inherit fonts; };
                
                phases = [ "unpackPhase" "buildPhase" "installPhase" ];
                buildPhase = ''
                  export PATH="${pkgs.lib.makeBinPath [ pkgs.coreutils pkgs.poppler-utils tex ]}";
                  export OSFONTDIR="${fonts}/share/fonts";
                  export TEXMFHOME="$(pwd)/.cache"
                  export TEXMFVAR="$(pwd)/.cache/texmf-var"
                  mkdir -p "$TEXMFVAR"
                  
                  luaotfload-tool --update
                  latexmk -interaction=nonstopmode -pdf -lualatex resume.tex
                '';
                installPhase = ''
                  mkdir -p $out
                  cp resume.pdf $out/
                '';
              };
            }
          );

          devShells = forAllSystems (system:
            let
              pkgs = nixpkgs.legacyPackages.${system};
              document = self.packages.${system}.default;
            in
            {
              default = pkgs.mkShell {
                inputsFrom = [ document ];
                shellHook = ''
                  export OSFONTDIR="${document.passthru.fonts}/share/fonts"
                  export TEXMFHOME="$(pwd)/.cache"
                  export TEXMFVAR="$(pwd)/.cache/texmf-var"
                  mkdir -p "$TEXMFVAR"
                  
                  # Speed Optimization: Only run font DB refresh if the cache is actually missing
                  if [ ! -f "$TEXMFVAR/luatex-cache/generic/names/luaotfload-names.luc" ]; then
                    echo "Updating LuaLaTeX font cache in background..."
                    luaotfload-tool --update -q &
                  fi

                  export SOURCE_DATE_EPOCH=$(date +%s);
                  printf "\n\t%s\n\t%s\n\n" "Hello LaTeX" "run: latexmk -interaction=nonstopmode -pdf -pvc -lualatex resume.tex"
                '';
              };
            }
          );
        };
    }

Running `nix build` will produce a dir called `result` with your document in it (lamport.pdf in this case). Or you are free to run `nix develop` and have (hoepfully) everything you'd need to run the commands manually. `-pvc` flag means generate a continuous preview which `latexmk` will try to open with `acroread` which has been removed from nixpkgs for security reasons. You can probably hack `latexmk` to open another program instead but maybe another time? Instead, just create a `~/.latexmkrc` file and set `$pdf_previewer = 'zathura'`. Soon we'll explore creating HTML from these tex docs and still use nix so everything is sorta contained. Notice `lwarp` in the flake above; it is the program we will use to create an HTML file from our tex doc.

Maybe...

