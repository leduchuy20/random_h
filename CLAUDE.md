# CLAUDE.md — Lớp Học Vui Vẻ (trò chơi ngẫu nhiên cho lớp học)

## Tổng quan
Web app một file (`index.html`, trước 02/11/2026 tên là `tro-choi-lop-hoc.html`) để giáo viên chiếu lên máy chiếu và gọi ngẫu nhiên học sinh hoặc số thứ tự qua **12 trò chơi + 10 trò mini**. App còn có nhiều lớp, điểm danh, bảng điểm, đồng hồ đếm ngược và xuất/nhập dữ liệu. Người dùng mở file bằng cách bấm đúp (`file://`). Không có build step, server hay framework.

**Ngôn ngữ:** UI hoàn toàn bằng tiếng Việt có dấu. Code, tên biến và comment kỹ thuật có thể dùng tiếng Anh.

**Lưới chọn trò** (`nav#tabs`, thay cho thanh tab từ 31/10): 24 thẻ `button[data-tab]` có `<i>`icon, `<b>`tên, `<small>`mô tả, xếp `grid auto-fill minmax(150px,1fr)` (8 cột ở 1366px, 2 cột trên điện thoại). Thứ tự: **Danh sách lớp (thẻ đầu tiên)** → 12 trò chính → 10 thẻ trò mini (`data-tab="mini" data-mini="k"`, nền vàng `.tmini`) → Bảng điểm (nền xanh `.tcls`, thẻ Danh sách lớp cũng nền xanh). **Mở app luôn vào tab `roster`** (không nhớ tab cũ) và cuộn về đầu trang (`history.scrollRestoration='manual'`). Lưới luôn hiện phía trên; bấm thẻ thì trang tự cuộn xuống nếu sân khấu bị che. Lưới chọn trò cũ bên trong tab Trò mini (`#miniPick`) bị ẩn.
| Tab (`section#tab-…`) | Phím số | Mô tả |
|---|---|---|
| `roster` 📋 Danh sách lớp | 1 | Thiết lập: danh sách, lớp, điểm danh, thời gian chạy, thử thách/phần thưởng |
| `wheel` 🎡 Vòng quay | 2 | Trò chơi |
| `slot` 🎰 Máy jackpot | 3 | Trò chơi |
| `duck` 🦆 Đua vịt | 4 | Trò chơi |
| `marble` 🎱 Đua bi | 5 | Trò chơi |
| `ufo` 🛸 UFO | 6 | Trò chơi |
| `farm` 🐔 Bắt gà (3D) | 7 | Trò chơi |
| `box` 🎁 Hộp bí mật | 8 | Trò chơi |
| `group` 👥 Chia nhóm | 9 | Trò chơi |
| `lotto` 🔢 Quay số | 0 | Trò chơi |
| `guess` 🤔 Đoán số | — | Trò chơi |
| `bomb` 💣 Bom số | — | Trò chơi |
| `dice` 🎲 Xúc xắc | — | Trò chơi |
| `mini` 🎮 Trò mini | — | 10 thẻ riêng trong lưới (xem phần Trò mini) |
| `score` 🏆 Bảng điểm | — | Bảng điểm |

## Files
- `index.html`: toàn bộ app (HTML + `<style>` + `<script>` inline), khoảng 1 MB, trong đó ~57 KB là font nhúng và ~608 KB là **three.js r149** (MIT) nhúng trong một `<script>` riêng ngay trước script chính (biến toàn cục `THREE`). **Giữ nguyên dạng một file.**
  - **Cập nhật three.js:** `npm pack three@0.149.0`, lấy `build/three.min.js` (bản UMD cuối cùng trước khi bỏ `build/three.min.js` ở r150+; phải kiểm tra file không chứa chuỗi `</script`). Code dùng `outputEncoding` mặc định (linear) và `ColorManagement.legacyMode` mặc định nên mã màu hex hiện đúng như viết.
- `tro-choi-lop-hoc.html`: chỉ còn là trang chuyển hướng nhỏ sang `index.html` (giữ cho đường dẫn/bookmark cũ). **Mọi sửa đổi làm trên `index.html`.**
- `CLAUDE.md`: file này.
- **Git/GitHub:** người dùng tự commit & push (repo + GitHub Pages, `index.html` ở gốc repo để link Pages gọn `https://<user>.github.io/<repo>/`). Claude chỉ sửa file, không push.

Thứ tự trong `<script>`: helpers (`rint`, `rand01`, `shuffle`, `store`, `Dur`, `copyText`, `toast`) → `Sound` → `Confetti` → `Modal` → state, classes & roster → mở rộng đường đua (`Expand`) → tabs & phím tắt → các game theo thứ tự tab → `Timer` → `Score` → header/roster init → boot.

## Ràng buộc bắt buộc
- **Chạy 100% offline từ `file://`, không gửi bất kỳ request mạng nào.** Font "Baloo 2" (SIL OFL) được nhúng base64 ở đầu `<style>`, gồm 2 subset `vietnamese` + `latin` với `unicode-range`. Không thêm CDN, Google Fonts hay fetch.
  - **Cập nhật font:** `npm pack @fontsource-variable/baloo-2`, lấy `files/baloo-2-{vietnamese,latin}-wght-normal.woff2` rồi base64.
- **Không có file âm thanh hay ảnh.** Âm thanh được tổng hợp bằng WebAudio (`Sound`). Hình ảnh dùng emoji, CSS, canvas hoặc inline SVG.
- **Dễ đọc trên máy chiếu:**
  - chữ to, tương phản cao, nút lớn, font dùng `clamp()`;
  - ở 1366×768 header nằm trên 1 dòng;
  - ở 1024×768 vòng quay và đường đua vừa màn hình, không cần cuộn;
  - layout vẫn dùng được trên điện thoại (~390px).
- **Mọi truy cập `localStorage` đi qua `store.get/set`** (đã bọc try/catch, prefix key `lhvv_`).
- **Random dùng `rint(n)` / `rand01()` / `shuffle(arr)`** (dựa trên `crypto.getRandomValues`).
  - `rint` dùng rejection sampling. **Đừng viết `x % n` trực tiếp:** khi n lớn kết quả lệch về số nhỏ. Lỗi này từng làm ô vòng quay lệch 16–25%.
  - `Math.random()` chỉ dùng cho hiệu ứng, hoặc cho mô phỏng mà mọi người chơi có cùng phân phối (tốc độ vịt, cú đẩy bi).
