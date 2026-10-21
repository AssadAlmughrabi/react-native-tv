# Testing at 720p, 1080p, 4K, and on a Low-End Device

Run the same focus and navigation checks across different resolutions and hardware levels, because TV apps behave differently on each.

## Why resolution matters

| Resolution | What to watch for |
|---|---|
| 720p (1280x720) | Cramped layout, text overflow, fewer items per row, scaled focus pushed past the screen edge |
| 1080p (1920x1080) | The usual design baseline; confirm everything works as designed here first |
| 4K (3840x2160) | Some runtimes upscale from 1080p coordinates, others expose the real size; check sizes, spacing, images, and focus outlines are not tiny or misplaced |

**Check at each resolution:**

- Focus indicator is clearly visible
- Focused (scaled) items are not clipped at the edges
- Rows and grids scroll correctly as focus moves
- Text and images are sharp and the right size

## Why a low-end device matters

Developer machines and high-end TVs hide performance problems. Budget TVs have slower CPUs, less memory, and older web engines.

**Problems that mainly show up on low-end devices:**

- Focus lags behind the key press
- Key presses are dropped during rapid input
- Scrolling stutters or animations drop frames
- Large lists or images cause slowdowns or crashes
- Focus is lost because a re-render takes too long

The rapid-press test (mash Down) is most useful on a low-end device.

## Tips

- Use the emulator or simulator for quick resolution checks, then confirm on a real TV.
- Keep one older or budget TV as your slowest reference device.
- If it works on the low-end device, it will almost always work on faster ones.