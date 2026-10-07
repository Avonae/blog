---
title: Some Extensions in Chrome Are No Longer Available
date: '2024-10-22'
lastmod: '2024-10-22'
tags:
- Brave
- Chrome
- browser extensions
- Manifest V3
categories:
- News
translationKey: many-brave-extensions-have-broken
aliases:
- /2024-10-22-many-brave-Extensions-have-broken/
---

Today I woke up to find that a good third of my extensions in Brave had stopped working. Same deal with Chrome. So, what now?

# What Happened
Google rolled out a new version of the API, called "Manifest V3," which puts some heavy limits on ad blockers—and sometimes even makes them unusable. A major casualty of this update is the popular uBlock Origin, which relies on user-updated lists of bad sites. These lists used to update automatically, but Manifest V3 doesn’t allow that anymore. I use Brave, and uBlock Origin still works for now, but who knows for how long.

You can [read more about it the news](https://www.theverge.com/2024/10/15/24270981/google-chrome-ublock-origin-phaseout-manifest-v3-ad-blocker).

To fix the broken extensions, just head over to the "Extensions" page and click the "Repair" button for each of them. The extensions will be reinstalled.

![This is what broken extension looks like](grammarly-broken-extension.png)

However, this fix didn't work for Grammarly. Turns out it was straight up removed from the extension store. I couldn’t find any news on why. So, I guess we’re just waiting for updates...

![For some reason, the extension was completely removed from Chrome Web Store](grammarly-removed-chrome-web-store.png)

Link to the extension: [https://chromewebstore.google.com/detail/grammarly-grammar-checker](https://chromewebstore.google.com/detail/grammarly-grammar-checker/kbfnbcaeplbcioakkpcpgfkobkghlhen)