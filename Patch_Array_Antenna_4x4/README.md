# Patch antenna and 4×4 patch array for Keysight ADS Momentum

Two independent packages built from the RF & Microwave Engineering Lab workbook, PDF pages 75–86:

| Package | Workbook | Result in ADS |
|---|---|---|
| `Single_Patch_Antenna.zip` | Part 1, pages 75–81 | cell `Single_Patch_Antenna`: inset-fed 29.2 mm patch, 1 port |
| `Patch_Array_Antenna_4x4.zip` | Part 2, pages 82–84 | cell `Patch_Array_Antenna_4x4`: 16 patches on a 50 mm pitch, 16 ports |

Each package holds an AEL script that builds the native layout inside ADS. Neither depends on the other.

**Nothing here has been run in ADS.** No ADS installation was available, so there are no simulated results in these packages, and no claim that the script loads without error on your version. What was checked, and how, is in section 6.

## 1. Which ADS version this targets

ADS 2011 or later, with a Layout and a Momentum licence. That is every release that uses workspaces (`*_wrk`) and OpenAccess libraries.

- The script calls 32 AEL functions. All 32 are documented in Keysight's ADS 2011.01 AEL manual, and the ones that create the design are also in the ADS 2025 help, marked "ADS 2011 and newer versions".
- The workbook screenshots show "Agilent Momentum Visualization", which dates them to ADS 2011–2014. The files written here use the oldest form of each format, taken from workspaces of that era, because newer ADS opens old workspaces.
- ADS 2009 or earlier (`*_prj` projects) cannot run the script. Use the import files in section 7.

## 2. How to build each project

Do this once per package. The steps are the same; only the names differ.

1. Unzip to a folder you can write to, with no spaces in its path, for example `C:/ADS_Labs/Single_Patch_Antenna`. ADS stores a compiled copy of the script beside it.
2. In ADS: **File > New > Workspace**. Name it `Single_Patch_Antenna_wrk` (or `Patch_Array_Antenna_4x4_wrk`). Keep the default library. For the layout technology choose **Standard ADS Layers, 0.0001 millimeter layout resolution**. This is workbook step 2.1; AEL has no documented way to create the technology, so it stays manual.
3. In the ADS Main window: **Tools > Command Line…**, type the line below with your own path and forward slashes, then press **Apply**.

   ```
   load("C:/ADS_Labs/Single_Patch_Antenna/build_Single_Patch_Antenna.ael");
   ```
   ```
   load("C:/ADS_Labs/Patch_Array_Antenna_4x4/build_Patch_Array_Antenna_4x4.ael");
   ```
4. The workspace closes and reopens, the layout window appears, and a dialog lists what was written and what to do next. Lines starting with `!` are steps that did not complete.

The script refuses to run if the cell already has a layout, and it never overwrites an existing substrate, materials file, EM setup or data display.

## 3. How to run the EM simulation

In the layout window:

1. **EM > Simulation Setup…** Check that Substrate shows `Patch_FR4_1p6mm` and the frequency plan shows the adaptive sweep (2.1–2.7 GHz for the patch, 2.3–2.5 GHz for the array).
2. **Frequency plan > Add**, Type **Single**, **2.4 GHz**. For the patch this is workbook step 3.3. For the array the workbook does not ask for it; add it anyway so that 2.4 GHz is certain to be a solved frequency for the far field.
3. Click **Simulate**.

Ports need no setup. ADS creates one 50 Ω port per layout pin, which is what the workbook relies on (it never opens the port editor).

## 4. Where to see the results

**S-parameters.** ADS opens its own S-parameter display when the simulation ends. For the other plots open **File > Open > Data Display** and pick the file the script copied into the workspace:

| File | Contents |
|---|---|
| `Single_Patch_Antenna_Results.dds` | S11 in dB (fitted curve and solved points, as in the workbook plot), VSWR, Smith chart, input impedance, minimum S11, return loss, resonant frequency |
| `Patch_Array_Antenna_4x4_Results.dds` | S(1,1) fitted and calculated, the complete 16×16 matrix `dB(S)`, all 16 reflection coefficients, coupling into corner element 1 and inner element 6, Smith chart |

If a plot is red, the dataset name differs on your version: choose `<cell>_MomUW_a` in the dataset list at the top of the display.

