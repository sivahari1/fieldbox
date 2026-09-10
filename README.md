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

## Running locally

It's a single self-contained HTML file with no build step. Open `index.html` directly in a browser, or serve the folder with any static file server:

```bash
python3 -m http.server 8000
```

## Tech

Plain HTML, CSS, and JavaScript. No framework. Uses [JSZip](https://stuk.github.io/jszip/) (loaded from a CDN) to bundle the YOLO and Pascal VOC exports.
"# fieldbox" 
