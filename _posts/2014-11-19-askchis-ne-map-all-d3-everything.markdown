---
layout: post
nav: Blog
status: publish
published: true
title: 'AskCHIS NE map: All D3 Everything'
writer:
  display_name: Andy Woodruff
  login: andy
  email: andy@axismaps.com
  url: http://www.cartogrammar.com/blog
author_login: andy
author_email: andy@axismaps.com
author_url: http://www.cartogrammar.com/blog
wordpress_id: 1855
wordpress_url: http://www.axismaps.com/blog/?p=1855
date: '2014-11-19 14:59:48 +0000'
date_gmt: '2014-11-19 19:59:48 +0000'
categories:
- Portfolio
tags: []
comments: []
project: chis
---

<p>Last week saw the launch of <a href="http://askchisne.ucla.edu/ask/_layouts/ne/dashboard.aspx#/" target="_blank">AskCHIS Neighborhood Edition</a>, a product of The California Health Interview Survey and the UCLA Center for Health Policy Research, with whom we worked to develop map and chart components for this new interactive tool. The short story of AskCHIS NE is that it is a tool for searching, displaying, and comparing various California health estimates at local levels such as zip codes and legislative districts. <a href="http://askchisne.ucla.edu/ask/_layouts/ne/dashboard.aspx#/" target="_blank">Take a look at it</a> if you feel like setting up an account, or you can watch a <a href="https://connectpro72759986.adobeconnect.com/_a782517175/p4martnp8f6?launcher=false&fcsContent=true&pbMode=normal" target="_blank">demo at its launch event</a> (demo begins at 14:00).</p>
<p><a href="{{ site.baseurl }}/media/posts/2014/11/chis_screenshot.jpg" target="_blank"><img src="{{ site.baseurl }}/media/posts/2014/11/chis_screenshot_med.jpg" alt="AskCHIS Neighborhood Edition" width="600" height="514" class="aligncenter size-full wp-image-1875" /></a></p>
<!--break-->
<p>The long story is interesting too, as this is fairly sophisticated for a public-facing tool, so we'd like to share a few details of how the map and charts were made.</p>
<p>We built that big component with the map, bar chart, and histogram. It lives in an iframe, and as a user makes selections in the table above, various URL hash parameters are sent to the iframe telling it what data to load. The map and bar chart then make requests to a data API and draw themselves upon receiving data. Almost everything here uses <a href="http://d3js.org" target="_blank">D3</a>, either for its common data-driven graphic purposes or simply for DOM manipulation. As usual, this much D3 work made it a fun learning experience. And it once again expanded my awe for Mike Bostock and Jason Davies and their giant brains: more than once during this project, I asked about apparent TopoJSON bugs far beyond my comprehension, and each time they fixed the bug almost immediately.</p>
<p><img src="{{ site.baseurl }}/media/posts/2014/11/commit.png" alt="Davies/Bostock mega-brain" width="503" height="47" class="aligncenter size-full wp-image-1856" /></p>
<h3>Tiled maps in D3</h3>
<p>The map shows one of six vector layers on top of a tiled basemap that is a variation on Stamen's <a href="http://maps.stamen.com/toner" target="_blank">Toner</a> map, derived from Aaron Lidman's <a href="https://github.com/aaronlidman/Toner-for-Tilemill" target="_blank">Toner for TileMill</a> project. Normally we use Leaflet for such tiled maps, but our needs for the vector overlays were complex enough that it made more sense to adapt the basemap to the vector layers' framework (D3) rather than the other way around. Tiled maps in D3 are pretty painless if you follow the examples of <a href="http://bl.ocks.org/mbostock/4132797" target="_blank">Mike Bostock</a> and <a href="http://bl.ocks.org/tmcw/3943330" target="_blank">Tom MacWright</a>. The six vector layers are loaded from <a href="https://github.com/mbostock/topojson/wiki" target="_blank">TopoJSON</a> files and drawn in the usual fashion in the same Mercator projection as the basemap.</p>
<h3>Dynamic simplifcation</h3>
<p>The most interesting technical detail of the map (to me, anyway) is that it uses dynamic scale-dependent simplification. This is a nice design touch, but more importantly it ensures that the map performs well. Zip codes need to have reasonable detail at city scale, but keeping that detail at state level would slow panning, etc. tremendously. We took (once again) <a href="http://bl.ocks.org/mbostock/7755778" target="_blank">one of Mike Bostock's examples</a> and, after some trial and error, got it to work with the tiled map. Here's a <a href="http://bl.ocks.org/awoodruff/10750267" target="_blank">stripped-down example</a>. Simplification is based on the standard discrete web map zoom levels. As the user zooms, the vector layers redraw with appropriate simplification when those thresholds are crossed, with basic SVG transformations for scaling in between those levels. Keep your eye on the black-outlined county in the gif below, and you should be able to see detail increasing with zoom.</p>
<p><img src="{{ site.baseurl }}/media/posts/2014/11/chis_simplification.gif" alt="Dynamic simplification" width="361" height="361" class="aligncenter size-full wp-image-1858" /></p>
<h3>Pooled entities</h3>
<p>One of the more sophisticated capabilities of this tool is combining geographies for pooled estimates. For example, maybe you want to look at Riverside and San Bernardino Counties together as a single unit:</p>
<p><a href="{{ site.baseurl }}/media/posts/2014/11/pooled.jpg" target="_blank"><img src="{{ site.baseurl }}/media/posts/2014/11/pooled.jpg" alt="Pooled entities" width="300" height="188" class="aligncenter size-medium wp-image-1862" /></a></p>
<p>The API does the heavy lifting and delivers pooled data; our challenge was to display pooled geographies as a single entity on the map. Although TopoJSON seems to support combining entities (after all, it does know topology), I had no success, apparently because of a problem with the order of points. Instead we use some trickery of masking, essentially duplicating the features and using the duplicates to block out interior borders, as outlined below. If you wonder why we couldn't just skip step 2, it's because our polygons are somewhat transparent, so the interior strokes would still be visible if we simply overlaid solid non-stroked polygons.</p>
<p><img src="{{ site.baseurl }}/media/posts/2014/11/pooled_shapes.jpg" alt="Combining SVG shapes while preserving transparency" width="600" height="225" class="aligncenter size-full wp-image-1860" /></p>
<h3>Image export</h3>
<p>The biggest source of headaches and discovery was the image export function. Users can export their current map or bar chart view to a PNG image, for example the map below. (Click it.)</p>
<p><a href="{{ site.baseurl }}/media/posts/2014/11/export.jpg" target="_blank"><img src="{{ site.baseurl }}/media/posts/2014/11/export_med.jpg" alt="Map export" width="300" height="403" class="aligncenter size-full wp-image-1891" /></a></p>
<p>I learned that converting an HTML canvas element to PNG data is <a href="https://developer.mozilla.org/en-US/docs/Web/API/HTMLCanvasElement.toDataURL" target="_blank">built-in functionality</a>, but getting this to work in a user-friendly way was not so simple. For one thing, our map is rendered as SVG, not canvas. Luckily there is the excellent <a href="https://code.google.com/p/canvg/" target="_blank">canvg library</a> to draw SVG to canvas, so we build the entire layout above in SVG, then convert the whole thing to canvas (except the basemap tiles, which are drawn directly to the canvas). Pretty cool. Thanks, canvg!</p>
<p>The nightmares begin when trying to trigger a download, not just display an image on screen. Firefox and Chrome support a magic "<a href="http://html5-demos.appspot.com/static/a.download.html" target="_blank">download</a>" attribute on &lt;a&gt; elements, which does exactly what we want, downloading the linked content (image data, in this case) instead of opening it in the browser. If only everyone used those browsers! Cutting to the chase, we couldn't find any way to ensure the correct download behavior across browsers without <a href="http://greenethumb.com/article/1429/user-friendly-image-saving-from-the-canvas/" target="_blank">sending the image data to a server-side script</a> that returns it with headers telling the browser to download. The final task was showing a notification while the image is being processed both client-side and server-side, which can take long enough to confuse the user if there is no indication that something is happening. Failing to detect the download start through things like D3's <a href="https://github.com/mbostock/d3/wiki/Requests#xhr" target="_blank">XHR methods</a>, we ended up using a <a href="http://stackoverflow.com/a/4168965" target="_blank">trick involving browser cookies</a>.</p>
<p><a href="http://i.giphy.com/xMbC6ANm2vJrq.gif" target="_blank"><img src="{{ site.baseurl }}/media/posts/2014/11/cookie.gif" alt="COOKIE?!?!?!?!?!?!?!" width="499" height="281" class="aligncenter size-full wp-image-1878" /></a></p>
<p>Phew! Turns out making a static graphic from an interactive is complicated! But it was a valuable experience—subsequently I've even used JavaScript and D3 deliberately to produce a static map. It works, so why not? (Most of <a href="http://bostonography.com/wp-content/uploads/2014/11/boston_gov_2014.jpg" target="_blank">this dot map</a> was exported from the browser after drawing it using <a href="http://bl.ocks.org/awoodruff/94dc6fc7038eba690f43" target="_blank">this method</a>.)</p>
<h3>ALL the D3</h3>
<p>For all of the above, it's hard to post code snippets that would make sense out of context, but we hope that the links and explanations are helpful, and are happy to talk in more detail if you are working on similar tasks. There's a lot more going on than what I've described here, too. Suffice it to say, D3 is a pretty amazing library for web mapping!</p>