**Radiation pattern** (workbook step 4; this is a post-processing action in the GUI and cannot be scripted):

1. **EM > Post-Processing > Far Field**, select **2.4 GHz**, click **Compute**.
2. In the visualization window: **Plot Properties > Far Fields > Antenna Parameters** gives gain, directivity, radiated power and efficiency.
3. **Far Field Cut** tab: enable, select **Phi**, click **Display Cut in Data Display**. ADS then builds the gain, directivity and efficiency display itself.

Menu names are from the workbook's ADS version and may be worded differently in newer releases.

**Array excitation.** The simulation solves the 16 ports one at a time and produces a 16×16 S-matrix. That alone is not an array pattern. The far field is a superposition in which you choose each port's source, and the Far Field Computation Setup dialog lists every port with an amplitude, a phase and a source impedance.

- For the workbook's pattern, a single beam normal to the board with four side lobes, every port must have the same amplitude and phase: **1 V, 0 deg on all 16 ports**. Select each port in the list and check it before pressing Compute. I could not confirm what the dialog defaults to for ports 2–16.
- The workbook screenshot shows a source impedance of 0 Ω and gives no other value, so leave it to reproduce the workbook. Entering 50 Ω on every port instead models sixteen matched 50 Ω generators; the pattern shape barely changes but gain and efficiency will.
- To see one element's pattern inside the array, set one port to 1 V and the other fifteen to 0 V with 50 Ω.

## 5. What is left for you to do

| Step | Why it is not automated |
|---|---|
| Create the workspace with the wizard | No documented AEL function sets up a library's layout technology |
| Add the Single 2.4 GHz frequency row | How ADS stores a "Single" row could not be confirmed from any real setup file, so it was not guessed |
| Compute the far field and the Phi cut | GUI post-processing with no documented script interface |
| Check the 16 port excitations for the array | Same dialog |

## 6. What was actually validated

All checks are static and were run by `_generator/generate_ads_packages.py`. The full list is in each package's `reference/VALIDATION_REPORT.txt`; every check passed.

- **Geometry.** The polygon equals the workbook list vertex for vertex, is simple (no self-intersection), has only horizontal and vertical edges, and has area 849.04 mm². Patch 29.2 × 29.2 mm, inset 12 mm, feed 3.0 mm, slots 1.4 mm. The array has 16 identical translated copies on the stated offsets, with 10.8 mm minimum clearance.
- **Ports.** Each pin sits on the midpoint of its 3.0 mm feed-line end edge, on layer cond, numbered P1–P16 row by row.
- **Substrate.** FR4 1.6 mm, Er 4.6, TanD 0.001, ground plane below, open air above, Copper 5.8e7 S/m, 35 µm, thick conductor set to intrude, mapped to layer cond. The script writes these files byte for byte as checked.
- **AEL script.** Brackets and quotes balance, every statement is terminated, and every function called is on the list verified against the Keysight manual.
- **EM setup and data displays.** Well-formed, and they use only elements and properties found in files that ADS itself saved.
- **Import files.** The GDSII file was read back and matches the geometry.

Confidence differs by part, because none of it was executed:

| Part | Basis | Confidence |
|---|---|---|
| Layout, copper, pins, save | Documented AEL functions, plus working public AEL code that uses the same calls | High |
| Substrate and materials files | Same structure as files ADS wrote in several public workspaces, including one FR4/Copper board of the workbook's era | Good |
| Data displays | Object blocks copied from an ADS-saved display; equations use documented functions | Good |
| EM setup written by the script | Derived from an ADS-saved setup, with frequencies, names and layer list changed and the cached port list emptied | Least certain |

If the EM setup gives an error when opened: close the workspace, delete the folder `<library>/%Single_%Patch_%Antenna/em%Setup` (or `%Patch_%Array_%Antenna_4x4/em%Setup`), reopen, and choose **EM > Simulation Setup** to let ADS create a fresh one. The substrate should already be selected (the script records it as the library default); if not, pick `Patch_FR4_1p6mm`. Then enter the frequency plan from section 3. To skip it from the start, set `SPA_WRITE_EM_SETUP = FALSE` (or `PAA_…`) at the top of the script.

