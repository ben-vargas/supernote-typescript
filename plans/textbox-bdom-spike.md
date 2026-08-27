# Text-box / Digest structural extraction research

## Question

Can the text-box and Digest content in
`textbox-n5-20260016-digest.note` be recovered as text, geometry, and styles
from the embedded MyScript iink `RECOGNFILE`, or from another `.note` record,
without reading the pixels in the affected regions?

## Conclusion

**Not from any structure identified in this fixture.** The strongest result of
this investigation is that `RECOGNFILE` is a handwriting-recognition package,
not the source model for the page's Supernote text boxes or Digest excerpt.
Its BDOM text fields, BINK labels, and captured-ink geometry all describe the
handwriting outside the page's `DISABLE` rectangles. They do not describe the
typeset content inside those rectangles.

The `.note` itself provides:

- exact page-pixel rectangles in `DISABLE`;
- the rendered pixels in the ordinary `MAINLAYER` Ratta-RLE bitmap;
- a page-level `PAGETEXTBOX` flag;
- for the Digest on page 5, an adjacent external-PDF `LINKO_` record containing
  the source filename and source page number.

It does **not** expose an identified text-box/Digest object record, original
Unicode body, font/paragraph styles, or Digest source-selection range. Raster
preservation therefore remains the only lossless, self-contained rendering
path.

## Fixture inventory

Affected pages:

| Page | `PAGETEXTBOX` | `DISABLE` (`x,y,w,h`) | Visible raster-only object |
| --- | ---: | --- | --- |
| 2 | 1 | `63,341,1735,616` | text box |
| 4 | 1 | `67,425,1769,1859` | large text box |
| 5 | 1 | `126,919,1622,174` | text box |
| 5 | 1 | `84,195,1506,126` | Digest excerpt |

Pages 2, 3, 4, 5, and 7 have `RECOGNFILE`; pages 3 and 7 have no text box and
`PAGETEXTBOX=0`. Conversely, `PAGETEXTBOX` supplies no object count or object
type. This establishes that recognition packages and text boxes are independent
page features.

The complete current page metadata was inventoried. Apart from normal layer,
recognition, orientation, and page fields, the only relevant keys are
`PAGETEXTBOX`, `DISABLE`, and (on recognized pages) `IDTABLE`. There is no
`TEXTBOX`, `DIGEST`, object-list, or style-address key. The footer likewise
contains only `PAGE*`, `TITLE_*`, `LINKO_*`, `STYLE_*`, `FILE_FEATURE`, and
`DIRTY`; there is no Digest-specific footer group.

On affected pages, both the handwriting and the typeset pixels are flattened
into one ordinary `MAINLAYER` Ratta-RLE bitmap. `LAYERPATH`,
`LAYERVECTORGRAPH`, and `LAYERRECOGN` are all `0`; there is no separate object
layer or attachment to inspect.

`IDTABLE` is not a text-object table. It has one entry per ink stroke sent to
recognition (162, 38, and 51 entries on pages 2, 4, and 5), matching each page's
BINK stroke count. `TOTALPATH` has 168, 41, and 54 records respectively because
it also contains records not sent to MyScript. The ID-table payload contains
base64-encoded IDs/opaque stroke identifiers, not text, bounds, or styles.

## `RECOGNFILE` package

`RECOGNFILE` is a length-prefixed ordinary ZIP archive. The inspected archives
contain:

```text
meta.json
rel.json
index.bdom
pages/<id>/meta.json
pages/<id>/page.bdom
pages/<id>/ink.bink
pages/<id>/style.css
```

| Page | ZIP page ID | `page.bdom` bytes | `ink.bink` bytes |
| --- | --- | ---: | ---: |
| 2 | `qexziywj` | 30,209 | 457,521 |
| 3 | `tmhvmsze` | 62,634 | 288,235 |
| 4 | `yvlnearm` | 9,474 | 100,145 |
| 5 | `hpmalcyl` | 14,723 | 43,255 |
| 7 | `inpaexhl` | 87,595 | 791,701 |

