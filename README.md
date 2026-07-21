# system-update

A safety-focused system update utility for CachyOS and Arch Linux that performs comprehensive audits, discovers available updates, and applies them with minimal user intervention.

## Features

- **Safety-first approach**: Performs extensive pre-update audits before any changes
- **Multi-package management**: Handles official repositories, AUR, and Flatpak updates
- **Automatic dependency management**: Removes orphaned packages and cleans package cache
- **Kernel awareness**: Detects when a reboot is required after kernel updates
- **Arch Linux news integration**: Integrates with Informant to display unread news
- **Color output**: Clear visual feedback with color-coded status messages
- **Comprehensive logging**: Tracks warnings, errors, and transaction statuses
- **Read-only modes**: Audit and plan modes for safe inspection

## Limitations

This script intentionally does NOT handle:

- Firmware or BIOS/UEFI updates
- Docker, pip, npm, cargo, gem, or ComfyUI packages
- Filesystem scrubbing or automatic database repair
- Snapshot creation, removal, or restoration
- Bootloader modifications or automatic service restarts
- Automatic resolution of AUR conflicts
- Automatic merging of pacnew/pacsave/pacorig files
- Automatic rollback functionality

## Requirements

### Core Dependencies
- `bash`
- `pacman`
- `sudo`
- `yay` (for AUR support)
- `flock`
- `awk`, `grep`, `sed`
- `findmnt`, `df`
- `getent`
- `systemctl`

### Optional Dependencies
- `pacman-conf` (repository parsing)
- `checkupdates` (update discovery, from `pacman-contrib`)
- `paccache` (cache cleaning, from `pacman-contrib`)
- `pacdiff` (config file discovery)
- `flatpak` (Flatpak support)
- `informant` (Arch news integration)
- `snapper` (snapshot checks)
- `shellcheck` (development only)

## Installation

### Manual Installation

1. Download the script:
```bash
git clone https://github.com/Cerviche/Arch-CachyOS_System-Update.git
```

2. Install with:
```bash
❯ sudo install -m 755 ~/Arch-CachyOS_System-Update/system-update /usr/local/bin/system-update
```

### Arch Linux Package (Future)

A PKGBUILD will be available in the AUR soon.

## Usage

### Basic Usage

```bash
# Full update with safety checks
system-update

# Read-only audit
system-update --audit

# Show update plan without applying
system-update --plan

# Display help
system-update --help

# Show version
system-update --version
```

### Modes Explained

| Mode | Description |
|------|-------------|
| **No option** | Full update mode: audits, discovers updates, applies official repo updates, AUR updates, Flatpak updates, removes orphans, and cleans cache |
| **--audit** | Read-only health check: verifies dependencies, repositories, storage, and system services |
| **--plan** | Shows what would be updated without making any changes |
| **--help** | Displays usage information |
| **--version** | Shows script version |

## How It Works

### 1. Pre-Update Audit

The script performs extensive safety checks:

- **Dependency verification**: Ensures all required commands are available
- **Repository integrity**: Verifies core repositories (core, extra, CachyOS) are enabled and accessible
- **Package database**: Checks database consistency with `pacman -Dk`
- **Network connectivity**: Verifies DNS resolution for archlinux.org
- **Storage availability**: Checks free space on root and separate /boot partitions
- **Kernel tracking**: Identifies running kernel and its package owner
- **Configuration files**: Finds pending pacnew/pacsave/pacorig files
- **System services**: Identifies failed systemd units
- **Snapper integration**: Checks snapshot configuration if installed

### 2. Update Discovery

- **Repository updates**: Uses `checkupdates` to find available official package updates
- **AUR updates**: Uses `yay -Qua` to discover AUR package updates
- **Flatpak updates**: Checks both system and user Flatpak installations

### 3. Arch News Integration

If Informant is installed:

- Checks for unread Arch Linux news
- Temporarily disables Informant hooks to allow unattended updates
- Prompts to read news after updates complete

### 4. Update Application

1. **Official repositories**: `pacman -Syu --noconfirm`
2. **AUR packages**: `yay -Sua` with non-interactive flags
3. **Flatpak**: System and user updates with unused runtime removal

### 5. Post-Update Maintenance

