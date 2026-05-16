# DopplrHub PHP CLI

A self-contained PHP CLI for the existing DopplrHub public API at `/api/v1`.

## Install

```bash
wget -O /usr/local/bin/dopplerhub https://raw.githubusercontent.com/DopplrHub/php-cli/main/dopplerhub
chmod +x /usr/local/bin/dopplerhub
export DOPPLERHUB_API_KEY=YOUR_API_KEY
```

If you run a self-hosted DopplrHub instance, the backend also serves the CLI at `/api/dopplerhub`.

For local development on Windows or Linux, you can also run it directly with PHP:

```bash
php ./dopplrhub -f jpg input.pdf
```

## Usage

Convert one file:

```bash
dopplrhub -f jpg input.pdf
```

Run OCR and save a DOCX result:

```bash
dopplrhub ocr -f ocr-docx --language eng scan.pdf
```

Compress a PDF with an explicit quality level:

```bash
dopplrhub pdf --operation compress --param quality=screen packet.pdf
```

Resize an image and change the output format:

```bash
dopplrhub image --operation resize --param width=1920 --param height=1080 --param fit=cover --param outputFormat=webp hero.png
```

Trim a video clip:

```bash
dopplrhub video --operation trim --param startTime=3 --param endTime=12 --param outputFormat=mp4 clip.mp4
```

Run ADA analysis or ATS resume analysis:

```bash
dopplrhub ada brochure.pdf
dopplrhub ats --job-description "Senior PHP engineer with API design experience" resume.pdf
```

Convert multiple files in one command and save the converted files next to the originals:

```bash
dopplrhub -f pdf *.txt
```

Point at a non-default API host:

```bash
DOPPLERHUB_BASE_URL=https://api.example.com/api/v1 dopplrhub -f png brochure.pdf
```

Delete the remote job after the local download finishes:

```bash
dopplrhub --delete -f jpg input.pdf
```

Merge multiple PDFs into one output file:

```bash
dopplrhub pdf --operation merge chapter-1.pdf chapter-2.pdf chapter-3.pdf
```

## Notes

- The script can call `POST /convert`, `POST /tools/ocr`, `POST /tools/pdf`, `POST /tools/image`, `POST /tools/video`, `POST /tools/ada/analyze`, and `POST /tools/ats/analyze`. For archive creation or social-media resizing, use one of the language SDKs instead.
- For `pdf`, `image`, and `video` commands, repeat `--param key=value` to pass operation-specific settings.
- Video `trim` and `extract` commands accept `--param startTime=...` and `--param endTime=...`; the CLI maps those to the API's nested trim payload.
- Output files are written into the same directory as each input file.
- The file extension is derived from the requested target format.
- The current backend is file-based. This CLI does not do webpage rendering.
- Requires PHP 8.1+ with the `curl` extension.
