---
layout: post
title: Patterns of use of Vello crate
---

This document tries to establish patterns among a list of crates and Github projects using the [Vello](https://github.com/linebender/vello/) renderer.

The crate list isn't meant to be exhaustive, but it's pretty large: I've sifted through maybe 40 or so reverse dependencies of the Vello repository, up to the point where most of the READMEs I read were along the lines of "WIP: quick experiment paint shapes with Vello".

Don't expect anything groundbreaking. My main focus is on common patterns among people using Vello's [`Scene`](https://docs.rs/vello/latest/vello/struct.Scene.html) API; most of this is going to be pretty dry.


## Projects

I've mostly noticed two types of projects with Vello as a direct dependency:

- **Engine code** bridging Vello with some other format or framework, or providing wrapper functions for Vello's API. Ex: `vello_svg`, `bevy_vello`, etc.
- **Painting code** that renders specific things with Vello.

(Though in practice, a few blurred the lines, and a lot of projects that I put in one category or the other mostly ended up using Vello for example code or for very basic painting.)

**Engine code** projects:

- https://github.com/linebender/xilem/tree/main/masonry
- https://github.com/linebender/bevy_vello/
- https://github.com/linebender/velato/
- https://github.com/linebender/vello_svg/
- https://github.com/itlaohuo/Graphite-latest-stable/
- https://github.com/lapce/floem/
- https://github.com/yutannihilation/vellogd-r/
- https://github.com/marcpabst/renderer/
- https://github.com/bella-project/bella/
- https://github.com/Anti-Alias/gewy/
- https://github.com/mhfan/inlottie/
- https://github.com/Noxime/vg/
- https://github.com/DasLixou/snowberry/
- https://github.com/ratmice/selvage/
- https://github.com/sansseriff/svelte-vello/
- https://github.com/voxell-tech/velyst
- https://github.com/oku-gui/oku-gui/
- https://github.com/mysangle/starfish/

**Painting code** projects:

- https://gitlab.com/cyloncore/cartography-rs/
- https://github.com/ducharmemp/mj/
- https://github.com/D3lta-2-1/chess_game/
- https://github.com/ccleavinger/thInk/
- https://github.com/carterisonline/dagrid/
- https://github.com/crockeo/ekad/
- https://github.com/rosefromthedead/ut3/
- https://github.com/cfagot/space_survival/
- https://github.com/yamgent/wlte/
- https://github.com/alliby/fuu/
- https://github.com/jasoneveleth/chop-editor/
- https://github.com/Rottenfront/fp-arcanoid/
- https://github.com/rj45/digilogic

The numbers (18 engine projects, 13 app-like projects) aren't too surprising if you're familiar with the "Rust has 50 game engines and 3 games" stereotype. They also match my "gut feeling" reading the code, where it feels like a lot more projects use Vello in a very systematized way, as some kind of middleware or optional backend than as a plug-and-play dependency to paint a bunch of shapes.

For instance, I saw almost no project calling the `Scene::fill()` function more than five times.

This is confounded by the fact that I only looked for direct dependents of Vello; I might have found more app projects looking for dependents of Masonry or bevy_vello. Maybe the painting-privitive-heavy code is in the dependents of one of those engine code projects I cited.

Graphite, for example, is a primitive-heavy 2D editor, but its use of Vello is bottlenecked through a middleware layer with its own internal representation.

In general, though, my impression is that the stereotype is mostly true.

Another interesting pattern is that code actually using Vello to paint things was often in amateur stub projects, which is good for our purposes: it tells us how people who have little experience with Vello end up using it.


## Scene API usage patterns

### Fill and stroke arguments

`Scene::fill()` and `Scene::stroke()` are the most used methods by a very wide margin.

As a reminder, their prototype is:

```rust
pub fn fill(
    &mut self,
    style: Fill,
    transform: Affine,
    brush: impl Into<BrushRef<'_>>,
    brush_transform: Option<Affine>,
    shape: &impl Shape,
);

pub fn stroke(
    &mut self,
    style: &Stroke,
    transform: Affine,
    brush: impl Into<BrushRef<'_>>,
    brush_transform: Option<Affine>,
    shape: &impl Shape,
);
```

A lot code calling them looks like this:

```rust
    // https://gitlab.com/cyloncore/cartography-rs/-/blob/bf1fe0b8269b5f8133c09ab6df2ceab1a43141d2/cartography/src/vello.rs#L489-508
    if fill_color.alpha() > 0.0
    {
        self.scene.fill(
            peniko::Fill::NonZero,
            vello::kurbo::Affine::IDENTITY,
            &fill_color.brushify(),
            None,
            &shape,
        );
    }
    if symbol.stroke_width > 0.0
    {
        self.scene.stroke(
            &kurbo::Stroke::new(symbol.stroke_width),
            vello::kurbo::Affine::IDENTITY,
            &(symbol.stroke_color)(rendering_state, feature).brushify(),
            None,
            &shape,
        );
    }
```

Note the heavy usage of default values:

- `Fill::NonZero` is the default fill setting for most paint APIs.
- `Stroke::new(width)` creates a stroke with rount joins and caps, no dashes, and the given width.
- `transform` is set to `Affine::IDENTITY` for both methods.
- `brush_transform` is set to `None` for both methods.

These patterns can be found throughout the projects I've linked. In total, I've counted:

- 16 projects using `Scene::fill()` with `Fill::NonZero`, `IDENTITY` and `None`.
- 10 projects using `Scene::stroke()` with `Stroke::new(x)`, `IDENTITY` and `None`.
- 7 projects using `Scene::fill()` with `Fill::NonZero`, a transform and `None`.
- 3 projects using `Scene::stroke()` with `Stroke::new(x)`, a transform and `None`.

Projects that used all the arguments of `fill()` or `stroke()` were rare, and were generally written as middleware code passing these arguments from another source. For example:

```rust
    // https://github.com/linebender/vello_svg/blob/0dc847383eb6a839a6ec82c8d24a2e3adf9d4161/src/render.rs#L49-L86

    let do_fill = |scene: &mut Scene, error_handler: &mut F| {
        if let Some(fill) = &path.fill() {
            if let Some((brush, brush_transform)) =
                util::to_brush(fill.paint(), fill.opacity())
            {
                scene.fill(
                    match fill.rule() {
                        usvg::FillRule::NonZero => Fill::NonZero,
                        usvg::FillRule::EvenOdd => Fill::EvenOdd,
                    },
                    transform,
                    &brush,
                    Some(brush_transform),
                    &local_path,
                );
            } else {
                error_handler(scene, node);
            }
        }
    };
    let do_stroke = |scene: &mut Scene, error_handler: &mut F| {
        if let Some(stroke) = &path.stroke() {
            if let Some((brush, brush_transform)) =
                util::to_brush(stroke.paint(), stroke.opacity())
            {
                let conv_stroke = util::to_stroke(stroke);
                scene.stroke(
                    &conv_stroke,
                    transform,
                    &brush,
                    Some(brush_transform),
                    &local_path,
                );
            } else {
                error_handler(scene, node);
            }
        }
    };
```

These projects tended to have one instance code calling each Scene method in the entire repository.


### Other methods

Most of the projects I've looked at used the `fill()` and `stroke()` API exclusively.
Few of them used `draw_image()`, `draw_glyphs()`, `push_layer()`, etc.

Those that did tended to be the "render any arbitrary SVG" kinds of projects.


## Recommendations

Based on the above, I'd recommend having Vello export the following API:

```rust
pub fn fill(
    &mut self,
    brush: impl Into<BrushRef<'_>>,
    shape: &impl Shape,
);

pub fn stroke(
    &mut self,
    width: f64,
    brush: impl Into<BrushRef<'_>>,
    shape: &impl Shape,
);

pub fn fill_at(
    &mut self,
    transform: Affine,
    brush: impl Into<BrushRef<'_>>,
    shape: &impl Shape,
);

pub fn stroke_at(
    &mut self,
    transform: Affine,
    width: f64,
    brush: impl Into<BrushRef<'_>>,
    shape: &impl Shape,
);

pub fn fill_with(
    &mut self,
    style: Fill,
    transform: Affine,
    brush: impl Into<BrushRef<'_>>,
    brush_transform: Option<Affine>,
    shape: &impl Shape,
);

pub fn stroke_with(
    &mut self,
    style: &Stroke,
    transform: Affine,
    brush: impl Into<BrushRef<'_>>,
    brush_transform: Option<Affine>,
    shape: &impl Shape,
);
```

In summary:

- `fill()` and `stroke()` use the minimum number of arguments.
- `fill_at()` and `stroke_at()` use an additional `transform` argument.
- `fill_with()` and `stroke_with()` use the full API.

With this API, the `cartography-rs` code I quoted would look like this:

```rust
    if fill_color.alpha() > 0.0
    {
        self.scene.fill(&fill_color.brushify(), &shape);
    }
    if symbol.stroke_width > 0.0
    {
        self.scene.stroke(
            symbol.stroke_width,
            &(symbol.stroke_color)(rendering_state, feature).brushify(),
            &shape,
        );
    }
```

This would also let us remove most of the helpers in `paint_scene_helpers.rs` in Masonry.
