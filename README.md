# 📸 Reel Momentos Photobooth — Automated Cloud Sync & Sharing Station

An end-to-end, automated digital delivery, solo-operator monetization, and social engagement system built for **Reel Momentos Photobooth** (Cabuyao, Laguna).

This system pairs local booth hardware with a background Cloudinary sync engine, a high-contrast guest-facing claiming table display, and a high-converting mobile web landing page hosted on GitHub Pages.

---

## 🏗️ System Architecture

```
[ KIOSK / MONITOR 2 ]               [ CLAIMING TABLE / PRIMARY LAPTOP ]
  • Guest-facing camera view          • Windows Default Photo Printer (Strips / 4x6)
  • Live countdown & posing           • sync_and_display.py Tkinter Kiosk Engine
  • Photo booth capture software      • High-DPI scaling aware & F11 Presentation Mode
              │                       • Generates high-contrast QR code per print
              │                       • Operator Reprint Controller (₱50/ea with + / −)
              │                                      │
              └──────────────► Local Folder ◄────────┘
                                     │
                        (Watchdog Event Trigger)
                                     │
                        (wait_for_file_ready Polling)
                                     ▼
                        [ CLOUDINARY CDN (Free 25GB) ]
                          • /reel-momentos/{EVENT_ID}/
                          • latest.json (Event Carousel Index)
                                     │
                                     ▼
                      [ GUEST SMARTPHONE / WEB APP ]
                        GitHub Pages (index.html)
                          ├── Protected Preview (Anti-Long-Press)
                          ├── Swiper.js Touch Carousel (GPU-Accelerated)
                          ├── 1-Tap Google Reviews (Zero-Login Barrier & Shuffle)
                          ├── Extra Prints Monetization (GCash QR Modal)
                          ├── 1-Tap Story Share (Auto-Copy Captions)
                          └── High-Res Download Gate (Support Modal)

```

---

## 🗂️ Project File Structure

```text
All About Photobooth/Automation/
├── config.py             # ⚙️ Central configuration (Credentials & active EVENT_ID)
├── sync_and_display.py   # 🖥️ Watcher engine + High-DPI Claiming GUI + Reprint Controller
├── cleanup_event.py      # 🧹 Post-event 1-click cloud storage cleanup tool
├── qrcodes/              # 📁 Auto-generated local QR images
└── test/                 # 📁 Watched folder for outputted photobooth prints

instant-download/ (GitHub Pages Repository)
├── index.html            # 📱 Mobile guest claiming & upsell web application
├── gcash-qr.png          # 📱 Operator GCash QR code image for reprint payments
├── logo.png              # 🟣 Brand circular icon
└── logo.jpg              # 🟣 Header brand image

```

---

## 🖥️ Physical Setup & Claiming Flow

### Hardware Placement

1. **Monitor 2 (Facing Backdrop):** Dedicated strictly to guests posing and the photobooth countdown software.


2. **Laptop (Operator & Claiming Table):** Sits on the print pickup table, slightly angled toward the guest line.


3. **Printer:** Sits beside the laptop, configured as the Windows Default Printer with your borderless strip/4x6 paper preset.



### Operational Step-by-Step

1. Guests take photos in front of Monitor 2.


2. When finished, guests walk over to the claiming table to wait for their physical prints.


3. While the operator cuts and sleeves the physical print, the laptop screen automatically displays:


* A sharp, scaled preview of their rendered print layout.


* A high-contrast QR code with a green status tag: `✓ Uploaded & Ready to Download`.


* A quick reprint counter (`−` / `+`) and dynamic reprint action button (`₱50` / print).


4. Guests scan the laptop screen with their phone camera and leave the camera area clear for the next group.



---

## ⚡ Station Desktop Engine Features (`sync_and_display.py`)

* **High-DPI Awareness:** Initializes `ctypes.windll.shcore.SetProcessDpiAwareness(1)` prior to window instantiation to prevent UI blurring and distortion across mixed-resolution dual displays.
* **Kiosk Presentation Mode:**
* `F11`: Toggles clean fullscreen kiosk mode, removing OS taskbars and borders.
* `Escape`: Exits fullscreen for administrative adjustments.


* **File-Lock Collision Prevention:** Implements `wait_for_file_ready()` with non-blocking byte reads to eliminate `PermissionError [WinError 32]` while your photobooth software writes high-resolution print files.
* **Tkinter Event Filtering:** Scopes `<Configure>` resize callbacks strictly to `event.widget == self.root` to eliminate recursive UI layout loops caused by child widget repainting.
* **Solo Operator Quick Reprint Station:**
* Interactive quantity selector (`−` / `+`) defaulting to 1 copy (safety capped at 10).
* Dynamic price label calculating total fee (`🖨️ REPRINT X COPIES (₱XX)`).
* Direct spooling via `os.startfile(..., "print")` with a 0.4s buffer sleep to prevent print spooler buffer choking.
* Auto-resets quantity back to 1 copy immediately after queuing to prevent accidental duplicate prints on future sessions.



