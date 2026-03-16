+++
title = "A bold colorscheme idea for my redesign"
date = 2026-03-16
summary = "An idea for a bold new colorscheme"
toc = false
readTime = true
autonumber = true
math = true
tags = []
showTags = true
hideBackToTop = false
draft = true
+++

```css
/* CSS HEX */
--ivory: #ffffebff;
--yellow: #fffb00ff;
--pure-red: #f20505ff;
--inferno: #a60303ff;
--pitch-black: #141400ff;

/* CSS HSL */
--ivory: hsla(60, 100%, 96%, 1);
--yellow: hsla(59, 100%, 50%, 1);
--pure-red: hsla(0, 96%, 48%, 1);
--inferno: hsla(0, 96%, 33%, 1);
--pitch-black: hsla(60, 100%, 4%, 1);

color-scheme: light dark;
--content-primary: light-dark(--pitch-black, --ivory);
--background: light-dar(--ivory, --pitch-black);
--primary: --yellow;
--accent: --pure-red;

a {
/* Links as marked blocks? */
    color: var(--pitch-black);
    background-color: var(--primary);
}

```