- **HTML động phải `esc()`** mọi chuỗi do người dùng nhập.
- **Dữ liệu giáo viên đã lưu phải luôn đọc được sau khi nâng cấp.**
  - `Classes.load()` tự chuyển dữ liệu bản một lớp cũ (key `roster`, `className`, `picked`, `history`, `lottoDrawn`) và tự bù field còn thiếu bằng `Classes.blank()`.
  - Thêm field mới cho lớp thì thêm vào `blank()`.
- **Công bằng là yêu cầu chính** (giáo viên dùng để gọi học sinh). Mọi cách chọn mới phải kiểm bằng thống kê (xem phần Kiểm thử).

## Mô hình dữ liệu
- `Classes.list`: mảng các lớp `{id, name, roster(text), picked:[id], history:[label], absent:{date, ids:[key]}, scores:{key:n}, calls:{key:n}, drawn:[số], weights:{key:hệ số ô vòng quay}, shuffle:{a,b,order,done}|null, wheelOrder:[key]|null, ufoOrder:[key]|null}`. `Classes.curId` là lớp đang mở.
- `State`: trạng thái lớp hiện tại trong bộ nhớ (`picked`, `history`, `absent` dạng `Set`/mảng), cùng các cài đặt chung (`noRepeat`, `display`, `wheelShort`, `challenges`, `rewards`, `tab`).
  - `loadClassState()` đọc trạng thái lớp vào `State`; `saveState()` ghi ngược lại.
  - **Mọi thay đổi `State.picked/history/absent` phải gọi `saveState()`.**
- `roster`: `[{id, name, no, key, short, gender}]`, được tạo lại bởi `applyRoster(text)` từ `parseNames()`:
  - `parseNames()` bỏ dòng tiêu đề Excel (`HEADER_RE`), ô chỉ có số, ô ngày sinh; lấy số thứ tự từ ô số hoặc tiền tố `12.`. Chỉ dùng số ghi sẵn khi **mọi dòng đều có và không trùng**, ngược lại thì đánh số theo dòng.
  - **Giới tính** (chỉ dùng để vẽ nhân vật UFO): ô cuối là `nữ`/`female` (phần tên còn lại ≥2 chữ) hoặc `nam`/`male` (phần tên còn lại **≥3 chữ**, để không nhầm cột Tên = "Nam" khi Excel tách Họ đệm | Tên) → `gender` = `'f'`/`'m'`, rồi bỏ ô đó khỏi tên. Ghi theo dạng `Trần Thị Bình, nữ` hoặc cột Giới tính từ Excel.
  - `id` = index dòng, chỉ dùng cho `picked`, reset khi danh sách đổi.
  - `no` = số thứ tự.
  - `key` = khóa ổn định theo tên viết thường (`'#'+no` nếu không có tên; trùng tên thì thêm `#no`), dùng cho **vắng mặt, điểm, số lần gọi, độ to ô**.
  - `short` = tên gọi ngắn: tên → đệm + tên nếu trùng → họ tên đầy đủ.
- `absent` chỉ có hiệu lực trong ngày: `absent.date !== todayStr()` thì coi như rỗng.
- `rosterVer`: tăng khi tập người chơi đổi (sửa danh sách, đổi lớp, Làm mới, đổi `noRepeat`, đổi chế độ hiển thị). Game lưu `this.ver` và build lại trong `onShow()` khi `this.ver !== rosterVer`.
  - Vắng mặt và lượt gọi **không** tăng `rosterVer`. Thay vào đó Box dùng `sync()`, Duck/Marble dùng `stale()`, còn Wheel được `updatePicked()` gọi `refresh()`.

### Các key trong localStorage (đều có prefix `lhvv_`)
- **Dữ liệu:** `classes`, `curClass`.
- **Cài đặt chung:** `noRepeat`, `display`, `wheelShort`, `wheelAutoMix`, `ufoAutoMix`, `sound`, `tab`, `challenges`, `rewards`, `durations`, `timerSec`.
- **Tùy chọn từng trò:** `genCount`, `slotNameOnly`, `duckCount`, `duckSkills`, `duckView`, `marbleCount`, `ufoCount`, `ufoLevel`, `ufoSpeak`, `farmCount`, `farmLevel`, `farmSpeak`, `farmCam`, `miniGame`, `miniTwist`, `miniMix`, `miniSpeak`, `boxSurprise`, `groupVal`, `groupMode`, `lottoMin`, `lottoMax`, `lottoMaxTouched`, `lottoCount`, `lottoMode`, `diceN`, `scoreSort`, `wheelSizesOpen`.
- **Key cũ** (chỉ đọc khi chuyển dữ liệu): `roster`, `className`, `picked`, `history`, `lottoDrawn`.

## API dùng chung
- **Hiển thị:**
  - `label(s, short?)`: text thuần;
  - `bigLabel(s)`: HTML có badge số, dùng cho modal;
  - `wheelLabel(s)`: dùng trên vòng quay.
  - Cả ba tôn trọng `State.display` (`'both'` | `'num'` | `'name'`) và tự fallback "Số N" khi không có tên. **Không hiển thị `s.name` trực tiếp.**
- **Chọn người:**
  - `present()`: những bạn không vắng;
  - `pool()`: có mặt và chưa được gọi (nếu `noRepeat`);
  - `ensurePool()`: trả `null` và tự báo khi rỗng;
  - `pick(s)`: đánh dấu đã gọi, ghi lịch sử, tăng `calls`;
  - `unpick(s)`: hoàn tác `pick`;
  - `setAbsent(s, on)`.
- **`personModal({emoji, title, s, sub, again, retry, retryLabel, onClose})`**: khung kết quả chuẩn khi gọi trúng một học sinh.
  - Tự thêm hàng cộng/trừ điểm và các nút [again] · ⏱ Bấm giờ · 🚫 Vắng – chọn lại · Đóng.
  - Nút "Vắng" chạy `unpick` + `setAbsent` rồi gọi `retry()`.
  - **Mọi game gọi người phải dùng hàm này thay cho `Modal.show`.**
