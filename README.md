```html
<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>스마트 답안지 생성기</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Noto+Sans+KR:wght@300;400;500;700&display=swap');
        
        body {
            font-family: 'Noto Sans KR', sans-serif;
            background-color: #f3f4f6;
        }

        @media print {
            .no-print {
                display: none !important;
            }
            body {
                background-color: white;
            }
            .print-area {
                box-shadow: none !important;
                margin: 0 !important;
                padding: 0 !important;
                width: 100% !important;
            }
            .page-break {
                page-break-after: always;
            }
        }

        .ox-btn {
            width: 2.5rem;
            height: 2.5rem;
            border: 1px solid #d1d5db;
            display: flex;
            align-items: center;
            justify-content: center;
            border-radius: 50%;
            cursor: pointer;
            transition: all 0.2s;
        }

        .ox-btn.selected-o {
            background-color: #ef4444;
            color: white;
            border-color: #ef4444;
        }

        .ox-btn.selected-x {
            background-color: #3b82f6;
            color: white;
            border-color: #3b82f6;
        }

        .mc-btn {
            width: 2rem;
            height: 2rem;
            border: 1px solid #d1d5db;
            display: flex;
            align-items: center;
            justify-content: center;
            border-radius: 50%;
            cursor: pointer;
            font-size: 0.875rem;
            transition: all 0.2s;
        }

        .mc-btn.selected {
            background-color: #1f2937;
            color: white;
            border-color: #1f2937;
        }

        /* 줄 눈금 효과 */
        .line-guide {
            background-image: linear-gradient(#e5e7eb 1px, transparent 1px);
            background-size: 100% 1.5rem;
        }
    </style>
</head>
<body class="p-4 md:p-8">

    <!-- 설정 패널 (인쇄 시 숨김) -->
    <div class="max-w-4xl mx-auto bg-white rounded-xl shadow-lg p-6 mb-8 no-print">
        <h1 class="text-2xl font-bold mb-6 text-gray-800 flex items-center gap-2">
            📝 답안지 설정 및 생성
        </h1>
        
        <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-4 gap-4 mb-6">
            <div class="space-y-2">
                <label class="block text-sm font-medium text-gray-700">시험 명칭</label>
                <input type="text" id="examTitle" value="통합 테스트 답안지" class="w-full border p-2 rounded focus:ring-2 focus:ring-blue-500 outline-none">
            </div>
            <div class="space-y-2">
                <label class="block text-sm font-medium text-gray-700">과목명</label>
                <input type="text" id="subjectName" value="정기 고사" class="w-full border p-2 rounded focus:ring-2 focus:ring-blue-500 outline-none">
            </div>
            <div class="space-y-2">
                <label class="block text-sm font-medium text-gray-700">이름</label>
                <input type="text" placeholder="성함 기입란" class="w-full border p-2 rounded bg-gray-50" disabled>
            </div>
            <div class="space-y-2">
                <label class="block text-sm font-medium text-gray-700">학번/번호</label>
                <input type="text" placeholder="번호 기입란" class="w-full border p-2 rounded bg-gray-50" disabled>
            </div>
        </div>

        <hr class="my-6">

        <h2 class="font-bold text-gray-700 mb-4">문항 수 설정</h2>
        <div class="grid grid-cols-2 md:grid-cols-3 lg:grid-cols-5 gap-4 mb-8">
            <div class="p-3 border rounded-lg bg-gray-50">
                <label class="text-[10px] font-bold text-gray-500 block uppercase">OX 문항</label>
                <input type="number" id="countOX" value="5" min="0" class="w-full text-lg font-bold bg-transparent outline-none">
            </div>
            <div class="p-3 border rounded-lg bg-gray-50">
                <label class="text-[10px] font-bold text-gray-500 block uppercase">객관식 (5지)</label>
                <input type="number" id="countMC" value="5" min="0" class="w-full text-lg font-bold bg-transparent outline-none">
            </div>
            <div class="p-3 border rounded-lg bg-gray-50">
                <label class="text-[10px] font-bold text-gray-500 block uppercase">빈칸 채우기</label>
                <input type="number" id="countFill" value="3" min="0" class="w-full text-lg font-bold bg-transparent outline-none">
            </div>
            <div class="p-3 border rounded-lg bg-blue-50 border-blue-200">
                <label class="text-[10px] font-bold text-blue-500 block uppercase">약술형 문항</label>
                <input type="number" id="countShort" value="2" min="0" class="w-full text-lg font-bold bg-transparent outline-none text-blue-700">
            </div>
            <div class="p-3 border rounded-lg bg-gray-50">
                <label class="text-[10px] font-bold text-gray-500 block uppercase">서술형 문항</label>
                <input type="number" id="countDescriptive" value="1" min="0" class="w-full text-lg font-bold bg-transparent outline-none">
            </div>
        </div>

        <div class="flex gap-4">
            <button onclick="generateSheet()" class="flex-1 bg-blue-600 text-white font-bold py-3 rounded-lg hover:bg-blue-700 transition shadow-md">
                답안지 업데이트
            </button>
            <button onclick="window.print()" class="px-8 bg-gray-800 text-white font-bold py-3 rounded-lg hover:bg-black transition flex items-center gap-2 shadow-md">
                PDF로 저장하기
            </button>
        </div>
    </div>

    <!-- 답안지 출력 영역 -->
    <div id="answerSheet" class="max-w-4xl mx-auto bg-white shadow-2xl p-10 print-area min-h-screen">
        <!-- 헤더 영역 -->
        <div class="border-b-4 border-gray-800 pb-4 mb-8">
            <div class="flex justify-between items-end">
                <div>
                    <h1 id="displayTitle" class="text-3xl font-black text-gray-900 mb-2 uppercase tracking-tighter">통합 테스트 답안지</h1>
                    <p id="displaySubject" class="text-xl text-gray-600 font-medium">과목명: 정기 고사</p>
                </div>
                <div class="flex gap-4 border-2 border-black p-4">
                    <div class="flex flex-col items-center">
                        <span class="text-[10px] font-bold border-b border-black w-12 text-center pb-1 mb-2">번호</span>
                        <div class="w-12 h-8 border border-gray-300"></div>
                    </div>
                    <div class="flex flex-col items-center">
                        <span class="text-[10px] font-bold border-b border-black w-20 text-center pb-1 mb-2">성명</span>
                        <div class="w-20 h-8 border border-gray-300"></div>
                    </div>
                    <div class="flex flex-col items-center">
                        <span class="text-[10px] font-bold border-b border-black w-12 text-center pb-1 mb-2">인</span>
                        <div class="w-12 h-8 border border-gray-300 flex items-center justify-center text-gray-200 text-[10px]">(인)</div>
                    </div>
                </div>
            </div>
        </div>

        <!-- 문항 렌더링 영역 -->
        <div id="questionsContainer" class="space-y-10">
            <!-- 자바스크립트로 생성됨 -->
        </div>

        <!-- 푸터 -->
        <div class="mt-12 pt-8 border-t border-gray-100 text-center text-gray-300 text-[10px] uppercase tracking-widest">
            Generated by Smart Answer Sheet System
        </div>
    </div>

    <script>
        function toggleOX(btn, type) {
            const parent = btn.parentElement;
            const buttons = parent.querySelectorAll('.ox-btn');
            buttons.forEach(b => {
                b.classList.remove('selected-o', 'selected-x');
            });
            if(type === 'O') btn.classList.add('selected-o');
            else btn.classList.add('selected-x');
        }

        function toggleMC(btn) {
            const parent = btn.parentElement;
            const buttons = parent.querySelectorAll('.mc-btn');
            buttons.forEach(b => b.classList.remove('selected'));
            btn.classList.add('selected');
        }

        function generateSheet() {
            const title = document.getElementById('examTitle').value;
            const subject = document.getElementById('subjectName').value;
            
            const counts = {
                ox: parseInt(document.getElementById('countOX').value) || 0,
                mc: parseInt(document.getElementById('countMC').value) || 0,
                fill: parseInt(document.getElementById('countFill').value) || 0,
                short: parseInt(document.getElementById('countShort').value) || 0,
                desc: parseInt(document.getElementById('countDescriptive').value) || 0
            };

            document.getElementById('displayTitle').innerText = title;
            document.getElementById('displaySubject').innerText = '과목명: ' + subject;

            const container = document.getElementById('questionsContainer');
            container.innerHTML = '';

            let globalIdx = 1;

            // 1. OX 문항
            if (counts.ox > 0) {
                const section = createSection("I. OX 문항", "참이면 O, 거짓이면 X에 표시하시오.");
                const grid = document.createElement('div');
                grid.className = "grid grid-cols-1 md:grid-cols-2 gap-x-12 gap-y-4";
                
                for (let i = 0; i < counts.ox; i++) {
                    const item = document.createElement('div');
                    item.className = "flex items-center justify-between border-b border-gray-100 pb-2";
                    item.innerHTML = `
                        <span class="font-bold text-gray-700">${globalIdx++}.</span>
                        <div class="flex gap-4 no-print">
                            <div class="ox-btn font-bold text-xl" onclick="toggleOX(this, 'O')">O</div>
                            <div class="ox-btn font-bold text-xl" onclick="toggleOX(this, 'X')">X</div>
                        </div>
                        <div class="hidden print:flex gap-8 px-4">
                            <span class="text-xl text-gray-200 border-2 border-gray-100 rounded-full w-8 h-8 flex items-center justify-center">O</span>
                            <span class="text-xl text-gray-200 border-2 border-gray-100 rounded-full w-8 h-8 flex items-center justify-center">X</span>
                        </div>
                    `;
                    grid.appendChild(item);
                }
                section.appendChild(grid);
                container.appendChild(section);
            }

            // 2. 객관식 문항
            if (counts.mc > 0) {
                const section = createSection("II. 객관식 문항", "가장 적절한 번호에 마킹하시오.");
                const grid = document.createElement('div');
                grid.className = "grid grid-cols-1 gap-y-3";
                
                for (let i = 0; i < counts.mc; i++) {
                    const item = document.createElement('div');
                    item.className = "flex items-center gap-4 border-b border-gray-100 pb-2";
                    let options = '';
                    for(let n=1; n<=5; n++) {
                        options += `<div class="mc-btn no-print" onclick="toggleMC(this)">${n}</div>`;
                        options += `<div class="hidden print:flex w-6 h-6 border border-gray-300 rounded-full items-center justify-center text-[10px] text-gray-400 font-bold mr-1">${n}</div>`;
                    }
                    item.innerHTML = `
                        <span class="font-bold text-gray-700 w-8">${globalIdx++}.</span>
                        <div class="flex gap-2 md:gap-4">${options}</div>
                    `;
                    grid.appendChild(item);
                }
                section.appendChild(grid);
                container.appendChild(section);
            }

            // 3. 빈칸 채우기
            if (counts.fill > 0) {
                const section = createSection("III. 단답형 / 빈칸 채우기", "질문에 알맞은 정답을 기입하시오.");
                const grid = document.createElement('div');
                grid.className = "grid grid-cols-1 md:grid-cols-2 gap-x-8 gap-y-4";
                
                for (let i = 0; i < counts.fill; i++) {
                    const item = document.createElement('div');
                    item.className = "flex items-center gap-3";
                    item.innerHTML = `
                        <span class="font-bold text-gray-700">${globalIdx++}.</span>
                        <div class="flex-1 border-b-2 border-gray-300 h-8 mt-1"></div>
                    `;
                    grid.appendChild(item);
                }
                section.appendChild(grid);
                container.appendChild(section);
            }

            // 4. 약술형 문항 (추가됨)
            if (counts.short > 0) {
                const section = createSection("IV. 약술형 문항", "핵심 내용을 요약하여 2~3문장 내외로 기술하시오.");
                
                for (let i = 0; i < counts.short; i++) {
                    const item = document.createElement('div');
                    item.className = "mb-6";
                    item.innerHTML = `
                        <div class="flex items-center gap-2 mb-2">
                            <span class="font-bold text-gray-700">${globalIdx++}.</span>
                        </div>
                        <div class="w-full border border-gray-200 rounded-md h-20 bg-gray-50/20 line-guide"></div>
                    `;
                    section.appendChild(item);
                }
                container.appendChild(section);
            }

            // 5. 서술형 문항
            if (counts.desc > 0) {
                const section = createSection("V. 서술형 문항", "주제에 대해 논리적으로 상세히 서술하시오.");
                
                for (let i = 0; i < counts.desc; i++) {
                    const item = document.createElement('div');
                    item.className = "mb-6";
                    item.innerHTML = `
                        <div class="flex items-center gap-2 mb-2">
                            <span class="font-bold text-gray-700">${globalIdx++}.</span>
                        </div>
                        <div class="w-full border-2 border-gray-200 rounded-lg h-48 bg-gray-50/20 relative">
                             <div class="absolute inset-0 p-2 pointer-events-none opacity-5 line-guide" style="background-size: 100% 2rem;"></div>
                        </div>
                    `;
                    section.appendChild(item);
                }
                container.appendChild(section);
            }
        }

        function createSection(title, subtitle) {
            const div = document.createElement('div');
            div.className = "section-block";
            div.innerHTML = `
                <div class="flex items-baseline gap-3 mb-4 border-l-4 border-gray-800 pl-3">
                    <h3 class="text-xl font-bold text-gray-800">${title}</h3>
                    <span class="text-xs text-gray-400 font-medium">${subtitle}</span>
                </div>
            `;
            return div;
        }

        window.onload = generateSheet;
    </script>
</body>
</html>

```
