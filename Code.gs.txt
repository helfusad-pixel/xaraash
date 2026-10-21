/**
 * Xaraash shared storage on Google Sheets + Drive (free).
 * Posts go to the "posts" sheet of the spreadsheet this script is attached to.
 * Photos go to the Drive folder "Xaraash photos" (shared as "anyone with the link can view").
 *
 * Setup: see README, section "Keydin Google Drive".
 */
var SHEET_NAME = 'posts';
var FOLDER_NAME = 'Xaraash photos';
var HEADERS = ['id', 'createdAt', 'cat', 'city', 'priceUsd', 'title', 'user', 'photos', 'status'];
var CATS = ['cars', 'devices', 'real', 'furniture', 'services', 'fashion', 'animals', 'jobs', 'games', 'coaching', 'food', 'gardens', 'tourism', 'trips', 'events', 'arts', 'coding', 'lost', 'projects'];
var MAX_PHOTOS = 5;
var MAX_PHOTO_CHARS = 700000;   // ~500 KB per photo after base64
var MAX_POSTS_PER_MINUTE = 20;  // whole site, protects your Drive quota
var LIST_LIMIT = 200;

// Run this once from the editor (Run > setup) to approve access to Sheets and Drive.
function setup() { sheet_(); folder_(); }

function sheet_() {
  var ss = SpreadsheetApp.getActiveSpreadsheet();
  var sh = ss.getSheetByName(SHEET_NAME);
  if (!sh) { sh = ss.insertSheet(SHEET_NAME); sh.appendRow(HEADERS); sh.setFrozenRows(1); }
  return sh;
}
function folder_() {
  var it = DriveApp.getFoldersByName(FOLDER_NAME);
  return it.hasNext() ? it.next() : DriveApp.createFolder(FOLDER_NAME);
}
function json_(o) { return ContentService.createTextOutput(JSON.stringify(o)).setMimeType(ContentService.MimeType.JSON); }

// Validates and trims an incoming post. Returns null when it should be rejected.
function clean_(b) {
  if (!b || typeof b !== 'object') return null;
  var title = String(b.title || '').replace(/[\u0000-\u001f]/g, ' ').trim().slice(0, 80);
  if (!title) return null;
  var cat = CATS.indexOf(b.cat) >= 0 ? b.cat : 'services';
  var city = String(b.city || '').replace(/[\u0000-\u001f<>]/g, '').trim().slice(0, 40);
  var price = b.priceUsd === null || b.priceUsd === '' || b.priceUsd === undefined ? '' : Number(b.priceUsd);
  if (price !== '' && (!isFinite(price) || price < 0 || price > 1e9)) return null;
  var photos = Array.isArray(b.photos) ? b.photos.slice(0, MAX_PHOTOS) : [];
  for (var i = 0; i < photos.length; i++) {
    if (typeof photos[i] !== 'string' || photos[i].indexOf('data:image/jpeg;base64,') !== 0 || photos[i].length > MAX_PHOTO_CHARS) return null;
  }
  return { title: title, cat: cat, city: city, price: price, photos: photos };
}

function doGet(e) {
  try {
    var sh = sheet_();
    var last = sh.getLastRow();
    if (last < 2) return json_({ ok: true, posts: [] });
    var from = Math.max(2, last - LIST_LIMIT + 1);
    var rows = sh.getRange(from, 1, last - from + 1, HEADERS.length).getValues();
    var posts = [];
    for (var i = rows.length - 1; i >= 0; i--) {
      var r = rows[i];
      if (r[8] === 'hidden') continue;
      var ids = [];
      try { ids = JSON.parse(r[7] || '[]'); } catch (x) { ids = []; }
      posts.push({ id: String(r[0]), ts: new Date(r[1]).getTime(), cat: r[2], city: r[3], priceUsd: r[4] === '' ? null : Number(r[4]), title: r[5], user: r[6], photos: ids });
    }
    return json_({ ok: true, posts: posts });
  } catch (err) {
    return json_({ ok: false, error: 'list_failed' });
  }
}

function doPost(e) {
  var lock = LockService.getScriptLock();
  try {
    lock.waitLock(10000);
    var body = JSON.parse((e && e.postData && e.postData.contents) || '{}');
    if (body.action !== 'add') return json_({ ok: false, error: 'bad_action' });

    var cache = CacheService.getScriptCache();
    var key = 'rl' + Math.floor(Date.now() / 60000);
    var n = Number(cache.get(key) || 0);
    if (n >= MAX_POSTS_PER_MINUTE) return json_({ ok: false, error: 'rate_limited' });
    cache.put(key, String(n + 1), 120);

    var p = clean_(body);
    if (!p) return json_({ ok: false, error: 'invalid' });

    var id = Utilities.getUuid().slice(0, 8);
    var ids = [];
    if (p.photos.length) {
      var f = folder_();
      for (var i = 0; i < p.photos.length; i++) {
        var bytes = Utilities.base64Decode(p.photos[i].split(',')[1]);
        var file = f.createFile(Utilities.newBlob(bytes, 'image/jpeg', id + '-' + i + '.jpg'));
        file.setSharing(DriveApp.Access.ANYONE_WITH_LINK, DriveApp.Permission.VIEW);
        ids.push(file.getId());
      }
    }
    sheet_().appendRow([id, new Date(), p.cat, p.city, p.price, p.title, 'Member ' + id.slice(0, 4), JSON.stringify(ids), 'visible']);
    return json_({ ok: true, id: id });
  } catch (err) {
    return json_({ ok: false, error: 'server_error' });
  } finally {
    try { lock.releaseLock(); } catch (x) {}
  }
}