- `addScore(key, d)`: cập nhật mọi `[data-k]` đang hiện. Các nút `button[data-d]` trong `#mcard` và `#scoreGrid` đã được delegate sẵn.
- `Modal.show({emoji, title, big, bigSmall, sub, confetti, sound, actions, onClose})`: `big`/`sub` là HTML, phải tự `esc()` trước. Ngoài ra còn `Confetti.burst(n)`, `toast(msg)`, `copyText(txt, msg)`.
- **`Sound`:** `tick/pop/thud/beep/up/down/buzz/win/boom` và `Sound.tone(freq, dur, type, vol, when)`.
- **`Dur.get(k)` / `Dur.ms(k)`:** thời gian chạy (giây / ms) mà người dùng chỉnh được. Mặc định, min, max và step nằm trong `DUR_DEF`; giá trị lưu ở key `durations`.
  - Mọi `input[data-dur="k"]` (ô trong từng trò lẫn danh sách chung ở tab Danh sách lớp) tự đồng bộ qua `Dur.sync()`.
  - **Animation mới phải lấy thời lượng từ `Dur`, không hard-code.**

  | k | mặc định (min–max) | quy đổi |
  |---|---|---|
  | `wheel` | 6 (1–30) | quay `T` ±8%, số vòng ≈ 1.2×giây |
  | `slot` | 4 (1–20) | 3 reel dừng ở 65% / 82% / 100% của `T` |
  | `duck` | 9 (3–60) | `RACE = T` (chưa tính 3-2-1) |
  | `marble` | 15 (6–90) | độ dài đường đua (xem phần Đua bi) |
  | `group` | 3 (0–20) | `T/n` mỗi người (0 = hiện ngay) |
  | `lotto` | 3 (0.5–20) | chữ số đầu dừng ở 35%, các chữ số sau chia đều phần còn lại; quay nhiều số thì mỗi số chạy 40% |
  | `dice` | 1.5 (0.3–10) | `T` + 0.15s mỗi viên |
  | `ufo` | 2.5 (0.8–8) | thời gian một vòng quét; bạn thứ 2 trở đi quét bằng 75% |
  | `farm` | 2.5 (1–8) | thời gian một lần rượt (lần rượt trúng = 1.1×) |
  | `mini` | 12 (5–40) | tổng thời gian một lượt trò mini |
- **`Timer.open(name, autoStart)`:** đồng hồ đếm ngược (overlay `#timer`), có các mốc 10s–3 phút và ô nhập số giây; còn 5 giây thì kêu bíp, hết giờ thì chuông.
- **Quản lý lớp:**
  - `switchClass(id)`, `newClass()` (dùng `prompt`), `deleteClass()` (dùng `confirm`, luôn còn ít nhất 1 lớp);
  - `exportData()` tải file `lop-hoc-vui-ve-YYYY-MM-DD.json` gồm mọi lớp + thử thách + phần thưởng;
  - `importData(text)` gộp theo tên lớp (trùng tên thì ghi đè).
- **`Expand.toggle('#raceBox' | '#marbleBox' | '#ufoBox' | '#farmBox' | '#miniBox', on?)`:** chế độ mở rộng đường đua (xem bên dưới).
- Bảng màu `COLORS` (8 màu); `LIGHT_BG` là các màu nền cần chữ tối. Class CSS dùng lại: `.panel`, `.btn` (`.big`, `.sm`, `.red`, `.blue`, `.green`, `.ghost`), `.chip-btn`, `.row`, `.field`, `.check`, `.hint`, `.stage` (khung chính + cột phải 320px), `.side`, `.empty`, `.nobadge`, `.countdown`.

## Tabs, phím tắt & khóa khi đang chạy
- Mỗi tab gồm `<section class="tab" id="tab-XXX">` và một nút `<button data-tab="XXX">` trong `nav#tabs`. Đăng ký `TABS.XXX = { onShow?, primary? }`.
- `ACTION_KEYS` = Space, Enter, PageDown, PageUp, →, ← (bút trình chiếu gửi các phím này):
  - Timer đang mở: phím hành động bật/dừng đồng hồ, Esc đóng.
  - Modal đang mở: phím hành động hoặc Esc đóng modal.
  - Đang mở rộng đường đua: Esc thu nhỏ.
  - Còn lại: phím hành động chạy `primary` của tab hiện tại. Bỏ qua khi focus input/textarea/select, và bỏ qua `e.repeat`.
- **Phím số:** `1`–`9`, `0` chuyển theo `TAB_ORDER` = 10 thẻ đầu tiên trong lưới **không tính thẻ trò mini** (`#tabs button:not([data-mini])`). Chèn thẻ mới sẽ làm các phím phía sau dịch chỗ.
- **⏹ Dừng lượt này** (`#stopBtn`, nổi góc phải dưới, z-index 46 để hiện cả khi Mở rộng; phím **Esc** khi có trò đang chạy): `stopGame()`.
  - `Run.ep` tăng lên → `sleep()` và `tween()` của lượt cũ **không bao giờ resolve** (các hàm async treo lại, không gọi `pick`/modal), các vòng `requestAnimationFrame` và `setTimeout` của Wheel, Slot, Duck (cả 2 kiểu), Group, Lotto.animate, Dice kiểm tra `ep!==Run.ep` rồi thoát; Marble dừng nhờ `racing=false`, Mini nhờ `run=null`, Farm nhờ `chaseCtl/walkTo=null`.
  - `Run.snap` (chụp trong vòng 150ms đầu lượt: `picked`, `history`, `calls`) được trả lại → bạn bị gọi trong lượt dở dang coi như chưa bị gọi.
  - `STOP[game]()` mở lại các nút, xóa trạng thái dở và dựng lại sân (`setup()`/`refresh()`/`fill()`/`render()`…). **Thêm trò mới có hoạt cảnh thì phải thêm một mục `STOP`** và cho vòng lặp kiểm tra `Run.ep`.
  - Bấm thẻ trò khác khi đang chạy: toast nhắc dùng nút Dừng, nút Dừng rung lên.
