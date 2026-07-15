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

Extract a document into structured JSON:

```bash
dopplrhub -f json invoice.pdf
```

The JSON output mirrors the document: headings nest into a `sections` tree, tables become arrays of
rows keyed by their column headers, and images are embedded as base64. Accepted sources are PDF,
DOC, DOCX, ODT, RTF, HTML, TXT, and CSV.

Pass conversion settings with `--setting key=value`. Values that look like JSON are decoded, so you
can hand the API a nested object. This example pins document keywords to your own property names and
skips base64 images:

```bash
dopplrhub -f json \
  --setting json='{"images":"omit","schema":{"fields":[
    {"property":"invoiceNumber","match":["Invoice Number","Invoice #"]},
    {"property":"totalDue","match":"Total Due","type":"number"}
  ]}}' \
  invoice.pdf
```

The resulting `invoice.json` carries a top-level `fields` object alongside the full section tree:

```json
{
  "document": { "title": "Acme Invoice", "sourceFormat": "pdf", "pageCount": 2 },
  "fields":   { "invoiceNumber": "INV-2026-42", "totalDue": 4500 },
  "sections": [ { "id": "s1", "heading": "Acme Invoice", "level": 1, "content": [ ... ] } ]
}
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
- For `convert`, repeat `--setting key=value` to pass `conversionSettings` entries. A value that parses as JSON is sent as a nested object; anything else is sent as a string.
- Video `trim` and `extract` commands accept `--param startTime=...` and `--param endTime=...`; the CLI maps those to the API's nested trim payload.
- Output files are written into the same directory as each input file.
- The file extension is derived from the requested target format.
- The current backend is file-based. This CLI does not do webpage rendering.
- Requires PHP 8.1+ with the `curl` extension.

### Pricing and the source media type

Jobs are priced from the **source** media type, not the target: `photo.jpg -> pdf` is an image job,
while `report.pdf -> jpg` is a document job. The CLI detects the source media type from the input
file's extension and sends it as `mediaType`, along with the file size, so each job is priced
correctly. It also normalizes extension aliases the way the API does (`.htm` -> `html`,
`.jpeg` -> `jpg`, `.dic` -> `dcm`).

If the API receives no `mediaType`, it falls back to pricing the job as one minute of audio, which
under-charges most document and image conversions. Older CLI versions never sent the field at all.

Override the detection with `--media-type` when an extension is missing or misleading:

```bash
dopplrhub -f json --media-type document report.bin
```

Unrecognised extensions are left for the API to classify, matching the previous behaviour.
