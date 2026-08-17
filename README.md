# Furniture planner

A single-file tool for working out whether your furniture fits before moving day.
Drag pieces around a to-scale plan of the unit, rotate them, edit their sizes, and
get told when something crosses a wall, lands on the bath, or won't go through a door.

Everything lives in **`index.html`**. No build step, no dependencies, no network calls.

## Using it

- **On the iPhone** — open `index.html` (AirDrop it, or serve the folder and browse to it),
  then *Share → Add to Home Screen* to get it as an app icon. It works offline.
- **On a Mac** — just open the file in any browser.

Your layout saves itself to that device automatically. To move a layout between the
phone and the laptop, use **Save → Export file** (or *Copy JSON*) and import it on the other.

### Gestures

| Action | Gesture |
|---|---|
| Move a piece | Drag it |
| Rotate | Drag the ⟳ grip, or use the ⟲/⟳ 90° buttons and the angle slider |
| Pan the plan | Drag empty space |
| Zoom | Pinch, or the +/− buttons |
| Select | Tap a piece; tap empty space to deselect |

Rotation snaps to 5°, and hard-snaps to square when it gets close. Dragging snaps
flush to walls when a piece is square-on (turn this off under **Save**).

### What gets checked

Each piece is continuously checked for:

- **Walls** — the footprint has to sit entirely inside one room.
- **Fixtures** — bath, shower, WC, vanity, linen press, kitchen benches, built-in robes,
  pantry and the shed are all modelled as occupied floor.
- **Other furniture** — overlaps are flagged.
- **Doorways** — the narrowest face of the piece is compared against every door on the
  route from outside to that room. So a wardrobe in Bed 1 is tested against the front
  door, the hall opening *and* the bedroom door, and you're told which one is tightest
  and by how many centimetres.

Optional **walkway clearance** (600/750/900 mm) draws a halo around each piece so you can
see whether you can still move around the room.

## About the plan itself

The geometry is traced from the marketing marketing floor plan at roughly 67 px per metre,
with the building's north-west corner as the origin. **Treat every number as approximate
until you've put a tape measure on it.** The tool is built around that caveat:

- Every room size and every door width is editable under **Rooms**.
- Where the drawing and the printed dimension disagree, the printed one is shown as a
  "plan says" chip so you can see the discrepancy rather than trusting one silently.

Known discrepancies between the drawing and the printed dimensions:

| Room | Traced | Plan prints | Why |
|---|---|---|---|
| Bed 1 | 4.20 × 2.85 | 4.3 × 3.1 | Printed length includes the built-in robe |
| Bed 2 | 3.85 × 2.70 | 4.1 × 2.7 | Printed length includes the robe recess |
| Living / Dining | 5.70 × 4.90 | 6.05 × 5.0 | Printed length runs ~0.35 m longer than the drawing |
| Kitchen | 2.85 × 1.70 | 3.0 × 2.45 | Printed width appears to measure through to the dining side |
| Garage | 5.85 × 2.70 | 6.0 × 2.9 | Rounding |
| Courtyard | wraps the north and west sides | 11.0 × 4.6 | The printed figure is the main strip along the top only |

**Door widths are estimates**, not measurements — standard 820 mm internal doors, 900 mm
front door, 770 mm to the wet areas. These drive the "will it fit through" check, so they
are the first thing worth measuring properly.

Resizing a room stretches that room and its fixtures only; it does not push the neighbouring
rooms out of the way, so a large change will make rooms overlap on screen.

### Checking the trace against the original

Under **Image** you can load the original marketing floor plan as a background underlay, set its
opacity, and drag/pinch it into alignment to see how well the traced walls match. The
default alignment already matches the original export, so it should line up immediately.
The image is stored on your device only.

## Development

Open `index.html` directly, or serve the folder:

```sh
python3 -m http.server 8899
```

The plan model — rooms, fixtures, doors and the routes used for the doorway check — is the
`PLAN` object at the top of the `<script>` block. Coordinates are metres, x east, y south.
