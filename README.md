<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>AI 주식 투자 비서</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.0.0/css/all.min.css">
    <!-- TradingView Widget Script -->
    <script type="text/javascript" src="https://s3.tradingview.com/tv.js"></script>
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Pretendard:wght@400;600;800&display=swap');
        body { font-family: 'Pretendard', sans-serif; background-color: #f8fafc; scroll-behavior: smooth; }
        .glass-card { background: rgba(255, 255, 255, 0.9); backdrop-filter: blur(10px); border: 1px solid rgba(255, 255, 255, 0.3); }
        .stock-row:hover { background-color: #f0f7ff; cursor: pointer; }
        .active-row { background-color: #eff6ff; border-left: 4px solid #3b82f6 !important; }
        
        /* 테이블 스타일 */
        table { width: 100%; border-collapse: separate; border-spacing: 0; }
        th { background-color: #f8fafc; padding: 12px; text-align: left; font-weight: 700; color: #475569; border-bottom: 2px solid #e2e8f0; }
        td { padding: 14px 12px; border-bottom: 1px solid #f1f5f9; }
        
        .badge-red { background: #fee2e2; color: #dc2626; }
        .badge-blue { background: #dbeafe; color: #2563eb; }
    </style>
</head>
<body class="min-h-screen text-slate-800 pb-20">

    <div class="max-w-5xl mx-auto p-4 sm:p-8">
        <!-- Header -->
        <header class="flex items-center justify-between mb-8">
            <div>
                <h1 class="text-3xl font-black text-blue-600 italic tracking-tighter">STOCK AI</h1>
                <p class="text-slate-500 text-sm font-medium">데이터 기반 저평가 우량주 추천 리스트</p>
            </div>
            <div class="text-right">
                <div class="bg-blue-600 text-white px-4 py-1 rounded-full text-xs font-bold shadow-lg">PRESET DATA V1.0</div>
                <p class="text-[10px] text-slate-400 mt-1">Last Update: 2024.05.20</p>
            </div>
        </header>

        <div class="grid grid-cols-1 lg:grid-cols-3 gap-8">
            <!-- Left: Stock List (2/3 width on large screens) -->
            <div class="lg:col-span-2 space-y-6">
                <div class="glass-card rounded-3xl p-6 shadow-xl border border-white">
                    <div class="flex items-center justify-between mb-6">
                        <h2 class="text-xl font-bold flex items-center gap-2">
                            <i class="fa-solid fa-list-check text-blue-500"></i>
                            국장 저평가 우량주 TOP 20
                        </h2>
                        <div id="pageIndicator" class="text-sm font-bold text-slate-400">1 / 2 Page</div>
                    </div>

                    <div class="overflow-hidden rounded-xl border border-slate-100 shadow-sm bg-white">
                        <table id="stockTable">
                            <thead>
                                <tr>
                                    <th>종목명</th>
                                    <th>코드</th>
                                    <th>PBR</th>
                                    <th>ROE</th>
                                    <th>상태</th>
                                </tr>
                            </thead>
                            <tbody id="stockTableBody">
                                <!-- Data injected by JS -->
                            </tbody>
                        </table>
                    </div>

                    <!-- Pagination -->
                    <div class="flex justify-center items-center gap-4 mt-6">
                        <button id="prevBtn" class="p-2 px-4 rounded-xl border border-slate-200 hover:bg-slate-50 disabled:opacity-30 disabled:cursor-not-allowed transition-all">
                            <i class="fa-solid fa-chevron-left mr-2"></i> 이전
                        </button>
                        <button id="nextBtn" class="p-2 px-4 rounded-xl border border-slate-200 hover:bg-slate-50 disabled:opacity-30 disabled:cursor-not-allowed transition-all">
                            다음 <i class="fa-solid fa-chevron-right ml-2"></i>
                        </button>
                    </div>
                </div>
            </div>

            <!-- Right: Detail & Chart (1/3 width) -->
            <div class="space-y-6">
                <!-- Detail Card -->
                <div id="detailCard" class="glass-card rounded-3xl p-6 shadow-xl border border-white sticky top-8">
                    <div id="noSelection" class="py-20 text-center text-slate-400">
                        <i class="fa-solid fa-arrow-left mb-4 text-3xl block"></i>
                        <p>종목을 선택하여<br>상세 분석을 확인하세요</p>
                    </div>

                    <div id="selectionContent" class="hidden space-y-4">
                        <div class="flex items-start justify-between">
                            <div>
                                <h3 id="detName" class="text-2xl font-black text-slate-800">---</h3>
                                <p id="detCode" class="text-blue-600 font-mono font-bold tracking-widest">000000</p>
                            </div>
                            <span id="detBadge" class="px-2 py-1 rounded text-[10px] font-black uppercase">Low PBR</span>
                        </div>
                        
                        <div class="grid grid-cols-2 gap-2 text-center py-2 border-y border-slate-100">
                            <div>
                                <p class="text-[10px] text-slate-400 uppercase">PBR</p>
                                <p id="detPbr" class="font-bold text-slate-700">--</p>
                            </div>
                            <div>
                                <p class="text-[10px] text-slate-400 uppercase">ROE</p>
                                <p id="detRoe" class="font-bold text-slate-700">--</p>
                            </div>
                        </div>

                        <div>
                            <h4 class="text-sm font-bold text-slate-800 mb-1 flex items-center gap-2">
                                <i class="fa-solid fa-circle-info text-blue-500 text-[10px]"></i> 투자 포인트
                            </h4>
                            <p id="detDesc" class="text-sm text-slate-600 leading-relaxed">---</p>
                        </div>

                        <!-- Mini Chart Area -->
                        <div class="rounded-2xl overflow-hidden border border-slate-100 h-[300px] mt-4 shadow-inner">
                            <div id="tradingview_widget" class="w-full h-full"></div>
                        </div>
                    </div>
                </div>
            </div>
        </div>
        
        <footer class="mt-12 text-center text-slate-400 text-xs">
            <p>본 데이터는 미리 분석된 정적 데이터이며, 실제 투자 시 최신 재무제표를 반드시 확인하시기 바랍니다.</p>
            <p class="mt-1">© 2024 STOCK AI. All rights reserved.</p>
        </footer>
    </div>

    <script>
        // 미리 준비된 20개의 정적 데이터 리스트
        const stockData = [
            { name: "삼성전자", code: "005930", pbr: 1.2, roe: 16.5, desc: "글로벌 메모리 반도체 1위 기업으로, 최근 HBM 및 파운드리 부문의 성장세가 기대됩니다. 압도적인 현금 흐름과 기술 장벽이 핵심입니다.", status: "우량" },
            { name: "현대차", code: "005380", pbr: 0.6, roe: 12.1, desc: "대표적인 저PBR 밸류업 수혜주입니다. 하이브리드 및 전기차 경쟁력을 바탕으로 역대급 실적을 경신 중이며 배당 성향 확대가 기대됩니다.", status: "저평가" },
            { name: "기아", code: "000270", pbr: 0.8, roe: 18.2, desc: "높은 영업이익률과 강력한 주주환원 정책(자사주 매입 및 소각)으로 시장의 주목을 받고 있습니다. 북미 시장 점유율이 견고합니다.", status: "저평가" },
            { name: "KB금융", code: "105560", pbr: 0.4, roe: 9.5, desc: "은행권 중 가장 높은 자본 건전성을 보유하고 있습니다. 저PBR 밸류업 프로그램의 핵심 대장주로 평가받고 있습니다.", status: "배당" },
            { name: "삼성물산", code: "028260", pbr: 0.7, roe: 7.8, desc: "그룹 지배구조의 핵심이며, 풍부한 자사주 소각 계획을 발표했습니다. 신사업(바이오, 친환경 에너지) 비중이 확대되고 있습니다.", status: "저평가" },
            { name: "SK하이닉스", code: "000660", pbr: 1.8, roe: 14.0, desc: "HBM3 시장 점유율 독보적 1위로 AI 반도체 수혜를 직접적으로 받고 있습니다. 메모리 사이클 반등의 최대 수혜주입니다.", status: "성장" },
            { name: "POSCO홀딩스", code: "005490", pbr: 0.6, roe: 6.5, desc: "철강 본업의 안정성과 리튬 등 이차전지 소재 부문의 중장기 성장성을 동시에 보유한 기업입니다. 밸류에이션 매력이 높습니다.", status: "저평가" },
            { name: "신한지주", code: "055550", pbr: 0.4, roe: 9.1, desc: "분기 배당 등 주주 환원에 가장 적극적인 금융지주사 중 하나입니다. 금리 환경 변화에 따른 이익 방어력이 우수합니다.", status: "배당" },
            { name: "LG전자", code: "066570", pbr: 0.9, roe: 10.2, desc: "가전 사업의 안정적 수익과 전장(VS) 사업부의 흑자 전환으로 재평가가 진행 중입니다. B2B 비중 확대로 체질 개선 중입니다.", status: "저평가" },
            { name: "하나금융지주", code: "086790", pbr: 0.4, roe: 10.5, desc: "우수한 비은행 부문 포트폴리오를 보유하고 있으며, 강력한 주주환원 의지를 표명한 대표적인 저PBR 종목입니다.", status: "배당" },
            { name: "삼성생명", code: "032830", pbr: 0.4, roe: 6.0, desc: "삼성전자 지분 가치 대비 현저히 저평가된 상태입니다. 금리 상승 시 IFRS17 하의 이익 체력이 강화되는 구조입니다.", status: "저평가" },
            { name: "SK", code: "034730", pbr: 0.5, roe: 5.5, desc: "SK그룹의 지주사로 자회사(하이닉스 등) 가치 대비 저평가되어 있습니다. 수소, 반도체 소재 등 미래 사업 투자가 활발합니다.", status: "지주" },
            { name: "한국전력", code: "015760", pbr: 0.3, roe: -2.0, desc: "에너지 가격 하향 안정화와 전기 요금 현실화로 턴어라운드가 기대됩니다. 극심한 저PBR 상태에서 해소 국면 진입 중입니다.", status: "턴어라운드" },
            { name: "LG화학", code: "051910", pbr: 1.1, roe: 8.5, desc: "양극재 등 배당 소재 사업의 고성장세와 석유화학 본업의 회복세가 맞물려 있습니다. 글로벌 톱티어 소재 기업입니다.", status: "성장" },
            { name: "삼성SDI", code: "006400", pbr: 1.5, roe: 12.5, desc: "수익성 중심의 질적 성장을 추구하는 배터리 기업입니다. 전고체 배터리 등 차세대 기술 개발에서 앞서 나가고 있습니다.", status: "성장" },
            { name: "KT&G", code: "033780", pbr: 1.1, roe: 11.2, desc: "안정적인 현금 창출 능력을 기반으로 높은 배당 수익률을 자랑합니다. 궐련형 전자담배(NGP) 수출 비중이 커지고 있습니다.", status: "배당" },
            { name: "우리금융지주", code: "316140", pbr: 0.3, roe: 10.8, desc: "가장 저평가된 금융지주사 중 하나로, 증권/보험사 인수를 통한 비은행 부문 강화가 주가 모멘텀이 될 전망입니다.", status: "배당" },
            { name: "메리츠금융지주", code: "138040", pbr: 1.4, roe: 28.5, desc: "국내 최고 수준의 주주환원(연결당기순이익 50%)을 실천하고 있습니다. ROE가 매우 높아 효율적인 경영이 돋보입니다.", status: "우량" },
            { name: "현대모비스", code: "012330", pbr: 0.5, roe: 7.2, desc: "현대차그룹의 핵심 부품사로 모듈 및 핵심 부품 제조 역량이 뛰어납니다. 지배구조 개편 시 수혜가 예상되는 저PBR주입니다.", status: "저평가" },
            { name: "KT", code: "030200", pbr: 0.6, roe: 9.8, desc: "통신 본업의 안정성과 더불어 클라우드, AI 등 DX(디지털 전환) 사업의 성장이 가시화되고 있습니다. 대표적인 배당주입니다.", status: "배당" }
        ];

        let currentPage = 1;
        const itemsPerPage = 10;

        const tableBody = document.getElementById('stockTableBody');
        const prevBtn = document.getElementById('prevBtn');
        const nextBtn = document.getElementById('nextBtn');
        const pageIndicator = document.getElementById('pageIndicator');
        const selectionContent = document.getElementById('selectionContent');
        const noSelection = document.getElementById('noSelection');

        function renderTable() {
            tableBody.innerHTML = '';
            const start = (currentPage - 1) * itemsPerPage;
            const end = start + itemsPerPage;
            const currentItems = stockData.slice(start, end);

            currentItems.forEach((item, index) => {
                const row = document.createElement('tr');
                row.className = 'stock-row transition-all';
                row.innerHTML = `
                    <td class="font-bold text-slate-700">${item.name}</td>
                    <td class="text-slate-400 font-mono text-xs">${item.code}</td>
                    <td class="font-semibold">${item.pbr}</td>
                    <td class="font-semibold text-blue-600">${item.roe}%</td>
                    <td><span class="px-2 py-0.5 rounded-full text-[10px] font-bold ${item.status === '저평가' ? 'badge-red' : 'badge-blue'}">${item.status}</span></td>
                `;
                row.onclick = () => selectStock(item, row);
                tableBody.appendChild(row);
            });

            pageIndicator.innerText = `${currentPage} / ${Math.ceil(stockData.length / itemsPerPage)} Page`;
            prevBtn.disabled = currentPage === 1;
            nextBtn.disabled = currentPage === Math.ceil(stockData.length / itemsPerPage);
        }

        function selectStock(item, rowElement) {
            // UI Update
            document.querySelectorAll('.stock-row').forEach(r => r.classList.remove('active-row'));
            rowElement.classList.add('active-row');
            
            noSelection.classList.add('hidden');
            selectionContent.classList.remove('hidden');

            document.getElementById('detName').innerText = item.name;
            document.getElementById('detCode').innerText = item.code;
            document.getElementById('detPbr').innerText = item.pbr;
            document.getElementById('detRoe').innerText = `${item.roe}%`;
            document.getElementById('detDesc').innerText = item.desc;
            
            const badge = document.getElementById('detBadge');
            badge.innerText = item.status;
            badge.className = `px-2 py-1 rounded text-[10px] font-black uppercase ${item.status === '저평가' ? 'badge-red' : 'badge-blue'}`;

            // TradingView Load
            initTradingView(item.code);
        }

        function initTradingView(code) {
            document.getElementById('tradingview_widget').innerHTML = '';
            new TradingView.widget({
                "autosize": true,
                "symbol": `KRX:${code}`,
                "interval": "D",
                "timezone": "Asia/Seoul",
                "theme": "light",
                "style": "1",
                "locale": "kr",
                "toolbar_bg": "#f1f3f6",
                "enable_publishing": false,
                "hide_top_toolbar": true,
                "save_image": false,
                "container_id": "tradingview_widget"
            });
        }

        prevBtn.onclick = () => { if(currentPage > 1) { currentPage--; renderTable(); } };
        nextBtn.onclick = () => { if(currentPage < 2) { currentPage++; renderTable(); } };

        // Initial Render
        window.onload = renderTable;
    </script>
</body>
</html>
