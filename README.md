# DreamActVLA — Project Report Website

A blog-style project report page for **DreamActVLA: Accurate Asynchronous Inference for Vision-Language-Action Models**.

Built with [Jekyll](https://jekyllrb.com/) and hosted on [GitHub Pages](https://pages.github.com/)

## How to Edit Content

All report content lives in **`index.md`**. It's standard Markdown with a few extras:

### Text & Headings

```markdown
# Section Title
## Subsection
Regular paragraph text with **bold** and *italic*.
```

### Math (KaTeX)

```markdown
Inline math: $E = mc^2$

Display math:
$$
\theta^* = \arg\max_\theta \mathbb{E}[R(\tau)]
$$
```

### Tables

```markdown
| Method       | SR (%) | Time (s) |
|--------------|--------|----------|
| Sync         | 93.5   | 9.02     |
| DreamActVLA  | 92.5   | 7.61     |
```

### Images

Place images in `assets/img/` and reference them:

```markdown
![Description of figure](assets/img/your-figure.png)
*Caption text below the image.*
```

### Videos

**Local video file** — place your `.mp4` in `assets/img/` (or create an `assets/video/` folder) and use:

```html
<video width="100%" controls>
  <source src="assets/img/your-video.mp4" type="video/mp4">
</video>
*Caption text below the video.*
```

**YouTube embed** — grab the video ID from the URL (the part after `v=`) and use:

```html
<div style="position:relative;padding-bottom:56.25%;height:0;overflow:hidden;">
  <iframe src="https://www.youtube.com/embed/VIDEO_ID"
    style="position:absolute;top:0;left:0;width:100%;height:100%;"
    frameborder="0" allowfullscreen></iframe>
</div>
*Caption text below the video.*
```

**Tip:** Keep local videos under 50MB — GitHub has a 100MB file size limit. For longer clips, upload to YouTube and embed instead.


### Callout Boxes

```html
<div class="callout callout-insight">
<strong>Key point:</strong> Your insight here.
</div>

<div class="callout callout-warning">
<strong>Note:</strong> Your caveat here.
</div>
```

### Adding TODO markers for collaborators

Use HTML comments that won't render on the page:

```markdown
<!-- TODO(Alice): Add results figure for dynamic benchmark -->
<!-- TODO: Write limitations section -->
```

---

## Run Locally (Optional)

If you want to preview changes before pushing:

```bash
# Install Jekyll (one-time)
gem install bundler jekyll

# Serve locally
bundle install
bundle exec jekyll serve
```

Then open `http://localhost:4000` in your browser. Changes auto-reload.

---

## File Structure

```
dream-act-vla.github.io/
├── _config.yml          # Site settings (title, URL, etc.)
├── _layouts/
│   └── default.html     # Page template (header, footer, KaTeX)
├── assets/
│   ├── css/
│   │   └── style.css    # All styling
│   └── img/             # Figures and images
├── index.md             # ← MAIN CONTENT — edit this file
├── Gemfile              # Ruby dependencies
└── README.md            # This file
```

---

## Tips for Collaborators

- **Edit `index.md`** for content changes — it's just Markdown.
- **Add images** to `assets/img/` and reference them in the Markdown.
- **Preview on GitHub** — GitHub renders Markdown previews, though math won't show until the site builds.
- **Avoid merge conflicts** — coordinate who's editing which section. Use `<!-- TODO(YourName): ... -->` to claim sections.
- **Push to `main`** — GitHub Pages rebuilds automatically on every push (~1-2 min).
