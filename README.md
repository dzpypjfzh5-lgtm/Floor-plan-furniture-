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

## The furniture

The measured list is in the `FURNITURE` constant near the top of the script, and loads
automatically on first run. **Save → Reload my furniture list** restores it if you wipe or
mangle the layout. Sizes are width × depth on the floor, in millimetres as measured:

| Piece | Size | Room |
|---|---|---|
| Double bed | 2000 × 1460 | Bed 1 |
| Side table × 2 | 520 × 300 | Bed 1 |
| Chest of drawers (adults) | 1090 × 520 | Bed 1 |
| Single bed × 2 | 2050 × 980 | Bed 2 |
| Couch, L-shaped | 3170 × 1770 overall, 950 deep legs | Living |
| Cube shelf | 1460 × 390 | Living |
| Bookshelf × 2 (matched) | 710 × 300 | Living |
| Narrow bookshelf | 400 × 300 | Living |
| Cupboard | 800 × 310 | Living |
| Chest of drawers (kids) | 1400 × 400 | Living |
| Change table | 870 × 580 | Living |
| Fridge | 700 × 700, 1720 high | Kitchen |

**Everything fits.** Floor coverage lands at 29% in Bed 1, 36% in Bed 2, 27% in the
living/dining and 7% in the kitchen, so there's room to move in all of them.

Assumptions made, all visible in the app:

- **Fridge depth is assumed 700 mm** — only height × width was noted. It's flagged in the
  Furniture panel.
- **The couch is modelled as a true L**, not a rectangle. Its bounding box is 5.6 m² but its
  real footprint is 3.8 m²; as a rectangle it would have been wrongly reported as not fitting.
- **The couch has to come in through the courtyard sliding door**, not the front door. At
  950 mm deep it won't clear the 900 mm front door, but the 1600 mm slider takes it with
  650 mm to spare. The app says so on the piece.
- **The three beds are flagged "on edge"**. At 1460 and 980 mm wide they won't pass the
  800 mm hall opening lying flat, so they need to go through on edge or come apart — which is
  normal, but it can't be confirmed without their thickness. Add a height to each bed and the
  check becomes definite.

### What gets checked

Each piece is continuously checked for:

- **Walls** — the footprint has to sit entirely inside one room.
- **Fixtures** — bath, shower, WC, vanity, linen press, kitchen benches, built-in robes,
  pantry and the shed are all modelled as occupied floor.
- **Other furniture** — overlaps are flagged.
- **Doorways** — tested against every door on the route from outside to that room, so a
  wardrobe in Bed 1 is checked against the front door, the hall opening *and* the bedroom
  door, and you're told which is tightest and by how many centimetres. Where more than one
  route exists the easiest is used, and if a piece can't come in the front door the app names
  the way it has to go instead.

  The doorway test reasons in three dimensions, because a piece too wide to go through flat
  can usually go through on edge: a rigid box clears an opening when its two smallest
  dimensions fit the opening's width and height (assumed 2040 mm). Give a piece a height and
  the answer is definite; leave the height off and anything too wide to go flat is flagged as
  needing to be tilted rather than being called a failure, since without the third dimension
  it genuinely can't be judged.

Furniture can be **L-shaped** rather than rectangular. Pieces carry a list of `parts`, which
are collided and drawn individually but move and rotate as one, so an L-couch is tested on
its real footprint instead of its much larger bounding box.

Optional **walkway clearance** (600/750/900 mm) draws a halo around each piece so you can
see whether you can still move around the room.

## About the plan itself

The model is **dimension-driven**. Every room the plan puts a number on is built at exactly
that number, because those figures come from someone measuring and are what the property is
advertised as. The drawing is used only for *topology* — which room adjoins which, and where
the doors sit.

That decision came from tracing the drawing first and finding it short almost everywhere:

| Room | Plan prints (built) | Drawing traced | Short by |
|---|---|---|---|
| Kitchen | 3.0 × 2.45 | 2.85 × 1.70 | **0.75 m** across |
| Living / Dining | 6.05 × 5.0 | 5.70 × 4.90 | 0.35 m |
| Bed 1 | 4.3 × 3.1 | 4.20 × 2.85 | 0.25 m |
| Bed 2 | 4.1 × 2.7 | 3.85 × 2.70 | 0.25 m |
| Garage | 6.0 × 2.9 | 5.85 × 2.70 | 0.20 m |

The kitchen settled it: traced at 1.70 m across, two 600 mm bench runs would leave a 500 mm
walkway, which is not a real kitchen. At the printed 2.45 m the walkway is 1.25 m and a
standard fridge fits between the benches. The bias is consistent and one-directional, which
is what you'd expect from a sketch drawn slightly tight rather than from measurement error.

Two printed figures need reading carefully, and the app says so on each room:

- **Bed 1 (4.3) and Bed 2 (4.1)** include the built-in robe recess. Clear floor is about
  3.7 m and 3.5 m respectively — the robes are modelled as occupied floor, so the fit checks
  already account for this.
- **Courtyard 11.0 × 4.6** is the main paved strip along the north side, built at exactly
  that. The paving also wraps down the west side and around toward the garage, which the
  printed figure doesn't cover.

Rooms with **no** printed dimension — bath, laundry, hall, and the wrap-around paving — carry
no advertised size, so they absorb the slack and are marked *"sized to fit"* in the app.
They're the least reliable numbers here. The hall in particular is pinned by arithmetic:
Bed 2 (4.1) + Bed 1 (3.1) plus walls leaves about 1.2 m for it.

**Door widths are estimates**, not measurements — standard 820 mm internal doors, 900 mm
front door, 770 mm to the wet areas. These drive the "will it fit through" check, so they
are the first thing worth measuring properly. All are editable under **Rooms**, as is every
room size.

Resizing a room stretches that room and its fixtures only; it does not push the neighbouring
rooms out of the way, so a large change will make rooms overlap on screen.

### Comparing against the original drawing

Under **Image** you can load the original marketing floor plan as a background underlay, set its
opacity, and drag/pinch it into alignment. Because the model is built to the printed
dimensions rather than to the drawing, **it will not line up perfectly** — no single scale
can, given the drawing runs short by varying amounts. Use it to sanity-check the layout, not
the sizes. The image is stored on your device only.

## Development

Open `index.html` directly, or serve the folder:

```sh
python3 -m http.server 8899
```

The plan model — rooms, fixtures, doors and the routes used for the doorway check — is the
`PLAN` object at the top of the `<script>` block. Coordinates are metres, x east, y south;
room rects are inside wall faces, with 0.10 m between rooms, so a printed dimension is the
clear internal size. Origin is the inside face of Bed 2's north-west corner.
