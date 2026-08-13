# design/ index

Aesthetic systems and the places to get raw material. Everything in this folder is
**opt-in**. A look should never fire on its own, the way a coding standard does.

| Skill | Use it for |
|---|---|
| **[apple-design](./apple-design/SKILL.md)** | Apple's visual language: glass materials, spring physics, SF Pro type, iOS/macOS system chrome. Load only when someone asked for that aesthetic by name |

Same layout as [`../skills/`](../skills/README.md): one folder per skill, named after the
skill's `name`, so `cp -r design/apple-design .claude/skills/` is the whole install.

---

## Resources

Links, not dependencies. Pull what you need, then own it in your repo.

### Components

| Source | What it is |
|---|---|
| [21st.dev](https://21st.dev) | Component registry aimed at React and Tailwind |
| [canvasui.dev/components](https://canvasui.dev/components) | Component collection |
| [toolfolio.com/tools/canvas-ui](https://toolfolio.com/tools/canvas-ui) | Canvas UI, via Toolfolio |

### Icons

| Source | What it is |
|---|---|
| [phosphoricons.com](https://phosphoricons.com) | Open-source icon family with several weights per glyph. Pick one weight and stay on it |

### Illustrations and imagery

| Source | What it is |
|---|---|
| [undraw.co/illustrations](https://undraw.co/illustrations) | Open-source illustrations, recolorable to a single brand hue before download |

### Backgrounds, gradients, patterns, 3D

| Source | What it is |
|---|---|
| [haikei.app](https://haikei.app) | Generates SVG backgrounds: blobs, waves, layered shapes |
| [meshgradient.com](https://meshgradient.com) | Mesh gradient generator |
| [colorflow.ls.graphics](https://colorflow.ls.graphics) | Mesh gradients with WebGL effects |
| [studio.zoxilsi.cc](https://studio.zoxilsi.cc) | Mesh gradient studio, 100+ presets |
| [shadergradient.co](https://shadergradient.co) | Animated 3D shader gradients |
| [backgrounds.supply/gradient-lab](https://backgrounds.supply/gradient-lab) | Animated gradients, exports to 4K |
| [colir.space/app](https://colir.space/app) | Curve-based gradient creator |
| [gradientsaas.blogspot.com](https://gradientsaas.blogspot.com) | Editorial CSS gradients, copied in one click |
| [tabbied.com/patterns](https://tabbied.com/patterns/) | Generative patterns |
| [spline.design](https://spline.design) | Browser-based 3D design, exports for the web |

### Color systems and accessibility

Where the gradient tools above make one surface look good, these decide whether the whole
palette holds up.

| Source | What it is |
|---|---|
| [realtimecolors.com](https://realtimecolors.com) | Previews a palette live on a real UI layout instead of on swatches |
| [ramps.studio](https://ramps.studio) | Color ramps and tokens, checked against WCAG |
| [colorable.jxnblk.com](https://colorable.jxnblk.com) | Contrast testing for a foreground and background pair |

---

## Before you ship any of it

- **Check the license.** These sources do not share one. Free to browse is not the same as
  free to ship in a commercial product, and the answer can differ per asset.
- **Vendor it, do not hotlink it.** An external URL in production is an outage you do not
  control and a tracking vector you did not agree to.
- **Budget the weight.** 3D scenes and layered SVG backgrounds are the two easiest ways to
  turn a fast page slow. Measure after you add one.
- **An animated gradient is a render loop.** A WebGL or shader background keeps running for
  as long as the page is open, which costs battery on a laptop and more on a phone. Gate it
  behind `prefers-reduced-motion`, and pause it when the tab is hidden.
- **Check contrast against the gradient, not against a flat color.** Text over a gradient
  passes at one end and fails at the other. Test the worst point, not the average.
- **Pick one icon set and one illustration style.** Mixing sources is the fastest way to
  make a competent design look amateur.

## Adding a resource

Add a row to the table it belongs in, with a description of what the source actually is.
No adjectives, no ranking. If a category does not exist yet, add the heading.
