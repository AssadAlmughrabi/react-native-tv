# Font Sizes, Contrast, and Scrims

Text on TV is read from ~3 m, on screens with blooming backlights, often on top of busy artwork. Three rules: **big text**, **strong contrast**, **scrims behind text on images**.

---

## 1. Font sizes

Sizes on the 1920×1080 canvas (apply `scale()`):

| Use | Size |
| --- | --- |
| Hero title | 56–72 px |
| Screen / section title | 36–48 px |
| Card title, buttons | 28–32 px |
| Body text | 24–28 px |
| Metadata (year, duration) | 20–24 px |
| **Minimum** | **20 px** |

- Use **regular, medium, or bold** weights. Thin/light weights break up from the couch.
- Short lines (max ~60 characters), max 2–3 lines on cards, then ellipsis (`numberOfLines`).
- Line height ~1.3× font size.

```tsx
<Text numberOfLines={2} style={{ fontSize: scale(28), lineHeight: scale(36), fontWeight: '500' }}>
  {item.title}
</Text>
```

---

## 2. Contrast

- Text vs background: at least **4.5:1** (WCAG), aim higher for small text.
- Large titles: at least **3:1**.
- Avoid **pure white on pure black** for big areas — it glows and tires the eyes. Prefer off-white (`#EDEDED`) on dark gray (`#121212`).
- Avoid saturated red/orange for text — it smears on many panels.
- Don't use color alone to show focus or state; add scale, border, or brightness change.
- TVs show colors differently (brightness, "vivid" modes): check on a real TV, not only your monitor.

---

## 3. Scrims over artwork

A **scrim** is a dark gradient between artwork and text, so text stays readable on any image.

```tsx
import LinearGradient from 'react-native-linear-gradient';

<ImageBackground source={{ uri: hero.image }} style={styles.hero}>
  <LinearGradient
    colors={['rgba(0,0,0,0.85)', 'rgba(0,0,0,0)']}
    start={{ x: 0, y: 0.5 }}
    end={{ x: 0.7, y: 0.5 }}
    style={StyleSheet.absoluteFill}
  />
  <Text style={styles.heroTitle}>{hero.title}</Text>
</ImageBackground>
```

- Put the gradient on the **side where text is** (left for hero text, bottom for card titles).
- Never trust the image to be dark enough — artwork changes, the scrim stays.
- Web build: the same gradient can be a CSS `linear-gradient` background.
- Keep gradients simple; big blur effects are slow on weak TVs.

---

## 4. Short note on aspect ratios

- The screen is **16:9**. Design all layouts for it.
- Common artwork ratios: **16:9** (landscape cards, hero), **2:3** (posters), **1:1** (square, channels/music).
- Use `aspectRatio` instead of fixed height, and `resizeMode="cover"` so images fill without stretching.

```tsx
<Image source={{ uri }} style={{ width: scale(320), aspectRatio: 16 / 9 }} resizeMode="cover" />
```

- Video content can be wider (2.39:1) — the player shows black bars; don't stretch it.
- Request artwork in the ratio you show, so important parts aren't cropped.

---

## Summary

- Body text 24–28 px, never below 20 px on the 1920 canvas; avoid thin weights.
- Contrast ≥ 4.5:1; off-white on dark gray; check on a real TV.
- Always put a scrim between artwork and text.
- Screen is 16:9; use `aspectRatio` and `cover`, never stretch images or video.
