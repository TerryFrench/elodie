# GEMINI.md

## Project Overview
Elodie is a personal EXIF-based photo, video, and audio assistant. It organizes media files into a customizable folder structure based on their metadata (date, location, etc.) without using a proprietary database.

## Setup Commands
- Install dependencies: `pip install -r requirements.txt`
- Install test dependencies: `pip install -r elodie/tests/requirements.txt`
- (Optional) Install ExifTool (required for core functionality):
  - OSX: `brew install exiftool`
  - Debian/Ubuntu: `apt-get install libimage-exiftool-perl`
  - Fedora/Redhat: `dnf install perl-Image-ExifTool`

## Running Elodie
- Import media: `./elodie.py import --destination="/path/to/destination" /path/to/source`
- Update media metadata: `./elodie.py update --location="Las Vegas, NV" /path/to/photo.jpg`
- Dry run: Add `--dry-run` to any command to see what would happen without making changes.

## Testing Instructions
- Run all tests: `pytest`
- Run a specific test file: `pytest elodie/tests/some_test.py`
- Run linting: `flake8 .`

## Code Style
- Follow PEP 8 guidelines.
- Use `pytest` for testing.
- Adhere to existing patterns in `elodie/` directory.
- Media handling logic is in `elodie/media/`.
- Filesystem operations are in `elodie/filesystem.py`.

## Important Files
- `elodie.py`: Main entry point.
- `elodie/media/`: Classes for different media types (photo, video, audio).
- `elodie/filesystem.py`: Logic for moving and organizing files.
- `elodie/config.py`: Configuration management.
- `elodie/geolocation.py`: Handles reverse geocoding using MapQuest API.

## Known Issues
1. **Unicode Filenames:** The program crashes when processing photos with Unicode characters in their names.
2. **Pre-Epoch Dates:** The program doesn't handle dates before the Unix epoch (1970-01-01).
