# Setup

## 1. Create the repository

The repository name must match your GitHub username **exactly**, including case.

```
github.com/new  ->  name: WnagarAryan
```

Make it public and initialize it with nothing. Then, from this folder:

```bash
git init -b main
git add .
git commit -m "feat: profile readme"
git remote add origin https://github.com/WnagarAryan/WnagarAryan.git
git push -u origin main
```

Delete `SETUP.md` and `preview.html` before pushing if you don't want them public —
neither affects rendering, they're just not part of the profile.

## 2. Editing the text later

`README.md` is fully filled in — no placeholders remain. Confirm with:

```bash
grep -o '{{[A-Z0-9_]*}}' README.md | sort -u    # prints nothing
```

The text baked into the SVGs lives here:

- `assets/header-terminal.svg` — `Aryan Nagar`, `AI System Architect | ML & Agentic AI`, `~/github/WnagarAryan`
- `assets/header-neon.svg` — `ARYAN NAGAR`, the three roles, `LOC:` and `STATUS:` labels
  (the font is embedded — adding a new character needs a new subset, see §6)
- `assets/footer-wave.svg` — the two footer lines
- `assets/banner-neurons-titled.webp` — text is burned into the frames, see §5

Changing the *length* of a line in `header-terminal.svg` means updating two things beside it.
The clip widths (`tb-c1`..`tb-c4`) are how far each line types out, and the `#tb-cursor` `x`
keyframes are where the cursor lands. Both are in pixels: this font measures **~10.5 px per
character at font-size 19**, so a line is `(glyphs x 10.5) + 10` wide, counting the `>` prompt
and the 10 px gap after it. A clip that is too narrow truncates the line mid-word; a cursor
value that is too large leaves it floating past the end of the text.

An ampersand in any of this text has to be written `&amp;` — these are XML files, and a bare
`&` makes the whole SVG fail to parse rather than degrade.

## 3. Enable the two game workflows

Both workflows run on push, on a schedule, and on manual dispatch. They need no secrets —
the built-in `GITHUB_TOKEN` is enough — but the repository must allow Actions to write:

```
Settings -> Actions -> General -> Workflow permissions
  -> Read and write permissions
```

Then run each one once by hand:

```
Actions -> "Generate Snake contribution graph"     -> Run workflow
Actions -> "Generate Pac-Man contribution graph"   -> Run workflow
```

They publish to **separate** branches on purpose:

| Workflow | Branch | Files |
|---|---|---|
| snake | `output` | `github-contribution-grid-snake{,-dark}.svg` |
| pacman | `output-pacman` | `pacman-contribution-graph{,-dark}.svg` |

Both force-replace their target branch, so pointing them at one shared branch makes each
run delete the other's output. That is why they are split.

Until the first successful run, those two images in the README are broken links. Expected.

## 4. Things that will break later

**`github-readme-activity-graph.vercel.app`** is the most failure-prone card in the file —
it is a hobby-tier Vercel deployment that regularly hits its limits and serves errors.
The same is true, less often, for `github-readme-stats.vercel.app` and
`github-profile-summary-cards.vercel.app`.

This is not hypothetical: **github-profile-trophy** and **github-readme-activity-graph** are
both returning HTTP 402 for every user as of the last check — their Vercel deployments are
over quota — and **github-readme-stats** was returning 503. The trophy and activity-graph
sections are commented out in `README.md` for that reason, and the stats cards were moved to
a mirror of the same codebase (`github-readme-stats-two-beta-28.vercel.app`).

Two options when one dies:

1. Delete or comment out that section.
2. Self-host it. Fork the upstream repo, deploy it to your own Vercel account for free,
   and swap the hostname in the README. This is exactly what the `Joaninnn` sample does
   (`github-readme-stats-two-beta-28.vercel.app` is their own fork).

The custom SVGs in `assets/` have no such dependency — they are served from your own repo
and cannot go down.

**Camo caching.** GitHub proxies every image through `camo.githubusercontent.com` and
caches it. After you edit an SVG and push, the README may keep showing the old version for
a while. Force a refresh with:

```bash
curl -s -X PURGE "<the camo url from the rendered page>"
```

Or just wait — it clears on its own.

## 5. Regenerating the masthead

The banner is `imlz5lae1m.mp4` cropped to 3:1 and encoded as an animated WebP. GitHub
strips `<video>` from README markdown, so an mp4 cannot play there — the clip has to become
an animated image. WebP is used rather than GIF because GIF caps at 256 colors per frame,
which visibly bands this clip's blue gradient, and because the whole frame moves every
frame, so GIF's inter-frame compression buys nothing (1.1 MB vs 5.5 MB for the same clip).

Clean banner:

```bash
ffmpeg -i imlz5lae1m.mp4 \
  -vf "crop=1920:640:0:220,fps=12,scale=1000:-1:flags=lanczos" \
  -c:v libwebp -q:v 65 -compression_level 6 -loop 0 -an \
  assets/banner-neurons.webp
```

