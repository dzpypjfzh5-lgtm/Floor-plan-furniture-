# Furniture planner

A single-file tool for working out whether your furniture fits before moving day.
Drag pieces around a to-scale plan of a two-bedroom unit, rotate them, edit their sizes, and
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
| Fine positioning | The ↑ ← → ↓ nudge pad; tap the middle to cycle 10 / 50 / 100 mm |

Rotation snaps to 5°, and hard-snaps to square when it gets close. Dragging snaps
flush to walls when a piece is square-on (turn this off under **Save**). The nudge pad
deliberately ignores wall snapping, so it can place a piece anywhere you want it.

If the adjustment panel eats too much of the phone screen, turn off **Open controls on select**
under **Save**. Selecting a piece then shows only a small chip with the item name, its fit
status and an **Adjust** button to raise the panel when you actually want it.

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
| Dining table | 1530 × 820 | Living |
| Bookshelf (airlock) | 970 × 290 | Living |
| Bookshelf (changing space) | 900 × 280 | Living |
| Fabric swing | 3000 mm circle (1500 mm radius) | see below |

**Everything fits except the swing.** Floor coverage lands at 29% in Bed 1, 36% in Bed 2 and
33% in the living/dining, so there's room to move in all of them.

- **The fabric swing needs a clear 3.0 m circle** on the floor — a 1500 mm radius swept around
  a single ceiling point, as you specified. Nothing indoors can take it once furnished. Bed 1
  is the only interior room that fits it *empty* (4.30 × 3.10); the garage misses by 100 mm at
  2.90 wide, and the living/dining's legs are only 2.50 m deep. It's parked in the courtyard,
  which takes it comfortably. It's modelled as a circle, not a square, so it collides
  correctly — and it skips the doorway check, since fabric packs into a bag.
- **The couch is modelled as a true L**, not a rectangle. Its bounding box is 5.6 m² but its
  real footprint is 3.8 m²; as a rectangle it would have been wrongly reported as not fitting.
- **The couch has to come in through the courtyard sliding door**, not the front door. At
  950 mm deep it won't clear the 900 mm front door, but the 1600 mm slider takes it with
  650 mm to spare. The app says so on the piece.
- **The three beds are flagged "on edge"**. At 1460 and 980 mm wide they won't pass their own
  820 mm bedroom doors lying flat, so they need to go through on edge or come apart — which is
  normal, but it can't be confirmed without their thickness. Add a height to each bed and the
  check becomes definite.
- **"Airlock" and "changing space"** are your names for those two bookshelves; I don't know
  which spots they refer to, so both start in the living room. Drag them where they belong.

### What gets checked

Each piece is continuously checked for:

- **Walls** — the footprint has to sit entirely inside one room.
- **Fixtures** — bath, shower, WC, vanity, linen press, built-in robes, the laundry tub and
  the shed are all modelled as occupied floor. The kitchen benches are not: the kitchen is
  blocked out wholesale instead.
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

Furniture is not limited to rectangles. A piece can carry a list of `parts` that are collided
and drawn individually but move and rotate as one, so the **L-shaped couch** is tested on its
real footprint instead of its much larger bounding box. A piece can also be a **circle**
(`circle: radius`), used for the swing's swept area, and can be marked `virtual` to mean "this
is clearance, not an object" — which skips the doorway check while still keeping the space
clear of everything else.

Optional **walkway clearance** (600/750/900 mm) draws a halo around each piece so you can
see whether you can still move around the room.

## About the plan itself

The model is **dimension-driven**: every room the plan puts a number on is built at exactly
that number, because those figures come from someone measuring and are what the property is
advertised as.

Re-traced carefully at **67 px per metre**, the drawing agrees with the printed figures:

| Room | Plan prints | Drawing traces to |
|---|---|---|
| Bed 1 | 4.3 × 3.1 | 4.30 × 3.13 |
| Bed 2 | 4.1 × 2.7 | 4.18 × 2.76 (including the robe recess) |
| Living / Dining | 6.05 × 5.0 | 6.05 × 5.07 (measured from the kitchen's west wall) |
| Garage | 6.0 × 2.9 | 5.82 × 2.69 |

An earlier version of this file claimed the drawing ran short by 0.2–0.75 m everywhere. That
was wrong — it was a tracing error, not a fault in the drawing. Walls had been read off at the
wrong pixels, most damagingly by inventing a full-width corridor between the two bedrooms.
The drawing is sound; the printed dimensions and the geometry agree.

Several printed figures still need reading carefully, and the app says so on each room:

- **Bed 1 (4.3) and Bed 2 (4.1)** include the built-in robe recess. Clear floor is about
  3.7 m and 3.5 m respectively — the robes are modelled as occupied floor, so the fit checks
  already account for it.
- **Living / Dining 6.05** is measured from the kitchen's west wall across to the east wall,
  which is why the room is modelled as an L wrapping above and beside the kitchen.
- **Kitchen 2.45** includes the pantry recess. The kitchen is blocked out anyway (see below).
- **Courtyard 11.0 × 4.6** is the main paved strip along the north side, built at exactly
  that. The paving also wraps down the west side and around toward the garage, which the
  printed figure doesn't cover.

### Layout notes worth knowing

- **There is no corridor between the bedrooms.** Bed 2 backs straight onto Bed 1 across a
  single wall. The hall is a small 1.70 × 0.90 lobby sitting east of Bed 2, between the bath
  above and Bed 1 below.
- **Bed 2's door is on its east wall**, opening into that lobby — which is where the hall
  dead-ends.
- **The kitchen is a blocked zone.** The fridge is the only thing going in there and it's
  known to fit, so the benches aren't modelled. It's drawn hatched, and furniture can be
  parked there as a holding space but is flagged as clashing.

Rooms with **no** printed dimension — bath, laundry, hall, and the wrap-around paving — carry
no advertised size, so they absorb the slack and are marked *"sized to fit"* in the app.
They're the least reliable numbers here.

**Door widths are estimates**, not measurements — standard 820 mm internal doors, 900 mm
front door, 770 mm to the wet areas. These drive the "will it fit through" check, so they
are the first thing worth measuring properly. All are editable under **Rooms**, as is every
room size.

Resizing a room stretches that room and its fixtures only; it does not push the neighbouring
rooms out of the way, so a large change will make rooms overlap on screen.

### Comparing against the original drawing

Under **Image** you can load the original marketing floor plan as a background underlay, set
its opacity, and drag/pinch it into alignment. The starting alignment assumes 67 px per metre
with the inside north-west corner of Bed 2 at pixel (447, 372), which is how the model was
traced, so it should line up closely. The image is stored on your device only.

## Development

Open `index.html` directly, or serve the folder:

```sh
python3 -m http.server 8899
```

The plan model — rooms, fixtures, doors and the routes used for the doorway check — is the
`PLAN` object at the top of the `<script>` block. Coordinates are metres, x east, y south;
room rects are inside wall faces, with 0.10 m between rooms, so a printed dimension is the
clear internal size. Origin is the inside face of Bed 2's north-west corner.
