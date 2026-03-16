# SwiftResize ⚡

**Lightning-fast image compression & resizing directly in your browser.**  

**Tagline:**  
> Try SwiftResize instantly — no upload required, 100% private, and works on any device.

---

## Features (Demo Version)

- ⚡ **Instant Compression:** Resize & compress images locally in your browser.  
- 🎨 **Multiple Formats:** PNG, JPEG, WebP, AVIF — choose automatically or manually.  
- 📦 **Batch Processing:** Compress multiple images at once and download as a ZIP.  
- 🔒 **100% Private:** All images are processed locally; nothing leaves your device.  
- 📱 **Works Everywhere:** Desktop, mobile, tablet — no installation needed.  
- 📈 **Metrics Display:** See size reduction and processing time for each image.

> **Note:** This demo handles **small to medium files only**. Large files are not supported in the public demo.

---

## Tech Stack

**Frontend:**  
- [Svelte + Vite](https://vitejs.dev/) – lightweight SPA  
- [TailwindCSS](https://tailwindcss.com/) – optional styling  
- [Squoosh WASM codecs](https://github.com/GoogleChromeLabs/squoosh) – client-side compression  
- [JSZip](https://stuk.github.io/jszip/) – batch ZIP downloads  

**No backend is required** — everything runs in the browser for maximum privacy.

---

## Installation / Setup

```bash
git clone https://github.com/arfeliu/SwiftResize.git
cd SwiftResize
npm install
npm run dev
# Access at http://localhost:5173
npm run build
# Production build in dist/
