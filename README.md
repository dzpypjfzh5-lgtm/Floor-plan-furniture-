# Furniture planner

A single-file tool for working out whether your furniture fits before moving day.
Drag pieces around a to-scale floor plan, rotate them, edit their sizes, and get told when
something crosses a wall, lands on the bath, or won't go through a door.

Two properties are modelled, picked from the selector in the top-left corner:

| | Rooms | Notes |
|---|---|---|
| **Vega** | 2 bed, 1 bath, garage, courtyard | Kitchen off the living room, hall lobby between the bedrooms |
| **Hydrae** | 2 bed, 1 bath, single garage, garden | Bedrooms over the wet areas, open lounge/dining, garage under the lounge |

Both carry the same measured furniture list, so you can try the same pieces in either.
Each keeps its **own** layout, room and door overrides, and plan-image underlay — switching
between them saves what you were doing and restores the other exactly as you left it, and the
one you were last on is the one that opens next time.

Everything lives in **`index.html`**. No build step, no dependencies, no network calls.

## Using it

- **On the iPhone** — open `index.html` (AirDrop it, or serve the folder and browse to it),
  then *Share → Add to Home Screen* to get it as an app icon. It works offline.
- **On a Mac** — just open the file in any browser.

Your layout saves itself to that device automatically, one layout per property. To move a
layout between the phone and the laptop, use **Save → Export file** (or *Copy JSON*) and
import it on the other. Exports record which property they belong to, so importing a Hydrae
file while Vega is open offers to switch first rather than dropping the furniture into the
wrong house.

The property selector is the name in the top-left corner — tap it to swap between **Vega**
and **Hydrae**. Switching is non-destructive: nothing is reset, and a property you have never
opened simply starts from the measured furniture list.

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
| Fabric swing | 2120 mm circle (1060 mm radius) | Living |

**Everything fits in both properties**, straight off the seeded layout. Floor coverage, as
the app computes it (real footprints, so the couch counts as its L and the swing as its
circle):

| | Vega | Hydrae |
|---|---|---|
| Main bedroom | 28% of 13.3 m² | 31% of 12.3 m² |
| Bed 2 | 36% of 11.1 m² | 41% of 9.8 m² |
| Living / dining | 47% of 24.4 m² | 43% of 27.1 m² |

Busy but workable either way. Hydrae's bedrooms are the tighter pair and its living space the
roomier one, which is the trade the two floor plans actually make.

- **The fabric swing needs a clear 2.12 m circle.** With 1.5 m of fabric from the ceiling
  carabiner and a 45° limit on how far it swings from vertical, the floor circle it sweeps is
  the base of that cone: 1.5 × sin 45° = **1.061 m radius**. Capping the angle matters — a full
  hemisphere would need a 3.0 m circle, which nothing indoors can take once furnished, whereas
  the 2.12 m circle fits the furnished living room with room to spare. It's modelled as a real
  circle, not a square, so it collides correctly, and it skips the doorway check since fabric
  packs into a bag.
- **The couch is modelled as a true L**, not a rectangle. Its bounding box is 5.6 m² but its
  real footprint is 3.8 m²; as a rectangle it would have been wrongly reported as not fitting.
- **The couch has to come in through the sliding door**, not the front door — in both
  properties. At 950 mm deep it won't clear either 900 mm front door, but both plans have a
  1600 mm slider off the living room, which takes it with 650 mm to spare. The app says so on
  the piece and names the door to use.
- **The three beds are flagged "on edge"**, again in both. At 1460 and 980 mm wide they won't
  pass their own 820 mm bedroom doors lying flat, so they need to go through on edge or come
  apart — which is normal, but it can't be confirmed without their thickness. Add a height to
  each bed and the check becomes definite.
- **"Airlock" and "changing space"** are your names for those two bookshelves; I don't know
  which spots they refer to, so both start in the living room. Drag them where they belong.

### What gets checked

Each piece is continuously checked for:

- **Walls** — the footprint has to sit entirely inside one room.
- **Fixtures** — bath, shower, WC, vanity, linen press, built-in robes, the laundry tub and
  (in Vega) the shed are all modelled as occupied floor. The kitchen benches are not: in both
  properties the kitchen is blocked out wholesale instead.
- **Other furniture** — overlaps are flagged.
- **Doorways** — tested against every door on the route from outside to that room, so a
  wardrobe in the main bedroom is checked against the front door, the hall opening *and* the
  bedroom door, and you're told which is tightest and by how many centimetres. Where more than one
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

## About the plans themselves

Both models are **dimension-driven**: every room the plan puts a number on is built at exactly
that number, because those figures come from someone measuring and are what the property is
advertised as. The drawing is used for the layout — which room is next to which, and where the
doors are — not for the sizes.

## Vega

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

### Vega layout notes worth knowing

- **There is no corridor between the bedrooms.** Bed 2 backs straight onto Bed 1 across a
  single wall. The hall is a small 1.70 × 0.90 lobby sitting east of Bed 2, between the bath
  above and Bed 1 below.
- **Bed 2's door is on its east wall**, opening into that lobby — which is where the hall
  dead-ends.
- **The kitchen is a blocked zone.** The fridge is the only thing going in there and it's
  known to fit, so the benches aren't modelled. It's drawn hatched, and furniture can be
  parked there as a holding space but is flagged as clashing.