- **Orphan removal**: Interactive removal of unneeded packages
- **Cache cleaning**: Keeps newest 3 versions, removes uninstalled packages
- **Health rescan**: Rechecks configuration files and system services
- **Reboot check**: Determines if a reboot is required

### 6. Summary

Provides a comprehensive summary of all actions taken, warnings, and errors.

## Configuration

### Environment Variables

- `AUR_CHECK_DEVEL`: Set to "true" to check development AUR packages (default: false)

### Modification

You can modify the script to change these constants:

| Constant | Description | Default |
|----------|-------------|---------|
| `ROOT_BLOCK_MIB` | Minimum free space on root (MiB) | 1024 |
| `BOOT_WARN_MIB` | Warning threshold for /boot (MiB) | 200 |
| `BOOT_BLOCK_MIB` | Blocking threshold for /boot (MiB) | 100 |

## Safety Features

- **Process locking**: Prevents concurrent script executions
- **Sudo keepalive**: Maintains credentials without timeout
- **Signal handling**: Proper cleanup on interrupt/termination
- **Blocking checks**: Stops updates if critical issues are found
- **Temporary files**: Secure temporary directory with automatic cleanup
- **No `set -e`**: Explicit error handling prevents unexpected exits

## Output Examples

### Audit Mode Output

```
[INFO] system-update 3.0.1
[INFO] Host: myhost
[INFO] Mode: audit

------------------------------------------------------------------
Dependencies
------------------------------------------------------------------
[OK] All required commands are available.

------------------------------------------------------------------
Repository integrity
------------------------------------------------------------------
[OK] /etc/pacman.conf exists and is readable.
[OK] Enabled repositories parsed with pacman-conf.
[INFO] Enabled repositories:
    core
    extra
    community
    cachyos
[OK] Required repository is enabled: core
[OK] Required repository is enabled: extra
[OK] At least one CachyOS repository is enabled.
[OK] Critical package resolves through enabled repositories: bash
[OK] Critical package resolves through enabled repositories: pacman
...
```

### Update Mode Summary

```
------------------------------------------------------------------
Summary
------------------------------------------------------------------
Repository transaction                        completed
Repository updates initially detected        15
AUR transaction                              completed
AUR updates initially detected              3
System Flatpak transaction                   not applicable; no refs
User Flatpak transaction                     completed
Orphans found                               2
Orphans removed                             2
Orphan removal declined                     no
Configuration files pending                 0
High-priority configuration files           0
Relevant failed units                       0
Ignored failed units                        1
Unread Arch news                            0
News reading declined                       no
Reboot required                             yes
Warnings                                    1
Errors                                      0
Duration                                    47s

Warnings:
    - The running kernel package changed from 6.6.2-zen1-1-zen to 6.7.0-zen1-1-zen
```

## Troubleshooting

### Common Issues

**"Pacman lock exists"**
- Another pacman process is running
- Wait for it to complete or remove the lock file if safe

**"Sudo authentication failed"**
- Ensure you have sudo access
- The script requires passwordless sudo or interactive authentication

**"Required command is missing: yay"**
- Install yay: `sudo pacman -S yay`

**"checkupdates is unavailable"**
- Install pacman-contrib: `sudo pacman -S pacman-contrib`

### Logging

The script creates a temporary directory for all output files (automatically cleaned up):
```
/tmp/system-update.XXXXXXXX/
```

## Contributing

Contributions are welcome! Please follow these guidelines:

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Run shellcheck on any bash changes
5. Submit a pull request

### Development Requirements

- `shellcheck` for linting
- Bash 4.0+ for associative arrays

## License

This project is licensed under the MIT License - see the LICENSE file for details.

## Acknowledgments

- Inspired by the Arch Linux philosophy of simplicity and transparency
- Built on the work of the Arch Linux, CachyOS, and open-source communities
- Uses `yay` for AUR support, `checkupdates` for safe update discovery

## Security Considerations

- The script requires sudo access for package management operations
- All commands are executed with explicit `sudo` calls for transparency
- Temporary files are created securely with `mktemp`
- No sensitive data is stored or transmitted
- Network checks are limited to DNS resolution

## Disclaimer

This script makes significant changes to your system. While it includes extensive safety checks, no guarantee is provided. Always backup important data before running system updates.