Package metadata identifies MyScript iink 3.0.3, document version 1.11,
format 4.0, Android, locale `de_DE`, and page type `Raw Content`.
`index.bdom` is only a document/page skeleton. `rel.json` has one generic
`rectangle/1` object on every inspected page regardless of actual rectangle
count, so it is not a text-box or Digest index.

### BDOM text

`page.bdom` is proprietary BDOM version 2. Its header is followed by a
length-prefixed schema dictionary (216 entries in these packages), with names
including `textField`, `textBlock`, `textLine`, `word`, `char`,
`charCandidate`, bounds-related names, and generic style/layout names.
Those names describe what the format *can* represent; they do not prove that a
Supernote text-box object was serialized into this package.

The understood subset identifies `textField` and `charCandidate` elements. A
selected character candidate stores an inline UTF-8 label after a binary string
token. Grouping candidates by field yields:

| Page | BDOM text fields |
| --- | --- |
| 2 | `a.`; `=`; `M`; `aids`; `smishing → Sms-Phishing\nWuling (Whale-Phishing) → c- Level Phishing\nWishing → Phone-Phishing\nCEO-Fraud` |
| 4 | `'odcast\nScale-Up 360°` |
| 5 | `Test. [n P von Digest`; `Digest mit Link zum Buch` |

The imperfect labels are the candidates stored by MyScript, not decoding
errors. They are the same handwriting labels already present in
`RECOGNTEXT` and in BINK `DIAGRAM` JSON.

The object counts are also decisive:

| Page | BDOM `textField` | BDOM `textBlock` | BINK text `DIAGRAM` | Raster-only regions |
| --- | ---: | ---: | ---: | ---: |
| 2 | 5 | 5 | 5 | 1 |
| 4 | 1 | 1 | 1 | 1 |
| 5 | 2 | 2 | 2 | 2 |

Every BDOM field has a generated name such as `Text1785482311075360` that
matches a BINK `DWContentFieldName` (`1/Text1785482311075360`). There are no
additional text fields/blocks for the raster-only regions and no BDOM `image`
elements. The BDOM `drawingField`/`shapeField` entries correspond to recognized
free drawing and generic page structures, not the disabled rectangles.

A raw contiguous-string search alone is insufficient for BDOM because its
recognized prose is stored as candidate records, often one character at a
time. The field/object inventory above avoids that false negative. Within the
known object graph, however, the visible typeset body is absent.

### BINK labels and geometry

`ink.bink` contains:

- captured X/Y/F/T ink channels (X and Y are declared in mm);
- the captured pen strokes;
- recognition-tree nodes such as `TEXT_STROKES`, `TEXT_LINE`, `TEXT_BLOCK`,
  `WORD`, `CHAR`, and `TEXT`;
- Supernote `DIAGRAM` JSON with `DWShape`, `DWContentFieldName`, `DWTagId`,
  `DWLabel`, alignment, line-gap, and reflow flags.

The BINK `DIAGRAM` labels exactly match the BDOM fields. Stroke-range attributes
on each recognition node make geometry recoverable for that recognized
handwriting. Converting the millimetre coordinates with the N5 recognition
scale (approximately 11.9 page pixels/mm) gives these field extents:

| Page | Recognized field | Approximate ink y-range (px) | Relevant `DISABLE` y-range (px) |
| --- | --- | ---: | ---: |
| 2 | four short fields near page top | 160–279 | 341–957 |
| 2 | phishing notes | 1,055–1,483 | 341–957 |
| 4 | `'odcast / Scale-Up 360°` | 114–384 | 425–2,284 |
| 5 | `Digest mit Link zum Buch` | 16–95 | 195–321 |
| 5 | `Test. [n P von Digest` | 800–915 | 919–1,093 |

