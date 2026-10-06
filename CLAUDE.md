# bitsnpicas (fork)

Notes for a Claude Code session working in this fork of Kreative BitsNPicas. The upstream is https://github.com/kreativekorp/bitsnpicas. This file is the owner's, not upstream's: it is tracked on the branch `private` alone, and must not get into a pull request.

## Why the owner cares

He makes Windows bitmap fonts (`.FON`) and pixel TrueType fonts of retro computers with `C:\github\public\retro-tools\pxfont.py`, and uses bitsnpicas as the GUI editor in between. `pxfont unpack FILE.fon` writes each font of the file as a `.fnt` beside its other pictures, bitsnpicas edits the `.fnt`, and `pxfont fon` packs the directory again; in the directory `fon.json` rules over the header of a `.fnt`, and `pxfont` only warns of the fields in which the file differs.

## Branches

- `master` only ever follows the upstream; no commit of the owner's goes there.
- One branch for each pull request, listed under the state below. A fix to a feature is committed on the branch of that feature.
- `private`, from `master`: what is the owner's alone and never goes upstream, which is this file and `.gitignore` for the built jars.
- `mine`: `master` with one merge of each of the other branches. It is what the owner builds and runs. Nothing is committed on it directly: after a commit on another branch, that branch is merged into `mine` again.

`rerere.enabled` is set in this clone, so a conflict resolved in one merge is resolved by itself in the next.

When the upstream accepts a pull request: update `master`; rebase each branch that is left onto it (commits accepted as they were drop out; for squashed ones name the range with `rebase --onto`; once `fnt-fixes` is in, `fnt-improvements` moves onto `master`); delete the accepted branch; then build `mine` anew, by resetting it to `master` and merging the branches that are left, `private` among them. `mine` is made again and not rebased, as it is nothing but merges. All of this rewrites history, so it is done only when the owner asks, with the commits named first.

## State, 2026-10-06

Six branches hold work meant for separate pull requests, written in the style of the upstream history (a short capitalized title, no prefix). The first three are stacked, each on top of the one before, as each touches lines of the one before, the next two are each on top of `font-properties`, and the last one is on `master` by itself; so the pull requests go in this order, or the later ones are rebased when an earlier one is refused.

`fnt-fixes`, from `master` at `43b11dc`: the plain bug fixes.

1. "Fix em size and internal leading of Windows FNT fonts". The importer took `dfPoints` for the em in pixels; the em is now the cell less `dfInternalLeading`. The exporter writes the internal leading that the em and the line height make, and the point size of that em at the resolution.
2. "Keep default and break chars of exported FNT fonts within the font". They were set to the entry after the last char. The break char falls back to the first char, the default char to the break char. A font with no chars in the chosen encoding is refused (it divided by zero).
3. "Import default char of FNT fonts as glyph of missing characters". The importer puts a copy of the glyph of `dfDefaultChar` into the font as the named glyph `.notdef`, which the TrueType exporter writes as glyph 0; without it a char that the font lacks drew nothing and had no width. The branches above `fnt-fixes` were rebased onto it on 2026-10-06; "Keep header fields of FNT fonts as properties" conflicts with it in the two lines of the importer that read the default and break chars (both are read), which `rerere` remembers.

`fnt-improvements`, on top of `fnt-fixes`:

1. "Prefer the glyph at 0x7F as default char of exported FNT fonts", as Windows fonts and the owner's fonts have it.
2. "Write FF_MODERN as family of fixed-pitch FNT fonts".
3. "Select encoding of imported FNT fonts by their character set". Without an encoding told, the importer takes that of `dfCharSet` (`getEncodingName()` in `FNTBitmapFontImporter`), and the import dialog offers it as the default. Charset 0 keeps the old built-in CP1252 mapping.
4. "Show the whole line height in cells of the glyph list". `edit/GlyphList.java` scaled a cell to the em alone and cut what is above it; this one touches all formats.
5. "Add average width option to FNT export". The export dialog has a panel of its own for FNT (`edit/exporter/BitmapExportFNTPanel.java`, the card "fnt"): the encoding and "Average Width (0 = Auto)", which goes to the exporter through `getFNTAverageWidth()` of `BitmapExportOptions`. The command line has no such option yet.
6. "Offer the encoding of the window when exporting FNT". The export menu item passes the name of the model that the font's window shows; when it is an encoding, the FNT panel offers it in place of CP1252. Other formats keep their defaults.

