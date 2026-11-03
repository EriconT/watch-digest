# Watch digest: project brief

A personal tool for sourcing not-running Seiko kinetic and AGS watches to repair (capacitor swap) and resell. Twice a day, a scheduled task searches Japan (Yahoo Auctions and Mercari, bought through Buyee) and eBay US, then publishes the results to a private web page.

**This repo is the master copy.** Make every change here first, then publish it to the live page or task. Commit after each change with a short message, and add a line to CHANGELOG.md.

## How the parts connect

```
Google Sheet ──read──> Scheduled task ──writes──> digest data ──shown by──> Digest page
(price limits)         (runs in Chrome on          (two docs in the          (page/index.html)
                        the owner's PC)             page's database)
```

### 1. Google Sheet: "Watch Flipping Tracker"
- Spreadsheet ID `1Qu6cx2y_0R5iSvuouoD7dodNk27Ij31Goaa3JA1KZuE`.
- Tab **Watches** is the owner's inventory and P&L. Never touch it.
- Tab **Caliber targets** holds the buying limits. The task reads range A4:P60.
  - Assumptions B5:B16: B5 yen per dollar, B6 Buyee fee, B7 domestic shipping, B8 consolidation, B9 international shipping, B10 duty, B11 handling, B12 parts, B13 eBay fee share, B14 shipping out, B15 minimum profit, B16 Japan fixed cost per watch.
  - US assumptions: F5 sales tax, F6 default inbound shipping.
  - Table rows 19–33 (one per caliber/version):
    - E: eBay median sale
    - K: Japan max bid in yen
    - L: Japan verdict
    - O: US max price including shipping
    - P: US verdict
  - The owner edits B12 and B15 himself. Don't overwrite his values.

### 2. Scheduled task
- Name "Watch digest Update", ID `trig_01QgRZ3NbgRTeFKYaaXJ3NWG`.
- Runs at 6:43am and 7:43pm Los Angeles time (`CRON_TZ=America/Los_Angeles 43 6,19 * * *`). It is bound to the owner's computer ENRICOPC and uses Claude in Chrome.
- Its full instructions are in `task/prompt.md`. To change them:
  1. Edit that file.
  2. Commit.
  3. Push the branch.
  4. From a cloud Claude Code session, `update_trigger` with a new prompt is refused (`needs_device_approval`), because the task is bound to ENRICOPC. Don't attempt it. Instead, end your reply with **one ready-to-paste prompt** in a code block for the owner to paste as-is into a new Cowork chat. That's the only step. Don't give alternatives or extra instructions. Use this form, filling in the branch:
     ```
     Update my scheduled task "Watch digest Update" (trig_01QgRZ3NbgRTeFKYaaXJ3NWG): set its prompt to the full contents of task/prompt.md on branch <branch> of the GitHub repo EriconT/watch-digest. Change nothing else, and don't edit or commit anything in the repo. Then confirm with get_trigger that the live prompt matches the file.
     ```
  - The same applies to anything else that has to be done on the Cowork side: one self-contained prompt to paste into a new Cowork chat, nothing more.
- The task's own push setting is OFF on purpose. The prompt sends exactly one PushNotification at the end of each run. The notification setting can only be set when a task is created.
- Deleting the task's entry in the claude.ai sidebar deletes the task itself. Warn the owner about this if it comes up.

### 3. Digest data
- The page's built-in database, managed with ArtifactData.
- `digest/latest`: Japan results.
- `digest/us`: eBay US results.
- `passes/<key>`: listings the owner hid with the Pass button. The page writes these; the task never touches them.
- The document shape is defined in `task/prompt.md` (steps A8 and B7). Keep the page and the prompt in agreement whenever a field changes.
- Size limit is 256 KiB per document. Thumbnails are about 3 KB each.

### 4. Digest page
- Source: `page/index.html`.
- Live link: https://claude.ai/artifact/3tZ2eidxkhKx8cyMK2NfjW (capability `db`).
- To publish:
  1. Read the artifact (Artifact action `read`) once in the session.
  2. Publish `page/index.html` with `url` set to the link above.
- Never publish without `url`, because that creates a second page with a new link.
- External images are blocked on the page, so photos are stored as data URIs by the task's thumbnail script.
- Features:
  - Japan / US (eBay) switch.
  - "Buy now only" filter.
  - Light green highlight on listings that weren't in the previous run.
  - Pass button, plus a "Show hidden listings" toggle.
  - Sort dropdown: Best profit first (default), landed price high-to-low / low-to-high, auction end (uses `ends_at`, falls back to the `ends` label).
  - Choices are remembered in localStorage (`jwd-region`, `jwd-filter`, `jwd-sort`).

## Owner's standing rules
- When something needs the Cowork side, give the owner one prompt to paste as-is into a new Cowork chat. Just that one step (see Scheduled task, step 4).
- Never bid, buy, make offers, or sign in to any site. Every purchase needs the owner's explicit approval of the specific item and amount, and he does it himself.
- Scheduled runs are read-only except for writing the two digest documents. If Chrome is unavailable, the run uses no other browser.
- Send exactly one notification per run, when the whole run is finished.
- eBay: listings located in the US only (no imports or tariffs). eBay's own location filter is unreliable; the task's extractor checks the "Located in United States" line.
- On eBay, keep 5M62, 5M63 and Arctura in the search. Never skip a listing because of eBay's "Parts Only" condition label, since that's what he wants.
- Use short English descriptions with no IDs, stock codes or raw Japanese titles. Put links behind words.
- Don't change the order items are listed in: best profit first, then watching (ending soonest), then lots.
- Limits: Worth a look shows at most 24 per market; Watching and Lots at most 3 each.

## Technical notes
- **Chrome tool output:** capped at about 1,000 characters, and URLs with query strings are redacted. Keep `javascript_tool` output compact and parse with `fetch()` + DOMParser inside the tab.
- **Yahoo search pages:** contain a `__NEXT_DATA__` JSON with auctionId, price, bids, buy-now price, end time and image.
- **Photos:** go through `/tmp/add_thumbs.py`, which is embedded in the prompt. Its hosts are allowlisted in the cloud workspace: `auc-pctr.c.yimg.jp`, `static.mercdn.net`, `i.ebayimg.com`.
- **Dead listings:** Yahoo search results can include listings whose item page is gone. Step A5a fetches each shown listing's Yahoo page (from page.auctions.yahoo.co.jp) and Buyee page (from buyee.jp) in the tab and drops the dead ones.
- **Buyee plans:** Lite (free) under ¥8,000; Standard (¥500) at ¥8,000 and over, and for lots. Standard lowers the max bid by ¥500.

## Known issues
- Watch whether the eBay side over-filters (few results, many skips).
