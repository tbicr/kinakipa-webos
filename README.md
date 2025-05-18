# Intro

This is a single-page web app for https://kinakipa.site focused on working on LG TV with webOS.

# Dev

## Intro

It was tested on WebOS 3.5 with Chrome 38. This means the code should work on outdated browsers that support ES5 only.

For this reason, it uses the following stack:

- https://mithril.js.org/ for rendering + additionally converted to ES5 with https://jstool.gitlab.io/babel-es6-to-es5/
- fetch polyfill, because `mithril.request` doesn't work well on old browsers
- smoothscroll polyfill, because `element.scrollTo` isn't supported in old browsers
- kinakipa.site API for getting lists of movies and series and video details
- main logic and styles are placed in index.html

Conceptually, the app has a global state. Component (or global) events and fetch operations handle and update this state. When the state is updated, either automatically or manually, `mithril.redraw` is run, which draws the appropriate components or page views using global CSS.

## Run

First update `LIST_URL` and `DETAILS_URL` with relevant urls.

To run on TV, look at the quite short guide: https://webostv.developer.lge.com/develop/getting-started

Run on emulator:

    ares-launch --simulator-path ~/Downloads/webOS_TV_6.0_Simulator_1.4.1/ -s 6.0 .

Run on TV:

    ares-package . && ares-install -d tv kinakipa_1.0.0_all.ipk && ares-launch -d tv kinakipa

## Main Logic

On load, the app fetches a list of movies and additionally enriches it with be-latn names used for search. After this, users see a list of movies where they can pick a movie or open the left search panel.

On the search panel, users can input a search prompt or pick a prompt from history stored in localStorage. When a prompt is entered, movies are filtered by substring and ordered by relevance (be, be-latn, en titles search supported), and the results are presented on the movies list page.

On the selected movie page, the app first loads a list of available videos. After that, users can see details about the movie and pick a video from available videos, seasons, and episodes. The selected season and episode are stored in localStorage.

On the selected video, users can play, pause, and rewind. Currently, only 720p videos are supported.

## Issues

As the app is designed for TV, it solves several issues:

- TVs may use old browsers where some syntax or features don't work, and the simulator may have a newer browser version
- Navigation using buttons (OK, LEFT, RIGHT, UP, DOWN, BACK) requires components to be in focus at the component level (especially important for input and video nodes). Otherwise, only the global event handler can catch events. Sometimes component focus is missed and the global event handler runs `mithril.redraw`, then the component level runs focus back if necessary. To avoid double redraw, component event handlers should use `event.stopPropagation()`
- Scrolling uses `scrollTo` to manually focus calculated elements. Scroll position is calculated manually to present better positioning and avoid jumps. In most cases, top/middle/bottom or left/right scroll cases are used. Use `event.stopPropagation()` to avoid scroll jump issues
- Video elements don't handle keys well, so manual event handling is used
- localStorage is used to remember search history and last episode watched