`font-properties`, on top of `fnt-improvements`:

1. "Add free-form properties to fonts". `Font` has a sorted map of properties, name to value, with `getProperty()`, `setProperty()`, `removeProperty()`, `containsProperty()` and `properties()`, in the manner of its names. The font makes nothing of them. The native formats (`.kbitx` and `.kpcax`) keep them among their `prop` elements; the ids of the eight metrics stay metrics.
2. "Edit font properties in the font info window". A third tab, Properties (`edit/FontInfoPropertiesPanel.java`): a text area, one `name=value` to a line; a line without `=` or without a name is dropped.
3. "Keep header fields of FNT fonts as properties". The importer sets `fnt.charSet`, `fnt.vertRes`, `fnt.horizRes`, `fnt.family` (DontCare, Roman, Swiss, Modern, Script, Decorative), `fnt.avgWidth`, and `fnt.defaultChar` and `fnt.breakChar` as codes of the chars (first char added). The exporter uses each where it still fits the font and falls back to its own logic otherwise; without an encoding given it takes that of `fnt.charSet`, and so does the FNT panel of the export dialog when the window shows no encoding.

`pxfont-png-export`, on top of `font-properties`:

1. "Add export of pxfont font sheets". `exporter/PxfontBitmapFontExporter.java` writes the PNG sheet that `pxfont.py` reads: the chars of a single-byte encoding, 32 to a row, each at its code modulo 32, rows from that of the first char to that of the last; navy on white and black on light gray by turns; a code without a char transparent. By the properties of a font imported from FNT: the cell at 0x7F has no paper when `fnt.defaultChar` is 127 and the cell is as wide as the empty places are; an empty place is as wide as `fnt.avgWidth` when the font is not of fixed pitch (else as most chars are); and the encoding is that of `fnt.charSet` when none is given. It is "PNG (pxfont font sheet)" in the export dialog and `pxfont` for `-f` of convertbitmap, and in the README's list of output formats.
2. "Add import of pxfont font sheets". `importer/PxfontBitmapFontImporter.java` reads a PNG sheet with nothing beside it (no `fon.json`), so it works out what `pxfont.py` is told: the height of a row (the largest for which no column of a row is in two colorings), the width of an empty place (that of most cells if it fits the transparent runs and the turns of the colorings, or else the widest that fits), and the first row (that of the cell without paper is the row of 0x7F; or else eight rows start at 0 and fewer at 32). A sheet in other colors is taken only when its file name tells the cell size with the width, as `zx 6x8px.png`, and the image is 32 such cells wide; its rows start at 32, seven at most. The ascent and the internal leading are measured on E, F, H, L, T, Z; the default char is 0x7F when the sheet shows it so (and always in other colors), set as `fnt.defaultChar` and as the `.notdef` glyph. The format is recognized in both import lists before the generic image import. A `.psd` cannot be read: Java has no reader for it. Tested by `pximp/pximptest.py` of the scratchpad: all 77 sheets of the corpus come back with the chars of their fonts, and with the same ascent; the internal leading and the default char are those of the font only where the sheet can tell them.

`fon-support`, on top of `font-properties` (not of `pxfont-png-export`): Windows `.FON` files, as the owner designed it on 2026-10-05; see the section on it below.

