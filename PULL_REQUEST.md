# Pull Request: Add configurable PhotoTranscoder path, improve Plex auth, and update Docker to Debian 12

## 📋 Summary

This PR adds several important improvements to ImageMaid:

1. **Configurable PhotoTranscoder path** - Fixes macOS compatibility issues
2. **Improved Plex authentication** - Better error handling and validation
3. **Docker improvements** - Updated to Debian 12 with automated builds

## 🎯 Changes

### 1. Add configurable PhotoTranscoder path via environment variable

- **New variable**: `PHOTO_TRANSCODER_PATH` in `.env`
- **Problem**: Fixed error "PhotoTranscoder Directory Not Found" on macOS and non-standard installations
- **Solution**: Users can now specify custom PhotoTranscoder path, defaults to `{PLEX_PATH}/Cache/PhotoTranscoder`
- **Files**: `imagemaid.py:69,141`, `config/example.env:13`

**Usage:**
```bash
PHOTO_TRANSCODER=True
PHOTO_TRANSCODER_PATH=/custom/path/to/PhotoTranscoder
```

### 2. Fix Plex authentication error handling and connection validation

- **Strip whitespace** from `PLEX_URL` and `PLEX_TOKEN` (fixes common .env copy-paste issues)
- **Auto-fix URL format** - automatically adds `http://` if missing
- **Pre-flight connectivity test** - tests server reachability before authentication
- **Enhanced error messages** - shows URL used, token length, and troubleshooting steps
- **Debug logging** - when `TRACE=True`, shows connection details without revealing secrets
- **Files**: `imagemaid.py:165-208`

**Before:**
```
[CRITICAL] | Plex Error: Plex token is invalid
```

**After:**
```
[INFO] Testing connectivity to http://192.168.1.12:32400...
[INFO] Server is reachable
[CRITICAL] | Plex Error: Authentication failed
  - URL used: http://192.168.1.12:32400
  - Token length: 20 characters
  - Verify your PLEX_URL and PLEX_TOKEN in .env file
  - Check for extra spaces in your configuration
```

### 3. Improve .dockerignore for optimized Docker builds

- Added comprehensive Python exclusions (cache, build artifacts)
- Added IDE and OS-specific file exclusions
- Prevents sensitive config files from being included in image
- Reduces Docker build context size
- **Files**: `.dockerignore`

### 4. Add GitHub Actions workflow and update Docker to Debian 12

**Docker updates:**
- Updated base image: `python:3.11-slim-buster` → `python:3.11-slim-bookworm` (Debian 12)
- Changed `KOMETA_DOCKER` → `IMAGEMAID_DOCKER` for consistency
- **Files**: `Dockerfile`

**GitHub Actions workflow:**
- Auto-builds Docker images on push to `master`, `develop`, and `claude/**` branches
- Publishes to GitHub Container Registry (`ghcr.io/nicolasrata/imagemaid`)
- Multi-platform support: `linux/amd64` and `linux/arm64`
- Auto-tags based on branch/tag names
- Intelligent layer caching for faster builds
- **Files**: `.github/workflows/docker-build.yml`

**Documentation:**
- Added comprehensive Docker usage guide
- **Files**: `DOCKER.md`

## 🧪 Testing

- [x] Code changes tested locally
- [x] Docker build successful with Debian 12
- [x] .dockerignore optimizations verified

## 📦 Docker Image

After merge, users can use:
```bash
docker pull ghcr.io/nicolasrata/imagemaid:latest
```

Instead of the unmaintained `kometateam/imagemaid`.

## 🔗 Related Issues

Fixes:
- PhotoTranscoder path issues on macOS
- False "token is invalid" errors from whitespace in .env
- Missing http:// in PLEX_URL causing connection failures
- Outdated Debian base image

## ⚙️ Breaking Changes

None - all changes are backward compatible.

---

## 📝 Commits in this PR

- `2c891ce` - Add configurable PhotoTranscoder path via environment variable
- `292e65b` - Fix Plex authentication error handling and connection validation
- `efdd7a0` - Improve .dockerignore for optimized Docker builds
- `2571e5d` - Add GitHub Actions workflow and update Docker to Debian 12
