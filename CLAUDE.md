# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview
bili2text is a Python application that converts Bilibili videos to text transcripts. The workflow: downloads videos → extracts audio → segments audio → uses OpenAI Whisper for speech-to-text conversion.

## Core Commands

### Setup and Installation

#### Using uv (Recommended)
```bash
# Install uv if not already installed
curl -LsSf https://astral.sh/uv/install.sh | sh

# Create virtual environment and install dependencies
uv sync

# Activate environment
source .venv/bin/activate  # Linux/macOS
# or
.venv\Scripts\activate     # Windows

# Install FFmpeg (required for audio processing)
# macOS: brew install ffmpeg
# Ubuntu: sudo apt install ffmpeg
```

#### Using pip (Traditional)
```bash
# Install dependencies
pip install -r requirements.txt

# Install FFmpeg (required for audio processing)
# macOS: brew install ffmpeg
# Ubuntu: sudo apt install ffmpeg
```

### Running the Application
```bash
# GUI version (recommended for users)
uv run python window.py
# or if environment is activated: python window.py

# CLI version (for testing/development)
uv run python main.py
# or if environment is activated: python main.py

# Using installed scripts (after uv sync)
bili2text-gui    # Launch GUI
bili2text        # Launch CLI
```

### Dependencies Check
- Verify FFmpeg installation: `ffmpeg -version`
- Check Python packages: `uv pip list` or `pip list`

### Development Commands
```bash
# Install development dependencies
uv sync --all-extras

# Add new dependency
uv add package-name

# Add development dependency
uv add --dev package-name

# Update dependencies
uv sync --upgrade
```

## Architecture

### Core Modules
- **main.py**: CLI entry point for testing (BV input → text output)
- **window.py**: GUI application using ttkbootstrap, main user interface
- **utils.py**: Video downloading using you-get, handles BV number processing
- **exAudio.py**: Audio processing pipeline (video→MP3→segments), uses moviepy and pydub
- **speech2text.py**: Whisper integration for speech recognition
- **xunfei.py**: Alternative speech recognition service (iFlytek)

### Data Flow
1. **Input**: BV number or Bilibili video URL
2. **Download**: `utils.download_video()` → saves to `bilibili_video/{BV}/`
3. **Audio Extraction**: `exAudio.convert_flv_to_mp3()` → saves to `audio/conv/`
4. **Segmentation**: `exAudio.split_mp3()` → 45-second chunks in `audio/slice/{timestamp}/`
5. **Transcription**: `speech2text.run_analysis()` → processes segments with Whisper
6. **Output**: Combined text saved to `outputs/{timestamp}.txt`

### Directory Structure Created During Runtime
```
bilibili_video/{BV_NUMBER}/    # Downloaded videos
audio/conv/                    # Converted MP3 files
audio/slice/{timestamp}/       # Audio segments (45s each)
outputs/                       # Final transcription text files
```

### Whisper Integration
- Model loading: `load_whisper(model)` where model = "tiny"|"small"|"medium"|"large"
- CUDA detection and auto-fallback to CPU
- Custom prompt support for better recognition accuracy
- Sequential processing of audio segments with proper ordering

### GUI Components (window.py)
- BV/URL input field with automatic BV extraction
- Whisper model selection dropdown
- Real-time log display with stdout redirection
- Threading for non-blocking video processing
- Model loading status and CUDA capability detection

## Development Notes

### File Processing
- Video integrity checking via FFmpeg before conversion
- Automatic cleanup of XML metadata files after download
- Timestamp-based unique folder naming prevents conflicts
- Proper file sorting ensures correct transcript order

### Error Handling
- Video download failure detection and reporting
- Audio file corruption validation
- Missing file checks at each processing stage
- User confirmation dialogs for long-running operations

### External Dependencies
- **you-get**: Bilibili video downloading (must be in PATH)
- **FFmpeg**: Required for audio processing and integrity checks
- **Whisper**: OpenAI's speech recognition model
- **ttkbootstrap**: Modern GUI theming

### Memory and Performance
- Audio segmentation prevents memory issues with long videos
- Model caching after initial load
- CUDA acceleration when available
- Background processing prevents GUI freezing