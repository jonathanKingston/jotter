---
title: "A Case for poster on Canvas"
categories:
  - Web Standards
  - HTML
  - JavaScript
  - accessibility
---

The HTML `<canvas>` element has a gap that `<video>` solved years ago: there's no declarative way to provide a static visual representation. When JavaScript hasn't run, can't run, or the page is being archived, canvas content vanishes. I think a `poster` attribute could fix this.

![](/images/canvasposter/archive-canvas.svg)

## The Problem

One of HTML's strengths is that rendered state is largely recoverable from serialised form. Save a web page, and most of what you see can be reconstructed: text lives in the DOM, images have `src` attributes, video has `poster` for a static frame. Even dynamically modified DOM is capturable.

Canvas breaks this. The `<canvas>` element is just a rectangle with dimensions. The pixels exist only in memory, produced by JavaScript. Serialise the DOM and you get:

```html
<canvas width="400" height="300"></canvas>
```

The chart, the game frame, the visualisation: gone.

This isn't a minor edge case. Canvas powers data visualisations, image editors, games, PDF viewers, signature pads, and map overlays. All of this becomes a black hole when archiving. The Internet Archive, national libraries, legal discovery: all struggle with canvas-heavy pages because there's no declarative way to capture what was rendered.

## The User Experience Problem

Canvas heavy pages leave users staring at blank rectangles while JavaScript loads. A dashboard with multiple charts, a page with an interactive map, a document viewer: all show nothing until scripts execute and draw. On slower connections or devices, that's a noticeable wait with no visual feedback.

A `poster` attribute would change this. The poster image loads and renders like any other image, so users see meaningful content immediately while JavaScript does its work in the background. The page feels faster because something is there from the start.

This also helps with Core Web Vitals. A blank canvas contributes nothing to Largest Contentful Paint (LCP), but a poster image becomes an LCP candidate immediately.

## The Idea

Add a `poster` attribute to `<canvas>`, mirroring `<video>`:

```html
<canvas width="400" height="300" poster="chart-snapshot.png">
  <table><!-- accessible data for screen readers --></table>
</canvas>
```

The poster displays before JavaScript draws to the canvas, then disappears once drawing begins, just like video. Fallback content stays separate. The poster provides a visual placeholder; the fallback table provides semantic data for screen readers. A chart needs both.

For archiving, when you save a page (via "Save As", web archive formats, or tools like the Wayback Machine), the serialisation step would capture the current canvas bitmap and store it in the poster attribute. The saved page then has a declarative record of what was drawn, viewable without re-executing JavaScript.

<figure class="polaroid">
  <img src="/images/canvasposter/screen-recording.gif" alt="Demo of canvas poster in action">
  <figcaption>Poster visible immediately, replaced once canvas draws</figcaption>
</figure>

I've put together an [explainer](https://github.com/jonathanKingston/canvas-poster/) with the full proposal. There are also prototype patches for [Chromium](https://github.com/jonathanKingston/canvas-poster/blob/main/patches/chromium-canvas-poster.patch), [Firefox](https://github.com/jonathanKingston/canvas-poster/blob/main/patches/firefox-canvas-poster.patch) and [WebKit](https://github.com/jonathanKingston/canvas-poster/blob/main/patches/webkit-canvas-poster.patch) if you want to try it out.

## Get Involved

If you work on web archiving, browser implementation, or accessibility tooling and this resonates, I'd welcome feedback.

The web is increasingly dynamic, but static representations remain essential for preservation, printing, and graceful degradation. Canvas shouldn't be a black hole where content goes to become unarchivable.
