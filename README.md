# blog @ bt dot io

Static blog built with [bss](https://github.com/rjpcasalino/bss).

## Local build

Build and preview locally with `bss`:

```sh
bss build -s
```

Generated files are written to `_site/` (configured in `manifest.ini`).

## Deployment

The blog runs as an Nginx container on AWS ECS (`us-west-2`), managed from the `AWWWWNIXOSBOX` repository.

1. Commit and push changes to `master` on `github.com/rjpcasalino/blog_bt`.
2. From `AWWWWNIXOSBOX/ECS`, run:

```sh
./deploy.sh blog
```

`deploy.sh` resolves the latest `master` commit from GitHub, builds an `aarch64-linux` Docker image via Nix (`image-blog.nix`, which pulls `bss` from its flake, runs `bss build`, and bundles `_site/` with Nginx), pushes the `blog-latest` tag to AWS ECR, and forces a new deployment on the `nixos-web-service-medium` ECS service.