- `markNav()` tô sáng thẻ đang chơi (với thẻ trò mini: chỉ thẻ có `data-mini === Mini.gk`); gọi trong `goTab()` và `Mini.markPick()`. Click thẻ trò mini → `goTab('mini')` rồi `Mini.choose(k)`; đang chơi dở thì không đổi trò.
- Nút bấm được `blur()` sau click để phím Space/Enter không kích hoạt lại nút đó.
- **Khóa khi đang chạy:** tab bị ẩn (`display:none`) làm trình duyệt **hủy CSS transition**. Reel jackpot và cube xúc xắc nhảy thẳng tới kết quả trong khi `setTimeout` vẫn chạy, nên khi quay lại tab thì kết quả đã hiện sẵn.
  - Cách xử lý: `runningGame()` trả về game đang chạy (dựa trên các cờ `Wheel.spinning`, `Slot.spinning`, `Duck.racing`, `Marble.racing`, `Ufo.busy`, `Lotto.rolling`, `Dice.rolling`, `Group.busy`).
  - `blockedByRunning()` chặn `goTab()` và việc đổi lớp, kèm toast nhắc.
  - Một `setInterval` 150ms gắn `body.busy` để làm mờ các tab khác.
- **Mở rộng đường đua:** `.expanded` = `position:fixed`, phủ kín cửa sổ, z-index 40 (modal là 50, timer 55, toast 70). Bên trong có `.xbar` với nút Bắt đầu / Thu nhỏ; nút Bắt đầu tự ẩn khi `body.busy`. Chuyển tab thì `Expand.close()`. Sau khi bật/tắt phải gọi lại `Duck.layoutLanes()`, `Marble.resize()` và `Ufo.resize()`, việc này `Expand` đã làm sẵn.

