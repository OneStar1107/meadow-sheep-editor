# Recreation Prompt (English)

> This is a complete recreation prompt distilled from the final build. Paste it into an AI to generate the same single-file three.js scene from scratch.

You are a frontend engineer who is excellent at three.js and stylized scenes. Build an interactive miniature meadow scene as **one HTML file**, in a hand-painted paper-diorama / soft-clay style. Do not aim for photorealism.

## Hard constraints

- Deliver a single index.html that opens by double-clicking. Load three.js 0.160 via an importmap from unpkg, with OrbitControls. No build step, no npm.
- Generate every texture (grass, paddy soil, island mask) procedurally with Canvas 2D. Do not reference external images, models, or fonts.
- The page needs internet access to load three.js; show a Simplified Chinese fallback if it fails.
- All visible UI copy must be Simplified Chinese.

## Composition and art direction

- A 44 × 29 m rounded-rectangle turf island, with a Canvas alpha fuzzy edge, a thin dark-green side wall, and a little cream fringe, like a floating patch of turf.
- Camera at (0, 52.5, 34.5) looking at (0, 2, 0), FOV 35. Use a frontal high-angle view, not a 45° side view, otherwise the rectangular turf turns into a diamond and gets cropped.
- Warm directional light + hemisphere light, ACES tone mapping, VSM soft shadows (2048, blurSamples 12), plus a warm page vignette.
- Scene elements: 8 trees (one purple), 3 rocks, 3 clouds, a river from lower-left to upper-right, a beige dirt path, and a rice paddy at the lower right.

## Turf and ground

- A 2048 × 1400 hand-painted grass texture: light green base, dark/light grass strokes, a dirt path, scattered tiny yellow flowers.
- Build the island mask from a rounded rectangle with a jittered outline and blurred fringe. Inject it into the grass, river, and path shaders via onBeforeCompile so they fade at the island edge.
- Build the river and path from CatmullRom curves and ribbon geometry with soft edges. They must not extend beyond the turf.

## River

- Use a custom ShaderMaterial for the water: two noise layers scrolling downstream along v, pale-blue glints, bank foam, and a restrained sun specular.
- Mask the water with the island alpha so it never leaves the banks. Keep the water saturated blue, not a big white PBR highlight.
- Wind affects the water: slightly change the surface undulation, flow speed, glints, and foam, but keep the flow continuous and smooth. Do not let the ripples break into short white diagonal dashes.

## Rice paddy

- The paddy is an organic low mound, about 7 cm above the surrounding grass, with a smooth edge.
- Fill it with about 3200 instanced rice tufts. Each tuft has 8 deep-green leaves (0.6–0.95 m, gently arched outward) and 3 crossed-ribbon panicles that arch up and then droop. The panicles are amber-gold; the leaf tips are slightly lighter.
- Use InstancedMesh, disable castShadow and receiveShadow to avoid dirty grey shadows, and animate the sway in a vertex shader (leaves and panicles bend downwind, with traveling gust waves).
- Use a golden paddy-soil texture underneath, leaving a little soil visible. No photoreal textures.
- Sheep wandering targets must avoid the paddy.

## Sheep

- 11 sheep with cream wool puffs, black faces, and short legs. The hierarchy must be root → bob → body / neck / legs[4] / tail. Overall scale 1.25.
- Behavior state machine: idle, graze, walk, jump. A jump lasts about 0.82 s and reaches a world-space apex of about 0.62 m, then lands back in idle. Squash/stretch the body and tuck the legs while airborne.
- Sheep cast and receive shadows; a very soft contact shadow under each is fine.

## Interactions

- Left-click a sheep: select it, showing 8 handles, a blue label (“width × height m · distance m”), and a toolbar (idle / graze / walk / jump | undo / redo / duplicate / delete).
- Drag the blue selection box to move the sheep; the camera must not orbit while dragging.
- Hold the left mouse button on the grass and draw a loop: on release, spawn a new sheep inside that area without overlapping existing sheep.
- Left-drag on empty space or right-drag anywhere: orbit. Wheel: zoom. Ctrl/Cmd+Z: undo; add Shift for redo.
- Implement pointer states, disabled buttons, and a history stack. Duplicate increments the count; delete decrements it.

