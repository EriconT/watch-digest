# Changelog

## 2026-11-03
- Japan: new step A5a checks each listing's Yahoo page and Buyee page before it's shown, and drops dead ones ("page not found"), filling the gap with the next runner-up. Prompted by a 5M85 Sportura listing that kept appearing but led to a missing page.
- Task: fixed a blank "New Tab" left open in Chrome after each run. Setup now reuses the blank tab Claude's tab group starts with, and Part C closes every tab in that group. Applied to the live task by the owner.
- CLAUDE.md: noted that prompt changes need the owner's approval on ENRICOPC.
- Merged the A5a live check and the tab fix into one prompt, so the live task has both.

## 2026-11-02
- Page version 6: the US empty-state text now gives the correct run times (6:43am or 7:43pm).
- Moved the project into this repo. The page source and task instructions match what's live: page version 5, and the task prompt as of 2026-11-02 3:27pm PT.
- Worth a look raised from 15 to 24 listings per market.
- Added the Pass button and the "Show hidden listings" toggle (stored in the page's `passes` collection).
- Added the light green highlight for listings that are new since the previous run.
- Merged the Japan and eBay tasks into one task with a single notification at the end.
- eBay: US-located listings only, checked per listing.

## Earlier (before this repo)
- Built the "Caliber targets" tab, the Japan digest, photos stored as thumbnails, the Buy now only filter, the Buyee plan advice, and the US (eBay) market switch.
