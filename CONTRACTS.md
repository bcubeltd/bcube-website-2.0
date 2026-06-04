# CONTRACTS.md — Career Page Refactor

## Goal
Refactor `career.html` so that:
1. Each job listing lives in its own HTML fragment file under `jobs/`
2. `career.html` has a single JS config array where jobs are enabled/disabled with one flag
3. No more hunting for commented-out `<div>` blocks — toggling a job is one character change

---

## File Structure to Create

```
careers/
├── career.html              ← refactored (modify in place)
└── jobs/
    ├── fpga-firmware.html
    ├── junior-hardware.html
    └── hardware-engineer.html
```

---

## Task 1 — Create `jobs/` directory and job fragment files

Create the following files. Each file contains **only** the inner `<div class="icon-box ...">` content — no wrapper divs, no Bootstrap grid columns. The loader in `career.html` will wrap each fragment in the grid column div automatically.

### `jobs/fpga-firmware.html`
```html
<h4>Design Engineer FPGA/PL Firmware</h4>
<p><b>Qualification:</b> 4 year Degree in Electrical/Computer Engineering or relevant field</p>
<p><b>Experience:</b> 4 to 8 years</p>
<br>
<a href="jdsrfw.html" class="btn btn-apply">Apply Now</a>
```

### `jobs/junior-hardware.html`
```html
<h4>Junior Design Engineer Hardware</h4>
<p><b>Qualification:</b> 4 year Degree in Electrical/Computer Engineering or relevant field</p>
<p><b>Experience:</b> Fresh to 1 year</p>
<br>
<a href="jdh.html" class="btn btn-apply">Apply Now</a>
```

### `jobs/hardware-engineer.html`
```html
<h4>Design Engineer Hardware</h4>
<p><b>Qualification:</b> 4 year Degree in Electrical/Computer Engineering or relevant field</p>
<p><b>Experience:</b> 2 to 3 years</p>
<br>
<a href="hd.html" class="btn btn-apply">Apply Now</a>
```

> All future job descriptions follow the same pattern. Add a new file to `jobs/` and one line to the config array in `career.html`.

---

## Task 2 — Refactor `career.html`

Replace the entire `<!-- Embedded Department Careers -->` section and everything inside `<div id="services" class="services">` with the following structure.

### 2a — Replace the static job rows with a single mount point

Find this block:
```html
<!-- Embedded Department Careers-->
<div class="row">
  ... (all the col-lg-4 divs) ...
</div>
<br>
```

Replace with:
```html
<!-- =====================================================
     JOB LISTINGS
     To enable/disable a role: set enabled: true / false
     To add a new role: add an entry + create jobs/<slug>.html
     ===================================================== -->
<div id="job-listings" class="row"></div>
```

### 2b — Add the loader script just before `</body>`

Insert this block immediately before the closing `</body>` tag (after all vendor JS includes):

```html
<script>
  // ─── JOB CONFIGURATION ──────────────────────────────────────────────────────
  // Toggle a job on/off by flipping `enabled`. Order here = order on the page.
  const JOBS = [
    { slug: "fpga-firmware",     enabled: true  },
    { slug: "junior-hardware",   enabled: true  },
    { slug: "hardware-engineer", enabled: true  },

    // Examples — currently disabled. Set enabled: true when ready to post.
    // { slug: "junior-it-admin",          enabled: false },
    // { slug: "production-engineer",      enabled: false },
    // { slug: "mech-design-engineer",     enabled: false },
    // { slug: "junior-rf-microwave",      enabled: false },
    // { slug: "rf-microwave-engineer",    enabled: false },
    // { slug: "senior-rf-microwave",      enabled: false },
    // { slug: "embedded-software",        enabled: false },
  ];
  // ────────────────────────────────────────────────────────────────────────────

  const mount = document.getElementById("job-listings");

  const activeJobs = JOBS.filter(j => j.enabled);

  if (activeJobs.length === 0) {
    mount.innerHTML = `
      <div class="poster row">
        <h1 class="display-5">Currently, We Don't Have Any Job Openings</h1>
        <p class="lead">Stay tuned and keep visiting our website for future openings.</p>
        <p><a href="https://www.bcubelimited.com" class="website" target="_blank">Visit our website</a></p>
      </div>`;
  } else {
    activeJobs.forEach(job => {
      fetch(`jobs/${job.slug}.html`)
        .then(r => {
          if (!r.ok) throw new Error(`Fragment not found: jobs/${job.slug}.html`);
          return r.text();
        })
        .then(html => {
          const col = document.createElement("div");
          col.className = "col-lg-4 col-md-6 d-flex align-items-stretch";
          col.setAttribute("data-aos", "zoom-in");
          col.setAttribute("data-aos-delay", "100");
          col.innerHTML = `<div class="icon-box iconbox-blue">${html}</div>`;
          mount.appendChild(col);
        })
        .catch(err => console.error(err));
    });
  }
</script>
```

---

## Task 3 — Clean up `career.html`

Remove all commented-out job listing blocks (the large `<!-- ... -->` sections containing old role divs). They are now superseded by the `jobs/` files. This reduces `career.html` from ~300 lines to under 120.

---

## Task 4 — Remove the now-redundant static "no openings" poster div

The following block in `career.html` can be deleted — the JS loader handles the empty state automatically:

```html
<!-- <div class="poster row">
  <h1 class="display-5">
    Currently, We Don't Have Any Job Openings
  ...
</div> -->
```

---

## Workflow After Refactor

### Disable a job
```js
{ slug: "hardware-engineer", enabled: false },
```

### Enable a job
```js
{ slug: "hardware-engineer", enabled: true },
```

### Add a brand-new role
1. Create `jobs/new-role-slug.html` with the JD content
2. Add one line to the JOBS array:
   ```js
   { slug: "new-role-slug", enabled: true },
   ```

---

## Notes

- `fetch()` requires the page to be served over HTTP/HTTPS (not `file://`). If testing locally without a server, run `python3 -m http.server` in the project root.
- Fragment files contain no `<html>`, `<head>`, or `<body>` tags — just the inner card content.
- The "no openings" fallback triggers automatically when all jobs have `enabled: false`.
