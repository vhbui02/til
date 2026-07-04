# Fish Learning

1. `~/.config/fish/conf.d/*.fish`: designed for drop-in, modular configurations. The preferred target for 3rd-party package managers (e.g., Homebrew, Nix, ...) and configuration management tool like Ansible. Evaluate before `config.fish`

2. `~/.config/fish/config.fish`: acts as the final word, overwrite aliases, paths, variables set by modular files in `conf.d/`. Evalaute after `conf.d`. In a perfectly automated environment, `config.fish` should be empty
