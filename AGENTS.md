# Codex Instructions

This repository is a Jekyll/Chirpy GitHub Pages blog.

## Required Context

Before editing, read `docs/BLOG_GUIDE.md` to understand the blog structure and working rules.

## Repository Rules

- Repo root: `C:\Dev\01_Project\Tech-Blog\kang-hyesung.github.io`
- Blog posts are under `_posts/`.
- For serialized technical posts, keep the two-digit sequence number in both the category folder and post filename when the surrounding series uses it, for example `01-basic-syntax` and `2025-11-11-01-title.md`.
- When asked to write or edit a blog post, follow the Markdown style and Chirpy syntax examples in `_posts/blogging/github_blogging/2019-08-08-text-and-typography.md`.

## Front Matter Rules

- `categories` in front matter are split by spaces into category depth.
- Use `tags` for broad reusable topics, not one-off index keywords.
- Prefer Korean tag names for Korean posts, but keep common technical abbreviations such as `JVM` and `GC`.
- Tags may contain spaces, for example `기본 문법`, `내부 클래스`, `타입 변환`, and `클래스 로딩`.
- Mermaid posts must include `mermaid: true` in front matter.

## Workflow Rules

- Before making changes, run `git status -sb`.
- Do not revert user changes.
- Do not run local build verification unless the user explicitly asks for it.
- Before finishing, run at least `git diff --check`.

## Local Preview

The user runs the local blog with Docker Desktop and checks it at:

`http://127.0.0.1:4000`

## Writing Style

- Write posts in Korean unless the user asks otherwise.
- Prefer clear technical explanations with runnable examples.
- Keep headings structured and consistent with existing posts.
- Use code blocks with language tags.

## Mermaid Diagram Style

- Design Mermaid diagrams for quick visual scanning rather than placing full explanations or formulas inside a single large node.
- Give each distinct concept or data component its own node. For composition diagrams, separate items such as headers, payloads, packets, segments, and trailers instead of joining them with `+` inside one box.
- Put transformations such as `헤더 추가`, `헤더 제거`, or `페이로드` on edges, and keep node labels to a short name plus a brief role when needed.
- Choose the overall direction to match the concept. Use a horizontal layout for sender-to-medium-to-receiver flows, while keeping each sender or receiver stack vertical inside its own subgraph.
- Use subgraphs for meaningful boundaries such as sender and receiver, and reverse the receiver stack when that makes encapsulation and decapsulation visually symmetrical.
- Use consistent colors for the same layer or role across a diagram. Make the main conceptual path visually stronger than secondary component branches when helpful.
- Add explicit `<br/>` breaks to long labels so Mermaid does not wrap Korean text one character at a time. Avoid diagrams that become an unnecessarily tall single column when a balanced multi-column layout communicates the relationship better.
- Check that the diagram remains understandable in Chirpy's dark and light modes and at both desktop and mobile widths.

## Lecture-To-Blog Workflow

When the user sends a lecture script and asks to turn it into a blog post, use these references:

- Lecture project files: `C:\Dev\02_Study\Network_inflearn`
- Lecture PDF: `C:\Dev\02_Study\Network_inflearn\섹션 1-네트워크 기초 개념 PDF.pdf`

Lecture scripts may arrive one lecture at a time in the form `섹션 1의 4. 강의 제목` followed by a timestamped transcript. Treat the section, rather than each individual lecture, as the editorial unit.

- Before writing, identify the matching section and read every existing post in that section.
- Do not assume that one lecture must become one post.
- Merge the new lecture into an existing post when it contributes to the same central reader question, continues the same conceptual flow, or would be too thin as a standalone article.
- Create a new post when the lecture addresses an independent question, has distinct search intent, or needs enough explanation, examples, calculations, or diagrams to stand on its own.
- When merging, reorganize the existing article so that it reads as one coherent post. Do not simply append a transcript or create visible lecture-by-lecture fragments.
- Supplement the lecture with relevant concepts, operational caveats, troubleshooting knowledge, commands, or examples that are widely used in real-world development or infrastructure work, even when the lecture does not mention them.
- Keep practical additions directly connected to the article's central topic, distinguish them naturally as real-world context, and avoid unrelated tangents or padding added only to increase length.
- Inspect the relevant lecture PDF pages when a diagram, screenshot, or other visual could materially improve the explanation.
- Prefer recreating simple slide diagrams as original Mermaid diagrams, tables, or text-based explanations instead of copying lecture slide screenshots into the public blog.
- Reuse an image from the lecture PDF only when the visual cannot be reproduced effectively, the user has the right to publish it, and it adds substantial explanatory value. Crop it to the relevant content, optimize it for the web, store it under a descriptive `assets/img/` path, and provide meaningful alt text and source attribution when appropriate.
- Do not publish copyrighted paid-course slides or watermarked lecture images without permission. When rights are unclear, create an original visual that communicates the same underlying concept.
- Check recreated visuals for readability in Chirpy's light and dark modes and at mobile widths. Avoid adding a visual when the existing prose, table, or diagram already communicates the same information clearly.
- Preserve the section's folder, filename numbering, front matter, category, and tag conventions when creating or updating posts.
- After handling each script, report whether it was merged into an existing post or used to create a new post, along with the reason and changed file path.
- When the user says that a section is complete, review every post in that section for overlap, missing explanations, unbalanced length, title quality, ordering, categories, and tags, then make the final editorial adjustments.

Do not simply transcribe the lecture script. Convert it into a structured technical article with explanations, examples, and section headings where useful. Preserve the lecture's intent, but improve readability for blog readers. Organize the result in a style consistent with existing blog posts.
