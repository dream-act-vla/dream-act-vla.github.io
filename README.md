# DreamActVLA — Project Report Website

A blog-style project report page for **DreamActVLA: Accurate Asynchronous Inference for Vision-Language-Action Models**.

A single static HTML page — open `index.html` directly in a browser (no build step required), or host it on [GitHub Pages](https://pages.github.com/).

## How to Edit Content

All report content lives in **`index.html`**. It's plain HTML with a few conventions:

### Text & Headings

```html
<h1 id="section-title">Section Title</h1>
<h2 id="subsection">Subsection</h2>
<p>Regular paragraph text with <strong>bold</strong> and <em>italic</em>.</p>
```

### Math (KaTeX)

```html
Inline math: $E = mc^2$

Display math:
$$
\theta^* = \arg\max_\theta \mathbb{E}[R(\tau)]
$$
```

KaTeX renders these client-side — no build step needed.

### Tables

```html
<table>
  <thead>
    <tr><th>Method</th><th>SR (%)</th><th>Time (s)</th></tr>
  </thead>
  <tbody>
    <tr><td>Sync</td><td>93.5</td><td>9.02</td></tr>
    <tr><td>DreamActVLA</td><td>92.5</td><td>7.61</td></tr>
  </tbody>
</table>
```

### Images

Place images in `assets/img/` and reference them:

```html
<div class="figure">
  <img src="assets/img/your-figure.png" alt="Description of figure">
  <p class="caption">Caption text below the image.</p>
</div>
```

### Videos

**Local video file** — place your `.mp4` in `assets/img/` (or create an `assets/video/` folder) and use:

```html
<video width="100%" controls>
  <source src="assets/img/your-video.mp4" type="video/mp4">
</video>
<p class="caption">Caption text below the video.</p>
```

**YouTube embed** — grab the video ID from the URL (the part after `v=`) and use:

```html
<div style="position:relative;padding-bottom:56.25%;height:0;overflow:hidden;">
  <iframe src="https://www.youtube.com/embed/VIDEO_ID"
    style="position:absolute;top:0;left:0;width:100%;height:100%;"
    frameborder="0" allowfullscreen></iframe>
</div>
<p class="caption">Caption text below the video.</p>
```

**Tip:** Keep local videos under 50MB — GitHub has a 100MB file size limit. For longer clips, upload to YouTube and embed instead.

### Callout Boxes

```html
<blockquote>
  <p><strong>Key Insight:</strong> Your insight here — renders blue.</p>
</blockquote>

<div class="note-quote">
  <blockquote>
    <p><strong>Note:</strong> Your caveat here — renders warm yellow.</p>
  </blockquote>
</div>
```

### Adding TODO markers for collaborators

Use HTML comments that won't render on the page:

```html
<!-- TODO(Alice): Add results figure for dynamic benchmark -->
<!-- TODO: Write limitations section -->
```

---

## Preview Locally

Just open `index.html` in your browser — double-click it, or run:

```bash
open index.html
```

No server, build step, or dependencies required. Changes to `index.html` or `assets/css/style.css` are visible on a simple page refresh.

---

## File Structure

```
dream-act-vla/
├── index.html           # ← MAIN CONTENT — edit this file
├── assets/
│   ├── css/
│   │   └── style.css    # All styling
│   └── img/              # Figures and images
├── .nojekyll             # Tells GitHub Pages to serve files as-is
└── README.md             # This file
```

---

## Tips for Collaborators

- **Edit `index.html`** for content changes — it's plain HTML.
- **Add images** to `assets/img/` and reference them with relative paths.
- **Avoid merge conflicts** — coordinate who's editing which section. Use `<!-- TODO(YourName): ... -->` to claim sections.
- **Push to `main`** — GitHub Pages serves the updated `index.html` automatically (~1 min).
