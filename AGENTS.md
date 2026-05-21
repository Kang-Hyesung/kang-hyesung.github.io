# Codex Instructions

This repository is a Jekyll/Chirpy GitHub Pages blog.

Before editing, read `docs/BLOG_GUIDE.md` to understand the blog structure and working rules.

- Repo root: `C:\Dev\01_Project\Tech-Blog\kang-hyesung.github.io`
- Blog posts are under `_posts/`.
- `categories` in front matter are split by spaces into category depth.
- Mermaid posts should use `mermaid: true` in front matter.
- The user runs the local blog with Docker Desktop and checks it at `http://127.0.0.1:4000`.
- Do not run local build verification unless the user explicitly asks for it.
- Before making changes, run `git status -sb` and do not revert user changes.
- Before finishing, run at least `git diff --check`.
- When asked to write or edit a blog post, follow the Markdown style and Chirpy syntax examples in `_posts/blogging/github_blogging/2019-08-08-text-and-typography.md`.
- When the user sends a lecture script and asks to turn it into a blog post, use the lecture project files under `C:\Dev\99_Resources\Lecture-Files\독하게 시작하는 Java - Part 3\독하게 시작하는 Java - Part 3 - 상` and the lecture PDF `C:\Dev\99_Resources\Lecture-Files\독하게 시작하는 Java - Part 3\독하게 시작하는 Java 프로그래밍 - Part 3 상 v1.0.pdf` as supporting reference material. Organize the result in a style consistent with the user's existing blog posts.