1. "Read and write Windows FON files". The package `fon/`: `FONFile` (the container: the fonts as bytes, the rest as properties), `FONDirectory` (split and merge), `FNTHeader` (what the container needs of a font).
2. "Add splitfon and mergefon commands". `main/SplitFON.java`, `main/MergeFON.java`, and a section of the README.
3. "Import fonts of Windows FON files". `importer/FONBitmapFontImporter.java`, in `main/BitmapInputFormat.java` and `edit/importer/ImportFormat.java`. The window that asks for the encoding (`EncodingSelectionFrame`, which got an optional note for this) tells that the fonts are opened as copies.
4. "Export font as Windows FON file". `exporter/FONBitmapFontExporter.java`: one font, as FNT of version 2; `fon` for `-f` of convertbitmap, "FON (Windows 3.x)" in the export dialog, with the FNT panel.
5. "Split and merge Windows FON files from File menu". The submenu Windows FON (`CommonMenuItems.WindowsFONMenu`) after Open in all ten File menus; the actions are `splitFON()` and `mergeFON()` of `edit/Main.java`.

`ttf-line-height-em`, from `master`, independent of the others:

1. "Add option to use line height as em size of TTF fonts". The em of an exported TrueType font is the em of the bitmap font; with the option it is the line height, as `pxfont ttf` has it, so that a font with a cell of 8 pixels is exact at 8, 16, 24 pixels and not at 7, 14, 21 (the em of a font imported from FNT is the cell less the internal leading). `-l` of convertbitmap (`-L` is the default), a check box in the TTF panel of the export window, the fourth argument of the constructor of `TTFBitmapFontExporter`. The OTB exporter is as it was.

`mine` is `master` with a merge of each of these six, in the order above, and of `private` last: so the first parent of the tip of `mine` has every change and nothing of `private`, and that is the commit to tag for a release.

Every commit of the six branches compiles by itself at the Java level of the Makefile (checked on 2026-10-06, 20 commits), and none has a file outside `src/` and `README.md`.

Jars in the fork's root, ignored by git on `private` and `mine`: `BitsNPicas-mine.jar` is a build of `mine`; the other `BitsNPicas-*.jar` are earlier builds, and `BitsNPicas.jar` an old one of 2023-02-02.

## What is still open

- `.FON`: a file of several fonts cannot be saved from the editor; it is split, changed and merged. A second font cannot be added to a `.FON` but as a `.fnt` file in the folder. The two menu items were not operated by hand: the dialogs were never seen, only the code under them was tested through the commands.
- `.FON`: other strings of a version resource than the eight, a second string table, the file flags and the padding of a font resource beyond its `dfSize` are dropped by a split.
- Of a `.fnt` header, what is still not kept: `dfPoints` (worked out from the em and the resolution, which differs from the file for 11 of the 77 test fonts), the device name, `dfPixWidth` of a font declared fixed-pitch that has chars of other widths, and `dfMaxWidth` when the file tells another than the widest char.
- The properties have no editor but the text area, and nothing checks what is typed there.
- The average width option is in the dialog alone; convertbitmap has none.
- Names are read and written as CP1252 whatever the encoding of the chars is.

## Sources

Under `main/java/BitsNPicas/src/com/kreative/bitsnpicas/`:

- `importer/FNTBitmapFontImporter.java` and `exporter/FNTBitmapFontExporter.java`: bare `.fnt` files, versions 2 and 3.
- The format is registered in four lists: `main/BitmapInputFormat.java`, `main/BitmapOutputFormat.java`, `edit/importer/ImportFormat.java`, `edit/exporter/BitmapExportFormat.java`.
- `main/DebugWinFNT.java`: a dump tool with the charset constants.
- `fon/`: the `.FON` container. `FONFile.read()` and `FONFile.write()`, `FONDirectory.split()` and `FONDirectory.merge()`. A property of the container is one that `FONFile.isProperty()` knows.

The sources are indented with tabs, and a blank line inside a method keeps the indent of its neighbors.

## Building and testing

There is no `make` on this machine; the Makefile's steps by hand, from `main/java/BitsNPicas/`:

    find src -name '*.java' > files.txt
    javac -nowarn -source 8 -target 8 -encoding UTF-8 -classpath dep/ual.jar -d BIN @files.txt
    (copy every file of src/ that is not .java into BIN/, at the same path)
    "/c/Program Files/Java/jdk-22/bin/jar.exe" cmf dep/MANIFEST.MF OUT.jar -C BIN com/kreative/unicode -C BIN com/kreative/bitsnpicas

