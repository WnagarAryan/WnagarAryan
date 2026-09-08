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
ffmpeg -i imlz5lae1m.mp4 -vf "\
crop=1920:640:0:220,fps=12,scale=1000:-1:flags=lanczos,\
drawbox=x=0:y=0:w=1000:h=333:color=black@0.38:t=fill,\
drawtext=fontfile='C\:/Windows/Fonts/seguibl.ttf':text='ARYAN NAGAR':fontcolor=white:fontsize=62:x=(w-text_w)/2:y=110:shadowcolor=black@0.85:shadowx=0:shadowy=3,\
drawbox=x=310:y=182:w=380:h=2:color=0x9ECBFF@0.9:t=fill,\
drawtext=fontfile='C\:/Windows/Fonts/consolab.ttf':text='AI SYSTEM ARCHITECT  |  ML \& AGENTIC AI':fontcolor=0xBFD8FF:fontsize=20:x=(w-text_w)/2:y=196" \
  -c:v libwebp -q:v 65 -compression_level 6 -loop 0 -an \
  assets/banner-neurons-titled.webp
```

`crop=1920:640:0:220` takes a 640 px tall band starting 220 px down the 1080p frame. Move
that last number to reframe; raise `-q:v` for better quality and a bigger file.

### The forest in the About Me sidebar

`assets/forest-mist.svg` is a still photo animated in SVG rather than re-encoded as a video.
The image is cropped to 576x860, downscaled to 440 px wide, saved as JPEG and embedded as a
base64 data URI, so the file is self-contained at 32 KB and the motion — the breath, the
three drifting fog banks, the floating motes — is CSS on top of it.

Swapping in a different image means regenerating the file:

```bash
ffmpeg -i <your-image> -vf "crop=576:860:0:340,scale=440:-2:flags=lanczos" -q:v 4 forest.jpg
```

then base64 it and replace the `href="data:image/jpeg;base64,..."` value in the SVG. Crop
values must stay within the source dimensions or ffmpeg refuses the filter outright.

Remote images do not work here. GitHub serves the SVG through its camo proxy, and an SVG
loaded as an `<img>` cannot fetch anything external — that is the same restriction the
Pac-Man workflow works around by inlining its ghost sprites.

## 6. Optional extras

Not wired up, but drop-in if you want them:

- **WakaTime coding stats** — needs a WakaTime account, an API key in repo secrets, and
  the `athul/waka-readme` action writing into a marked block in the README.
- **Spotify now-playing** — `novatorem/novatorem` deployed to your own Vercel.
- **Latest blog posts** — `gautamkrishnar/blog-post-workflow` reading an RSS feed.
