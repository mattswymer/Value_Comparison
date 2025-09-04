# Value Comparisons — README

A single‑file, offline‑friendly web app for a classroom activity: students compare two real‑world items, choose which is more expensive, then record the price and product URL for **both** items. Designed for speed, simplicity, accessibility, and clean PDF export.

---

## ✨ Highlights

- **Zero dependencies** — one self‑contained `index.html` (HTML+CSS+JS)
- **Autosave** to `localStorage` (per device)
- **Sticky footer** with consistent navigation (no layout jump)
- **No-crop images** — normalized card frames that **resize** images without cropping
- **Smart URL input** — accepts partial domains (`target.com`) and normalizes to `https://target.com`
- **Validation** — “Save” and “Save & Next” enable only when both sides are valid
- **Clean Summary → Print/PDF** — hides UI chrome for a screenshot/PDF‑perfect sheet
- **A11y** — keyboard navigation, live regions, focus styles, semantic structure
- **Fast** — system fonts, minimal JS, no frameworks

---

## 🧭 Quick Start

### Option A: Just open locally
1. Save `index.html` (and optionally an `assets/` folder if you use local images).
2. Double‑click `index.html` to open in your default browser.

### Option B: Serve locally (recommended for consistent behavior)
You can use any static server (VS Code Live Server, Python, etc.):

```bash
# From the folder that contains index.html
python -m http.server 8080
# Then browse to http://localhost:8080
```

> **Note:** Remote images are allowed and the app sets `referrerpolicy="no-referrer"` to avoid most hotlink rejections. Some third‑party hosts may still block embedding; the app falls back to a “No image” placeholder.

---

## 🗂️ Project Structure

```
.
├── index.html        # Single-file app (HTML + CSS + JS)
└── assets/           # Optional local images (if you choose to self-host)
```

Everything lives inside `index.html`:
- **CSS** — in a single `<style>` block
- **JS** — in a single `<script>` block
- **Data** — an `items` array at the top of the script (images & labels)

---

## 🧩 How It Works

### Data model: `items → pairs`
At the top of the script:

```js
const items = [
  {
    left:  { name: "Microsoft Xbox Series X – 1TB Digital Edition", emoji: "🎮", image: "..." },
    right: { name: "Sony PlayStation 5 (PS5) Digital Console Slim", emoji: "🎮", image: "..." }
  },
  // ...more pairs
];
```

The app derives `pairs` from `items` and uses `pairs.length` as the total step count.  
**Edit the `items` array** to change names and images—no other changes needed.

### State & persistence
- In‑memory state + debounced autosave to `localStorage`.
- Storage key: `value-comparisons.v6` (versioned to avoid migration issues).
- **Schema** (per pair):
  ```ts
  {
    choice: 'left' | 'right' | null,
    leftPrice: string,  // "12.99"
    leftUrl:   string,  // "https://..."
    rightPrice:string,
    rightUrl:  string
  }
  ```
- Meta:
  ```ts
  { studentName: string, date: string (YYYY-MM-DD) }
  ```

### Validation rules
- A pair is **complete** when:
  - A choice (left/right) is selected
  - Both prices parse to **non‑negative** numbers (2 decimals on blur **only if digits were typed**)
  - Both URLs are **valid http(s)** URLs
- **Smart URL normalization while typing** — `target.com` → stored/validated as `https://target.com`.  
  (On blur, the visible text updates to the normalized value.)

### Navigation
- **Previous** — always enabled (except on the first pair)
- **Save** — explicit save (autosave exists too)
- **Save & Next** — enables only if the current pair validates
- On the **last pair**, the label changes to **“Save & Summary”**
- On the **Summary** view, the sticky footer is **hidden**

### Images (no cropping)
- Each card has a consistent **frame** (`aspect-ratio: 4 / 3` by default)
- Images are **centered** and **scaled down** with `max-width/height` and `object-fit: contain`
- A built‑in **SVG placeholder** appears automatically on broken or blocked image URLs
- **Prefetches** the next pair’s images for snappy "Next" transitions

---

## 👩‍🏫 Student Flow

1. **Start/Resume** — if prior work is found, resume at the first incomplete pair.
2. **Pick** which item is more expensive (click the image card).
3. **Enter price + URL for both items**.  
   - Prices accept `$` and commas; normalize to `12.34` on blur if digits were entered.  
   - URLs accept partial domains and normalize on the fly.