The separation is exact in intent, not merely a missing label: BINK has no
captured strokes in the typeset regions. MyScript is recognizing the
handwriting around each object, while `DISABLE` prevents those raster-only
objects from participating in the vector/recognition ink path.

Consequently, a fuller BINK decoder can provide character/word/line geometry
for handwriting, but cannot recover the text-box or Digest body from this
fixture.

### Styles

All five extracted `style.css` files are byte-identical (SHA-256
`407fd2739d932773051b8e84532222109b4ff0190c08010b51ac3b57d75f787a`).
They contain a large generic MyScript/Supernote stylesheet: default text rules,
heading classes, named color classes, math/diagram styles, selection styles,
and pen styles.

For the affected pages, BINK references only generic recognition styles such
as `defaultPenBrushStyle`, `black-color`, `pen-035`, and `raw-content` before
its recognition tree. Its text `DIAGRAM` JSON has `DWAlignment: "Left"`,
`DWLineGap: 0`, and recognition flags, but no text-box font or paragraph style.
The BDOM object stream has no per-field `style` elements for the disabled
objects.

The CSS therefore describes renderer capabilities and recognized ink defaults;
it is not evidence of the font/size/weight used by the visible Supernote text
boxes. The text-box and Digest styles are only observable in their pixels in
this fixture.

## Digest-specific evidence

Page 5 has one useful structural record adjacent to the upper disabled region:

```text
LINKRECT       913,321,759,70
LINKFILE       /storage/emulated/0/Document/bok_978-3-658-51112-8.pdf
OBJPAGE        61
FULLTEXT       —bok_978-3-658-51112-8.pdf - Page 61
FONTPATH       /system/fonts/DroidSansFallbackFull.ttf
FONTSIZE       40.000000
ITALIC         1
```

The disabled Digest rectangle ends at y=321 and the link starts at y=321.
Together with the rendered page, this is strong evidence that the link is the
Digest's source citation. It permits recovery of the source PDF path and page
number, but:

- no field explicitly types the disabled rectangle as `Digest`;
- no metadata links that rectangle ID to the `LINKO_` record;
- the source PDF is not embedded in the `.note`;
- no source-text range, source-page rectangle, or extracted paragraph is
  stored in the identified metadata;
- `FONTPATH`/`FONTSIZE` style the displayed link label, not the excerpt.

The second page-5 disabled region has no adjacent source link and is an
ordinary text box. Spatial adjacency can therefore support a Digest heuristic,
but not a general or authoritative object classifier.

The standalone `digest-n5-20230015-test.mark` fixture does not fill the gap. It
is a normal `markSN_FILE_VER_20230015` page/layer/TOTALPATH container for
handwritten Digest notes. It has no `RECOGNFILE`, `DISABLE`, source-document
record, or original Digest prose. A `.mark` file and an imported Digest excerpt
inside a `.note` should not be assumed to share a structured text format.

## Text searches and journal history

Exact visible body phrases (for example `Belohnungen`, `Gutscheine`,
`Goodies`, `Bonusspiele`, `Resilienz`, and the page-4 body prose) were searched
in:

- the raw `.note` as UTF-8, UTF-16LE/BE, and Latin-1;
- every decompressed ZIP entry;
- BINK printable payloads;
- decoded BDOM text fields.

Only the handwritten recognition labels and the external PDF link label are
present structurally. The main-layer Ratta-RLE naturally contains glyph pixels,
not searchable text.

The note is append-only/journaled and contains historical page/footer snapshots.
Those snapshots show page insertion/reordering and the same page-5 external
link under earlier page numbers. They do not introduce a hidden Digest/text-box
metadata group or plaintext body. This lowers, but cannot eliminate, the chance
that an unreferenced proprietary block has been missed.

## Confidence

### High confidence

- `RECOGNFILE` in this fixture models handwriting recognition, not the
  disabled text-box/Digest objects.
