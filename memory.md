# Dozer Dash project memory

Last updated: 2026-11-23

## Project

Dozer Dash is a mobile-first arcade game about driving a friendly mechanical dozer through an expanded city and varied terrain, crushing clutter, collecting people and coins, and growing.

The prototype is implemented in [`index.html`](./index.html), with city assets selected by [`city-model-shortlist.json`](./city-model-shortlist.json). The main scene uses real 3D WebGL rendering through Three.js; the HUD and minimap remain HTML/CSS and Canvas respectively.

## Current architecture

- Three.js `0.149.0`, GLTFLoader, and BufferGeometryUtils are loaded from jsDelivr. The page needs an internet connection and access to the project asset files; if local file access is blocked, serve the folder with `python3 -m http.server 8000`.
- Asset packs are kept intact under `assets/city`, `assets/characters`, and `assets/environment`; the original visual references are in `assets/references`. The game and model browser remain at the project root.
- The game canvas is WebGL. The 3D world spans 5,600 × 4,092 gameplay units, with roads 176 units wide on a 560-unit grid. It contains residential and downtown blocks, open parks, industrial yards, drylands, a waterfront, buildings, trees, streetlights, clutter, people, coins, rival dozers, shadows, fog, and a perspective camera that follows the player.
- Buildings, trees, street furniture, clutter, and reward coins use the 61 selected Crayon GLB models in the project shortlist. People use all five GLB characters in `assets/characters/NPC/GLB`; road surfaces, terrain, traffic, and dozers remain procedural. The city loader supports Crayon buildings, props, and trees and requires at least one of each.
- The player is assembled from 3D meshes, including tracks, cab, friendly face, and upper/lower toothed crusher jaws that slide together and apart.
- The jaws' forward protrusion and reach stroke are half their previous lengths; contact and crusher effects follow the shortened jaws.
- The player dozer does not cast a shadow; other scene shadows remain enabled.
- Road markings use instancing, box meshes share geometry, and selected GLB meshes are merged where materials are compatible. Clones share their immutable geometry and materials, while buildings have separate materials for occlusion fading. Off-screen city groups are culled. Shadows update every other rendered frame; coarse-pointer devices use a 512-pixel shadow map. Rendering pixel ratio is capped at 1.5.
- The game loop advances up to 100 ms of elapsed time using steps of at most 35 ms, so moderate frame drops do not slow gameplay; longer interruptions are capped.
- The game starts at a downtown intersection, with a small trail of starter clutter, nearby people, and moving traffic. Foreground buildings fade when they obstruct the camera's view of the player.
- Rounds last three minutes. Each completed person or object collection adds one growth point; buildings add four. The dozer grows by 0.0375 per point, levels up every 12 points, and reaches its maximum 2.8× size at 48 points.
- Gameplay coordinates remain in the original 2D world plane and are mapped to 3D X/Z coordinates for rendering. Selected city models use a common 0.3 scene-unit scale per source unit, with slight tree-size variation. Object dimensions come from their rendered local bounds; the dozer's bounds include its animated jaw position. An object's longest local dimension must be strictly smaller than the dozer's to be collected. Jaw contact uses oriented ground footprints; smaller buildings shatter on forward contact and larger buildings block the dozer.
- The jaws extend toward nearby eligible objects without pulling them. On contact, an object is held between the jaws, breaks into blocky fragments that move into the crusher, then produces coins from the rear exhausts. Growth and coins are credited when processing finishes.
- Smaller buildings use the same staged crusher sequence on forward contact; larger buildings remain obstacles. Active crusher jobs finish before the round-end results and are cleared on replay.
- People flee the approaching dozer with a fear indicator and struggle briefly during front-jaw intake, moving intact without fragments or shredding. Each completed collection emits seven coins from the rear; pending collections and visible coins finish before results, and replay restores people.
- The NPC GLBs have no animation clips. Their rigs are posed and baked into shared, merged walking and fear morph targets at load time; people use conservative bounds across the walking poses for size and contact checks, and off-screen characters are not rendered.
- Sixteen cars, taxis, vans, trucks, and buses drive along road lanes and wrap at the map edges. Vehicles follow the same size-gated crusher rules as other props and stop moving once captured.
- Movement supports keyboard controls and a free touch/mouse joystick that starts wherever the player presses.
- Clutter, trees, streetlights, vehicles, rival dozers, and people can be collected when small enough. Terrain remains traversable and buildings are the only world geometry that blocks the dozer. The map targets 80 clutter pickups, 35 scenery objects, 16 moving vehicles, and 56 people. Buildings occupy at most one lot per block; scenery is spaced from buildings and other scenery, and tree canopies may overhang roads.
- Joystick and keyboard directions are translated from screen space using the fixed camera basis. The camera follows player position without rotating around the dozer; steering immediately turns the dozer toward the requested direction and redirects its speed along that heading.
- Building collision clips movement to the dozer's circular body footprint, allowing sliding along walls and escape if growth causes overlap. Driving speed remains responsive when a movement component is blocked.
- The game fills portrait screens and uses a centered 9:16 play area in landscape, leaving room for a future desktop layout. The renderer and HUD size to that play area; keyboard and mouse controls remain available. On short, touch-first landscape screens, a rotate prompt pauses gameplay until portrait returns.
- A camera settings button opens live starting-distance, tilt, and rotation sliders. Starting camera distance ranges from 5 to 18 world units and smoothly scales with the dozer's size; tilt ranges from 15° (low, GTA-like) through 90° (top-down); rotation ranges from -180° to 180°. Defaults are 9.0, 49°, and -42°.
- Save stores these settings in browser local storage for automatic reload and writes a versioned `camera-settings.json` file through the browser's save picker, with a download fallback.
- The start/results card links to [`model-browser.html`](./model-browser.html), a separate 3D catalog of 142 Crayon and Kenney city models. Users can search, filter by pack, rotate and zoom previews, choose Crayon variants, save a browser-local shortlist, and export it as JSON. The game loads the project-folder shortlist at startup; editing a browser-local shortlist alone does not change it. Exported project files should be saved in the project folder; browsers may require the user to select it or move a downloaded file.

