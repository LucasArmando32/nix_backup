# Lucas Declarative macOS Configuration

This repository contains the full declarative configuration for my macOS setup, managed by Nix and `nix-darwin`. The goal is a reproducible and version-controlled system.

---

## Directory Layout & Key Files

The configuration is structured into several key directories.

-   hosts/: Contains the main entry point for each specific machine.
    -   `darwin/default.nix`: The main file for this Mac, which assembles all the necessary modules.
-   modules/: Contains reusable pieces of the configuration.
    -   `darwin/`: Modules with settings that *only* apply to macOS.
        - `casks.nix`: A list of graphical `.app` files to install via Homebrew.
        - `packages.nix`: A list of command-line tools for macOS.
        - `system.nix`: macOS-specific system settings (`system.defaults`).
    -   `shared/`: Modules with settings that apply to *all* systems.
        - `home-manager.nix`: The main configuration for user-level programs like Git, Zsh, and Neovim.
-   overlays/: Used for advanced package customizations (optional).

## Bootstrap a New Mac

To set up a new Mac from scratch:

1.  Install Nix: Run the Determinate Systems Nix Installer script.
    ```bash
    curl --proto '=https' --tlsv1.2 -sSf -L [https://install.determinate.systems/nix](https://install.determinate.systems/nix) | sh -s -- install
    ```

2.  Get a temporary Git shell:
    ```bash
    nix-shell -p git
    ```

3.  Clone this repository:
    ```bash
    git clone [https://github.com/your-username/nixos-config.git](https://github.com/your-username/nixos-config.git) ~/nixos-config
    cd ~/nixos-config
    ```

4.  Perform the initial build and activation:
    ```bash
    # First, build the configuration to create the './result' link
    nix build '.#darwinConfigurations.aarch64-darwin.system'

    # Second, run the activation script from the result with sudo
    sudo ./result/sw/bin/darwin-rebuild switch --flake .#aarch64-darwin
    ```
5.  Perform Initial App Setup:
    These are one-time manual steps after the first `switch` has completed.

    * Enable Kubernetes in OrbStack:
        1.  Launch OrbStack from your Applications folder.
        2.  Click the OrbStack icon in the menu bar -> **Settings**.
        3.  Navigate to the **Kubernetes** section and check the **"Enable Kubernetes"** box.

    * Set `kubectl` Context:
        1.  Open a new terminal tab.
        2.  Run the following command to point `kubectl` to your local OrbStack cluster:
            ```bash
            kubectl config use-context orbstack
            ```

    * Configure 1Password:
        1.  Launch the 1Password app from your Applications folder and sign in.
        2.  Open **Settings** (`⌘,`) and navigate to the **Developer** tab.
        3.  Check the box for **"Integrate with 1Password CLI"**.
        4.  Verify the setup in a new terminal by running `op account list`. It should prompt for your fingerprint (Touch ID) and then show your account details.

## Daily Workflow: Applying Local Changes

Any time I edit a `.nix` file to change a setting, add an application, or modify the configuration, I follow this process:

1.  Commit the changes (Best Practice):
    ```bash
    git add .
    git commit -m "feat: add new tool and tweak settings"
    ```

2.  Activate the new configuration:
    *(This alias is defined in `modules/shared/home-manager.nix`)*
    ```bash
    nix-switch
    ```
    *(Or, the full command: `sudo darwin-rebuild switch --flake .#aarch64-darwin`)*

## Weekly Workflow: Updating Dependencies

To update all underlying software (Nixpkgs, Home Manager, etc.) to their latest versions:

1.  Update the flake's lock file:
    ```bash
    nix flake update
    ```

2.  Apply the updates to the system:
    ```bash
    # First, commit the new flake.lock
    git add flake.lock
    git commit -m "chore: update flakes"

    # Then, switch to the new configuration
    nix-switch
    ```

## System Maintenance: Garbage Collection

Nix stores every version of every package. Over time, this can use a lot of disk space. To clean up old, unused versions (generations) of your system:

1.  List current system generations:
    ```bash
    darwin-rebuild --list-generations
    ```

2.  Delete all old (not currently booted) generations:

    ```bash
    nix-collect-garbage -d
    ```
