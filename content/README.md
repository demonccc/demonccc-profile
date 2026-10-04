# Content

Canonical professional content lives here using one structure for every item, regardless of whether it is classified as a post, article, note, talk or something else.

```text
<date>-<slug>/
├── content.<language-code>.md
└── assets/
    └── [optional files]
```

Language codes and content classifications are defined by this profile in [`../settings.yaml`](../settings.yaml).

The date is derived from the bundle directory and the language from the Markdown filename. They are not repeated in metadata.

When structured metadata is needed, YAML front matter is the first block in the Markdown file so standard tooling can parse it directly.

Relationships between content items use `related_content` and should point to the content bundle directory unless the relationship is explicitly language-specific.

External platforms such as LinkedIn or Medium are publication channels. This repository keeps the owned, versioned source of the ideas themselves.
