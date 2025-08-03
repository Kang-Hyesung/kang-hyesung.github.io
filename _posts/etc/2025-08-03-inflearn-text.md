---
title: 텍스트 복사
date: 2025-08-03 17:44 +0900
author: hyesung
---
```javascript
// --- 스크립트 수집 도우미 코드 ---

let collectedLines = new Map();
let observer;

// DOM 변경을 감지하고 데이터를 수집하는 함수
function startObserver() {
    const scrollContainer = document.querySelector('.mantine-ScrollArea-viewport');
    const targetNode = scrollContainer ? scrollContainer.querySelector('div[style*="display: table"]') : null;

    if (!targetNode) {
        alert("감시할 대상 노드를 찾지 못했습니다. 페이지 구조가 변경되었을 수 있습니다.");
        return;
    }
    
    console.clear();
    console.log("✅ 스크립트 수집기가 활성화되었습니다.");
    console.log("   이제 페이지의 스크립트 창을 마우스로 천천히 '끝까지' 스크롤하세요.");
    console.log("   스크롤하는 동안 데이터가 자동으로 수집됩니다.");
    console.log("   스크롤이 끝나면 2단계 안내에 따라주세요.");

    // 이미 화면에 있는 내용 먼저 수집
    targetNode.querySelectorAll('div[data-index]').forEach(line => processLine(line));

    // DOM 변경 감지 설정
    const config = { childList: true, subtree: true };

    const callback = (mutationsList) => {
        for(const mutation of mutationsList) {
            mutation.addedNodes.forEach(node => {
                if (node.nodeType === 1) { // Element node
                    if (node.matches('div[data-index]')) processLine(node);
                    node.querySelectorAll('div[data-index]').forEach(line => processLine(line));
                }
            });
        }
    };

    observer = new MutationObserver(callback);
    observer.observe(targetNode, config);
}

// 라인 처리 및 데이터 저장 함수
function processLine(line) {
    const index = line.dataset.index;
    if (!collectedLines.has(index)) {
        const timestampEl = line.querySelector('.mantine-Badge-inner');
        const scriptTextEl = line.querySelector('.mantine-Text-root');
        
        const timestamp = timestampEl ? timestampEl.innerText.trim() : '';
        const scriptText = scriptTextEl ? scriptTextEl.innerText.trim() : '';
        
        collectedLines.set(index, `${timestamp}\t${scriptText}`);
        console.log(`수집됨 (${collectedLines.size}개)`);
    }
}

// 수집기 시작
startObserver();
```

```javascript
// 2단계에서 사용할 결과 복사 함수
function copyResults() {
    if (observer) observer.disconnect(); // 관찰 중지

    if (collectedLines.size === 0) {
        alert("수집된 내용이 없습니다. 페이지를 스크롤한 후 다시 시도해주세요.");
        return;
    }

    console.log("--------------------------------------------------");
    console.log(`✅ 최종 수집 완료! 총 ${collectedLines.size}개의 라인을 복사합니다.`);

    const sortedKeys = Array.from(collectedLines.keys()).sort((a, b) => parseInt(a, 10) - parseInt(b, 10));
    const fullScript = sortedKeys.map(key => collectedLines.get(key)).join('\n');

    console.log(fullScript); // 최종 결과물을 콘솔에 출력

    try {
        navigator.clipboard.writeText(fullScript);
        alert(`✅ 전체 스크립트가 클립보드에 복사되었습니다! (총 ${collectedLines.size} 라인)`);
    } catch (err) {
        alert('클립보드 복사는 실패했지만, 콘솔에 전체 내용이 출력되었습니다.');
    }
}
```

`copyResults();`