4. **Save & Next** — repeat until all pairs are complete.
5. **Summary → Print/Save PDF** — submit to your LMS.

---

## 🖨️ Printing / PDF

- The **Summary** page is styled for clean printing:
  - Hides navigation and interactive UI
  - Standard 0.5" margins: `@page { margin: 12.7mm }`
  - Avoids page breaks inside a single pair
- Use your browser’s **Print → Save as PDF** to produce the uploadable file.

---

## ♿ Accessibility

- **Single `<h1>`**; headings are semantic
- **Keyboard**:  
  - **Left/Right arrow** to switch selection  
  - Tabbing/focus styles for inputs and cards
- **Live regions** (`aria-live="polite"`) announce progress and validation
- **Asterisk** (`*`) marks the chosen item in Summary, with `aria-label="chosen"`

---

## 🎨 Customization

### Change the item images & names
Edit the `items` array only; total count and UI adjust automatically.

### Image frame shape
In the CSS (`.choice__img`):

```css
/* default */
aspect-ratio: 4 / 3;

/* or square */
aspect-ratio: 1 / 1;

/* or widescreen */
aspect-ratio: 16 / 9;
```

### Colors / theme
Adjust CSS variables in `:root` (light & dark modes supported):

```css
:root{
  --accent: #2563eb;   /* primary brand color */
  --bg: #ffffff;
  --fg: #0f172a;
  /* ... */
}
```

### Button labels
In the `render()` function:

```js
nextBtn.textContent = (state.currentIndex === TOTAL_PAIRS - 1)
  ? 'Save & Summary'
  : 'Save & Next →';
```

### Validation rules
Update `parsePrice`, `normalizeUrlInput`, and `validatePair` to tighten/relax requirements.

---

## 🧰 Troubleshooting

**Images look cropped**  
- They shouldn’t be: the CSS uses a uniform frame + `max-width/height` and `object-fit: contain`.  
- If a source host blocks hotlinking, you’ll see the **placeholder**—try another image URL or host locally.

**Progress not at 100%**  
- Typically one pair is missing a normalized URL or a price (try focusing the field and tabbing out).  
- The app now normalizes while typing, but the field’s **visible text** updates on blur.

**Autosave not persisting**  
- Some browsers in private mode restrict `localStorage`. Serve via `http://localhost` for consistency.

**“Left/Right” wording**  
- The live UI uses **item names**, not “Left/Right.” The side identifiers only exist in code/state.

---

## 🔒 Privacy

- Data is stored **locally** in the browser’s `localStorage` under the key `value-comparisons.v6`.
- Nothing is sent to a server unless you deploy behind one.

---

## 🗺️ Roadmap (nice-to-haves)

- **Enter‑to‑advance** on the final valid field
- **Quick pair jump** (dropdown) for review
- **Open link** icon next to each URL (only when valid)
- **Hover zoom** on images for closer inspection
- **Import/Export JSON** for backups or cross‑device transfer

---

## 🧾 Changelog (recent)

- **No-crop image sizing** via fixed frame + `max-*` (unified card heights, no distortion)
- **Final CTA** becomes **“Save & Summary”** on the last pair; navbar hidden on Summary
- **Smart URL** validates **while typing**; normalizes on blur
- **Price inputs** stay blank unless digits were typed (no auto `0.00`)
- **Progress** shows **Completed X of N** and reaches **100%** when all pairs validate
- **State model** keyed by pair ID; **Previous** never clears future answers
- **Sticky footer** and reserved input space prevent layout jumps

---

## 🛠️ Development Notes

- Single‑file app; no build step required.
- Tested in current Chrome/Edge/Safari/Firefox. For the smoothest results in class, use updated Chrome/Edge.
- If you change the state schema, **bump the `STORAGE_KEY`** (e.g., `value-comparisons.v7`) to avoid loading older incompatible saves.

---

## 📄 License

Choose a license for your classroom/school context (e.g., MIT).  
Add `LICENSE` to the repo, then reference it here.

---

## 🙌 Credits

- Activity concept and content: **Matt Swymer**  
- UI/UX and engineering: This repo’s single‑file app