Checklist for the first run in ADS:

1. The layout shows the patch (or 16 patches) with a pin on each feed end. Measure one patch: 29.2 mm wide, feed 3.0 mm.
2. **EM > Substrate…** shows FR4 1.6 mm over a cover, with `cond` on top as Copper 35 µm, intruding.
3. EM Setup lists 1 port (or 16), each 50 Ω.
4. The simulation finishes and S11 dips near 2.4 GHz.

For comparison, the workbook prints these values (they are the workbook's, not results from these files): patch S11 about −23 dB at 2.4 GHz, directivity 6.29 dBi, gain 4.65 dBi, efficiency 68.6 %; array S(1,1) about −9.4 dB near 2.396 GHz.

## 7. If the script cannot be used

`fallback_import/` holds the copper as GDSII and DXF, plus a coordinate list.

1. Create the workspace and an empty layout cell.
2. **File > Import**, choose GDSII (layer 1 maps to `cond`) or DXF (units millimetre, layer `cond`).
3. Insert a pin on `cond` at each position in `<name>_coordinates.txt`.
4. **EM > Substrate…**: new substrate from the 25 mil Alumina template; dielectric FR4, 1.6 mm, Er 4.6, TanD 0.001; `cond` as Copper, 5.8e7 S/m, Intrude Into Substrate, 35 micron. `ads_files/` has the same definition as files, to copy into the library folder instead.
5. Continue from section 3.

## 8. How the workbook was interpreted

- **Closing vertex.** The workbook list ends by repeating (0, 0). The polygon is drawn with the 12 distinct vertices and closed by ADS, which gives the same shape without a zero-length edge.
- **Pin position.** Page 77 shows the pin at the centre of the open end of the feed line, which is (14.6, −10) mm. Part 2 says "center of feed line"; its figure shows the pins at the same place, so the element is shifted to put that point at (0, 0).
- **Port numbering.** P1–P4 are the top row left to right, then P5–P8 and so on downwards, which is the order the workbook's copy steps produce.
- **Array sweep type.** The workbook gives only 2.3–2.5 GHz. Its plot shows fitted and calculated traces, so the sweep is adaptive.
- **No feed network.** The workbook puts a pin on every element, so the array has 16 separate ports and nothing combining them.
- **Loss tangent.** TanD 0.001 is used as printed. Real FR4 is nearer 0.02, so a built antenna would be less efficient than this model.
- **Element spacing.** The workbook calls 50 mm "0.75 λ". That is the wavelength in the substrate; in free space 50 mm is 0.4 λ at 2.4 GHz.
- **Names.** The workbook calls its workspace `Lab5_PatchAntenna_wrk` and its cell `Lab1_Patch`. The cells here carry the package names. The workspace name is yours to choose.
- **Ground and dielectric** extend to infinity sideways. That is how Momentum models a substrate and is what the workbook simulates.

## 9. Package contents

```
<name>/
  build_<name>.ael              the script to load in ADS
  ads_files/
    emStateFile.xml, master.tag EM setup, installed by the script
    <name>_Results.dds          data display, copied into the workspace by the script
    materials.matdb             reference copies of what the script writes
    Patch_FR4_1p6mm.subst
  fallback_import/              .gds, .dxf, coordinate list
  reference/                    geometry picture, VALIDATION_REPORT.txt
```

`_generator/` (next to the zips, not inside them) holds the Python script that produced everything and the two ADS-saved reference files it uses as templates.

## 10. Sources used to verify functions and file formats

- Keysight, *ADS 2011.01 – AEL* and *ADS 2011.01 – EM Simulation* manuals: `edadownload.software.keysight.com/eedl/ads/2011_01/pdf/ael.pdf` and `…/em.pdf`
- ADS 2025 online help (AEL function pages and Measurement Expressions)
- Dr. Mühlhaus Consulting, *Create customized artwork cells using AEL code*: `muehlhaus.com/support/ads-application-notes/ael-artwork`
- Public ADS workspaces on GitHub used as format references: `VolkerMuehlhaus/RFIC-Inductor-Toolkit-Open`, `lq2394/TNE089`, `vais09hnav/Microstrip-Patch-Antenna-ADS`, `danieltyukov/tud-electromagnetics`
