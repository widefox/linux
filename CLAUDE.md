# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This is the Linux kernel source tree. The Linux kernel is a monolithic Unix-like operating system kernel written primarily in C, with some assembly and (increasingly) Rust.

## Build System

The kernel uses Kbuild, a custom build system based on GNU Make.

### Basic Build Commands

```bash
# Configure the kernel (required before first build)
make defconfig              # Use default configuration for current architecture
make menuconfig             # Interactive configuration (ncurses)
make oldconfig              # Update existing .config for new kernel version

# Build
make                        # Build kernel image and modules (use -j$(nproc) for parallel build)
make vmlinux                # Build just the kernel image
make modules                # Build just the modules
make dir/file.o             # Build specific object file
make dir/file.ko            # Build specific kernel module

# Architecture-specific builds
make ARCH=x86_64 defconfig  # Configure for specific architecture
make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- vmlinux

# Verbose builds
make V=1                    # Show full commands during build

# Clean
make clean                  # Remove most generated files
make mrproper               # Remove all generated files including config
```

### Static Analysis and Code Quality

```bash
# Run checkpatch on changes (CRITICAL before submitting patches)
scripts/checkpatch.pl --file path/to/file.c
scripts/checkpatch.pl mypatch.patch

# Other static analysis
make coccicheck             # Run Coccinelle semantic patches
make clang-analyzer         # Run clang static analyzer
make clang-tidy             # Run clang-tidy

# Generate tags for navigation
make tags                   # ctags
make cscope                 # cscope
make gtags                  # GNU GLOBAL
```

### Testing

```bash
# Kernel self-tests (selftests)
cd tools/testing/selftests
make                        # Build all tests
make run_tests              # Run all tests
cd <subsystem>; make        # Build/run specific subsystem tests

# Boot testing requires actual kernel boot or QEMU
```

## Code Style

**CRITICAL**: The kernel has strict coding style rules in Documentation/process/coding-style.rst

Key points:
- Tabs are 8 characters (hard tabs, not spaces)
- Lines should be under 80 characters where possible
- K&R brace style for functions and control structures
- Case labels align with switch statement (not double-indented)
- Always run `scripts/checkpatch.pl` before submitting patches

## Directory Structure

```
arch/           - Architecture-specific code (x86, arm64, riscv, etc.)
block/          - Block layer (I/O scheduling, bio handling)
crypto/         - Cryptographic API and implementations
drivers/        - Device drivers (largest directory)
fs/             - Filesystems (ext4, btrfs, nfs, vfs core, etc.)
include/        - Kernel headers
  include/linux/    - Core kernel headers
  include/uapi/     - User-space API headers
init/           - Kernel initialization code
io_uring/       - io_uring async I/O subsystem
ipc/            - Inter-process communication
kernel/         - Core kernel subsystems (scheduler, time, locking, etc.)
lib/            - Library routines (string functions, data structures)
mm/             - Memory management
net/            - Network stack
rust/           - Rust infrastructure and bindings
samples/        - Sample code and documentation
scripts/        - Build scripts and development tools
security/       - Security modules (SELinux, AppArmor, etc.)
tools/          - Userspace tools for kernel development and testing
Documentation/  - Kernel documentation
```

## Subsystem Organization

The kernel is organized into subsystems, each typically with:
- Core code in a top-level directory (e.g., mm/, fs/, net/)
- Headers in include/linux/ or include/subsystem/
- Architecture-specific code in arch/*/
- Drivers in drivers/subsystem/
- Documentation in Documentation/subsystem/

Each subsystem has designated maintainers listed in the MAINTAINERS file.

## Development Workflow

### Understanding the Patch-Based Workflow

The kernel uses a decentralized, email-based patch workflow:
1. Changes are submitted as patch series via email
2. Patches must follow strict format and style guidelines
3. Each patch should be self-contained and atomic
4. Commit messages must follow Documentation/process/submitting-patches.rst

### Key Development Principles

- **One logical change per patch**: Each commit should do one thing
- **Build at every step**: Every patch in a series must build and boot successfully
- **Subsystem trees**: Most development happens in subsystem-specific git trees, not mainline
- **Maintainer hierarchy**: Changes flow through subsystem maintainers to Linus

### Important Scripts

```bash
scripts/checkpatch.pl           # Style and format checker (ALWAYS RUN THIS)
scripts/get_maintainer.pl       # Find maintainers for a file/patch
scripts/decode_stacktrace.sh    # Decode kernel stack traces
scripts/Lindent                 # Reformat code to kernel style
```

## Configuration System (Kconfig)

The kernel uses Kconfig for configuration. Options are defined in Kconfig files throughout the tree and selected during configuration (menuconfig, defconfig, etc.). The resulting .config file controls what gets built.

## Module Development

```bash
# Build external module against kernel tree
make M=/path/to/module modules

# Install modules
make modules_install            # Install to /lib/modules/$(uname -r)/

# Module info
modinfo module.ko               # Show module information
```

## Rust Support

The kernel is gradually adding Rust support:
- Rust code goes in rust/ for infrastructure, or alongside C code in subsystems
- Enable with CONFIG_RUST
- Requires specific Rust toolchain version
- Check requirements: scripts/rust_is_available.sh

## Key Documentation

Essential reading in Documentation/process/:
- submitting-patches.rst - How to submit patches
- coding-style.rst - Kernel coding style
- 4.Coding.rst - Coding guidelines and pitfalls
- changes.rst - Minimum tool versions required

## Common Pitfalls

1. **Do NOT submit coding style cleanup patches for existing code** - these are considered noise
2. **Do NOT break existing userspace APIs** - kernel-to-userspace interfaces are stable
3. **Always test on real hardware when possible** - emulation doesn't catch everything
4. **Follow subsystem conventions** - different subsystems may have additional rules (see Documentation/process/maintainer-handbooks.rst)
5. **Build for multiple architectures** - changes often have arch-specific implications
