# Aspen development

## Create your local dev environment

To enable cross-platform development, Aspen recommends:

1. Install [Devbox](https://www.jetify.com/docs/devbox/installing_devbox/)
  - Requires the Nix package manager
2. Install [Docker Desktop](https://www.docker.com/products/docker-desktop/)
  - Note: for Mac users on Apple Silicon
    - Once installed, run Docker Desktop, then open `Settings` > `General` 
    - Turn on `Rosetta for x86/amd64 emulation for Apple Silicon`
3. Ensure that `devbox version` and `docker version` work in your terminal

With this setup you get:

- nix (that package manager)
  - for reproducible, declarative builds in your local environment
- Devbox
  - to define Aspen's dependencies
  - to create an isolated shell environment
- [`process-compose`](https://github.com/F1bonacc1/process-compose)
  - a [TUI](https://en.wikipedia.org/wiki/Text-based_user_interface) for orchestrating Aspen's services

## Getting started

In your terminal, navigate to your local copy of Aspen and run the following command to check that everything's working:

```bash
# enter your isolated dev environment
devbox shell
# verify dependencies
bun --version
# Launch `process-compose` TUI and start all services
devbox services up
```

Aspen uses [Bun](https://bun.sh/docs) and the [Bun Shell](https://bun.sh/docs/runtime/shell) to write cross-platform scripts.