---

## ⚙️ Configuration (`config.py`)

All scripts read from this single local configuration file. Update this before each event:

```python
# Cloudinary Credentials
CLOUD_NAME = "YOUR_CLOUD_NAME"
API_KEY = "YOUR_CLOUDINARY_API_KEY"
API_SECRET = "YOUR_CLOUDINARY_API_SECRET"

# Active Event Name (Used across uploader, cloud folders, and cleanup)
EVENT_ID = "2026-10-corporate-event"

# URLs & Local Directories
GITHUB_BASE_URL = "https://markloren-swe.github.io/instant-download/"
WATCH_FOLDER = r"C:\Users\MarkLoren\Downloads\All About Photobooth\Automation\test"

```

---

## 📱 Mobile Landing Page Features (`index.html`)

### 1. Protected Photo Preview

* Direct long-press saving is suppressed via `-webkit-touch-callout: none` and `pointer-events: none`.


* Context menus and image dragging are blocked (`oncontextmenu="return false;"`, `draggable="false"`).


* Directs guest traffic cleanly into your high-res download and social funnel.



### 2. Hardware-Accelerated Touch Carousel (Swiper.js)

* Renders event memories via GPU 3D transforms (`translate3d`), eliminating iOS WebKit auto-scroll cancellations and sub-pixel freezing.
* Native finger flick and swipe physics with momentum across iPhone and Android.
* True seamless infinite looping (`loop: true`) with autoplay that pauses on touch or hover.
* Tapping any carousel thumbnail loads it into the main preview for instant downloading.

### 3. Dynamic Google Review Funnel (Zero Login Barrier)

* Routes directly to **Google Reviews** (`GOOGLE_REVIEW_URL`), leveraging persistent mobile browser Google logins and avoiding social media login friction.
* **Sampling Without Replacement:** Draws unique combinations across distinct thematic pools (`printFeatures`, `serviceFeatures`, `overallFeatures`) to eliminate repetitive openers or closers.
* **🔄 Shuffle Action:** Allows guests to reroll 3 fresh, unique review chips on demand.
* 1-tap copy handler copies the exact selected chip and navigates to the Google rating dialog with a 700ms smooth delay.

### 4. Solo Operator Monetization (GCash Upsell Funnel)

* Dedicated on-page upsell card informing guests that extra physical magnetic prints are available at the counter for ₱50.
* Responsive modal displaying `gcash-qr.png`.
* **📥 Save QR to Photos** button that programmatically downloads the QR image to the guest's mobile gallery for GCash's native *Scan QR → Upload QR* workflow.

### 5. Social Sharing & Download Gates

* **Download High-Res Photo:** Displays the interstitial support modal encouraging guests to follow Reel Momentos on Facebook before triggering high-res blob downloads.


* **Share to Stories / IG:** Bypasses the modal to prevent drop-off. Automatically copies pre-formatted promotional captions to clipboard and triggers the native Web Share sheet (`navigator.share`) with the image attached.



---

## 🚀 Event-Day Runbook

### 1. Pre-Event Setup (At Home or Venue)

1. Open `config.py` in your code editor.


2. Update `EVENT_ID` to match the booking.


3. Ensure `WATCH_FOLDER` points to your photobooth software's print output folder.


4. Set Windows display settings to **Extend** (`Win + P`).
5. Ensure your photo printer is set as the Windows Default Printer with proper borderless paper defaults.
6. Launch the station:
```cmd
python sync_and_display.py

```


7. Move the station window to your primary laptop screen and press `F11` to enter kiosk presentation mode.

### 2. During the Event

* The script works 100% autonomously in the background.


* Each time a photo renders, it waits for file readiness, uploads to Cloudinary in ~1s, updates `latest.json`, creates a local high-contrast QR code, and updates the display.


* **Handling Extra Reprint Requests:**
* *Current Photo:* Tap `+` to set the quantity and tap `🖨️ REPRINT X COPIES (₱XX)` after receiving Cash or GCash payment.
* *Past Photo:* Guest finds their print in the mobile carousel, shows it at the counter, and you trigger the print from the watch folder.



### 3. Post-Event Storage Cleanup

To keep your Cloudinary account cleanly within the free tier:

1. Run the cleanup script:


```cmd
python cleanup_event.py

```


2. Review the target event name printed on screen.


3. Type `yes` to confirm. All photos and JSON assets for that event folder are purged immediately.



---

## 🛠️ Required Dependencies

Install all necessary Python libraries with:

```cmd
pip install cloudinary qrcode[pil] pillow watchdog

```

---

## 🔗 Official Brand Endpoints

* **Facebook Page:** [facebook.com/reelmomentos](https://www.facebook.com/reelmomentos/)

* **Messenger Inquiries:** [m.me/reelmomentos](https://www.google.com/search?q=https://m.me/reelmomentos)

* **Google Reviews:** Set via `GOOGLE_REVIEW_URL` in `index.html`
