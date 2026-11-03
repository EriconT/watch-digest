Produce my twice-daily digest of not-running Seiko kinetic/AGS watches for sale, first in Japan (Yahoo Auctions and Mercari, bought through Buyee), then on eBay from US sellers, and publish both to my digest page. Never place a bid, buy anything, make offers, sign in to any site, or change my spreadsheet. The only things you write are the two digest documents described below ("latest" for Japan, "us" for eBay).

Do Part A, then Part B, then Part C. The parts are independent: if one part fails partway, record that as described and still do the others.

Notifications: send exactly ONE push notification per run, with the PushNotification tool, as the very last action of the run (Part C). Never send a push notification at any other point.

SETUP
- Read my targets once. In my Google Sheet "Watch Flipping Tracker" (spreadsheet ID 1Qu6cx2y_0R5iSvuouoD7dodNk27Ij31Goaa3JA1KZuE), read tab 'Caliber targets' range A4:P60 with the Google Sheets get_values tool. Assumptions: B5 = yen per dollar, B10 = US duty on Japan buys, B12 = parts per watch ($), B13 = eBay fee share, B14 = shipping out ($), B15 = minimum profit ($), B16 = Japan fixed costs per watch ($), F5 = US sales tax, F6 = default inbound shipping ($) for eBay listings that don't show it. From row 19 down, each row is Caliber (A), Version (B), eBay median sale in USD (E), Japan max bid in yen (K), Japan verdict (L), US max price incl. shipping in USD (O), US verdict (P).
- Load the chrome-browser skill, then the Chrome tools in one ToolSearch call (include javascript_tool and navigate). Call tabs_context_mcp and open one new tab; work only in that tab for both parts. Chrome tool output is cut off at about 1,000 characters and URLs with query strings are hidden, so always return compact text, a few lines per call.
- Load ArtifactData and PushNotification with ToolSearch ("select:ArtifactData,PushNotification"). The page is https://claude.ai/artifact/3tZ2eidxkhKx8cyMK2NfjW.
- Write the script below to /tmp/add_thumbs.py exactly as shown (if PIL is missing, run: pip install --break-system-packages pillow). Usage: python3 /tmp/add_thumbs.py <in.json> <out.json> <previous.json>. It fills each item's "thumb" from its "img_url", removes "img_url", and sets "is_new" to true for every listing that was not in the previous digest (compared by src_url), so the page can highlight new listings. Failed photos stay null. Run it at most once per part.

import json, sys, io, os, base64, urllib.request
from PIL import Image
UA = "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 Chrome/130 Safari/537.36"
def thumb(url):
    for _ in range(2):
        try:
            req = urllib.request.Request(url, headers={"User-Agent": UA, "Referer": "https://auctions.yahoo.co.jp/"})
            raw = urllib.request.urlopen(req, timeout=20).read()
            im = Image.open(io.BytesIO(raw)).convert("RGB"); im.thumbnail((144, 144))
            buf = io.BytesIO(); im.save(buf, "JPEG", quality=70, optimize=True)
            return "data:image/jpeg;base64," + base64.b64encode(buf.getvalue()).decode()
        except Exception as e:
            err = e
    print("failed:", url[:80], err, file=sys.stderr)
def load(p):
    d = json.load(open(p))
    return d if "look" in d else d.get("data", d)
d = load(sys.argv[1])
prev = set()
if len(sys.argv) > 3 and os.path.exists(sys.argv[3]):
    p = load(sys.argv[3])
    prev = {i.get("src_url") for s in ("look", "watch", "lots") for i in p.get(s, []) if i.get("src_url")}
ok = new = 0
for s in ("look", "watch", "lots"):
    for it in d.get(s, []):
        u = it.pop("img_url", None); it["thumb"] = thumb(u) if u else None; ok += bool(it["thumb"])
        it["is_new"] = bool(prev) and it.get("src_url") not in prev; new += it["is_new"]
json.dump(d, open(sys.argv[2], "w"), ensure_ascii=False)
print(f"thumbs {ok}, new {new}, size {len(json.dumps(d))} bytes")

