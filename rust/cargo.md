# Rust Cargo & CLI Tools Cheat Sheet

## Project Management

### Creating Projects
```bash
cargo new my_project           # Create new binary project
cargo new my_lib --lib         # Create new library project
cargo init                     # Initialize project in current directory
cargo init --lib               # Initialize library in current directory
```

### Building & Running
```bash
cargo build                    # Build project (debug mode)
cargo build --release          # Build project (optimized/release mode)
cargo run                      # Build and run binary
cargo run --bin my_bin         # Run specific binary
cargo run -- arg1 arg2        # Pass arguments to your program
cargo check                    # Check code without building (faster)
```

## Testing

### Running Tests
```bash
cargo test                     # Run all tests
cargo test test_name           # Run specific test
cargo test --lib              # Run library tests only
cargo test --bin my_bin       # Run binary tests only
cargo test --release          # Run tests in release mode
cargo test -- --nocapture     # Show println! output in tests
cargo test -- --test-threads=1 # Run tests sequentially
```

### Documentation Tests
```bash
cargo test --doc              # Run documentation tests
cargo doc                     # Generate documentation
cargo doc --open              # Generate and open documentation
```

## Dependencies

### Managing Dependencies
```bash
cargo add serde               # Add dependency
cargo add serde --features derive # Add with specific features
cargo add tokio --dev         # Add development dependency
cargo add clap --build        # Add build dependency
cargo remove serde            # Remove dependency
```

### Updating Dependencies
```bash
cargo update                  # Update all dependencies
cargo update serde            # Update specific dependency
cargo tree                    # Show dependency tree
cargo outdated                # Show outdated dependencies (requires cargo-outdated)
```

## Publishing & Registry

### Package Management
```bash
cargo publish                 # Publish to crates.io
cargo package                 # Create distributable package
cargo login                   # Login to crates.io
cargo owner --add username    # Add package owner
cargo yank --vers 1.0.0       # Yank a version
```

### Registry Operations
```bash
cargo search regex            # Search crates.io
cargo install ripgrep         # Install binary crate globally
cargo uninstall ripgrep       # Uninstall global binary
cargo install --path .        # Install current project globally
```

## Cargo Workspaces

### Workspace Commands
```bash
cargo build --workspace       # Build all workspace members
cargo test --workspace        # Test all workspace members
cargo run -p member_name      # Run specific workspace member
cargo build -p member_name    # Build specific workspace member
```

## Formatting & Linting

### Code Quality
```bash
cargo fmt                     # Format code
cargo fmt -- --check         # Check if code is formatted
cargo clippy                  # Run Clippy linter
cargo clippy -- -D warnings  # Treat warnings as errors
cargo clippy --fix           # Auto-fix some issues
```

## Advanced Cargo Commands

### Profile & Features
```bash
cargo build --features feat1,feat2  # Build with specific features
cargo build --all-features          # Build with all features
cargo build --no-default-features   # Build without default features
cargo build --profile dev            # Use specific profile
```

### Target Management
```bash
cargo build --target x86_64-pc-windows-gnu  # Cross-compile
rustup target add wasm32-unknown-unknown    # Add compilation target
rustup target list                          # List available targets
```

### Cleaning & Cache
```bash
cargo clean                   # Remove build artifacts
cargo clean --release        # Remove only release artifacts
cargo clean -p package_name  # Clean specific package
```

## Useful Cargo Extensions

### Installing Extensions
```bash
cargo install cargo-watch    # Watch for changes and rebuild
cargo install cargo-expand   # Show macro expansions  
cargo install cargo-audit    # Security audit
cargo install cargo-outdated # Check for outdated dependencies
cargo install cargo-edit     # Enhanced dependency management
```

### Using Extensions
```bash
cargo watch -x run           # Watch and run on changes
cargo watch -x test          # Watch and test on changes
cargo expand                 # Show macro expansions
cargo audit                  # Check for security vulnerabilities
cargo machete                # Find unused dependencies
```

## Rustup (Rust Toolchain Manager)

### Toolchain Management
```bash
rustup update                 # Update Rust toolchain
rustup default stable        # Set default toolchain
rustup toolchain list        # List installed toolchains
rustup toolchain install nightly # Install nightly toolchain
rustup component add clippy  # Add component
```

### Overrides
```bash
rustup override set nightly  # Use nightly for current directory
rustup override unset        # Remove override
rustup show                   # Show current toolchain info
```

## Environment Variables

### Common Variables
```bash
CARGO_TARGET_DIR=./target     # Set target directory
RUST_LOG=debug               # Set log level
RUST_BACKTRACE=1             # Enable backtraces
RUSTFLAGS="-C target-cpu=native" # Pass flags to rustc
```

## Cargo.toml Quick Reference

### Basic Structure
```toml
[package]
name = "my_project"
version = "0.1.0"
edition = "2021"

[dependencies]
serde = { version = "1.0", features = ["derive"] }
tokio = { version = "1.0", optional = true }

[dev-dependencies]
criterion = "0.4"

[features]
default = []
async = ["tokio"]
```

### Useful Sections
```toml
[[bin]]
name = "my_bin"
path = "src/bin/my_bin.rs"

[profile.release]
opt-level = 3
lto = true

[workspace]
members = ["crate1", "crate2"]
```

## Common Patterns

### Development Workflow
```bash
# Typical development cycle
cargo check          # Quick syntax check
cargo test           # Run tests
cargo clippy         # Check for issues
cargo fmt            # Format code
cargo build          # Final build
```

### Release Preparation
```bash
cargo test --release  # Test in release mode
cargo clippy -- -D warnings # Ensure no warnings
cargo fmt -- --check # Ensure formatted
cargo audit          # Security check
cargo package        # Test packaging
cargo publish        # Publish to crates.io
```

## Tips & Tricks

- Use `cargo check` during development for faster feedback
- Set up `cargo watch` for automatic rebuilds during development
- Use `cargo tree` to debug dependency conflicts
- Add `--jobs 1` to build commands for single-threaded builds (useful for debugging)
- Use `RUST_LOG=debug cargo run` for verbose logging
- Create `.cargo/config.toml` for project-specific settings
- Use `cargo --list` to see all available commands and extensions