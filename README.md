# Halos Raspberry Pi OS Distribution

Halos is the Hat Labs Operating System, a custom Raspberry Pi OS distribution designed to help end-users run and maintain both Hat Labs and generic Raspberry Pi hardware with ease.

Halos is built on top of Raspberry Pi OS Lite, adding tools for web-based management, hardware monitoring, and containerized services.

This repository acts as a monorepo for all the components that make up Halos.

## Layout

- halos-pi-gen/ - Custom Raspberry Pi image builder (submodule)
- casaos-marine-store/ - Custom marine app store repository for CasaOS (submodule)
- casaos-docker-service/ - systemd service files and Debian packaging for CasaOS Docker deployment (submodule)
- apt.hatlabs.fi/ - Custom APT repository for Halos packages (submodule)