## Wind

- A global wind field: target strength changes randomly every 2–6 s (about 0.05–1.2), the direction swings slowly, and gust noise is layered on top.
- Light objects (tree canopies, rice leaves, panicles) always sway. Heavy objects (tree trunks) tilt only when the wind exceeds 0.5 (max about 3.4°), and must stay exactly still in light wind.
- Tree yaw is random; transform the world wind direction into each tree's local space before applying the rotation.
- River, rice, and trees share the same wind so direction and feel stay consistent.

## Time & weather

- Add a fixed Chinese time/weather panel in the top-right: a 24-hour clock, phase label (dawn/day/dusk/night), a 0–24 time slider, and sunny/rain/snow buttons. The panel is semi-transparent: its background is sampled from the current sky color (about 40% opacity) with a backdrop blur, and text turns light on dark skies.
- Start at 08:00 and complete a day in about 8 minutes; dragging the slider jumps to any time.
- The sun travels on an east-to-west arc. Dawn is peach, day is cream, dusk is orange-pink, night is deep blue with cool moonlight. Sky color, cloud color, exposure, and water tint follow the same time curve; page text turns light at night.
- Rain: hide the sun and moon, turn the sky grey-blue, darken the clouds, add about 2200 wind-slanted rain streaks and a faint rain haze; grass, trees, sheep, the paddy and the road darken and look wet, with lower roughness on grass and road.
- Snow: falling flakes; a snow layer thickens across the island, caps the tree crowns, and turns the paddy and road pale; the river freezes gradually into pale blue-white ice with frost lines and thaws slowly after the snow stops.
- Randomize the weather (sunny/rain/snow) on every page load. Show a sun in the daytime sky and a moon at night, moving along a time arc. They are fixed screen-space sky overlays and do not move with the camera. Rain fully hides both; snow dims them. Feather the riverbank and path edges so they blend naturally across weather states.
- Sunny/rain/snow transition smoothly over about 2 seconds; all visible UI copy stays Simplified Chinese.

## Hoofprints & default walking

- On page open every sheep starts in walk. Clicking a sheep starts it walking, and a lasso-spawned sheep also starts walking.
- While walking on snow (enough cover) or wet ground (enough wetness), a sheep leaves alternating hoofprints at about a 0.8m stride; snow prints are blue-grey, wet prints are dark. They fade in about 6–8 seconds and disappear within 1–2 seconds after switching to sunny. Prints sit on terrainH and must not sink through the ground.
- Snow on the paddy must look natural: snow sits on the field surface and the panicles, white caps cover the panicle tips, and leaves/stems still show through; it must not become a flat, detail-free white slab.

## Acceptance

- Double-click index.html opens directly; zero console errors; the page needs internet.
- The island is a rounded turf patch with a fuzzy edge; the river and path never leave it.
- Clicking a sheep shows 8 handles, the label, and the toolbar; dragging the box moves the sheep; clicking empty space deselects; lasso spawns a new sheep without overlap; undo/redo/duplicate/delete all work.
- Jump apex is about 0.62 m world space, duration about 0.82 s, landing back in idle.
- In light wind the trunk angle stays 0 while canopies move; only strong wind tilts trunks; rice and water react to wind.
- Dragging the time slider shows dawn, dusk, and night; sunny/rain/snow switch smoothly; rain shows streaks and wetness, snow shows cover/tree caps/frozen river and detailed paddy snow; the sun appears by day and the moon at night, weather randomizes on every load; page-open/selected/lasso-spawned sheep default to walking and leave short-lived hoofprints on snow/wet ground; river/path edges are feathered; the panel follows the sky color.
- All UI copy is Simplified Chinese.

## Output

Output one complete index.html only. No explanation, no extra files.