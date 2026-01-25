# Installing and verifying `csvkit` CLI tool

`csvkit` v2 is best installed via `pipx` to avoid polluting system Python. This guide covers installation on major platforms.

## Prerequisites

- Python 3.10 or later
- `pipx` (recommended) or a Python virtual environment

## Installation by platform

### macOS

```shell
# Install pipx if not already installed
brew install pipx
pipx ensurepath

# Install csvkit
pipx install csvkit

# Restart your terminal or run:
source ~/.zshrc
```

**Alternative (Homebrew):**

```shell
brew install csvkit
```

### Ubuntu / Debian

```shell
# Install pipx
sudo apt update
sudo apt install pipx
pipx ensurepath

# Install csvkit
pipx install csvkit

# Restart your terminal or run:
source ~/.bashrc
```

### RHEL / CentOS / Fedora

```shell
# Install pipx
sudo dnf install pipx
pipx ensurepath

# Install csvkit
pipx install csvkit

# Restart your terminal or run:
source ~/.bashrc
```

### Windows (PowerShell)

```powershell
# Install pipx (requires Python already installed)
python -m pip install --user pipx
python -m pipx ensurepath

# Restart PowerShell, then:
pipx install csvkit
```

## Verification

After installation, verify `csvkit` is accessible and reports version 2.x:

```shell
csvcut --version
```

Expected output:

```
csvcut 2.0.1
```

## Troubleshooting

### Command not found

If `csvcut` is not found after installation:

1. **PATH not updated**: Run `pipx ensurepath` again and restart your terminal.

2. **Shell config not sourced**: Manually source your shell config:
   ```shell
   source ~/.bashrc   # Linux
   source ~/.zshrc    # macOS (zsh)
   ```

3. **Check pipx bin directory**: Verify the pipx bin path is in your PATH:
   ```shell
   echo $PATH | tr ':' '\n' | grep pipx
   ```
   
   If missing, add it manually:
   ```shell
   export PATH="$HOME/.local/bin:$PATH"
   ```

### Wrong version (1.x instead of 2.x)

If you see version 1.x, you may have an older system-installed version taking precedence:

```shell
# Check which csvcut is being used
which csvcut

# If it's not from pipx, uninstall the system version or adjust PATH priority
```

### Permission errors on Linux

Avoid `sudo pip install`. If you encounter permission issues:

```shell
# Ensure pipx is installed for your user, not system-wide
python3 -m pip install --user pipx
```

## Summary table

| Platform             | Recommended            | Alternative            | Discouraged        |
|----------------------|------------------------|------------------------|--------------------|
| macOS                | `pipx install csvkit`  | `brew install csvkit`  | system `pip`       |
| Ubuntu/Debian        | `pipx install csvkit`  | Python virtualenv      | `sudo pip install` |
| RHEL/CentOS/Fedora   | `pipx install csvkit`  | Python virtualenv      | system `pip`       |
| Windows              | `pipx install csvkit`  | Python virtualenv      | global `pip`       |