PART A: JAPAN (document "latest")
A1. Targets: rows whose Japan verdict (L) contains "Good target" or "Usually sells above max".
A2. Navigate the tab to https://auctions.yahoo.co.jp/.
   - Yahoo Auctions: for each target caliber, fetch() inside the tab '/search/search?p=' + encodeURIComponent('seiko <CALIBER> ジャンク') + '&n=100', and the same with 不動 instead of ジャンク. Parse with DOMParser, JSON.parse the script with id "__NEXT_DATA__", and walk it for objects with both auctionId and imageUrl. Each gives auctionId, title, price, bidCount, buyNowPrice (0 or missing when none), isFixedPrice, endTime (Japan time, +09:00) and imageUrl. Keep the photo as a short path: the part of imageUrl's pathname after "/images.auctions.yahoo.co.jp/image/", with no host and no query string.
   - Mercari Japan items on sale: https://jp.mercari.com/search?keyword=seiko%20<CALIBER>%20ジャンク&status=on_sale (navigate there if fetch() does not return listings). Read item ID (starts with "m"), title and price.
   For diver/Scuba rows keep only titles with diver words (ダイバー, スキューバ, SCUBA, DIVER, 200m, プロスペックス); for Dress/sport rows exclude them.
A3. Filter out water damage (水入り), aftermarket parts (社外), dial-only or face-only parts (文字盤のみ, フェイスのみ), movement-only, cases-only, non-kinetic quartz, ended auctions, and titles whose caliber doesn't match. Multi-watch lots (まとめ) go in Lots, with the calibers you can read.
A4. Buyee plan: "Standard" (¥500: inspection plus insured delivery) at ¥8,000 or more and for every lot; "Lite" (free) under ¥8,000; "Lite now, Standard over ¥8,000" for cheap auctions likely to finish above ¥8,000.
A5. Landed cost in USD = price_yen / B5 × (1 + B10) + B16, plus 500 / B5 when the plan is Standard. Expected profit = E × (1 − B13) − B14 − B12 − landed cost. Limit = K, minus ¥500 when the plan is Standard. Fixed-price listings are Mercari items and Yahoo listings with isFixedPrice true. A Yahoo auction whose buy-it-now price is at or below the limit is worth a look, priced at its buy-it-now price.
   - Worth a look: at or below the limit; best profit first; at most 24.
   - Watching: Yahoo auctions above the limit but at most 1.3 × K, ending within 3 days; soonest first; at most 3.
   - Lots: cheapest per watch first; at most 3.
   Remember each section's total before trimming.
A5a. Live check. Search results can include listings whose page no longer exists, so check every listing before it is shown. Take the trimmed Worth a look, Watching and Lots lists plus up to 10 runners-up per section, and check them in batches of at most 15 IDs, keeping output to one line per batch:
   - Yahoo: navigate the tab to https://page.auctions.yahoo.co.jp/ and run, with IDS = the batch of Yahoo auction IDs:
     (async(IDS)=>{const bad=[],err=[];for(const id of IDS){try{const r=await fetch('/jp/auction/'+id,{credentials:'include'});const t=await r.text();if(r.status!==200||!t.includes(id)||/ページが見つかりません|このオークションは終了しています|削除されました|出品が取り消されました/.test(t))bad.push(id)}catch(e){err.push(id)}}return 'checked '+IDS.length+' gone:'+(bad.join(',')||'none')+' err:'+(err.join(',')||'none')})(IDS)
   - Buyee (both Yahoo and Mercari listings, because the Buyee link is the one I open): navigate the tab to https://buyee.jp/ and run, with P = the batch of Buyee paths ("/item/yahoo/auction/<auctionID>" or "/mercari/item/<itemID>"):
     (async(P)=>{const bad=[],err=[];for(const p of P){const id=p.split('/').pop();try{const r=await fetch(p,{credentials:'include'});const t=await r.text();if(r.status!==200||!r.url.includes(id)||/Page Not Found|ページが見つかりません|お探しのページ|This auction has (already )?ended|has been deleted/i.test(t))bad.push(id)}catch(e){err.push(id)}}return 'checked '+P.length+' gone:'+(bad.join(',')||'none')+' err:'+(err.join(',')||'none')})(P)
   Drop every "gone" listing and move the next runner-up into its place, keeping the order rules. Listings under "err" could not be checked: keep them. If a batch of 3 or more comes back with every listing gone, the check itself is probably broken (for example a changed error page): run that batch once more, and if it still says all gone, keep the batch and continue. Count dropped listings in "skipped" as "no longer available" and subtract them from the section totals.