- BDOM selected text, BINK labels, and `RECOGNTEXT` are redundant recognition
  representations of the same handwriting.
- BINK geometry belongs entirely outside the affected `DISABLE` regions.
- The original typeset bodies and their styles are absent from all identified
  page/footer/ZIP structures.
- `DISABLE` plus `MAINLAYER` pixels is the only self-contained, lossless render
  source currently identified.
- Page 5's `LINKO_` record recovers an external PDF filename and page 61.

### Medium confidence

- `PAGETEXTBOX` is a broad page flag and does not distinguish text boxes from
  Digests.
- The page-5 upper disabled rectangle and adjacent external link form one
  Digest object/citation pair.
- No recoverable high-level text-box model remains in the saved `.note` after
  Supernote rasterizes the object.

### Unknown / requires controlled fixtures

- Whether another firmware/device version stores text-box source in a new key
  or BDOM object type.
- Whether an unparsed app-private block outside the current address graph
  contains edit state.
- Whether the device/Partner app keeps editable text in a separate database
  not copied into the `.note`.
- How `DISABLE` rectangle order relates to object creation or z-order.

## Viable extraction approaches

### 1. Preserve the raster overlay — production recommendation

Keep the exact `DISABLE` pixels from `MAINLAYER` and paint them after vector
ink. This is complete, offline, browser/Node-compatible, and retains the source
font and layout. It is not selectable text.

### 2. OCR each disabled rectangle

The regions are exact and the content is typeset, so cropped OCR should be more
accurate than whole-page handwriting recognition. Emit OCR text as a hidden
search layer while continuing to display the original pixels. This does not
recover authoritative source text or styles, but is the best self-contained
searchability path.

### 3. Resolve a Digest's external source

When a disabled rectangle is spatially adjacent to a `LINKO_` citation, decode
`LINKFILE` and `OBJPAGE`, obtain the external PDF, extract that source page's
text, and match the rectangle OCR against it. This could correct OCR and recover
Unicode accurately for page 5. It remains heuristic because the `.note` has no
source selection range and usually does not carry the linked PDF.

### 4. Expose recognition text/geometry separately

BDOM candidate extraction or BINK tree parsing can expose recognized
handwriting and its bounds. This is useful in its own right, but must be named
and documented as recognition output—not text-box/Digest extraction. It should
not replace `RECOGNTEXT` unless it demonstrably adds needed candidate data.

### 5. MyScript SDK export — validation only for this question

A licensed MyScript iink SDK can likely open the Raw Content package and export
JIIX/SVG/text. Android artifacts are native/ABI-specific and
`Engine.create()` is certificate-gated, so this is not a production Node/browser
solution. More importantly, the package's object and geometry inventory shows
that the disabled bodies were never included. SDK export is useful to validate
the BDOM interpretation, but is unlikely to recover missing text-box content.

### 6. Controlled differential corpus

Before attempting more proprietary parsing, save successive copies after:

1. adding a one-character text box;
2. editing its text without moving it;
3. changing font, size, color, alignment, and line spacing individually;
4. moving/resizing it;
5. inserting a Digest from a known PDF selection;
6. moving or deleting the source citation;
7. reopening and editing after a device restart.

Compare all newly appended blocks, not only current footer addresses. If only
`MAINLAYER`, `DISABLE`, thumbnails, and checksums change, structural extraction
from `.note` can be treated as unavailable for that firmware. If a new address
or object block appears, that controlled corpus will identify it far more
reliably than further blind BDOM token decoding.

## Recommendation

Keep raster overlays as the rendering source. If selectable/searchable text is
needed now, OCR only the `DISABLE` crops and retain the raster visually. Add an
optional external-PDF matching path for Digest citations when the linked source
is available. Treat BDOM/BINK extraction as handwriting-recognition support,
not as a route to the missing text-box or Digest body, unless a new controlled
fixture demonstrates that another firmware actually serializes those objects.
