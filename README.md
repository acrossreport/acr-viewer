# ACR Viewer

English | [日本語](README.ja.md) | [Français](README.fr.md)

ACR Viewer is an application that stays in the system tray and automatically prints and saves (PDF/PNG) ACR (AcrossReport) report data received through a watch folder or HTTP.

## Features

- Runs in the system tray (no window needed)
- **Watch folder**: just drop a data file to render, print, and save automatically
- **HTTP API**: send report data from other systems to print and save
- Output: PDF / PNG / both, and automatic printing
- Settings are changed in a browser-based settings page (Japanese / English / Français)
- Automatic deletion of received data and output files after a retention period

## System Requirements

| OS | Status |
|---|---|
| Windows x64 | Supported (this release) |
| macOS (Apple Silicon) | Planned |
| macOS (Intel) | Planned |
| Linux x64 | Planned |

- Supported Windows versions: Windows 11 or later
- Automatic printing requires an application that can open PDF files (the default PDF app). Printing goes to the OS default printer

## Download

Download the file for your OS from [Releases](https://github.com/acrossreport/acr-viewer/releases).

- Windows x64: `acr_viewer-v0.1.0-win-x64.zip`

## Installation and Launch

1. Extract the downloaded zip to any folder
2. Run `acr_viewer.exe`. An icon appears in the system tray
3. Choose "Open Web Settings" from the tray icon menu to open the settings page in your browser (`http://localhost:8765/`)

On first launch, the following folders are created automatically under "Documents" (you can change them on the settings page).

| Folder | Purpose |
|---|---|
| `AcrViewer\Watch` | Watch folder (drop files here) |
| `AcrViewer\Templates` | Design definition folder |
| `AcrViewer\Output` | PDF / PNG output |
| `AcrViewer\Processed` | Processed files |
| `AcrViewer\Error` | Data that could not be processed |

## Tray Menu

| Menu Item | Description |
|---|---|
| Auto Print | Toggle automatic printing on/off |
| Save PDF | Toggle PDF saving on/off |
| Open Web Settings (Folders, Printer, Port) | Open the settings page in your browser |
| Exit | Quit AcrViewer |

The display language follows the language chosen on the settings page (日本語 / English / Français). The change takes effect after restarting AcrViewer.

## Usage

### Method 1: Specify the definition in the data file (recommended)

1. Place the design definition file (e.g. `invoice.json`) in the **design definition folder**
2. Write the definition file name in the header (`Parameters`) of the data file

```json
{
  "Parameters": {
    "TemplateFile": "invoice.json",
    "...": "...",
    "Data": [ ... ]
  }
}
```

3. Drop this data into the **watch folder** as `name.data.json`; it is rendered, printed, and saved automatically
4. Successfully processed data is moved to the processed folder; data whose definition is not found or that fails to render is moved to the error folder

Do not use `TemplateFile` as a report field name (it is reserved).

### Method 2: Drop the definition and data as a pair

Drop a pair with the same name into the watch folder. It is processed once both files are present.

```
name.template.json   … design definition
name.data.json       … data
```

If a `.template.json` with the same name is in the watch folder, Method 2 takes priority.

### Output file name

```
yyyymmddhhmmss_definitionname.pdf
```

### HTTP API

- `POST /api/print`: send `{ "template": {...}, "data": {...}, "design_name": "..." }` to render, save, and print
  - If `template` is omitted, the definition is read from the design definition folder using `Parameters.TemplateFile` in `data`

The HTTP server listens on `0.0.0.0`. Use it in an environment that cannot be accessed from outside networks.

## About Output

If the license is not registered, output (PDF, PNG, printing) includes a watermark. Register your email address and license key under "License Registration" on the settings page to output without a watermark. See the [official website](https://acrossreport.com) for details.

## Links

- ACR Designer: https://github.com/acrossreport/acr-designer
- ACR Generator: https://github.com/acrossreport/acr-generator
- ACR specification (JSON template): https://github.com/acrossreport/acr-spec
- Official website: https://acrossreport.com

## License

The source code of this software is not publicly available. Please see [LICENSE](LICENSE) for the terms of use.

## Contact

across.support@gmail.com

---

© Across Systems Corporation
The intermediate drawing instruction architecture of ACR is patent pending.
