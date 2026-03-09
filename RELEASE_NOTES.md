# Release v1.0.5

## 🎉 New Features

- **Office Pro Edition**: Add Dockerfile.office for non-technical users with full document processing capabilities

## 🔧 Improvements

- Modernize Makefile help UI with grouped commands
- Simplify Makefile UI for cleaner output
- Optimize Makefile with unified version selection
- Adopt OpenClaw official Dockerfile best practices
- Adopt multi-stage build for optimized image size

## 🐛 Bug Fixes

- Add pandoc and texlive for document processing
- Add unzip to builder stage for bun installation
- Copy docs directory to fix missing AGENTS.md
- Move chown after COPY to fix directory not found
- Remove invalid shell syntax from COPY instruction
- Use base image's node instead of copying from builder
- Use npm to install pnpm instead of corepack
- Add unzip for bun installation
- Pass OPENCLAW_IMAGE env to setup script

## 📦 Internal

- Set help-full as default make target

---

**Full Changelog**: https://github.com/hrygo/openclaw-devkit/compare/v1.0.4...v1.0.5