## Change history

1. Created the original single-file mobile game with driving, debris collection, rescues, coins, growth, HUD, minimap, sound toggle, and restart.
2. Reworked the playfield into an isometric Canvas illustration and added the floating, anywhere-on-screen joystick.
3. Added responsive mobile layouts, safe-area handling, and closer portrait/landscape camera framing.
4. Fixed building collision and added building damage so larger structures do not keep the dozer wedged.
5. Replaced radial debris attraction with front-only suction; coins now emerge behind the dozer and are credited as emitted.
6. Reworked the bite into parallel upper/lower jaws with inward-facing teeth instead of scissor-like rotation.
7. Replaced the 2D scene renderer with an actual Three.js WebGL scene and 3D meshes while preserving the game HUD and controls.
8. Reduced movement speed and dozer scale; then tuned both further after the WebGL preview to make the vehicle easier to control and smaller relative to city objects.
9. Slowed movement again after feedback; corrected the free joystick's isometric direction transform so it preserves input magnitude instead of boosting movement.
10. Fixed joystick direction alignment: camera yaw is now stable, screen-space steering maps to the camera's ground-plane axes, and the dozer turns and moves along the selected direction. The floating joystick remains centered at the initial touch point.
11. Added an in-game camera settings panel with live view-distance and camera-tilt sliders, a reset button, and support for a true top-down camera angle.
12. Added a camera rotation slider and Save settings action. Saved camera settings are validated and restored on reload, with a JSON file export for backup or placement in the project.
13. Added size-based instant building shatters on contact. Shard debris spawns in the front suction cone, is consumed by the jaws, and produces rear-emitted coins; larger buildings retain the existing higher-level damage mechanic.
14. Expanded the playfield to 5,200 × 3,800 units, placed the initial spawn at an open park-road intersection, and added distinct park, industrial, dryland, and waterfront districts. Increased clutter and rescue targets for the larger world, added district-specific building palettes and terrain props, and updated the minimap to show the terrain.
15. Increased driving top speed moderately for the expanded world: 110 units/second at smaller sizes and 90 at larger sizes, retaining the existing acceleration and deceleration tuning.
16. Refreshed the opening with a downtown spawn, starter pickups, a nearby rescue, crosswalks, more detailed buildings and dozer, camera-following sunlight, and warm construction-themed HUD styling. Added building occlusion fading and corrected maximum-level growth and progress, as well as minimap display of destroyed buildings.
17. Made the play area portrait-first, centered it on wider screens, and added a pause-and-rotate prompt for phones in landscape. Joystick positioning now accounts for the play area's offset.
18. Made driving more responsive with higher top speed, faster acceleration and stopping, closer camera following, and a shorter joystick throw. Oversized clutter no longer slows the dozer.
19. Unified collection around rendered object size, adding trees, rocks, streetlights, and rival dozers. Removed damage to oversized buildings and persistent cosmetic rubble; collected scenery and rivals return on replay.
20. Replaced magnetic attraction with extending jaws and contact-based biting, visible boxy fragments drawn into the dozer, and delayed rear coin payouts. Buildings use the same sequence and in-progress jobs finish at round end.
21. Widened roads across the city and increased block spacing and world dimensions. Crosswalks adapt to the road width, and pickup and rescue counts preserve density.
22. Added moving road traffic and more people, with protective-bubble rescues to the cab instead of exhaust coin effects.
23. Replaced bubble rescues with intact, contact-based person intake and rear coin payouts, without the object's shredding effect.
24. Batched repeated road and building details, shared box geometry, reduced shadow and high-DPI rendering costs, and corrected slow-frame simulation timing.
25. Added district-specific architecture and landscaping, shopfronts, regional plant shapes, and additional collectible street objects.
26. Extended rounds to three minutes and made each collection contribute to more visible growth, including people; added numeric progress and a larger maximum size.
27. Added a separate city-model browser for previewing and shortlisting assets before deciding which to integrate.
28. Saved the user's 61-model Crayon shortlist in the project and replaced city buildings, trees, street objects, and reward coins with those assets, retaining procedural roads, terrain, characters, and vehicles.
29. Opened up the city, reduced scenery and traffic density, slowed growth, and based size gates and rotated jaw contact on the models' rendered bounds.

