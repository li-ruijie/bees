# bees

Automated builds of [Zygo/bees](https://github.com/Zygo/bees) for x86_64 and aarch64.

## About bees

bees (Best-Effort Extent-Same) is a block-oriented userspace deduplication agent for btrfs filesystems.

## Downloads

Pre-built binaries are available in [Releases](../../releases).

| Architecture | Static | Dynamic |
|--------------|--------|---------|
| x86_64 (amd64) | `bees-<version>-x86_64-static.tar.gz` | `bees-<version>-x86_64-dynamic.tar.gz` |
| aarch64 (arm64) | `bees-<version>-aarch64-static.tar.gz` | `bees-<version>-aarch64-dynamic.tar.gz` |

### Static vs Dynamic

- **Static**: Self-contained, no runtime dependencies. Larger binary.
- **Dynamic**: Smaller binary, requires compatible system libraries. Each release includes `LIBRARIES.txt` documenting linked library versions.

## Installation

```bash
# Download and extract
tar -xzf bees-<version>-<arch>-<linking>.tar.gz

# Install binary
sudo install -m 755 bees /usr/local/bin/

# Optional: install systemd service
sudo install -m 644 beesd@.service /etc/systemd/system/
sudo install -m 755 beesd /usr/local/sbin/
```

## Build Triggers

- **Manual**: Dispatch workflow with specific tag
- **Scheduled**: Weekly check for new upstream releases

## Build Environment

Builds use Debian Bookworm in Docker containers. Dynamic builds document library versions in release notes.

## Upstream

- Repository: https://github.com/Zygo/bees
- Documentation: https://github.com/Zygo/bees/tree/master/docs
- License: GPL-3.0
