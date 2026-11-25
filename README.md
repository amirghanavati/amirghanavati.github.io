# Go Knowledge Base

Build locally with `zensical build --clean --strict`. Run `zensical serve` to preview the site.

Add each syllabus topic as a Markdown page under both `docs/en/` and `docs/fa/`, using the same relative path and a localized title. Add the pages to the matching language sections in `zensical.toml` in syllabus order.

Each topic starts as `TODO`. To update both language versions, set its relative path without `.md` in `[project.extra.documentation_status]` to `todo`, `in_progress`, or `done`. Record any course learning percentages under `[project.extra.learning_progress]` using the same topic path and a number from 0 to 100.