## Các game
| Tab | Object | Ghi chú |
|---|---|---|
| `roster` | (init) | Textarea danh sách (tự lưu, debounce 350ms), quản lý lớp, xuất/nhập, điểm danh (bấm tên trong danh sách xem trước), tạo danh sách số 1→N, thời gian chạy các trò, sửa thử thách/phần thưởng. |
| `wheel` | `Wheel` | Vẽ bằng canvas. Ô có **độ to khác nhau** theo `weights[key]` (`WEIGHT_STEPS` 0.25–5, mặc định 1). `layout()` tính `g={starts,sizes,ws,total}`. Kết quả chọn theo trọng số bằng `rand01()*total` **trước**, rồi mới tính góc dừng trong ô đó, nên **xác suất = phần ô trên vòng**. `indexAt(rot)` tìm ô theo `starts`. Có tùy chọn `wheelShort`. **Trộn vị trí ô**: `Classes.cur().wheelOrder` (mảng key, `null` = theo số thứ tự); `arrange(p)` xếp `pool()` theo đó (bạn mới thêm nằm cuối). Nút 🔀 `mix()` có hoạt cảnh đảo vài lần rồi `newOrder()`; ↺ thì đặt `null`; tùy chọn `#wheelAutoMix` (key `wheelAutoMix`) trộn lại trước mỗi lần `spin()`. Trộn chỉ đổi chỗ, không đổi trọng số nên xác suất giữ nguyên; người thắng lấy từ `this.items[t]` (đã sắp xếp). |
| `slot` | `Slot` | 3 reel. Chiều cao item = `.reel-win.clientHeight/3`. `r.last` giữ 3 item đang hiện để lượt sau nối liền. Có chế độ "Chỉ quay tên". |
| `duck` | `Duck` | Giao diện "giải đua vịt ao làng": băng rôn `.race-head` có cờ đuôi nheo và bình luận viên `say(msg)` (`#duckTalk`, báo khi đổi người dẫn đầu, giới hạn ~1.3s/lần, gần đích, về đích). Vịt là SVG `<symbol id="duckSym">` bọc trong `.dbody` cùng **phù hiệu số `.dnum`** trên bụng (số thứ tự; chế độ "Chỉ tên" thì là chữ cái đầu). Mọi animation/hiệu ứng lắc, xoay áp vào `.dbody` để số đi theo vịt, còn filter màu (sét, rong, ngủ) chỉ áp vào `svg` để số luôn trắng, dễ đọc. Mũ vịt có màu `currentColor` trùng chấm màu trên thẻ tên (mũ vàng thì đổi sang màu tối). Bến gỗ, sóng nước chạy (`flow`), dây phao, vệt nước theo `--spd`, 💨 khi tăng tốc / 💦 khi vấp (`.fx`), 👑 trên vịt dẫn đầu, vịt về đích nhảy (`hop`). `.water` **không dùng `overflow:hidden`** (sẽ cắt mất vương miện); `.tcol` mới cắt. **Kỹ năng** (`DUCK_SKILLS`, chọn mức Tắt/Ít/Vừa/Nhiều ở `#duckSkills`, key `duckSkills`): tự tăng tốc 🚀🍞🌊, tự làm chậm 🌀🌿💤, chơi xấu 🍌 (vịt gần nhất phía trước) và ⚡ (vịt dẫn đầu, trọng số thấp). Mỗi vịt nhận kỹ năng theo tiến trình Poisson cùng một tốc độ (`rate` = mức × hệ số ngẫu nhiên 0.4–1.6 mỗi lượt), có giới hạn số hiệu ứng cùng lúc (`cap` = max(3, n/4)). Không có mục tiêu thì đổi sang kỹ năng cho bản thân. Hiệu ứng là class `sk-<k>` trên `.lane` cùng icon `.sk`. `RACE` được nhân 1.04/1.08/1.12 theo mức để bù việc vịt thắng thường vừa được tăng tốc. **Vật lý tách riêng:** `makeRace()` + `stepRace(st, now, dt)` + `cast()` không đụng DOM; `run()` chỉ vẽ theo sự kiện `{crossed, casts, ended}`. Kiểm thử thì gọi trực tiếp hai hàm này. Tối đa 60 vịt. Tốc độ đổi ngẫu nhiên mỗi 250–900ms (có tăng tốc / vấp); dừng khi đủ top 3 (`podium`). `layoutLanes()` chia đều chiều cao còn trống cho các làn (14–90px) và tính `maxX`; phải gọi lại khi resize hoặc mở rộng. Trên 30 vịt thì render 2 cột `.tcol` song song, mỗi cột là một block riêng. **Không dùng CSS grid** cho phần này vì grid từng làm các hàng giãn sai. Nhiều vịt chạm vạch cùng frame thì xếp theo thời điểm chạm vạch nội suy (`r.cross`). Bấm "Vắng" thì `announce(i+1)` lấy vịt về sau, không đua lại. | **Hai kiểu hiển thị** (`#duckView`, key `duckView`: `auto`/`lanes`/`river`; `auto` = Dòng sông khi ≥15 vịt) — xem phần "Đua vịt kiểu Dòng sông".
| `marble` | `Marble` | Đua bi — xem phần riêng bên dưới. |
| `ufo` | `Ufo` | **UFO bắt trả bài** (lấy ý tưởng từ "UFO Khảo bài" của KROMA, nhưng tự vẽ và tự viết). Canvas khu phố ban đêm: trời sao, trăng, nhà, bãi cỏ; mỗi bạn trong `pool()` là một nhân vật (`kids`, xếp theo `arrange()`: thứ tự đã trộn `Classes.cur().ufoOrder` hoặc theo `no`; nút 🔀/↺ gọi `mix()` → `reorder(ms)` cho các bạn nhảy sang chỗ mới; tùy chọn `#ufoAutoMix` (key `ufoAutoMix`) đổi chỗ trước mỗi lượt, trừ lượt bắt bù khi có bạn vắng; `layout()` tự chọn số hàng sao cho nhân vật to nhất). Ngoại hình `look(s)` cố định theo `key`; nam/nữ ưu tiên `s.gender` → tên riêng phổ biến (`UFO_GIRL`/`UFO_BOY`) → đệm Thị/Văn → hash. Hash phải dùng `>>>` (`>>` từng làm chỉ số âm → nhân vật mất tóc/màu áo). **Chọn trước bằng `choose(p,k)` = `shuffle(p).slice(0,k)`** (đã kiểm chi-square), hoạt cảnh chỉ để diễn: bay vào → `sweep()` quét đèn vàng (nhân vật trong đèn giơ tay "!") theo `UFO_SWEEPS` (Nhẹ 1 / Hồi hộp 2 / Cực căng 3 vòng; Cực căng có thêm `decoy()` lượn nhầm một bạn khác) → `capture()` tia hút xanh kéo nhân vật lên tàu. Gọi 1–10 bạn/lượt (`#ufoCount`). `#ufoStatus` là dòng trạng thái. Khoang hành khách `#ufoCabin`: nút 🚫 → `absent(key)` (unpick + setAbsent + `launch({k:1, quick:true, append:true})`); đổi `rosterVer` thì khoang tự xóa. Một bạn thì dùng `personModal`, nhiều bạn thì modal liệt kê `.ufo-list`. Đọc tên bằng `speechSynthesis` (`vi-VN`, ưu tiên giọng `vi*` có trong máy; nếu không có thì `#ufoVoiceHint` báo cách cài). `tween(ms, fn)` + `ease2` là helper hoạt cảnh; vòng vẽ `loop()` chỉ chạy khi tab UFO đang mở hoặc đang bay. |
| `farm` | `Farm` | **Bắt gà 3D** (three.js) — xem phần riêng bên dưới. |
| `mini` | `Mini` + `MINI[k]` | **10 trò mini** — xem phần riêng bên dưới. |
| `box` | `Box` | Lật 3D. Trộn ~12% ô bất ngờ (`SURPRISES`, không `pick`). `sync()` làm mờ hộp của bạn đã gọi hoặc vắng mà không xáo lại. `primary` mở một hộp ngẫu nhiên. |
| `group` | `Group` | Dùng `present()`; chia vòng tròn nên các nhóm lệch nhau tối đa 1; người đầu mỗi nhóm là nhóm trưởng; sao chép kết quả. |
| `lotto` | `Lotto` | `drawn` lưu theo lớp. `nameOf(x)` tìm theo `no`, không theo index. Khi bật "Hiện tên" thì bỏ qua số của bạn vắng. Max mặc định = `no` lớn nhất, trừ khi người dùng đã tự sửa (`lottoMaxTouched`). **Hai chế độ** (`mode`, key `lottoMode`, nút `#lottoMode`): *Quay từng số* như cũ, và *Trộn cả dãy*: `shuffleRun()` lấy mọi số từ Từ→Đến (tối đa 200 số; bỏ số của bạn vắng khi bật "Hiện tên"), xáo trước bằng `shuffle()` rồi diễn vài lượt `flip()` (thẻ bay đổi chỗ, kỹ thuật FLIP; số lượt theo `Dur('lotto')`). Dãy lưu theo lớp ở `Classes.cur().shuffle = {a, b, order, done}`. `next()` gọi số kế tiếp chưa gọi và không vắng → `pick` + `personModal` ("Vắng – gọi số kế tiếp"); bấm thẻ thì đánh dấu/bỏ đánh dấu tay; có nút sao chép dãy. Phần tử chỉ thuộc một chế độ có class `.roll-only` / `.shuf-only` và được ẩn bằng `hidden` (CSS `[hidden]{display:none !important}`). Space: đang ở Trộn thì còn số → `next()`, hết số → trộn mới. |
| `guess` | `Guess` | Báo lớn hơn/nhỏ hơn, thanh hiển thị khoảng còn lại. Người đoán ngẫu nhiên lấy từ `present()` (không `pick`). |
| `bomb` | `Bomb` | Lưới 5–300 số; trúng bom thì flash, rung màn hình, kèm thử thách. Người chọn lấy từ `present()`. |
| `dice` | `Dice` | Cube CSS 3D (1–6 viên); đưa mặt v ra trước bằng `ORI[v]`, xoay thêm bội số 360°. Mặt 1 và 4 chấm đỏ. |
| `score` | `Score` | Bảng điểm theo lớp; sắp xếp theo STT / điểm / ít được gọi; sao chép TSV để dán vào Excel. |

### Đua bi (`Marble`) — chi tiết
- **Thế giới:** rộng `W=600` đơn vị; canvas scale `s=min(cw/W, ch/380)`, camera theo bi dẫn đầu (`updateCam`). Tối đa 60 bi; bán kính 12 / 10.5 / 9 / 8 tùy số bi.
- **Vật lý:** `G=1400`, `SUB=4` bước con mỗi frame.
  - **Lực cản không khí ~v²** dạng ẩn (`DRAG = G/VT²`, tốc độ giới hạn `VT=430`, chặn `VMAX=750`) để bi không lao vụt qua đoạn trống.
  - Va chạm bi–đoạn thẳng (dốc, tường; cánh quạt quay có tính vận tốc bề mặt), bi–cọc, bi–nấm (`e=1.1`, nảy mạnh) và bi–bi.
  - Vật tĩnh được chia ô theo trục y (`bk`, ô 80 đơn vị) để chỉ kiểm tra va chạm ở gần.
