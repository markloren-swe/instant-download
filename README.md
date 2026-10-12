# 📸 Reel Momentos Photobooth — Automated Cloud Sync & Sharing Station

An end-to-end, automated digital delivery and social engagement system built for **Reel Momentos Photobooth** (Cabuyao, Laguna). 

This system pairs local booth hardware with a background Cloudinary sync engine, a high-contrast guest-facing claiming table display, and a high-converting mobile web landing page hosted on GitHub Pages.

---

## 🏗️ System Architecture

```
[ KIOSK / MONITOR 2 ]               [ CLAIMING TABLE / LAPTOP ]
  • Guest-facing camera view          • Photo printer (2x6 / 4x6 prints)
  • Live countdown & posing           • sync_and_display.py Tkinter GUI
  • Photo booth capture software      • Generates live QR code per print
              │                                      │
              └──────────────► Local Folder ◄────────┘
                                     │
                             (Watchdog Trigger)
                                     ▼
                        [ CLOUDINARY CDN (Free 25GB) ]
                          • /reel-momentos/{EVENT_ID}/
                          • latest.json (Event Carousel)
                                     │
                                     ▼
                      [ GUEST SMARTPHONE / WEB APP ]
                        GitHub Pages (index.html)
                          ├── Protected Preview (Anti-Long-Press)
                          ├── Download Gate (Support Modal)
                          ├── 1-Tap Story Share (Auto-Copy Captions)
                          ├── 1-Tap Anti-Spam FB Review Generator
                          └── Live Rolling Carousel
```

---

## 🗂️ Project File Structure

```text
All About Photobooth/Automation/
├── config.py             # ⚙️ Central configuration (Credentials & active EVENT_ID)
├── sync_and_display.py   # 🖥️ Watcher engine + Claiming Table GUI
├── cleanup_event.py      # 🧹 Post-event 1-click cloud storage cleanup tool
├── qrcodes/              # 📁 Auto-generated local QR images
└── test/                 # 📁 Watched folder for outputted photobooth prints

instant-download/ (GitHub Pages Repository)
├── index.html            # 📱 Mobile guest landing page
└── logo.png              # 🟣 Brand circular icon
```

---

## 🖥️ Physical Setup & Claiming Flow

### Hardware Placement
1. **Monitor 2 (Facing Backdrop):** Dedicated strictly to guests posing and the photobooth countdown software.
2. **Laptop (Operator & Claiming Table):** Sits on the print pickup table, slightly angled toward the guest line.
3. **Printer:** Sits beside the laptop.

### Operational Step-by-Step
1. Guests take photos in front of Monitor 2.
2. When finished, guests walk over to the claiming table to wait for their physical prints.
3. While the operator cuts and sleeves the physical print, the laptop screen automatically displays:
   - A sharp preview of their rendered print layout.
   - A high-contrast QR code with a green status tag: `✓ Uploaded & Ready to Download`.
4. Guests scan the laptop screen with their phone camera and leave the camera area clear for the next group.

---

## ⚙️ Configuration (`config.py`)

All scripts read from this single file. Only edit this file before each event:

```python
# Cloudinary Credentials
CLOUD_NAME = "YOUR_CLOUD_NAME"
API_KEY = "YOUR_CLOUDINARY_API_KEY"
API_SECRET = "YOUR_CLOUDINARY_API_SECRET"

# Active Event Name (Used across uploader, cloud folders, and cleanup)
EVENT_ID = "2026-demo-event"

# URLs & Local Directories
GITHUB_BASE_URL = "https://markloren-swe.github.io/instant-download/"
WATCH_FOLDER = r"C:\Users\MarkLoren\Downloads\All About Photobooth\Automation\test"
```

---

## 🚀 Event-Day Runbook

### 1. Pre-Event Setup (At Home or Venue)
1. Open `config.py` in your code editor.
2. Update `EVENT_ID` to match the booking (e.g., `2026-10-lamcor-compliance` or `2026-11-santos-wedding`).
3. Ensure `WATCH_FOLDER` points to your photobooth software's print output folder.
4. Launch the station:
   ```cmd
   python sync_and_display.py
   ```
5. Maximize or place the window on your laptop screen facing the claiming line.

### 2. During the Event
- The script works 100% autonomously in the background.
- Each time a photo renders, it uploads to Cloudinary in ~1 second, syncs the carousel `latest.json`, generates a local high-contrast QR code, and refreshes the display.
- Photos and QR codes remain on screen indefinitely until the next session drops.

### 3. Post-Event Storage Cleanup
To stay safely within Cloudinary's free 25 GB tier:
1. Run the cleanup script:
   ```cmd
   python cleanup_event.py
   ```
2. Review the target event name printed on screen.
3. Type `yes` to confirm. All photos and JSON assets for that event folder will be deleted immediately, returning your storage quota to zero.

---

## 📱 Mobile Landing Page Features (`index.html`)

### 1. Protected Photo Preview
- Direct long-press saving is suppressed via `-webkit-touch-callout: none` and `pointer-events: none`.
- Context menus and image dragging are blocked (`oncontextmenu="return false;"`, `draggable="false"`).
- Guests must interact with the action buttons, directing traffic through your brand funnel.

### 2. Strategic Action Button Routing
- **Download High-Res Photo:** Always triggers the **"Enjoying Your Photos?"** support modal asking guests to follow `facebook.com/reelmomentos/` for candid shots before initiating the file download (with an option to skip).
- **Share to Stories / IG:** Bypasses the modal to eliminate drop-off. Silently copies your branded caption to the clipboard, displays a toast alert (`✓ Caption copied! Paste it into your story!`), and launches the device's native share sheet with the photo attached.

### 3. Anti-Spam Dynamic Review Generator
To prevent Facebook from shadowbanning or flagging repetitive recommendations:
- Selecting 4 or 5 stars displays three review suggestion chips.
- Tapping a chip combines randomized sentences from three internal pools (openers, features, closings) creating over **125+ natural variations**.
- Copies the custom text to the clipboard and redirects guests directly to:
  `https://www.facebook.com/reelmomentos/reviews/`
- Guests simply paste and post without typing.

### 4. Live Rolling Event Carousel
- Reads `latest.json` from Cloudinary to display recent prints from the current event.
- Continuously auto-scrolls horizontally; pauses on finger touch or mouse hover.
- Tapping any carousel card swaps it into the main preview area for instant re-download.

---

## 🛠️ Required Dependencies

Install all necessary Python libraries with:

```cmd
pip install cloudinary qrcode[pil] pillow watchdog
```

---

## 🔗 Official Brand Endpoints

- **Facebook Page:** [facebook.com/reelmomentos](https://www.facebook.com/reelmomentos/)
- **Facebook Reviews:** [facebook.com/reelmomentos/reviews/](https://www.facebook.com/reelmomentos/reviews/)
- **Messenger Inquiries:** [m.me/reelmomentos](https://m.me/reelmomentos)
