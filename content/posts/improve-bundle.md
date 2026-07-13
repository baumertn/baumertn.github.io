+++
draft = true
title = "Shrinking a multi-page JavaScript bundle by 50.3% (on average) and achieving a (up to) 98.4% speed increase"
summary = "Fixing bugs, lazy loading, and conditional builds."
date = 2026-07-13
toc = true
readTime = true
autonumber = true
math = true
tags = []
showTags = true
hideBackToTop = false
+++

I shrunk our multi-page `webpack` bundle by 50.3% on average and even 61.8–98.4% faster.
The 98.4% are very optimisitc with much caching. But cache was not enabled before!

Here are the numbers:

### Build Times
| Run | Runtime |
|--------|--------
| Original | ∞ (Crashed) |
| Original, with`--max-old-space-size=4096` | 170445 ms |
| After, empty cache| 65182 ms |
| After, with cache | 2744 ms |


- **Empty cache ~61,8 % faster**
- **With cache ~98,4 % faster**


### Bundle Size
| Run    | Minimum (MiB) | Maximum (MiB) | Avergae (MiB) |
|--------|---------------|---------------|---------------|
| Before | ~0.52         | 9.67[1]       | ~5.78         |
| After  | ~0.52         | 4.49[2]       | ~2.87         |

- **On average 50.3 % smaller bundles**
- **The largest bundle is 53.6 % smaller**
- **The smallest bundle stayed the same**

Notes:
- [1]: A wrong import was made here to import dev tooling
- [2]: The page from [1] reduced 9.67 to 4.26 MiB (still a large!)



## Motivation
Despite this not having anything directly to do with our infrastrcture I took a look because our self-hosted CI stalled.

It seemed that the workers entered a zombie state where the GitHub UI showed them offline but on the servers they supposedly still did work that didn't progress.

I realized the source for these stalls was our `npm run build` (⇒ `webpack --mode=production`).

Running this localy lead to `FATAL ERROR: Ineffective mark-compacts near heap limit Allocation failed - JavaScript heap out of memory`.

The same error was also occuring on a staging server so the team lead used `NODE_OPTIONS=--max-old-space-size=4096 webpack --mode=production` as a workaround to build there.
This woraround was pushed to the repo.


## Changes

### Source Maps

Our build generates source maps to be used with Sentry. But we generated them everytime `--mode=production` was set. Even in the CI.

The fix: just gate the source maps behind the existance of the Sentry token.

This alone was enough to resolve the crash on my machine.

### IDE Auto-Import

Two modules that where used on one page imprted the whole `sass` stack for `Exception`. It was used like this: `throw Exception('…')`.
I assume someone was in a Python mindset and had their IDE auto-import this.
This code was probably either not tested at all or the missing `new` threw an exception and the devloper was happy either way.

The fix, of course, is easy: `throw new Error('…')`.

### Lazy Loading
As our bundles are per page, lazy loading is mostly not needed. But we have a `ready.js` file that runs on every page to do some shared initialization.

Would be a shame, if we loaded something heavy there, right? All fixed include just lazy loading heavier dependencies that are in the critical path.

For development the team uses `AXE`. This was imported in the `ready.js` and neatly gated behind a flag. Well, the import was not.

The software offers a bug report tool that takes screenshots of the page with `html2canvas`. This is a large depnendency that was loaded on every page as well.
No it's lazily loaded in the function that needs it. While it makes that funtion slower, it's a worthwhile tradeoff.

Last but not least, we have a `onAfterSetup` callback function that calls functions after everything else is done. I've added some imports here as well for heavier modules (one loads a whole Markdown stack).


### Small Changes
- Activated Webpacks file cache
- Replacing the JavaScript bundle of FontAwesome with just the CSS and Webfonts. Small improvement, but the JS was not used.



