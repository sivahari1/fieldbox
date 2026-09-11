# Fieldbox

A browser-based image annotation tool: draw bounding boxes, label them by class, and export a dataset in COCO, YOLO, or Pascal VOC format.

Built for annotating potholes, trees, and plants, but the class list is fully editable, so it works for any bounding-box annotation task.

**Live tool:** https://sivahari1.github.io/fieldbox/

## Features

- Upload any number of images (drag-and-drop or file picker) — JPEG, PNG, WebP, GIF, BMP
- Draw a box by dragging on the image; move it by dragging the body, resize with the corner handles, nudge with arrow keys
- Manage classes: add, rename, recolor, delete — defaults are Pothole, Tree, Plant
- Assign a box's class by clicking a class in the sidebar, or with number keys 1–9
- Navigate images with the arrow buttons or N / P, zoom with + / − / 0
- Export the full dataset as:
  - **COCO** — single `.json` file
  - **YOLO** — `.zip` with a `.txt` per image, `classes.txt`, and `data.yaml`
  - **Pascal VOC** — `.zip` with a `.xml` per image

## How data is stored

Images and annotations are saved in the browser's local storage (IndexedDB) on whichever device opens the page. That means work is kept between sessions on the same browser and device, but does not sync across devices or browsers on its own. Exporting is the durable, portable copy of your work — worth doing regularly, since browser storage can be cleared.

The page opens with one sample image (a simple illustration, clearly marked "Sample") with a few example boxes, so it's usable right away. Delete it once you add your own images.

## Global usage panel (optional)

The sidebar can show a live "Global usage" panel — total visits, images annotated, and boxes drawn, counted across everyone who opens the page, updating in real time. It's off by default. To turn it on:

1. Go to [console.firebase.google.com](https://console.firebase.google.com) and create a project (free Spark plan, no credit card).
2. In the project, click the `</>` (Web) icon to register a web app. Copy the `firebaseConfig` object it shows you.
3. In the left menu, open **Firestore Database → Create database**. Pick any region, start in production mode.
4. In the **Rules** tab, replace the default rules with:
   ```
   rules_version = '2';
   service cloud.firestore {
     match /databases/{database}/documents {
       match /stats/global {
         allow read: if true;
         allow create, delete: if false;
         allow update: if
           request.resource.data.keys().hasOnly(['visits','imagesAnnotated','boxesDrawn']) &&
           request.resource.data.visits is number &&
           request.resource.data.imagesAnnotated is number &&
           request.resource.data.boxesDrawn is number &&
           request.resource.data.visits >= resource.data.visits &&
           request.resource.data.imagesAnnotated >= resource.data.imagesAnnotated &&
           request.resource.data.boxesDrawn >= resource.data.boxesDrawn &&
           request.resource.data.visits <= resource.data.visits + 1 &&
           request.resource.data.imagesAnnotated <= resource.data.imagesAnnotated + 1 &&
           request.resource.data.boxesDrawn <= resource.data.boxesDrawn + 1;
       }
       match /{document=**} { allow read, write: if false; }
     }
   }
   ```
   This lets any visitor's browser bump the three counters by exactly one at a time and read them, but nothing else in the database — no one can overwrite, spike, or delete the numbers.
5. Back in the **Data** tab, manually create one document: collection `stats`, document ID `global`, with three number fields all set to `0`: `visits`, `imagesAnnotated`, `boxesDrawn`.
6. In `index.html`, find the `firebaseConfig` object near the top of the script and paste in the six values from step 2.
7. Push the change. The panel appears automatically once it can reach Firestore — no code changes beyond the config.

Counting logic: a "visit" is counted once per page load; a "box drawn" is counted every time a labeled box is confirmed; an "image annotated" is counted once per image, the first time it receives a box (deleting and redrawing its only box won't double-count it). There's no login, so these are site-wide totals rather than a per-person breakdown.

## Running locally

It's a single self-contained HTML file with no build step. Open `index.html` directly in a browser, or serve the folder with any static file server:

```bash
python3 -m http.server 8000
```

## Tech

Plain HTML, CSS, and JavaScript. No framework. Uses [JSZip](https://stuk.github.io/jszip/) (loaded from a CDN) to bundle the YOLO and Pascal VOC exports.