Titled banner — the name is drawn into the pixels, so rerun this whenever it changes:

```bash
# ab.ttf / asb.ttf must sit in the working directory - see section 6.
# A Windows absolute path breaks drawtext: ffmpeg splits filter options on ":"
# and the drive letter's colon is eaten before any escaping applies.
ffmpeg -i imlz5lae1m.mp4 -vf "\
crop=1920:640:0:220,fps=12,scale=1000:-1:flags=lanczos,\
drawbox=x=0:y=0:w=1000:h=333:color=black@0.42:t=fill,\
drawtext=fontfile=ab.ttf:text='ARYAN NAGAR':fontcolor=white:fontsize=88:x=(w-text_w)/2:y=94:shadowcolor=black@0.85:shadowx=0:shadowy=4,\
drawbox=x=220:y=202:w=560:h=3:color=0x9ECBFF@0.9:t=fill,\
drawtext=fontfile=asb.ttf:text='AI SYSTEM ARCHITECT  |  ML \& AGENTIC AI':fontcolor=0xCFE2FF:fontsize=27:x=(w-text_w)/2:y=222" \
  -c:v libwebp -q:v 65 -compression_level 6 -loop 0 -an \
  assets/banner-neurons-titled.webp
```

`crop=1920:640:0:220` takes a 640 px tall band starting 220 px down the 1080p frame. Move
that last number to reframe; raise `-q:v` for better quality and a bigger file.

### The forest in the About Me sidebar

`assets/forest-mist.svg` is a still photo animated in SVG rather than re-encoded as a video.
The image is cropped to 576x1094, downscaled to 440 px wide, saved as JPEG and embedded as a
base64 data URI, so the file is self-contained at 36 KB and the motion — the breath, the
three drifting fog banks, the floating motes — is CSS on top of it.

Swapping in a different image means regenerating the file:

```bash
ffmpeg -i <your-image> -vf "crop=576:1094:0:110,scale=440:-2:flags=lanczos" -q:v 4 forest.jpg
```

then base64 it and replace the `href="data:image/jpeg;base64,..."` value in the SVG. Crop
values must stay within the source dimensions or ffmpeg refuses the filter outright.

The aspect ratio is deliberate. The README renders this at `width="100%"` so it fills its
table cell, and roughly 1:1.9 is what it takes to match the height of the code block beside
it. A squarer crop leaves dead space under the image; a taller one pushes the row past the
code block and moves the gap to the other side.

Remote images do not work here. GitHub serves the SVG through its camo proxy, and an SVG
loaded as an `<img>` cannot fetch anything external — that is the same restriction the
Pac-Man workflow works around by inlining its ghost sprites.

## 6. Fonts

Everything with type in it uses **Archivo** - `Archivo Black` for display lines, `Archivo
SemiBold` for supporting ones. Both come from Google Fonts under the OFL:

```bash
curl -L -o ab.ttf  "https://github.com/google/fonts/raw/main/ofl/archivoblack/ArchivoBlack-Regular.ttf"
curl -L -o var.ttf "https://github.com/google/fonts/raw/main/ofl/archivo/Archivo%5Bwdth%2Cwght%5D.ttf"
```

`asb.ttf` is that variable font pinned to wght 600 with `fontTools.varLib.instancer`. Both
TTFs are build inputs for the banner's ffmpeg command, not repository files.

The SVGs are a different problem. GitHub renders them as `<img>` through its camo proxy, and
an SVG in that position cannot fetch anything external - a Google Fonts `<link>` is silently
ignored and the text falls back to a system font. So the face has to travel inside the file,
as a base64 `@font-face` in the SVG's own `<style>`.

Shipping a whole font that way would be wasteful, so each is subsetted to the 84 characters
actually used: Archivo Black lands at 10.6 KB, SemiBold at 7.2 KB.

```python
from fontTools import subset

CHARS = ("ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz"
         "0123456789 .,:;!?'\"|/()&-+@#\u00b7\u2013\u2014")

opts = subset.Options()
opts.flavor = "woff"          # not woff2 - that encoder needs the brotli extension
opts.layout_features = ["kern"]
font = subset.load_font("ab.ttf", opts)
s = subset.Subsetter(options=opts)
s.populate(text=CHARS)
s.subset(font)
subset.save_font(font, "archivo-black.woff", opts)
```

A character outside `CHARS` renders as a missing glyph, not as a fallback - so widen `CHARS`
and regenerate both subsets before adding an accent or an em dash to any SVG.

`header-terminal.svg` stays monospace on purpose. A terminal set in a proportional face stops
reading as a terminal, and its typing animation is built on a fixed 10.5 px advance per
character - every clip width and cursor keyframe in section 2 derives from that number.

## 7. Project banners

`assets/project-fraudlens.svg` and `assets/project-kuberis.svg` are hand-built to each app's
own theme rather than pulled from `gh-card.dev`, whose cards render white and read as foreign
objects on a dark profile.

