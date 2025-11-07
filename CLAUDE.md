# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This repository contains a Clash subscription converter system with three main components managed as git submodules:

- **sub-web**: Vue.js frontend (port 8080 dev, 80 prod) - User interface for subscription conversion
- **subconverter**: C++ backend service (port 25500) - Converts between proxy subscription formats
- **ACL4SSR**: Configuration repository - Contains rulesets and configuration files

## Architecture

### Frontend (sub-web)
- Framework: Vue 2.6 + Element UI
- Build: vue-cli-service
- Main component: `src/views/Subconverter.vue` - handles subscription conversion interface
- Backend configuration: `.env` file (VUE_APP_SUBCONVERTER_DEFAULT_BACKEND)
- Two modes: Basic (基础模式) and Advanced (进阶模式)

### Backend (subconverter)
- Language: C++20
- Build system: CMake
- Web server: httplib (cpp-httplib)
- Main entry: `src/main.cpp`
- Key modules:
  - `src/generator/` - Config generation and templates
  - `src/parser/` - Subscription parsing
  - `src/handler/` - Request handling, web interface
  - `src/utils/` - Utilities (base64, regex, network)
- Configuration: `base/pref.ini` (from pref.example.ini)
- Supported formats: Clash, Surge, Quantumult X, Loon, V2Ray, SS/SSR, Singbox, etc.

### Configuration (ACL4SSR)
- Contains rulesets for various clients
- Referenced by subconverter through `base/config/*.ini` files

## Development Commands

### Quick Start with Docker Compose (Recommended)
```bash
# One-command deployment (builds local images)
docker-compose up -d --build

# Access services
# Frontend: http://localhost:8080
# Backend: http://localhost:25500

# Stop services
docker-compose down

# View logs
docker-compose logs -f

# Rebuild after code changes
docker-compose up -d --build
```

See `DEPLOY.md` for detailed Docker Compose usage.

### Frontend (sub-web/)
```bash
# Install dependencies
yarn install

# Development server (http://localhost:8080)
yarn serve

# Production build (outputs to dist/)
yarn build

# Lint
yarn lint

# Docker build (local image)
docker build -f Dockerfile.local -t subweb-local:latest .
docker run -d -p 8080:80 --restart always --name subweb subweb-local:latest
```

### Backend (subconverter/)
```bash
# Build with CMake (typical workflow)
mkdir build && cd build
cmake .. -DCMAKE_BUILD_TYPE=Release
cmake --build .

# Run (from build directory or installation path)
./subconverter

# Docker build (local image)
docker build -f Dockerfile.local -t subconverter-local:latest .
docker run -d --restart=always -p 25500:25500 subconverter-local:latest

# Check version
curl http://localhost:25500/version
```

### Git Submodules
```bash
# Initialize submodules (if not already done)
git submodule update --init --recursive

# Update all submodules to latest
git submodule update --remote

# Update specific submodule
cd sub-web && git pull origin main
```

## Configuration

### Backend Configuration (subconverter/base/pref.ini)
Key settings to configure before first run:
- `api_mode`: Set to true for production to prevent loading local files
- `api_access_token`: Change from default "password"
- `default_external_config`: Default config when none specified
- `clash_rule_base`, `surge_rule_base`, etc.: Template files for each client
- `exclude_remarks`: Filter out nodes matching patterns (e.g., expiry notices)
- `proxy_config`, `proxy_ruleset`, `proxy_subscription`: Proxy settings for downloading

### Frontend Configuration (sub-web/.env)
- `VUE_APP_SUBCONVERTER_DEFAULT_BACKEND`: Backend API URL (default: https://api.wcc.best)
- `VUE_APP_MYURLS_API`: URL shortening service
- `VUE_APP_CONFIG_UPLOAD_API`: Configuration file hosting service

## API Usage

The backend conversion API format:
```
http://127.0.0.1:25500/sub?target=<TARGET>&url=<URL>&config=<CONFIG>
```

Parameters:
- `target`: Output format (clash, surge, quanx, loon, v2ray, ss, ssr, singbox, etc.)
- `url`: URLEncoded subscription link(s), use `|` to merge multiple
- `config`: (Optional) URLEncoded external config file path/URL

Example:
```
http://127.0.0.1:25500/sub?target=clash&url=https%3A%2F%2Fexample.com%2Fsub
```

## Build Dependencies

### Backend (subconverter)
Required libraries:
- CURL >= 7.54.0
- yaml-cpp >= 0.6.3
- rapidjson
- toml11
- PCRE2
- QuickJS (for script support)
- libcron (for scheduled tasks)

### Frontend (sub-web)
- Node.js >= 20.0.0
- Yarn 1.x

## Important Notes

- The three submodules are forks maintained by duzefu (see .gitmodules)
- When modifying sub-web defaultBackend: edit `src/views/Subconverter.vue` directly OR update `.env` file
- External configs in `subconverter/base/config/` provide pre-configured rulesets (ACL4SSR variants)
- The backend uses templates from `base/*.tpl` for generating client configs
- For deployment, sub-web requires nginx or similar web server for the dist/ output
