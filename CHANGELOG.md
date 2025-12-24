# Changelog

All notable changes to DirettaRendererUPnP will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [1.1.0] - 2025-12-24

### Fixed
- **Critical**: Fixed format change freeze when transitioning between bit depths (e.g., 24-bit → 16-bit)
  - Issue: 4 residual samples in buffer never drained, causing 10-second timeout
  - Solution: Force flush with silence padding to push incomplete frames through pipeline
  - Result: Format changes now complete in ~200-300ms instead of 10s timeout
- Reduced drain timeout from 10s to 3s for faster error recovery
- Added tolerance for ≤4 residual samples (considered "empty enough")
- Increased hardware stabilization time from 200ms to 300ms for better reliability

### Changed
- Format change transitions are now much faster and more reliable
- Better logging during format change sequence with flush detection

### Technical Details
- Added Step 1.5 in `changeFormat()`: Force flush with 128 samples of silence padding
- Modified drain logic to accept small residual (≤4 samples) as successful drain
- Improved error messages and debugging output

### Added
- 🌐 **Multi-interface support** for multi-homed systems
  - New option: `--interface <name>` to bind to specific network interface (e.g., eth0, eno1)
  - New option: `--bind-ip <address>` to bind to specific IP address
  - Essential for 3-tier architecture configurations with separate control and audio networks
  - Fixes SSDP discovery issues on systems with multiple network interfaces

### Changed
- UPnP initialization now uses `UpnpInit2()` with interface parameter instead of `nullptr`
- Auto-detection remains default behavior (backward compatible)
- Improved error messages when UPnP initialization fails with specific interface
- Added interface information in startup logs

### Configuration
- Systemd configuration now supports `NETWORK_INTERFACE` parameter in `diretta-renderer.conf`
- Wrapper script (`start-renderer.sh`) automatically detects IP vs interface name

### Use Cases
- **3-tier architecture**: Control points on 192.168.1.x (eth0), Diretta DAC on 192.168.2.x (eth1)
  ```bash
  sudo ./bin/DirettaRendererUPnP --interface eth0 --target 1
  ```
- **VPN + Local network**: Bind to local interface while VPN is active
- **Multiple Ethernet adapters**: Specify which adapter to use for UPnP

### Documentation
- Added comprehensive multi-interface section in README
- Added troubleshooting guide for multi-homed systems
- Added examples for common network configurations

## [1.0.8] - 2024-12-23

### Fixed
- Fixed SEEK functionality deadlock issue
  - Replaced mutex-based synchronization with atomic flag
  - Implemented asynchronous seek mechanism
  - Seek now completes in <100ms without blocking

### Changed
- Improved seek reliability and responsiveness
- Better error handling during seek operations

## [1.0.7] - 2024-12-22

### Added
- Advanced Diretta SDK configuration options:
  - `--thread-mode <value>`: Configure thread priority and behavior (bitmask)
  - `--cycle-time <µs>`: Transfer packet cycle maximum time (default: 10000)
  - `--cycle-min-time <µs>`: Transfer packet cycle minimum time (default: 333)
  - `--info-cycle <µs>`: Information packet cycle time (default: 5000)
  - `--mtu <bytes>`: Override MTU for network packets (default: auto-detect)

### Changed
- Buffer size parameter changed from integer to float for finer control
  - Now accepts values like `--buffer 2.5` for 2.5 seconds
- Improved buffer adaptation logic based on audio format complexity
- Better MTU detection and configuration

### Documentation
- Added comprehensive documentation for advanced Diretta SDK parameters
- Added thread mode bitmask reference
- Added MTU optimization guide

## [1.0.6] - 2024-12-21

### Fixed
- Audirvana Studio streaming compatibility issues
  - Fixed pink noise after 6-7 seconds when streaming 24-bit content from Qobuz
  - Issue was related to HTTP streaming implementation vs Diretta SDK buffer handling
  - Workaround: Use 16-bit or 20-bit streaming settings in Audirvana

### Changed
- Improved buffer handling for HTTP streaming sources
- Better error detection and recovery for streaming issues

## [1.0.5] - 2024-12-20

### Fixed
- Format change handling improvements
  - Fixed clicking sounds during 24-bit audio playback
  - Removed artificial silence generation that caused artifacts
  - Proper buffer draining using Diretta SDK's `buffer_empty()` methods

### Changed
- Improved audio playback behavior during track transitions
- Better handling of sample rate changes

## [1.0.4] - 2024-12-19

### Added
- Jumbo frame support with 16k MTU optimization
- Configurable MTU settings for network optimization

### Fixed
- Network configuration issues with Intel i225 cards (limited to 9k MTU)
- Buffer handling improvements

## [1.0.3] - 2024-12-18

### Added
- Gapless playback support
- Improved track transition handling

### Changed
- Better buffer management during track changes
- Improved format detection and handling

## [1.0.2] - 2024-12-17

### Fixed
- DSD playback improvements
- Sample rate detection accuracy

## [1.0.1] - 2024-12-16

### Added
- Support for multiple Diretta DAC targets
- Interactive target selection
- Command-line target specification

### Fixed
- Target discovery reliability
- Connection stability improvements

## [1.0.0] - 2024-12-15

### Added
- Initial release
- UPnP MediaRenderer implementation
- Diretta protocol integration
- Support for PCM audio (16/24/32-bit, up to 768kHz)
- Support for DSD audio (DSD64/128/256/512/1024)
- AVTransport service (Play, Pause, Stop, Seek, Next, Previous)
- RenderingControl service (Volume, Mute)
- ConnectionManager service
- Automatic SSDP discovery
- Format-specific buffer optimization
- Systemd service integration

### Supported Formats
- PCM: 16/24/32-bit, 44.1kHz to 768kHz
- DSD: DSD64, DSD128, DSD256, DSD512, DSD1024
- Containers: FLAC, WAV, AIFF, ALAC, APE, DSF, DFF

### Supported Control Points
- JPlay
- Roon
- BubbleUPnP
- Any UPnP/DLNA control point

---

## Version Numbering

- **Major.Minor.Patch** (e.g., 1.0.10)
- **Major**: Breaking changes
- **Minor**: New features, backward compatible
- **Patch**: Bug fixes, backward compatible