| | FraudLens | Kuberis |
|---|---|---|
| ground | `#0e1215` | `#e4ddcf` ruled paper |
| ink | `#d9e1e6` | `#14293e` |
| accent | `#4d9eff` | `#a9832f` |
| keyline | `#e8e8e8` | `#c9bfa8` |
| display face | Archivo SemiBold (embedded) | Playfair Display Bold (embedded, 2.2 KB) |

Both are 520x230 and rendered at `width="100%"`, so they scale to whatever the table cell is.
Colours were sampled from the running apps, not guessed.

One caution learned the hard way: a CSS animation whose opacity starts at `0` left the Kuberis
subtitle invisible in every frame sampled, at every phase, with the keyframes both renamed and
simplified. The cause was never isolated, so that line is static. If you animate text in these
files, animate between two non-zero opacities and check a render before trusting it.

## 8. The photographic set

Four of the SVGs are built around photographs rather than drawn from scratch. Each embeds its
image as a base64 JPEG, for the same reason the sidebar does: an SVG rendered through camo
cannot fetch anything external.

| File | Source | Treatment |
|---|---|---|
| `banner-bust.svg` | `download (2).jpg` | keyed to alpha, split into three piece PNGs |
| `header-neon.svg` | `download (1).jpg` | scaled to 630x280, sat on the right, faded left |
| `header-terminal.svg` | `download (3).jpg` | cropped above the lettering, blurred, darkened |
| `footer-wave.svg` | `download (4).jpg` | scaled to 1000x333, cropped to a 170 px band |

```bash
ffmpeg -i "download (1).jpg" -vf "scale=630:280:flags=lanczos,eq=brightness=-0.04:contrast=1.06" -q:v 4 statue.jpg
ffmpeg -i "download (3).jpg" -vf "crop=518:134:110:0,scale=1000:280:flags=lanczos,gblur=sigma=4,eq=brightness=-0.14:contrast=0.95" -q:v 4 clouds.jpg
ffmpeg -i "download (4).jpg" -vf "scale=1000:333:flags=lanczos,crop=1000:170:0:96,eq=brightness=-0.05" -q:v 4 horses.jpg
```

Two of those treatments are load-bearing, not taste.

**The bust is keyed, not blended.** The banner cuts the sculpture into its three pieces and
slides them apart. Two earlier attempts failed: screen-blending the JPEG left a visible rectangle
because the photo's background is dark but not black, and crushing the blacks with a `curves`
filter worked only until a `brightness` lift after it pushed black back up to 9 and the blocks
returned. The fix is a real alpha channel — luminance under 0.14 fully transparent, over 0.30
fully opaque, smoothstep between, built in Pillow:

```python
LO, HI = 0.14, 0.30
lut = [0 if v/255 <= LO else 255 if v/255 >= HI else
       int(round((lambda t: t*t*(3-2*t))((v/255-LO)/(HI-LO))*255)) for v in range(256)]
alpha = im.convert("L").point(lut).filter(ImageFilter.GaussianBlur(0.5))
im.putalpha(alpha)
im.crop(alpha.getbbox()).save("bust.png", optimize=True)
```

`LO` is the number that matters. Below about 0.10 the ambient glow between the pieces survives
and bridges them into one mass.

**Horizontal bands cannot split this image.** Clipping the keyed PNG into three rows looked right
until you watched it: the skull occupies rows 103-207 of the source and the jaw starts at row 194,
so they overlap by thirteen rows and any single cut row hands part of the skull to the jaw, which
then drift together. `split_bust.py` labels connected components in the alpha instead, keeps the
three largest, assigns loose specks to the nearest one, and writes a PNG per piece. Each is then
positioned by its own bounding box and animated on its own clock — no clip paths at all.

A caution on verifying this. Chrome's `--virtual-time-budget` advances SMIL but paints CSS
animations at their **first frame**, so a headless screenshot cannot show CSS-driven motion at all,
however long the budget. To check geometry, freeze the pieces at their extremes instead:

```css
#bb-p1 { transform: translateY(7px); }
#bb-p2 { transform: translateY(-9px); }
#bb-p3 { transform: translateY(6px); }
```

**The clouds' crop.** `download (3).jpg` has "GOD'S PLAN" set across its middle. Scaled to fill
the header, that lettering lands exactly on the typed `user` and `role` lines. Blurring it was
not enough - it stayed legible and fought the terminal text - so the crop takes the clouds above
it instead. If you want the lettering back, it needs to sit clear of x 46-540, y 86-215.

## 9. Optional extras

Not wired up, but drop-in if you want them:

- **WakaTime coding stats** — needs a WakaTime account, an API key in repo secrets, and
  the `athul/waka-readme` action writing into a marked block in the README.
- **Spotify now-playing** — `novatorem/novatorem` deployed to your own Vercel.
- **Latest blog posts** — `gautamkrishnar/blog-post-workflow` reading an RSS feed.