## Current tuning

- Player top speed: 185 world units/second at smaller sizes and 165 at larger sizes, before the small size multiplier.
- Input acceleration: 12; passive deceleration: 15, using frame-rate-independent exponential smoothing.
- The floating joystick reaches full input at a 48-pixel drag and keeps its thumb within the joystick ring.
- Player model scale: `0.65 * player.size`.
- Rival dozer model scale: `0.55`.
- Building collision radius: `36 * player.size` world units, matched to the reduced model.
- Camera uses a high-angle perspective view and sizes its projection to the play area.

## Validation notes

- Browser-tested the WebGL renderer, gameplay initialization, and joystick activation/release.
- The current level-one full-input speed is about 188 world units/second; keyboard and joystick diagonals preserve the same speed cap.
- Verified the rendered model scale is about 0.65.
- Reloaded the game, drove with a touch-style drag, and confirmed the timer advanced and the browser reported no JavaScript errors.
- Simulated a rightward drag; verified the floating joystick appeared at its touch origin and the dozer faced right on the screen.
- Opened the camera panel in the browser, verified the tilt slider updates its displayed value live, and confirmed Reset view restores the default 9.0 distance and 49° tilt.
- Verified rotation changes the live camera value. Tested Save settings with a stubbed file picker: the JSON contains version, distance, tilt, and rotation; reloaded the page and confirmed those values restored from browser storage.
- Parsed the updated inline game script in the browser with no syntax or runtime page errors.
- Updated building collision and destruction flow to shatter size-matched buildings on contact and route spawned debris through front suction before rear coin payout.
- Expanded terrain generation, free off-road traversal, district-specific scenery, and minimap for the 5,200 × 3,800 world; reset now restores the park-intersection spawn and expanded clutter set.
- Browser-checked the expanded 3D scene and park-road starting view; the rescue HUD reports 0/34 initially and the page reports no JavaScript errors.
- Increased movement speed by about 23% at level one while retaining the existing acceleration/deceleration; browser smoke-tested game initialization and movement input without page errors.
- Browser-tested the downtown opening at desktop and 390 × 844, joystick input and release, pickup rewards, and camera controls. Checked starter placements and restart restoration, and verified the max-level size no longer shrinks and the progress bar stays full.
- Measured driving acceleration and stopping, equal cardinal/diagonal speeds, and joystick dead zone, analog response and full-input threshold.
- Checked strict smaller/equal/larger size boundaries, vehicle and scenery collection, rival collection, replay restoration, and small/large building collision behavior.
- Tested no distant attraction, bite/fragment/payout timing, simultaneous objects, building processing, round-end completion, and replay cleanup; visually checked the bite and fragments in the browser.
- Checked the widened grid for buildings overlapping roads, clear starter positions, crosswalk generation, replay counts, and browser movement and pickups.
- Checked traffic lanes, wrapping, vehicle size limits and crushing, simultaneous bubble rescues, cab tracking, rescue rewards, round-end completion, and replay restoration. Visually checked that the rescue bubble lifts clear of the jaws.
- Checked front-only person intake without shredding, simultaneous collections, rear coin payout, moving coins after the timer expires, replay during and after intake, and unchanged clutter shredding.
- In the browser test environment, average scene draw time fell from about 26 ms to 14 ms and renderer geometry count from about 3,865 to 838. Verified slow-frame timing, size gates, pickup behavior, and replay; tested WebGL, the pixel-ratio cap, and touch movement at an emulated 390 × 844 viewport with device pixel ratio 3.
- Checked new prop and scenery types, size-based collection and payouts, shop signs, replay restoration, and the downtown and park views. Average scene draw time remained about 14 ms in the same browser environment.
- Tested growth from people and props, level and maximum-size thresholds, strict size gates, larger-building unlocks, the three-minute timer, round expiry, replay, and the starter route.
- Checked all five model-pack counts and representative 3D previews, flat appearance, search, shortlist filtering, persistence, JSON export, removal, model-load and storage errors, mobile sizing, game navigation, and game startup. Verified the Crayon manifest paths exist.
- Checked all 61 selected assets appear, building footprints avoid roads, starter objects remain collectible, crushing and growth work for props and buildings, shared resources survive collection, and replay restores the city. Checked game startup, movement, timer, model-loading errors, JSON export, and a 390 × 844 viewport. In the browser test environment, compatible-material merging reduced a desktop opening view from 784 to 533 draw calls; a portrait opening view used about 295.
- Checked all 61 models remain present in the less-dense city, measured 235 model bounds against their gameplay dimensions, verified equal-size rejection and rotated jaw contact, tested slower growth thresholds and building size gates, and checked spacing and three replays.
- Checked all five NPC variants load, animate, fit the starting size gate, enter intact for seven coins, and return on replay; checked portrait rendering and WebGL status.
- Checked the fear indicator and struggle animation for all five NPC variants, including intact intake, unchanged size gates, seven-coin payout, and replay.
- Checked all 61 relocated shortlist paths, startup with current and legacy paths, all five city-pack previews, and saved-shortlist path migration.
- Tested responsive canvas sizing at 320×568, 390×844, 430×932, 844×390, and 568×320. When emulating viewport changes in the browser test harness, dispatch a `resize` event after changing the viewport.
- No automated test suite is configured for this standalone prototype.

Keep this file current when changing gameplay, rendering, controls, dependencies, or important tuning values.
