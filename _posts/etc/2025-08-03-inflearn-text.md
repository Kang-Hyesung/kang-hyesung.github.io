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


```javascript
(function() {
    // --- 설정 ---
    const CONFIG = {
        rowSelector: 'div[data-index]',
        timestampSelector: '.mantine-Badge-inner',
        textSelector: '.mantine-Text-root',
        scrollStep: 300,    // 한 번에 스크롤할 높이
        scrollInterval: 100 // 스크롤 속도 (ms)
    };

    let collectedLines = new Map();
    let observer;
    let autoScrollInterval;
    let statusDiv;

    // 1. 스크롤 가능한 부모 요소 찾기 (핵심 수정 사항)
    function getScrollParent(node) {
        if (node == null) return null;
        if (node.scrollHeight > node.clientHeight) {
            return node;
        } else {
            return getScrollParent(node.parentNode);
        }
    }

    // 2. 초기화 및 타겟 선택 유도
    function init() {
        alert("🎯 [타겟 지정 모드]\n\n확인을 누른 뒤, 스크롤이 되어야 하는 '본문 영역'을 마우스로 클릭해주세요.\n클릭하면 자동으로 수집이 시작됩니다.");
        
        document.body.style.cursor = "crosshair"; // 커서 변경
        
        const clickHandler = (e) => {
            e.preventDefault();
            e.stopPropagation();

            // 클릭된 요소에서 가장 가까운 스크롤 영역 찾기
            const targetScrollContainer = getScrollParent(e.target);

            if (!targetScrollContainer) {
                alert("⚠️ 스크롤 가능한 영역을 감지하지 못했습니다. 다시 시도하거나 더 넓은 영역을 클릭해보세요.");
                return;
            }

            // 이벤트 제거 및 커서 복구
            document.removeEventListener('click', clickHandler, true);
            document.body.style.cursor = "default";

            // 수집 시작
            startScraping(targetScrollContainer);
        };

        document.addEventListener('click', clickHandler, true);
    }

    // 3. UI 생성
    function createStatusUI() {
        if (document.getElementById('script-scraper-ui')) return;
        
        statusDiv = document.createElement('div');
        statusDiv.id = 'script-scraper-ui';
        Object.assign(statusDiv.style, {
            position: 'fixed', bottom: '20px', right: '20px', zIndex: '9999',
            backgroundColor: '#222', color: '#fff', padding: '15px',
            borderRadius: '8px', boxShadow: '0 4px 12px rgba(0,0,0,0.3)',
            fontFamily: 'sans-serif', fontSize: '14px', minWidth: '200px'
        });
        statusDiv.innerHTML = `
            <div style="margin-bottom:10px; font-weight:bold;">📜 스크립트 수집기</div>
            <div id="scraper-count">수집된 라인: 0개</div>
            <div id="scraper-status" style="color:#4ade80; margin-bottom:10px;">자동 스크롤 중...</div>
            <button id="scraper-stop-btn" style="cursor:pointer; background:#e11d48; color:white; border:none; padding:5px 10px; border-radius:4px; width:100%;">중지 및 복사</button>
        `;
        document.body.appendChild(statusDiv);
        document.getElementById('scraper-stop-btn').addEventListener('click', stopAndCopy);
    }

    // 4. 라인 처리
    function processLine(line) {
        const index = parseInt(line.dataset.index, 10);
        if (isNaN(index) || collectedLines.has(index)) return;

        const timestampEl = line.querySelector(CONFIG.timestampSelector);
        const scriptTextEl = line.querySelector(CONFIG.textSelector);
        
        const timestamp = timestampEl ? timestampEl.innerText.trim() : '';
        const scriptText = scriptTextEl ? scriptTextEl.innerText.trim() : '';
        
        collectedLines.set(index, `${timestamp}\t${scriptText}`);
        
        const countEl = document.getElementById('scraper-count');
        if (countEl) countEl.innerText = `수집된 라인: ${collectedLines.size}개`;
    }

    // 5. 실제 수집 및 스크롤 로직
    function startScraping(scrollContainer) {
        console.log("✅ 타겟 설정 완료:", scrollContainer);
        createStatusUI();

        // DOM 감지 설정
        observer = new MutationObserver((mutations) => {
            for (const mutation of mutations) {
                mutation.addedNodes.forEach(node => {
                    if (node.nodeType === 1) {
                        if (node.matches && node.matches(CONFIG.rowSelector)) processLine(node);
                        if (node.querySelectorAll) node.querySelectorAll(CONFIG.rowSelector).forEach(processLine);
                    }
                });
            }
        });
        
        // 화면에 이미 있는 것들 먼저 수집
        scrollContainer.querySelectorAll(CONFIG.rowSelector).forEach(processLine);
        
        // 관찰 시작 (상위 요소 관찰)
        observer.observe(scrollContainer, { childList: true, subtree: true });

        // 자동 스크롤 시작
        let lastScrollTop = scrollContainer.scrollTop;
        let stuckCount = 0;

        autoScrollInterval = setInterval(() => {
            scrollContainer.scrollBy(0, CONFIG.scrollStep); // 상대적 스크롤

            // 스크롤이 더 이상 안내려가는지 확인
            // (오차 범위 2px 허용 - 고해상도 화면 대응)
            if (Math.abs(scrollContainer.scrollTop - lastScrollTop) < 2) {
                stuckCount++;
                if (stuckCount > 15) { // 약 1.5초 동안 멈춰있으면 끝난 것으로 간주
                    stopAndCopy();
                }
            } else {
                lastScrollTop = scrollContainer.scrollTop;
                stuckCount = 0;
            }
        }, CONFIG.scrollInterval);
    }

    // 6. 종료 및 복사
    async function stopAndCopy() {
        if (observer) observer.disconnect();
        if (autoScrollInterval) clearInterval(autoScrollInterval);

        const statusText = document.getElementById('scraper-status');
        if (statusText) statusText.innerText = "완료! 복사 중...";

        const sortedKeys = Array.from(collectedLines.keys()).sort((a, b) => a - b);
        const fullScript = sortedKeys.map(key => collectedLines.get(key)).join('\n');

        try {
            await navigator.clipboard.writeText(fullScript);
            alert(`✅ 수집 완료! 총 ${collectedLines.size}줄이 복사되었습니다.`);
        } catch (err) {
            const textArea = document.createElement("textarea");
            textArea.value = fullScript;
            document.body.appendChild(textArea);
            textArea.select();
            document.execCommand("copy");
            document.body.removeChild(textArea);
            alert(`✅ (백업 방식) 총 ${collectedLines.size}줄 복사 완료!`);
        }
        
        if (statusDiv) statusDiv.remove();
    }

    // 실행
    init();
})();
```