- **The kitchen has one sliding door, dead centre of its north wall** onto the dining end
  (confirmed, not traced). The kitchen runs 4.40–6.85 across, so the door is centred on 5.625.
  Since the kitchen is blocked and no route to any other room passes through it, this door
  doesn't affect any fit result; it's there for the drawing.

## Hydrae

The printed figures are:

| Room | Plan prints | Built as (north-south × east-west) |
|---|---|---|
| Master | 3.5 × 3.5 | 3.50 × 3.50 |
| Bed 2 | 2.8 × 3.5 | 2.80 × 3.50 (including the robe recess) |
| Dining | 3.0 × 2.8 | 3.00 × 2.80 |
| Kitchen | 3.0 × 2.4 | 2.90 × 2.40 — see below |
| Lounge | 3.6 × 5.2 | 3.60 × 5.20 |
| Garage | 3.0 × 5.3 | 3.00 × 5.30 |

Reading the drawing, the first number of each pair is the **north-south depth** and the second
the east-west width. That is what makes the kitchen sit tall and narrow beside the dining, and
the lounge wide and shallow below both — and it is why the dining (2.8 wide) plus the kitchen
(2.4 wide) come to exactly the lounge's 5.2.

### Hydrae layout notes worth knowing

- **The lounge and dining are one room.** The drawing shows no wall between them, so they are
  modelled as a single L-shaped space carrying both printed sizes. Splitting them into two
  rooms would put an invisible wall through the middle and wrongly fail anything standing
  across the join — a sofa backing onto the dining end, for instance.
- **The kitchen is built 2.90 m deep, 100 mm shy of its printed 3.0.** Its south wall has to
  land on the same line where the dining opens into the lounge, and once a wall thickness is
  allowed for, the printed kitchen depth and the printed dining depth cannot both hold. The
  app flags the 100 mm on the room, so the discrepancy is visible rather than buried. The
  kitchen is blocked out anyway, for the same reason as Vega's.
- **The bedrooms sit over the wet areas.** Bed 2's door is on its *south* wall at the top of
  the hall; the master's is at its south-east corner. The hall is an L — a long leg down the
  east side of the master, and a short leg turning west underneath it to reach the master and
  laundry doors.
- **The garage has no internal door.** The car is drawn lengthways east-west in a 3.0 m deep
  garage, so the roller door has to be on the 3.0 m west wall facing the drive; there is no
  connecting door to the house, so anything going in the garage goes through the roller door.
  That inference is noted on the room.
- **Two ways in besides the front door** — the 1600 mm lounge slider and an 820 mm side door
  at the dining end, both onto the paving on the west. The app picks the easiest route that
  actually works for each piece.
- The **garden and paving** carry no printed size at all: a planted strip the full length of
  the western boundary, paving beside the lounge and dining that narrows past the garage to
  reach the garage door, and the small entry porch on the east.

## What both plans hedge on

Rooms with **no** printed dimension — in Vega the bath, laundry, hall and the wrap-around
paving; in Hydrae the bath, laundry, hall and the garden — carry no advertised size, so they
absorb the slack and are marked *"sized to fit"* in the app. They're the least reliable
numbers in either plan.

**Door widths are estimates**, not measurements — standard 820 mm internal doors, 900 mm
front door, 770 mm to the wet areas. These drive the "will it fit through" check, so they
are the first thing worth measuring properly. All are editable under **Rooms**, as is every
room size.

Resizing a room stretches that room and its fixtures only; it does not push the neighbouring
rooms out of the way, so a large change will make rooms overlap on screen.

### Comparing against the original drawings

Under **Image** you can load the original marketing floor plan as a background underlay, set
its opacity, and drag/pinch it into alignment. The image is stored on your device only, and
each property remembers its own.

The starting alignment is the tracing each model was built from — 67 px per metre with the
inside north-west corner of Bed 2 at pixel (447, 372) for Vega, and 232 px per metre with the
inside north-west corner of the master at pixel (821, 101) for Hydrae — so a photo of the same
plan should land close and only need nudging.

## Development

Open `index.html` directly, or serve the folder:

```sh
python3 -m http.server 8899
```

Each plan model — rooms, fixtures, doors and the routes used for the doorway check — is one
object at the top of the `<script>` block: `VEGA` and `HYDRAE`, listed in `PROPERTIES`, with
`PLAN` pointing at whichever is on screen. **Adding a third property is a matter of writing
one more of those objects and dropping it in the list**; the picker, the per-property storage
and the export tagging all follow from it.

Coordinates are metres, x east, y south; room rects are inside wall faces, with 0.10 m between
rooms, so a printed dimension is the clear internal size. Origin is the inside face of Bed 2's
north-west corner in Vega and of the master's in Hydrae.

The two share **room ids** (`bed1`, `bed2`, `bath`, `ldry`, `hall`, `kitchen`, `living`,
`garage`, `court`) even where the names differ — Vega's "Bed 1" is Hydrae's "Master" — which
is what lets the one `FURNITURE` list seed either property unchanged.

Saved layouts live under `floorplan-planner-v2:<property>` in `localStorage`, with the plan
underlay under `floorplan-planner-img-v1:<property>`. A layout saved by the single-property
version of this file is migrated to Vega's key once, on first run.