- **Đường đua** (`build`): cửa xuất phát → 5 hàng cọc "xáo trộn" → các đoạn ghép ngẫu nhiên từ `pegs` / `zigzag` / `bumpers` / `funnel` (phễu + cánh quạt) → vạch đích → sàn.
  - Đoạn được ghép tới khi tổng thời gian ước tính `EST` ≈ `Dur.get('marble')`. Hiện tại `EST = {zigzag:4.7, pegs:2.4, funnel:1.6, bumpers:1.6}`, cộng 1.6s cho phần đầu.
  - **Đổi vật lý, đổi kích thước đoạn hoặc thêm loại đoạn thì phải đo lại `EST`** (xem Kiểm thử). Lỗi cũ: ma sát lăn quá cao làm đoạn zigzag mất 20 giây.
  - `this.bounds` lưu `y0/y1` của từng đoạn để đo.
- **Công bằng:**
  - chỗ xuất phát được `shuffle`, rải đều trong 60% giữa bề ngang (sát tường thì bi chậm);
  - có lệch vị trí ngẫu nhiên và `kick()` lúc mở cửa.
  - Kết quả đo: theo học sinh p = 0,63–0,98; theo chỗ xuất phát p = 0,085 ở đường ngắn, còn ở đường dài chỗ chính giữa lợi nhẹ (tối đa ~1,5 lần).
- **An toàn:**
  - chống kẹt: đứng yên 1.2s thì được hích;
  - timeout: quá max(3T, T+30)s thì xếp hạng theo vị trí;
  - thứ tự về đích tính theo thời điểm chạm vạch trong bước con (`ct`).
- **`advance(dt)`** chạy vật lý một frame mà không vẽ, nên dùng được cho mô phỏng nhanh khi test. Khi mở rộng thì vẽ thêm bảng top 5 lên canvas. Podium + `announce(i)` giống Duck.

## Đua vịt kiểu "Dòng sông" (lớp đông)
- Lấy ý tưởng từ Duck Race của online-stopwatch.com: không chia làn, cả đàn bơi chung một dòng sông vẽ bằng canvas `#riverCanvas` (trong `#river`, class `.rivermode` trên `#raceBox` ẩn `#track`). **Dùng chung `makeRace`/`stepRace`** với kiểu làn nên công bằng như nhau; `runRiver(st,t0)` chỉ khác phần vẽ (bình luận, âm thanh giống `run()`).
- `setupRiver()`: mỗi vịt một chỗ `slot{t (0..1 từ trên xuống), col, ph}` theo thứ tự xáo trộn; trước giờ đua xếp so le 3 hàng (4 hàng nếu >24 vịt) sau vạch, phần so le mờ dần trong ~11% quãng đường đầu (`rPos`).
- `resizeRiver()` → `rg = {W,H,d,bank,map,top,bot,band,dh,dw,X0,L,skew}`: vịt cao `dh = clamp(band×0.2 (0.17 nếu >40), 40, 104)`, đường đua thế giới `L = max(1500, 2.3W)`, vạch xuất phát/đích xiên (`skew` 0.32) và vị trí vịt xiên theo cùng độ nghiêng.
- Camera `rv.camX` giữ vịt dẫn đầu ở 62% màn hình, cuối cùng dừng khi vạch đích ở 54% (chừa chỗ bên phải cho bảng kết quả); đua xong thì `riverIdle()` trượt nốt.
- Vẽ: bờ cỏ + bụi cây + biển 25/50/75%, sóng, vịt sắp theo `y` (xa trước gần sau), vịt gần to hơn (`k` 0.84–1); bong bóng số (`bubbleText`: số, hoặc tên ngắn ở chế độ Tên) tô vàng/bạc/đồng cho top 3, 👑 vịt dẫn đầu, biểu tượng kỹ năng, `filter` cho zap/weed/sleep, vịt về đích trôi qua vạch 1,8%; viên "◀ n vịt phía sau"; **bản đồ nhỏ** dưới đáy (chấm màu từng vịt + khung phần đang thấy).
- DOM chồng lên canvas: `#riverTop` (dải top 3–5 theo bề rộng, tên ngắn `chipName`, cập nhật 200ms) và `#riverBoard` (bảng kết quả đầy đủ sau khi đua, cuộn được, nút ▾ thu gọn).
- Ảnh vịt: `duckImgs()` dựng 8 ảnh từ `#duckSym` (thay `currentColor` bằng màu mũ `capCol(i)`) qua data-URL SVG.

## Bắt gà 3D (`Farm`)
- **Cách chơi:** sân trại 3D có hàng rào gỗ, nhà kho đỏ, cây, rơm và một **chuồng gà** sát hàng rào (phía sau bên phải). Mỗi bạn trong `pool()` là một chú gà có huy hiệu số + tên (sprite canvas, đổi theo `State.display`). Bác nông dân rượt hụt vài con (mức `light/mid/max` = hụt 1/2/3 lần trước bạn đầu tiên, các bạn sau hụt tối đa 1), rồi rượt trúng, "CHỘP!", ôm gà ra hàng rào và quăng theo đường vòng cung vào chuồng.
- **Công bằng:** `choose(avail,k) = shuffle(avail).slice(0,k)` chọn **trước**; gà bị rượt hụt chọn bằng `rint`. Lúc rượt trúng, gà mục tiêu chạy chậm dần (`tired`) để chắc chắn bị bắt; lần rượt hụt thì gà nhảy né (`dodge`). Hoạt cảnh không ảnh hưởng kết quả.
- **Dựng cảnh lười:** `gl()` chỉ tạo WebGL lần đầu mở tab hoặc bấm chơi. Không có WebGL thì `#farmStatus` báo lỗi, các tab khác vẫn chạy bình thường.
- **Hiệu năng:**
  - Các khối tĩnh được gộp bằng `bake(parts)` thành một mesh tô màu theo đỉnh (`M.vc`, `part = [geo, màu, sx,sy,sz, x,y,z, rx,ry,rz]`): cả sân trại là 1 mesh. Mỗi gà có 6 mesh (thân, đầu, 2 cánh, 2 chân) + 1 sprite, và geometry thân/đầu được cache theo kiểu lông trong `BC`. 30 gà ≈ 220 draw call.
  - `setQuality(q)` có 4 mức: 0 đầy đủ; 1 tắt bóng đổ; 2 tạo lại renderer không khử răng cưa (thay `#farmCanvas` bằng canvas mới); 3 pixelRatio 0.7.
  - `software()` phát hiện SwiftShader/llvmpipe nên vào thẳng mức 2. `perf(raw)` lấy trung vị 30 khung hình, nếu > 34ms thì hạ một mức.
  - Vòng `loop()` chỉ chạy khi đang ở tab `farm` hoặc đang `busy`; `dt` tối đa 0,1s.
