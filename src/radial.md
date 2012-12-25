Caterwaul Radial | Spencer Tipping
Licensed under the terms of the MIT source code license

# Introduction

Radial is a canvas-based UI library for Caterwaul. It allows you to represent user interfaces as immutable data structures, which makes it possible to manipulate interfaces with pure functions over
previous UI states and event objects. Radial provides several layers of abstraction, each built on the ones above it:

    1. Low-level rendering combinators. This simplifies the process of drawing on a context.
    2. Compositional offscreen buffer allocation. This simplifies composited rendering operations.
    3. Bounded components with rendering and event handling. Component hierarchies are not supported; any hierarchical elements need to be flattened prior to rendering.

Radial can also allocate <canvas> elements using caterwaul's jquery[] macro.

    caterwaul.module('radial', ':all', function ($) {
      $.radial = wcapture [

# Rendering combinators

These functions are first invoked with parameters, then the results are invoked on the graphics context. They do everything from building paths to manipulating the transformation matrix.

## Options handling

Some combinators, stroke() and fill() for example, accept a parameter specifying options that influence how the stroke or fill is done. This parameter is either a string (in which case it's interpreted
as a color), or an object containing one or more of these keys:

    1. line_width
    2. line_cap
    3. line_join
    4. miter_limit
    5. color (both strokeStyle and fillStyle will be set to this value)

You can also specify stroke_style and fill_style separately, and you can use the normal camelCased names as an alternative to the underscore_cased ones above.

      install_options_on(options, c) = options.constructor === String ? c.strokeStyle = c.fillStyle = options
                                                                      : options /pairs *![x0 = x[0].replace(/_(\w)/, given [_, x] in x.toUpperCase()),
                                                                                          x0 === 'color' ? c.strokeStyle = c.fillStyle = x[1] : c[x0] = x[1]] -seq,

## Path construction combinators

These are wrappers around methods used to build paths. You can construct paths by using the path() function, which takes a list of commands. These commands are invoked sequentially on the context
within the context of a new path. Afterwards the context's state is restored, so side-effects don't escape from the path() invocation. For example:

    path(prect(0, 0, 10, 10), fill())

The path() function is also used to introduce localized transformations:

    path(translate(0, 10), prect(0, 0, 10, 10), fill())

These transformations are undone after path() completes, and the path is erased as well. The only side-effect of a path() call is rendered output.

      path(fs = +arguments -seq)(c) = (c.save(), c.beginPath(), fs *![x.call(this, c)] -seq, c.restore()),

      fill(options)(c)   = (c.save(), options /-install_options_on/ c, c.fill(),   c.restore()),
      stroke(options)(c) = (c.save(), options /-install_options_on/ c, c.stroke(), c.restore()),
      pclose(c)          = c.closePath(),

      pmove(x, y)(c) = c.moveTo(x, y),  pquad(cx, cy, x, y)(c)               = c.quadraticCurveTo(cx, cy, x, y),           parcto(x1, y1, x2, y2, r)(c) = c.arcTo(x1, y1, x2, y2, r),
      pline(x, y)(c) = c.lineTo(x, y),  pbezier(c1x, c1y, c2x, c2y, x, y)(c) = c.bezierCurveTo(c1x, c1y, c2x, c2y, x, y),  parc(x, y, r, sa, ea, ac)(c) = c.arc(x, y, r, sa, ea, ac),
      prect(x, y, w, h)(c) = c.rect(x, y, w, h),

## Image rendering

A function to render an image from one element to another. Doesn't include any support for repositioning or scaling the image, but this can be achieved by using the transformation matrix.

      image(source, options, options = options || {})(c) = c.drawImage(source, 0, 0),

## Text rendering

You can easily render text from inside a path, but text operations themselves are not path transformations. Instead, the canvas context provides two new methods, fillText() and strokeText(). These
methods are represented here as a pair of functions, fill_text() and stroke_text(). You can specify options for each of these functions, which is useful for setting the font and specifying how the text
will be aligned. Text positioning should be done using translation.

      stroke_text(s, options)(c) = (c.save(), options /-install_options_on/ c, c.strokeText(s, 0, 0), c.restore()),
      fill_text(s, options)(c)   = (c.save(), options /-install_options_on/ c, c.fillText(s, 0, 0),   c.restore()),

# Compositing framework

This is a mechanism by which intermediate canvases are allocated and compositing is done between them. We do this by setting up a 'pipeline', which is a series of canvas elements linked with render
functions. Each render function takes a source and a destination canvas and is expected to perform the compositing for that stage. Once a pipeline is constructed, you can invoke it by using a compositing
render function:

    render = path(translate(10, 20), stroke_text('foo', {color: 'red'}));
    screen_canvas = document.createElement('canvas');
    composited_layer = composite(400, 300, screen_canvas, "image(composited_layer)(_)".qf);       // copy from composite layer to screen canvas
    composited_layer(render);

The last call kicks off the whole process. Once the render combinator structure returns, we then render the composite layer downwards onto the screen canvas.

    ]});