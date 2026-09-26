# WARP.md

This file provides guidance to WARP (warp.dev) when working with code in this repository.

## Project Overview

This is **rust-mcp-filesystem**, a high-performance MCP (Model Context Protocol) server implemented in Rust for filesystem operations. It's a complete rewrite of the JavaScript-based `@modelcontextprotocol/server-filesystem` with enhanced performance, security, and feature set.

## Key Architecture

### Core Components

- **CLI (`src/cli.rs`)**: Command-line argument parsing with security-focused options:
  - `--allow-write` / `-w`: Enables write operations (read-only by default)
  - `--enable-roots` / `-t`: Allows dynamic directory access control via MCP Roots
  - `allowed_directories`: Space-separated list of permitted directories

- **Handler (`src/handler.rs`)**: Main MCP protocol handler implementing `ServerHandler` trait
  - Manages MCP initialization, tool listing, and tool execution
  - Handles MCP Roots protocol for dynamic directory updates
  - Enforces write access controls

- **FileSystem Service (`src/fs_service/`)**: Core filesystem operations
  - `core.rs`: Main service logic and path validation
  - `io/`: File I/O operations (read, write, edit)
  - `search/`: File search and content search functionality
  - `archive/`: ZIP archive creation and extraction
  - `utils.rs`: Utility functions for file operations

- **Tools (`src/tools/`)**: 22 MCP tools for various filesystem operations
  - Read operations: `ReadTextFile`, `HeadFile`, `TailFile`, `ReadFileLines`
  - Directory operations: `ListDirectory`, `CreateDirectory`, `DirectoryTree`
  - Search operations: `SearchFiles`, `SearchFilesContent`
  - File management: `MoveFile`, `WriteFile`, `EditFile`, `GetFileInfo`
  - Archive operations: `ZipFiles`, `UnzipFile`, `ZipDirectory`
  - Analysis tools: `CalculateDirectorySize`, `FindDuplicateFiles`, `FindEmptyDirectories`

## Development Commands

### Building
```bash
# Standard build
cargo build

# Release build
cargo build --release
```

### Testing
```bash
# Run all tests
cargo test

# Alternative: If cargo-nextest is installed (preferred by project)
cargo nextest run --no-tests=pass
```

### Code Quality
```bash
# Format code
cargo fmt --all -- --check

# Lint with Clippy
cargo clippy --all-targets -- -D warnings

# Fix Clippy issues automatically
cargo clippy --fix --allow-dirty

# Run all quality checks (if cargo-make is installed)
cargo make check
```

### Running the Server
```bash
# Read-only mode with specific directories
cargo run -- /path/to/allowed/dir1 /path/to/allowed/dir2

# Enable write operations
cargo run -- --allow-write /path/to/allowed/dir

# Enable MCP Roots support (dynamic directory management)
cargo run -- --enable-roots

# Combination of flags
cargo run -- --allow-write --enable-roots /path/to/initial/dir
```

## Security Model

The server follows a **secure-by-default** approach:
- **Read-only by default**: Write operations require explicit `--allow-write` flag
- **Directory restrictions**: Only specified directories are accessible
- **MCP Roots support**: Optional dynamic directory access via MCP protocol
- **Path validation**: All paths are validated against allowed directories

## Tool Categories

### Write Operations (require `--allow-write`)
- `CreateDirectory`, `MoveFile`, `WriteFile`, `EditFile`
- `ZipFiles`, `UnzipFile`, `ZipDirectory`

### Read Operations (always available)
- All other tools including search, listing, and analysis operations

## Development Notes

- Uses **Rust 2024 edition** with async/await throughout
- Built on `rust-mcp-sdk` for MCP protocol implementation
- Uses `tokio` for async runtime and `clap` for CLI parsing
- Extensive test coverage in `tests/` directory
- Uses `cargo-nextest` as preferred test runner (when available)
- Uses `cargo-make` for streamlined development workflow (when available)

## Dependencies

The project uses several key dependencies:
- `rust-mcp-sdk`: MCP protocol implementation
- `tokio`: Async runtime
- `clap`: CLI argument parsing
- `walkdir`: Directory traversal
- `async_zip`: ZIP archive operations
- `glob-match`: Pattern matching for file searches

## Testing

Tests are organized in `tests/` directory:
- `test_fs_service.rs`: Core filesystem service tests
- `test_tools.rs`: Individual tool functionality tests
- `test_cli.rs`: CLI argument parsing tests
- `common/`: Shared test utilities

Run specific test files:
```bash
cargo test test_fs_service
cargo test test_tools
```