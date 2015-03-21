I use this website to test technical ideas and concepts that may make it to Flightfox. Some of it's for performance, some just for fun. See below for a list of the, albeit rudimentary, concepts tested.

## Design

<ul>
<li>Perfect vertical rhythm of all sections, text and images</li>
<li>SVG logo that scales, no need for separate 'retina' images</li>
<li>Only using 'rem' units, no px, em or pt</li>
<li>Icon font for the little 'read more' icon</li>
<li>Fully responsive layout and high-DPI resolution friendly</li>
<li>Increased browser text-size and zoom friendly</li>
<li>Optimized CSS for EOT, SVG, TTF and WOFF font formats</li>
<li>Uses Open Sans font, including the condensed version for headings</li>
<li>Uses HTML5 elements, including datetime and pubdate attributes</li>
<li><del>Larger intro paragraph text</del></li>
</ul>

## Performance

<ul>
<li>All content is static HTML</li>
<li><del>All resources gzipped and cached for 1 year</del> (now hosted on GitHub)</li>
<li><del>Custom Nginx MIME types to gzip fonts</del> (now hosted on GitHub)</li>
<li>Cache bursting in URI, opposed to query string</li>
<li>Minimal HTML5 markup, no compatability concessions</li>
<li>No Javascript, other than the Google Analytics snippit</li>
<li>All LessCSS files compiled into a single, minified file</li>
</ul>

## Just For Fun

<ul>
<li>No ID or classes in CSS, all element selectors (I know, I know)</li>
<li>No Javascript, other than the Google Analytics snippit</li>
</ul>


## To Do

<ul>
<li>Use a separate cookieless domain for assets</li>
<li>Build blog processor, minify HTML, make life easier</li>
<li>Microcache HTML pages with NGINX, maybe with Memcache</li>
<li>Convert to SSL and configure SPDY</li>
</ul>
