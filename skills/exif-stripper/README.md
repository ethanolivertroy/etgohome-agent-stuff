# EXIF Stripper Skill

An agent skill that strips sensitive EXIF metadata from images before they're published.

## Why

Digital photos embed metadata including GPS coordinates, camera serial numbers, software used, and more. Publishing images without stripping this data is a privacy risk. This skill automates the cleanup.

## Strategy

**Strip all, restore essentials.** All metadata is removed, then only orientation (so photos don't display sideways) and ICC color profile (so colors render correctly) are restored.

## Prerequisites

- [exiftool](https://exiftool.org/) — `brew install exiftool`

## Usage

Set `EXIF_SKILL_DIR` to the absolute directory containing this skill's `SKILL.md`,
whether it is in a checkout or a runtime's skills directory. Run the commands from
your project directory so relative image paths refer to your project.

```bash
export EXIF_SKILL_DIR="/absolute/path/to/exif-stripper"
```

### Strip

Strip sensitive EXIF metadata from images at the given path (defaults to current directory, recursive).

```bash
bash "$EXIF_SKILL_DIR/scripts/strip-exif.sh"                     # Strip all images in current directory
bash "$EXIF_SKILL_DIR/scripts/strip-exif.sh" ./content/images    # Strip images in specific directory
bash "$EXIF_SKILL_DIR/scripts/strip-exif.sh" photo.jpg           # Strip a single file
```

### Check

Audit images for sensitive metadata without modifying them.

```bash
bash "$EXIF_SKILL_DIR/scripts/check-exif.sh"                     # Check all images in current directory
bash "$EXIF_SKILL_DIR/scripts/check-exif.sh" ./static/images     # Check images in specific directory
```

## Automation

Custom Git hooks or publishing workflows can call the scripts above. Audit first,
strip metadata if needed, and re-stage changed images before committing.

## What's Stripped vs Preserved

| Stripped (Privacy) | Preserved (Display) |
|-|-|
| GPS coordinates | Orientation |
| Camera serial numbers | ICC color profile |
| Software / editing history | |
| Owner / artist names | |
| Thumbnail / preview images | |
| Maker notes | |

## Supported Formats

JPG, JPEG, PNG, TIFF, WebP

## Scripts

The scripts can be used standalone outside an agent runtime:

```bash
# Strip metadata from images
bash "$EXIF_SKILL_DIR/scripts/strip-exif.sh" /path/to/images

# Audit images for sensitive metadata
bash "$EXIF_SKILL_DIR/scripts/check-exif.sh" /path/to/images
```
