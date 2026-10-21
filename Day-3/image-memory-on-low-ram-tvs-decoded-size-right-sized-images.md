# Image Memory on Low-RAM TVs

Images are the biggest memory user in a TV app. What counts is not the **file size** but the **decoded size** — the raw pixels kept in memory to draw the image.

---

## Decoded size

```text
decoded bytes = width × height × 4   (RGBA, 4 bytes per pixel)
```

| Image | File (JPEG) | Decoded in memory |
| --- | --- | --- |
| 3840×2160 (4K) | ~1 MB | **~33 MB** |
| 1920×1080 | ~300 KB | **~8 MB** |
| 320×180 (card size) | ~20 KB | **~0.2 MB** |

- A 300 KB file still costs **8 MB** once decoded.
- Home with 200 cards using 1080p images: 200 × 8 MB = **1.6 GB** → app killed on a 1–2 GB TV.
- Same 200 cards with right-sized images: 200 × 0.2 MB = **~46 MB**.

---

## Right-sized images

Request images at the **pixel size they are shown**, not the original.

```ts
import { PixelRatio } from 'react-native';

function imageUrl(base: string, widthDp: number, heightDp: number) {
  const w = PixelRatio.getPixelSizeForLayoutSize(widthDp);
  const h = PixelRatio.getPixelSizeForLayoutSize(heightDp);
  return `${base}?w=${w}&h=${h}&fit=crop`; // image CDN resizes on the server
}

<Image source={{ uri: imageUrl(item.image, scale(320), scale(180)) }} style={styles.card} />
```

- Needs an **image CDN / resize service** that accepts width and height.
- Cards: card size. Hero/background: screen size (1920×1080), only **one** at a time.
- Format: **WebP** works on all target engines; avoid AVIF on old LG/Samsung (needs Chromium 85+).

---

## Other tips

- **Virtualise** rails so off-screen images are released.
- **Limit the image cache** (memory and disk) in your image library.
- Android: `resizeMethod="resize"` on `Image` downsamples big images when decoding.
- Show a **solid color placeholder** while loading — no extra image.
- Don't preload images for rails far below the screen.
- Measure memory on the weakest device while scrolling all rails.

---

## Summary

- Memory cost = width × height × 4, not file size.
- Request every image at its display size from an image CDN.
- One full-screen image at a time; cards stay small.
- Virtualise, limit the cache, and measure on the weakest TV.