Put BIN and the jar outside the repository. `-source 8 -target 8` is what `dep/minJavaVerOpts` gives with JDK 22; without it the jar needs Java 22 to run. The command-line converter runs without a window:

    java -Djava.awt.headless=true -jar OUT.jar convertbitmap -oe CP1251 -f fnt2 -o OUTDIR FILE.fnt

How the branch was tested, and how to test again:

- The corpus: the 73 `.fnt` files that `pxfont unpack` writes of the 20 `.FON` files in `C:\github\public\my-public\FonEd\stuff\`, and those of the owner's fonts (`C:\github\public\retro-tools\fonts\`, made with `pxfont fon --unpack` from the `.psd` sheets).
- Each file was imported and exported as FNT of the same version, with the output encoding of its charset and no input encoding told, and the result parsed with `parse_font()` of `pxfont.py`: all 77 keep their glyphs, chars and height; all have the default and break chars within the font; a `.kbitx` export shows the em equal to the cell less the internal leading for all 77.
- For the owner's fonts the saved `.fnt` packs with `pxfont fon` both as a lone file and in its directory, and the directory gives the original `.FON` back byte for byte. For his four current fonts the header saved by bitsnpicas equals `fon.json` in every field.
- With the properties, the header fields that still differ after a pass through bitsnpicas are `dfMaxWidth` (16 fonts), `dfReserved` (12), `dfPoints` (11), `dfPixWidth` (6), the pitch bit (4), `dfWeight` (4) and `dfItalic` (2); the default and break chars, the resolution, the family and the average width come back for all 77.
- The sheet exporter: each of the 77 fonts was exported as a sheet with no encoding told, the sheet put into a copy of the font's directory in place of its three pictures, and `pxfont fon` gave the original `.FON` back byte for byte for all 77; all 77 sheets equal the PNG of `pxfont.py` pixel for pixel. The owner's four fonts, exported and packed as lone sheets, give the same `.fon` as their `.psd` sheets do.
- The properties: a save and a load of `.kbitx` and of `.kpcax` keep them; the text area was rendered off-screen and its parsing tried.
- The glyph list was rendered off-screen by a small class that makes a `GlyphList` of an imported font and paints it into an image; this needs a display (the class `CommonMenuItems` fails headless).
- `.FON`: each of the 20 sample files was split with `splitfon` and merged with `mergefon`. All 20 give the `.fnt` files that `pxfont unpack` gives, byte for byte; `pxfont unpack` takes all merged files (19 cleanly, and `MODERN.FON` as it takes the original, whose fonts are vector ones); the version strings, the module name and the description are those of the original for all 20; a second split gives the same fonts and `fon.txt`; and Windows loads all 20 merged files with all their fonts (`AddFontResourceExW()` with `FR_PRIVATE`, which is for the asking process alone and installs nothing). 18 have the fonts in the order of the original; `DOSAPP.FON` and `SMALLE.FON` do not. No merged file equals its original byte for byte, which is not a goal.
- `.FON`, the rest: a char that the code page lacks, a wrong number, a text that is not UTF-8 and an unknown name in `fon.txt` are told as designed; a font of a one-font `.FON` keeps the properties through `.kbitx` and an export as `.FON`; the window that opens a `.FON` was rendered off-screen.

## .FON support

As the owner designed it on 2026-10-05, and as it is on `fon-support`. A `.FON` is treated as a bundle of `.fnt` files, with as little GUI as can be:

- Splitting (`splitfon`, or File, Windows FON, Split into FNT Files) writes each font of `X.FON` as a `.fnt` into the folder `X.FON.files/` beside it, with `fon.txt`. A folder that exists is not written into: the command refuses it unless `-f` is given, the menu item asks; then every file of the folder is deleted first, whatever it is, and a folder with a directory in it is refused all the same (`FONDirectory.split()` with `replace`). Merging (`mergefon`, or Merge FNT Files) makes `X.FON` of the folder `X.FON.files/`, or `NAME.fon` of another folder `NAME/`, with a container made anew. An existing `.FON` is asked about in the GUI and replaced by `mergefon`; no `.BAK` files.
- The `.fnt` files are copied byte for byte both ways, not parsed and made again, so nothing of a font is lost, and vector fonts pass through though bitsnpicas cannot edit them. A font resource is cut to its `dfSize`.
- There is no list of fonts: merging takes every `.fnt` of the folder, sorted by pixel height, then weight, then upright before italic, then average width, then file name, and numbers them from 1. No numbers go into file names. The fonts are named by face, size and style, as `pxfont.py` names them.
- What the container has and the fonts do not is in `fon.txt`, optional, one `name=value` to a line as the Properties tab has them, in UTF-8: `fon.companyName`, `fon.fileDescription`, `fon.fileVersion`, `fon.internalName`, `fon.legalCopyright`, `fon.originalFilename`, `fon.productName`, `fon.productVersion` (the eight version strings), `fon.fileVersionNumber` and `fon.productVersionNumber` (up to four numbers with periods), `fon.moduleName`, `fon.description` (the `FONTRES ...` line), `fon.windowsVersion` (as 3.10), `fon.language` (four hex digits) and `fon.codePage` (a number). A name that is not known, or a line that is not `name=value`, gives a warning and is left out.
- Without the file or a line of it: language `0409` and code page `1252`, whatever the charset of the fonts; `FileDescription` is the face name with " font" after it, `InternalName` and `ProductName` the face name; `LegalCopyright` the copyright of the first font; `OriginalFilename` the file name; both versions 1.0; the module name the letters and digits of the file name, eight at most; the description `FONTRES 100,` with the resolution, the face and the point sizes; Windows 3.10.
- The strings are converted to the code page that `fon.codePage` tells: it is a field of the version resource (the string table is keyed by language and code page, as `040904E4`), not a given, though all 20 sample files have 1252 there. A char that the code page lacks stops the merging, with a message that tells the char, its code point and its line and column in `fon.txt`.
- A `.FON` can be opened directly, which is an import: each font in a window of its own, and no saving back into that `.FON`; the window that asks for the encoding says so. The font of a file that has no other gets the `fon.` properties, and exporting a font as `.FON` (one font to a file) honors them.
- `pxfont.py` and bitsnpicas keep each its own unpacked form of a `.FON` (`fon.json` there, `fon.txt` here) and share only the `.fnt` files and the name of the folder; so replacing a folder on a split deletes the sheets and `fon.json` that `pxfont unpack` wrote there. `pxfont.py` is not to be changed for this: it also packs PNG and text sheets, which is another matter.

An exact copy of the original file is not a goal: the version resource of real files has quirks that a flat list cannot tell (NULs after a string, leftover bytes in padding), and the checksum of the NE header, the ordinals and the order of the fonts are made anew. That is what `pxfont.py` is for; it is also the reference for the format, as it reproduces the 20 sample files byte for byte and `pxfont unpack` shows every field of a file in `fon.json`.

## GitHub and releases

The owner's fork is https://github.com/mike-shevchenko/bitsnpicas, made on 2026-10-06; it is the remote `origin` of this clone, and the upstream is the remote `upstream`. Every branch and the release tag were pushed there on 2026-10-06, at the owner's word; a push is made only when he says so, in so many words. `private` and `mine` are public there too, by his decision.

A release is the jar alone, `BitsNPicas.jar`, built at Java 8 level from a clean export (`git archive`) of the tagged commit, with notes that list the changes by branch. The tag is on the first parent of the tip of `mine`, named after the upstream version: `v2.2.2-ms.1`, local, at `6853dd9`. Its release was made as a draft with `gh release create --draft` and published with `gh release edit --draft=false` once the tag was pushed: GitHub refuses a draft whose target is a commit that it does not have (HTTP 422), so a draft made before the push has `master` for target, which is not used once the tag exists. A later release is a detached merge of the feature branches onto `master`, tagged, built and published the same way.
