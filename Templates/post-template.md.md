<%*
const title = await tp.system.prompt("노트 제목을 입력하세요");
const date  = tp.date.now("YYYY-MM-DD");
const slug  = title.trim().replace(/\s+/g, "-").toLowerCase();

// 파일명만 변경: 경로 없이
await tp.file.rename(`${date}-${slug}`);

const datetime = tp.date.now("YYYY-MM-DD HH:mm ZZ");
tR += `---
title: "${title}"
date: ${datetime}
author: "hyesung"
description: "설명"
mermaid : "true"
---

`;
%>
