## Quick orientation — what this repo is

This repository contains a Microsoft Dataverse / Power Platform solution exported as files (unpacked solution). Key facts you can read from the files:

- Solution UniqueName: `MediaEventsVenues` (see `Other/Solution.xml`).
- Solution Version: `2.0.3.5` and Publisher CustomizationPrefix: `msmedia` (see `Other/Solution.xml`).
- The solution has external dependencies (e.g. `MediaCommon (2.0.2.0)`) listed in `Other/Solution.xml` under MissingDependencies — verify these before import.

## Top-level layout (important paths)

- `Entities/` — one folder per entity. Each entity folder contains `Entity.xml`, `FormXml/`, `Formulas/` (XAML rollups), `RibbonDiff.xml`, `SavedQueries/`, `Visualizations/`.
  - Example: `Entities/msmedia_MediaEvent/Entity.xml` (attributes, optionset references and `FormulaDefinitionFileName` entries).
- `CanvasApps/` — canvas apps (.msapp) + `.meta.xml`. Example: `msmedia_imagegallerymediavenue_1ded7_DocumentUri.msapp` and `msmedia_imagegallerymediavenue_1ded7.meta.xml`.
- `OptionSets/` — global option set XML files (e.g. `msmedia_eventformat.xml`, note `msmedi_timezones.xml` has a slightly different prefix).
- `WebResources/` — SVG images and their `.data.xml` metadata; also grouped SVG bundles like `msmedia_AttractionsSVG`.
- `Other/` — `Solution.xml`, `Customizations.xml`, `Relationships.xml` — the canonical solution manifest and missing-dependency info.
- `Resources/`, `Dashboards/`, `Workflows/`, `Extracts/` — other solution artifacts (dashboards are XML definitions, see `Dashboards/{guid}.xml`).

## Naming and conventions discovered in this repo

- Publisher prefix: `msmedia_` is the dominant prefix for custom entities, attributes and web resources. Keep this prefix when you add new custom components.
- OptionSet filenames generally start with `msmedia_` (one exception: `msmedi_timezones.xml`) — be careful to match the OptionSet logical names used by `Entity.xml` (the `OptionSetName` attribute).
- Canvas apps include `DatabaseReferences` in their `.meta.xml` (they declare which entities they depend on). Example: the image gallery app depends on `msmedia_mediavenue` and `msmedia_mediavenueimage`.

## Code / artifact patterns agents should know

- Entity custom attributes and metadata live in `Entities/<Entity>/Entity.xml`. Many attributes reference `OptionSetName` and/or `FormulaDefinitionFileName` (computed/rollup logic).
- Formulas are XAML workflows used as rollup/aggregate rules. Look under `Entities/<Entity>/Formulas/*.xaml` (e.g. `msmedia_mediaevent-msmedia_numberregistered.xaml`). These are not simple JS/Python — they are CRM workflow XAML.
- UI definitions: `Entities/<Entity>/FormXml/` holds form XML; `RibbonDiff.xml` holds ribbon customizations; `Visualizations/` has charts.
- Web resources: each `.svg` usually has a matching `.data.xml` file. Edit the `.svg` and keep the `.data.xml` in sync if you change ids, dimensions or metadata.

## Recommended minimal workflows (how humans/agents should make and validate edits)

1. Inspect `Other/Solution.xml` for MissingDependencies and the `CustomizationPrefix` before making changes.
2. Make a targeted change (example: small text change in an entity label) by editing `Entities/<Entity>/Entity.xml` and the appropriate `FormXml/*` file.
3. If you change an image, edit `WebResources/msmedia_/_imgs/*.svg` and keep its `.data.xml` paired file.
4. Do NOT hand-edit `.msapp` binary blobs. To modify CanvasApps use Power Apps Studio and re-export or use the Power Platform CLI / Solution Packager to unpack/pack Canvas apps and the solution.

Example (pack/unpack using Power Platform tooling — verify flags for your installed toolchain):

```
# pack solution (example)
pac solution pack --folder .\MediaEventsVenues --zipfile MediaEventsVenues.zip

# unpack solution
pac solution unpack --zipfile MediaEventsVenues.zip --folder .\MediaEventsVenues-unpack
```

If you don't have `pac`, SolutionPackager (SDK) can be used similarly — this repo stores the unpacked structure that SolutionPackager produces/consumes.

## What to avoid / gotchas (from repo inspection)

- MissingDependencies in `Other/Solution.xml` means the solution is not standalone — imports will fail unless required solutions exist in the target environment.
- Many attributes use `msmedia_` prefix and Application-level defaults; do not change the publisher prefix or schemaNames without understanding downstream impact.
- Formulas (XAML) are brittle if edited incorrectly — changes to `Formulas/*.xaml` should be tested in a sandbox environment.
- Canvas apps (`.msapp`) are binary; editing them directly in text will corrupt them.

## Where to look for examples in this repo

- Solution manifest: `Other/Solution.xml` (version, publisher, missing deps).
- Entity example: `Entities/msmedia_MediaEvent/Entity.xml` (attributes, option set usage, formula references).
- Canvas app example: `CanvasApps/msmedia_imagegallerymediavenue_1ded7.meta.xml` + `*.msapp`.
- Web resources: `WebResources/msmedia_/_imgs/*.svg` and `WebResources/*SVG` bundles.

---

If any section is unclear or you want the file to include CI/CD examples (pac auth, import to environment, or a scripted SolutionPackager workflow), tell me which tooling you use and I will add a short, exact checklist and commands for that toolchain.