- **Ngoại hình:** `look(s)` dùng lại `Ufo.look(s).girl` → gà mái (hen) hoặc gà trống (rooster, to hơn 12%, mào lớn, đuôi 3 màu); màu lông theo hash tên (`>>>`).
- **Mô phỏng:**
  - `step(now,dt)`: gà đi lang thang, mổ thóc, và bỏ chạy khi bác nông dân đang chạy trong bán kính 3,4; gà đẩy nhau ra cho khỏi chồng; bị giới hạn trong sân.
  - Trạng thái gà: `roam` → `held` → `fly` → `pen`.
  - `stepFarmer`: `chaseCtl` (tốc độ tự chỉnh để tới nơi đúng lúc hết `T`, lao chụp khi < 1,35 hoặc hết giờ) và `walkTo` (đi tới điểm).
- **Camera:** khi rảnh thì xoay chậm quanh sân. Khi bật `#farmCam` và đang chơi thì bám theo bác nông dân; tọa độ được làm mượt (`cp`).
- **Sân:** bán kính `R = clamp(4+√n·1.6, 7, 16)`, dựng lại trong `setup()` mỗi khi `stale()`; `stale()` so gà `roam` với `pool()` trừ gà đang trong chuồng.
  - Nếu tắt "Không gọi lại" thì lượt mới (không phải `append`) thả hết gà ra sân lại.
  - `avail` = các bạn có gà đang ở trên sân.
- **Vắng:** `#farmCabin` (🧺 Chuồng gà) có nút 🚫 gọi `absent(key)`: bỏ gà khỏi chuồng rồi chạy `launch({k:1, quick:true, append:true})` (quick = không rượt hụt). `results()`, `speak()` (dùng `Ufo.viVoice()`) và `renderCabin()` làm giống UFO.
- **Test:** Chromium headless cần `--use-angle=swiftshader --enable-unsafe-swiftshader`. Máy test (2 lõi) chỉ đạt khoảng 10 fps nên chỉ dùng để chụp ảnh và đo thời lượng (theo dõi `#farmStatus` bằng `MutationObserver`). Mức Hồi hộp, 1 bạn ≈ 11s tới lúc "CHỘP!".

## Trò mini (`Mini`, tab `mini`)
- Ý tưởng 10 trò lấy từ trang "Ms Alice's Random Name Picker" (của cô Lương Tâm); **hình và code tự viết**, vẽ hoàn toàn bằng canvas `#miniCanvas` (không dùng ảnh), dùng emoji cho vài chi tiết không có hướng (🍰 🐭 🐝 🧀 📺…). Không tự vẽ lại hình minh hoạ của trang gốc.
- `MINI_GAMES` = danh sách thẻ chọn trò `{k, icon, name, sub, go (chữ trên nút), result (tiêu đề modal)}`: `fish` Câu cá, `balloon` Bóng bay, `dino` Khủng long, `rocket` Tên lửa, `torch` Đèn pin, `card` Thẻ đỏ, `blind` Bịt mắt bắt dê, `light` Đèn xanh đèn đỏ, `cat` Mèo bắt chuột, `bee` Ong tìm hoa. Lưới thẻ `#miniPick` (class `.mgcard` — **đừng dùng `.mcard`, đó là thẻ modal**).
- **Công bằng:** `launch()` bốc `target = actors[rint(n)]` ngay từ đầu. `plan(target, T, twist, quick)` dựng dòng thời gian: dẫn 700ms → các đoạn `miss` (~2,8s/đoạn, tối đa 6) → đoạn `fake` 2,6s nếu bật **Lật kèo** → đoạn `win` 3s cuối (người thật lộ diện). Bạn bị "hụt" được chọn bằng `rint` trong 35% số bạn **gần chỗ đang đứng nhất**, lần hụt/lật kèo cuối chọn trong 25% số bạn gần người thật → người đi tìm không lao qua cả màn hình (đo: tốc độ đỉnh ~270–400 px/s trên sân 960px, trước đây 600–1200). Pha: move 42–45% → try/got → fail/twist/cele. Đoạn mới luôn bắt đầu từ vị trí đang vẽ (`hpos`). `quick` (sau khi bấm Vắng) = 2s, không hụt.
- `phase(seg,u)`: `move` → `try` → `fail` (hụt) | `move` → `got` → `twist` (lật kèo) | `move` → `got` → `cele` (thắng). `cur(now)` trả `{el, seg, u, ph, k}`; sau khi xong `done` giữ đoạn `win` ở `cele`.
- Mỗi trò `MINI[k]` có: `layout(A,W,H)` (đặt `a.x, a.y, a.cell` và kích thước riêng như `a.z`, `a.r`, `a.L` — **không ghi đè `a.s`, đó là học sinh**), `bg(c,W,H,now)`, `draw(c,now,cu,M)`, `txt` (câu trạng thái), tuỳ chọn `hunt:true` + `home()` + `anchor(a,base,now,ph)` (người đi tìm: lưỡi câu, khủng long, trọng tài, người bịt mắt, đèn pin, mèo, ong — `Mini.hunter()` nội suy vị trí trong pha `move`), `start(run)` (Bóng bay lên lịch nổ `R.pops`, Tên lửa `R.seed`, Đèn xanh reset `a.adv`), `onPhase` (thay câu trạng thái mặc định), `sfx(ph)`.
- **Trộn vị trí mỗi lượt:** xáo `actors`, `layout()` lại, `base(a,now)` nội suy từ chỗ cũ trong 750ms; thứ tự được nhớ trong `Mini.order` (theo `key`).
- Công cụ chung: `fit(n,x0,y0,w,h,ratio)` (lưới to nhất), `tag()` (viên tên; ô < 70px thì chỉ hiện số), `kid(c,x,y,size,look,now,pose)` với pose `idle|panic|sad|happy|blind|run|hide` (vẽ lại từ nhân vật UFO, `look = Ufo.look(s)`), `emoji()`, `bubble()`, `glow()`. Bịt mắt bắt dê xếp vòng elip **chia đều theo chiều dài cung** (2 vòng khi > 20 bạn); Tên lửa 1–3 hàng (≤ 20 tên lửa/hàng); Đèn pin vẽ bóng tối trên canvas phụ `dk` rồi khoét vùng sáng bằng `destination-out`.
- Đồng hồ `#miniClock` đếm ngược tổng thời gian, kêu bíp 3 giây cuối. Kết quả dùng `personModal` (nút Vắng → `launch({quick:true})`), đọc tên bằng `Ufo.viVoice()` nếu bật `#miniSpeak`. `Mini.busy` khóa chuyển tab.
- Thêm trò mini mới: thêm mục vào `MINI_GAMES` và object `MINI.k` theo giao diện trên.

