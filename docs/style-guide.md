# Visual and interaction style guide

SpaceDress exists to improve recognition. Its visual language should feel like **tailoring on top of Mission Control**, not a second window manager fighting for attention.

## Visual priorities

In order:

1. app/split identity;
2. user-assigned color or motif;
3. concise title;
4. document detail;
5. decorative artwork.

If decoration makes the underlying thumbnail harder to recognize, the styling is overdressed.

## Default composition

For a single full-screen app, the default synthesized style should be approximately:

- app icon in a consistent corner;
- one concise app title when useful;
- subtle border or accent derived from user choice or a deterministic neutral palette;
- no full-thumbnail tint unless it adds meaningful distinction.

The system app icon is a primary identity token and should not require an app-authored manifest.

## Split/tiled composition

A split Space should show **both** apps.

Preferred behavior when participant geometry is known:

- place each icon inside or adjacent to its participant region;
- clip app-supplied tint/artwork to that participant's region;
- permit a shared outer border only when it comes from a user-level whole-Space style;
- avoid implying that one app owns the entire Space.

When geometry is uncertain, use a neutral paired-icon treatment rather than guessing which side belongs to which app.

## Borders

Borders are among the safest high-recognition decorations because they preserve thumbnail content.

Guidelines:

- default width should scale modestly with thumbnail size and remain visually subordinate to the thumbnail;
- respect the native thumbnail corner radius rather than drawing visibly incompatible corners;
- avoid rapid animation;
- use high-contrast fallback behavior when macOS Increase Contrast is enabled.

A manifest may request a width/color; the renderer may clamp values to maintain usability.

## Color overlays

Tints should preserve the thumbnail's legibility.

- Treat manifest opacity as a request subject to user/accessibility policy.
- Prefer low-opacity color as an identity cue.
- Do not use an app manifest to make the thumbnail effectively opaque.
- Reduce or remove translucency when Reduce Transparency is enabled.

## Icons

Icon sources, in priority order:

1. explicit user-selected resource;
2. valid manifest resource;
3. actual runtime app bundle icon;
4. generic application fallback.

Do not identify an app by searching for an icon from another installation when the owning process already provides a bundle URL.

Icon backgrounds may use a macOS material/scrim for contrast. Avoid bespoke faux-macOS chrome.

## Text

Supported semantic sources include:

- application title;
- document/window title;
- user-defined title;
- manifest literal text where justified.

Text guidelines:

- use system typography by default;
- prefer one line;
- truncate rather than shrink into unreadability;
- use a material or scrim when text crosses complex imagery;
- never make document-title display mandatory;
- give users a global privacy switch for document/window titles.

### Terminology

Within SpaceDress, **title** means a displayed name. It never means rank, ownership, honorific, or role.

## Bitmap overlays

App-bundled bitmap artwork can add distinctive identity, but it is the most likely layer to obscure content.

Rules:

- resources must be local to the approved style package/app bundle;
- images are clipped to their permitted region;
- content mode is explicit (`fit` or `fill`);
- opacity is clampable by user policy;
- remote image URLs are not part of the base standard;
- animation is not part of manifest v0.1.

## Interaction

Mission Control owns the interaction.

SpaceDress overlays should:

- ignore mouse events by default;
- not steal keyboard focus;
- not create alternate click targets over native thumbnails;
- appear and disappear with Mission Control;
- avoid visible lag trails during dismissal;
- hide when frame alignment is not trustworthy.

A future interactive feature requires an explicit design decision because it changes this fundamental relationship.

## Motion

Use motion only to preserve attachment to the native thumbnail during system animation. Do not add decorative entrance, bounce, shimmer, or perpetual animation in the default experience.

Honor Reduce Motion.

## Accessibility

At minimum, test:

- Increase Contrast;
- Reduce Transparency;
- Reduce Motion;
- multiple display scales;
- light and dark desktop/app content;
- color-blind distinguishability when color is the only custom cue.

Icons and titles should keep Space identification possible when color differentiation is ineffective.

## Visual metaphor

The wardrobe vocabulary is useful for product navigation:

- a **style** is an outfit;
- user style collections may be called a **Wardrobe**;
- the visual editor may be a **Fitting Room**;
- this document is the **dress code**.

Do not carry the metaphor into low-level APIs where literal names are clearer. A type should be called `SpaceParticipant`, not `TrouserLeg`.
