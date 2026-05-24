# Codex Instructions

This repository is a Jekyll/Chirpy GitHub Pages blog.

## Required Context

Before editing, read `docs/BLOG_GUIDE.md` to understand the blog structure and working rules.

## Repository Rules

- Repo root: `C:\Dev\01_Project\Tech-Blog\kang-hyesung.github.io`
- Blog posts are under `_posts/`.
- When asked to write or edit a blog post, follow the Markdown style and Chirpy syntax examples in `_posts/blogging/github_blogging/2019-08-08-text-and-typography.md`.

## Front Matter Rules

- `categories` in front matter are split by spaces into category depth.
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

## Lecture-To-Blog Workflow

When the user sends a lecture script and asks to turn it into a blog post, use these references:

- Lecture project files: `C:\Dev\99_Resources\Lecture-Files\독하게 시작하는 Java - Part 3\독하게 시작하는 Java - Part 3 - 상`
- Lecture PDF: `C:\Dev\99_Resources\Lecture-Files\독하게 시작하는 Java - Part 3\독하게 시작하는 Java 프로그래밍 - Part 3 상 v1.0.pdf`

Do not simply transcribe the lecture script. Convert it into a structured technical article with explanations, examples, and section headings where useful. Preserve the lecture's intent, but improve readability for blog readers. Organize the result in a style consistent with existing blog posts.