A6. Write /tmp/jp_in.json in the document shape below, giving each item "img_url": for Yahoo "https://auc-pctr.c.yimg.jp/i/auctions.c.yimg.jp/images.auctions.yahoo.co.jp/image/<short path>?pri=l&w=300&h=300&up=0", for Mercari "https://static.mercdn.net/thumb/item/webp/<itemID>_1.jpg".
A7. "get" collection "digest", doc_id "latest" with out_dir "/tmp/prev_jp" (it saves the previous digest to /tmp/prev_jp/digest/latest.json) and note its version. Then run: python3 /tmp/add_thumbs.py /tmp/jp_in.json /tmp/jp_out.json /tmp/prev_jp/digest/latest.json
A8. "set" doc_id "latest" with file_path "/tmp/jp_out.json" and if_version set to the version from A7. Document:
   {"date_label": "Mon Nov 2", "run_label": "morning" or "evening" (or "test run" if asked), "updated_label": "6:51am PT", "rate": B5, "status": "ok", "message": "",
    "counts": {"look_total": n, "watch_total": n, "lot_total": n}, "look": [items], "watch": [items], "lots": [items],
    "not_shown": "Not shown: 4 more worth a look, 2 watching, 1 lot" (or ""), "skipped": "38 skipped: 19 priced above your max, 8 water damage, 6 parts only, 5 not kinetic"}
   Item: {"desc": short English description such as "5M62-0BL0 Scuba, black dial, not running" (never the raw Japanese title, stock codes, IDs or "1円スタート"), "site": "Yahoo" or "Mercari", "fixed": true for Mercari and Yahoo fixed-price, "price_yen", "buy_now_yen": Yahoo buy-it-now for auctions that have one else null, "bids", "ends": Los Angeles time like "Tue 6:05am" (null for Mercari), "urgent": ends within 12 hours, "max_yen": the limit, "plan", "landed_usd", "profit_usd" (Worth a look only), "over_yen" (Watching only), "calibers" (Lots only, else null), "buyee_url": https://buyee.jp/item/yahoo/auction/<auctionID> or https://buyee.jp/mercari/item/<itemID>, "src_url": https://auctions.yahoo.co.jp/jp/auction/<auctionID> or https://jp.mercari.com/item/<itemID> (always in exactly this form, so new and passed listings are matched correctly), "src_label": "Yahoo" or "Mercari", "img_url"}. The script adds "thumb" and "is_new".
   If nothing qualifies, publish empty arrays with "message": "Nothing under your max this run."

PART B: EBAY US (document "us"; never write "latest" in this part)
Only listings located in the United States are allowed. eBay's own location filter is not reliable (it lets UK listings through), so the script below checks each listing's "Located in United States" line and drops everything else. Never add a listing that the script dropped.
B1. Targets: rows whose US verdict (P) does not contain "Skip on eBay".
B2. Navigate the tab to https://www.ebay.com/. Build one phrase from the target calibers in lower case, joined by commas:
   seiko (kinetic,ags) (<calibers>,skj,"0a40","6b50","0b69") -(strap,band,link,links,manual,box,ladies,women) (parts,repair,broken,"not running","as is","needs capacitor","needs battery")
   Run this in the tab with Q = that phrase, once with EXTRA = '&LH_Auction=1&_sop=1' and once with EXTRA = '&LH_BIN=1&_sop=15'. It stores compact lines for US-located listings in window.__eb and reports how many non-US listings it dropped:
   (async(Q,EXTRA)=>{const h=await fetch('/sch/i.html?_nkw='+encodeURIComponent(Q)+'&LH_PrefLoc=1&_ipg=240'+EXTRA,{credentials:'include'}).then(r=>r.text());const d=new DOMParser().parseFromString(h,'text/html');window.__eb=window.__eb||[];window.__ebx=window.__ebx||new Set();const seen=new Set(window.__eb.map(l=>l.split('|')[0]));let n=0;for(const c of d.querySelectorAll('li.s-item, li.s-card')){const a=c.querySelector('a[href*="/itm/"]');if(!a)continue;const id=(a.getAttribute('href').match(/itm\/(\d+)/)||[])[1];if(!id||seen.has(id))continue;const L=[...c.querySelectorAll('span,div')].filter(e=>!e.children.length).map(e=>e.textContent.trim()).filter(Boolean);const title=((c.querySelector('.s-item__title, .s-card__title')||{}).textContent||'').replace(/Opens in a new window or tab|New Listing/g,'').trim();if(!title||title==='Shop on eBay')continue;if(!L.includes('Located in United States')){window.__ebx.add(id);continue}seen.add(id);const price=(L.find(t=>/^\$[\d,.]+$/.test(t))||'').replace(/[$,]/g,'');const sh=L.find(t=>/delivery|shipping/i.test(t))||'';const ship=/free/i.test(sh)?'0':((sh.match(/\$([\d.,]+)/)||[])[1]||'');const bids=(L.find(t=>/^\d+ bids?$/.test(t))||'').split(' ')[0];const end=(L.find(t=>/^\(.+\)$/.test(t))||'').replace(/[()]/g,'');const kind=EXTRA.includes('LH_Auction')?'AUC':'BIN';const img=c.querySelector('img');const im=((img&&(img.getAttribute('src')||img.getAttribute('data-src')))||'').match(/images\/g\/([^/]+)\//);window.__eb.push([id,price,ship,bids,end,kind,im?im[1]:'',title.slice(0,80)].join('|'));n++}return n+' kept from the US, '+window.__ebx.size+' skipped as not from the US so far'})(Q,EXTRA)
   Read window.__eb six lines at a time. Each line: itemID|price|shipping (blank = not shown, 0 = free)|bids|auction end in Los Angeles time|AUC or BIN|photo id|title.
B3. Judge only by the title. eBay labels every for-parts listing "Parts Only"; that is the condition I want, so never skip a listing for it. Read the caliber from the title; a 5M62, 5M43 or 5M23 with diver words (diver, scuba, 200m, prospex) is the diver row, otherwise dress/sport. Skip titles with no readable caliber (count them), calibers not in the targets, ladies' watches, and titles selling only a part (dial only, movement only, case only, crown, bezel insert, hands, caseback); a whole watch or watch head for parts is a watch. Titles with "lot", "lot of" or "bundle" go in Lots.
B4. Total = price + shipping (F6 when blank). Landed = total × (1 + F5). Profit = E × (1 − B13) − B14 − B12 − landed. Limit = O.
   - Worth a look: total at or below the limit; best profit first; at most 24.
   - Watching: auctions above the limit but at most 1.3 × the limit, ending within 3 days; soonest first; at most 3.
   - Lots: cheapest per watch first; at most 3.
B5. Write /tmp/us_in.json in the document shape below, giving each item "img_url": "https://i.ebayimg.com/images/g/<photo id>/s-l225.jpg" (or null).
B6. "get" collection "digest", doc_id "us" with out_dir "/tmp/prev_us" (it saves the previous digest to /tmp/prev_us/digest/us.json) and note its version. Then run: python3 /tmp/add_thumbs.py /tmp/us_in.json /tmp/us_out.json /tmp/prev_us/digest/us.json
B7. "set" doc_id "us" with file_path "/tmp/us_out.json" and if_version set to the version from B6. Document:
   {"date_label", "run_label", "updated_label" as in Part A, "tax": F5, "status": "ok", "message": "", "counts": {...}, "look": [...], "watch": [...], "lots": [...], "not_shown": "...", "skipped": "38 skipped: 18 above your max, 7 not shipping from the US, 6 no caliber in title, 4 parts only, 3 ladies"}
   Item: {"desc": short English description such as "5M43-0B69 Sports 100 titanium, not running" (no IDs, seller names or filler), "site": "eBay", "fixed": true for Buy It Now, "price_usd", "ship_usd" (number or null), "buy_now_usd": null, "bids", "ends": auction end like "Mon 3:30pm" (null for Buy It Now), "urgent": ends within 12 hours, "max_usd": the limit, "landed_usd", "profit_usd" (Worth a look only), "over_usd" (Watching only), "calibers" (Lots only, else null), "src_url": "https://www.ebay.com/itm/<itemID>" (always in exactly this form), "src_label": "eBay", "img_url"}. The script adds "thumb" and "is_new".
   If nothing qualifies, publish empty arrays with "message": "Nothing under your max this run."

PART C: FINISH
C1. Close the tab you opened.
C2. Send ONE push notification with the PushNotification tool, as the very last action of the run, for example:
   "Japan: 3 new. Top: 5M62 Scuba ¥12,800 (max ¥17,000, Standard). eBay: 1 new. Top: 5M23 Spirit $61 incl. shipping (max $78). Open the digest: https://claude.ai/artifact/3tZ2eidxkhKx8cyMK2NfjW"
   Use the number of new listings from each script run and the top pick from each market, or "nothing under your max" for that market. Then end with the same text as your final message. Do not send any other messages during the run.

FAILURES
- If Chrome or the Claude in Chrome extension cannot be reached, do not use any other browser or method. For each document that exists ("latest" and "us"), "update" it (with if_version) to "status": "error", "message": "This run was skipped because your computer or Chrome wasn't available.", and a new updated_label. Then send one push notification saying so, and stop.
- If one part fails partway (for example Chrome stops responding during Part B), update only that part's document with "status": "error" and a short plain message, keep the other part's results, and say in the push notification which market failed.
- If an ArtifactData write fails, put that market's top picks in the push notification instead (short English descriptions, no IDs) and say the page could not be updated.