## Thêm một game mới
1. Thêm nút trong `nav#tabs` và một `<section class="tab" id="tab-ten">`. Nếu chèn vào giữa thì báo người dùng rằng phím số bị dịch.
2. Viết `const Ten = { init(){…}, onShow(){…} }`:
   - lấy người bằng `ensurePool()`, hiển thị bằng `label()`;
   - có kết quả thì `pick(s)` rồi `personModal({...})`.
3. Đăng ký `TABS.ten = { onShow, primary }` và gọi `Ten.init()` ở phần boot.
4. Có animation thì:
   - thêm cờ đang chạy vào `runningGame()`;
   - thêm thời lượng vào `DUR_DEF` cùng một `<input data-dur="ten">` trong tab;
   - chống bấm hai lần bằng cờ + `disabled`.
5. Có random thì kiểm thống kê công bằng trước khi giao.

## Thứ tự boot & bẫy TDZ
`Classes.load(); loadClassState(); initHeader(); Wheel/Slot/Duck/Marble/Ufo/Farm/Mini/Box/Group/Lotto/Guess/Bomb/Dice.init(); Timer.init(); Score.init(); Dur.init(); initRosterTab(); switchClass(curId, true); goTab(...)`
- `applyRoster`, `updatePicked`, `setAbsent` và `runningGame` tham chiếu các object game (là `const`) qua `typeof`. **Trên `const` đang ở TDZ, `typeof` vẫn ném lỗi**, nên các hàm đó chỉ được chạy sau khi mọi object đã khai báo (tức là sau boot).
- `switchClass` gọi `Lotto.load()`, `Guess.nextTurn()`, `Bomb.nextTurn()`, nên phải chạy sau các `init()`.

## Kiểm thử
Không có test tự động trong repo. Khi sửa, kiểm tra bằng Playwright (Chromium headless).

**Chung:**
- Mở `file:///…/index.html`, nghe `pageerror`. Kiểm tra **không có request mạng** nào ngoài `file:`/`data:`.
- `page.on('dialog', d => d.accept())` (dùng cho `confirm`/`prompt`). Muốn bắt đầu sạch thì `localStorage.clear()` rồi reload.
- Nạp danh sách mẫu, đi qua cả 15 tab (tab Trò mini: chọn từng trò bằng `Mini.choose(k)`), chạy `primary` hoặc click từng game, chụp màn hình để xem.
  - Thời gian chờ với giá trị mặc định: wheel ~7s, slot ~4.5s, duck ~13s, marble ~18s (cả hai đều gồm 3-2-1), ufo 1 bạn mức Hồi hộp ~10s, 2 bạn mức Cực căng ~18s, quay 4 số ~6s.
  - **Đợi trò chạy xong rồi mới chuyển tab**, vì khóa chuyển tab sẽ chặn và làm script tưởng có lỗi.

**Các trường hợp phải thử:**
- paste từ Excel (cột STT, dòng tiêu đề, ngày sinh);
- danh sách chỉ có số;
- cả 3 chế độ hiển thị;
- đánh dấu vắng rồi bấm "Vắng – chọn lại" (wheel, duck, marble);
- gọi hết cả lớp;
- tạo lớp mới, xuất rồi nhập lại vào storage trống;
- chuyển dữ liệu từ bản cũ;
- đồng hồ hết giờ;
- lớp đông 55 người (duck 2 cột, marble), có và không mở rộng;
- các viewport 1366×768, 1024×768, 390×844.

**Công bằng** (chạy trong `page.evaluate`, dùng chi-square của scipy, p > 0,05 là đạt):
- `rint(n)`, `shuffle`, xúc xắc: đếm 200–300k lần.
- Vòng quay có trọng số: lặp lại đúng vòng chọn `rand01()*total` khoảng 400k lần, so tỉ lệ thực tế với `w/total`; đồng thời chạy spin thật và kiểm `indexAt(rot)` trùng người thắng.
- Đua vịt: gọi `Duck.makeRace(n, RACE, level, 0)` rồi lặp `Duck.stepRace(st, now, 1/60)` (đúng code thật). Mỗi mức kỹ năng chạy khoảng 3k cuộc đua, mức "Nhiều" chạy 20k; đếm làn về nhất (p > 0,05) và so thời gian về nhất giữa các mức (phải xấp xỉ nhau). Kết quả lần đo gần nhất: p = 0,10–0,90; mức Nhiều 20k lượt p = 0,39.
- UFO: lặp `Ufo.choose(pool(), 3)` 100k lần, đếm số lần mỗi bạn được chọn (lần đo gần nhất p = 0,105).
- Bắt gà: lặp `Farm.choose(30 bạn, k)` 60k lần với k = 1 và k = 3, đếm cả tổng số lần và người đầu tiên (lần đo gần nhất p = 0,33–0,63).
- Đua bi: `Marble.setup(); gateOpen=true; kick();` rồi lặp `advance(1/60)` tới khi `done`, khoảng 1.000 cuộc đua 8 bi; đếm người thắng theo **học sinh** và theo **thứ hạng chỗ xuất phát** (sắp theo y rồi x).

**Hiệu chỉnh thời gian Đua bi:**
- Thời gian theo đoạn: theo dõi bi dẫn đầu đi qua `Marble.bounds[i].y1` để đo từng loại đoạn.
- Tổng thời gian: với T = 6/15/30 và n = 5/15/30, thời gian bi đầu về đích phải ≈ T và không lượt nào timeout. Bi rơi tự do không vượt quá ~470 đơn vị/s.
