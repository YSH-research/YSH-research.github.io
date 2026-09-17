# **Projects — Research**

Research-driven project work: implementations, benchmarks, and experiments tied to my autonomous driving research.

---

## **Projects**

<div class="project-grid" markdown>

<div class="project-card" markdown>
<div class="project-card__thumb project-card__thumb--empty">No image</div>
<div class="project-card__body" markdown>

<div class="project-card__title">Project title goes here</div>

<div class="project-tags"><span class="tag">Tag</span><span class="tag">Tag</span></div>

</div>
</div>

<div class="project-card" markdown>
<div class="project-card__thumb project-card__thumb--empty">No image</div>
<div class="project-card__body" markdown>

<div class="project-card__title">Project title goes here</div>

<div class="project-tags"><span class="tag">Tag</span><span class="tag">Tag</span></div>

</div>
</div>

<div class="project-card" markdown>
<div class="project-card__thumb project-card__thumb--empty">No image</div>
<div class="project-card__body" markdown>

<div class="project-card__title">Project title goes here</div>

<div class="project-tags"><span class="tag">Tag</span><span class="tag">Tag</span></div>

</div>
</div>

</div>

---

## **How to add a project card**

Each project gets its own folder next to this file, following the `folder/folder.md` convention:

```text
Docs/Project_Research/
  Projects_Research.md
  My_Project/
    My_Project.md
    Thumb.png
```

Then copy one block into the grid above and edit three things — the image, the title link, and the tags:

```html
<div class="project-card" markdown>
![](My_Project/Thumb.png){ .project-card__thumb }
<div class="project-card__body" markdown>

[My Project Title](My_Project/My_Project.md){ .project-card__title }

<div class="project-tags"><span class="tag">CARLA</span><span class="tag">Diffusion</span></div>

</div>
</div>
```

Notes:

- **Image is optional.** With no image yet, keep `<div class="project-card__thumb project-card__thumb--empty">No image</div>` instead of the `![]()` line. Thumbnails are cropped to 16:9, so roughly 640×360 or larger works best.
- **Title is optional as a link.** Until the detail page exists, use `<div class="project-card__title">Title</div>`; once it exists, swap in the markdown link so the whole card becomes clickable.
- **Tags are free text** — one `<span class="tag">` per tag, as many as fit.
