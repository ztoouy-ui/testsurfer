<!DOCTYPE html>
<html lang="ko">
<head>
<meta charset="UTF-8">
<title>서핑 보드 자동 매칭 툴 - 실시간 버전</title>
<style>
    body { font-family: Arial, sans-serif; background:#f0f4f8; margin:0; padding:20px; }
    h1 { text-align:center; }
    .container { max-width:600px; margin:auto; background:#fff; padding:20px; border-radius:10px; box-shadow:0 0 10px rgba(0,0,0,0.1); }
    label { display:block; margin-top:10px; font-weight:bold; }
    select, input { width:100%; padding:8px; margin-top:5px; border-radius:5px; border:1px solid #ccc; }
    button { margin-top:15px; padding:10px; width:100%; background:#0077cc; color:white; border:none; border-radius:5px; font-size:16px; cursor:pointer; }
    button:hover { background:#005fa3; }
    .result { margin-top:20px; background:#eef; padding:15px; border-radius:5px; }
    .board-card { background:white; margin-top:10px; padding:10px; border-radius:5px; border:1px solid #ddd; }
    .board-card h3 { margin:0; }
    .pin { font-style:italic; color:#555; font-size:14px; }
</style>
</head>
<body>
<h1>🏄‍♂️ 실시간 서핑 보드 자동 매칭</h1>
<div class="container">
    <label>서핑 스팟</label>
    <select id="spot">
        <option value="namyeol">남열</option>
        <option value="busan">부산 송정</option>
    </select>

    <button onclick="getSurfData()">실시간 데이터 불러오기</button>

    <div id="liveData" style="margin-top:15px; display:none;"></div>
    <div id="result" class="result" style="display:none;"></div>
</div>

<script>
// 서핑 보드 데이터
const boards = [
    { name: "Sharp Eye Synergy", size: "6'0\" 30.5L", tail: "Round Tail", fin: "Thruster", desc: "중~큰 파도 퍼포먼스 숏보드, 파워 있는 클린 페이스에 강함" },
    { name: "Lost Sub Driver 3.0", size: "5'10\" 31.0L", tail: "Squash Tail", fin: "Thruster/Quad", desc: "약~중간 파도 하이브리드 퍼포먼스, 패들력·가속 우수" },
    { name: "Chilli Fade 2.0", size: "5'11\" 30.8L", tail: "Squash Tail", fin: "Thruster", desc: "올라운드 퍼포먼스 숏보드, 다양한 조건 대응 가능" },
    { name: "JS Monsta", size: "6'0\" 30.5L", tail: "Round Tail", fin: "Thruster", desc: "올라운드 퍼포먼스, 중~큰 파도 안정성·드라이브 우수" },
    { name: "Channel Islands Bobby Quad", size: "5'8\" 29.0L", tail: "Swallow Tail", fin: "Quad", desc: "퍼포먼스 피시, 작은~중간 파도 속도·유연성 강점" }
];

// 스팟 좌표
const spots = {
    namyeol: { lat: 34.605, lon: 127.287 },
    busan: { lat: 35.179, lon: 129.199 }
};

// 실시간 데이터 가져오기
async function getSurfData() {
    const spot = document.getElementById('spot').value;
    const { lat, lon } = spots[spot];

    // Open-Meteo API (풍속, 파고, 파 주기)
    const url = `https://www.wsbfarm.com/wavecam/WaveChartView?beach=GNH2`;
    const res = await fetch(url);
    const data = await res.json();

    // 현재 시간 기준 데이터
    const nowIndex = 0;
    const height = data.hourly.wave_height[nowIndex];
    const period = data.hourly.wave_period[nowIndex];
    const windSpeed = data.hourly.wind_speed[nowIndex];
    const windDirDeg = data.hourly.wind_direction[nowIndex];
    const windDir = degToDirection(windDirDeg);

    document.getElementById('liveData').style.display = "block";
    document.getElementById('liveData').innerHTML = `
        <h3>🌊 실시간 조건</h3>
        <p>파고: ${height.toFixed(1)} m</p>
        <p>피리어드: ${period.toFixed(1)} s</p>
        <p>바람: ${windDir} ${windSpeed.toFixed(1)} m/s</p>
    `;

    matchBoard(height, period, windDir, windSpeed);
}

// 풍향 변환
function degToDirection(deg) {
    const dirs = ['북', '북동', '동', '남동', '남', '남서', '서', '북서'];
    return dirs[Math.round(deg / 45) % 8];
}

// 보드 매칭
function matchBoard(h, p, windDir, windSpeed) {
    let picks = [];

    if (h <= 1.0 && p <= 8) {
        picks = [boards[4], boards[1], boards[2]];
    }
    else if (h > 1.0 && h <= 1.3 && p >= 8) {
        picks = [boards[0], boards[3], boards[2]];
    }
    else if (h >= 1.3 && p >= 9) {
        picks = [boards[0], boards[3]];
    }
    else if (h <= 0.9 && p <= 7 && windSpeed > 5) {
        picks = [boards[4], boards[1]];
    }
    else {
        picks = [boards[2], boards[1], boards[4]];
    }

    let html = `<h2>추천 보드</h2>`;
    picks.forEach((b, i) => {
        html += `<div class="board-card">
            <h3>${i+1}순위: ${b.name}</h3>
            <p>${b.size} / ${b.tail} / ${b.fin}</p>
            <p>${b.desc}</p>
            <p class="pin">핀 추천: ${b.fin.includes("Quad") ? "쿼드 Performer" : "Thruster Performer"}</p>
        </div>`;
    });

    document.getElementById('result').innerHTML = html;
    document.getElementById('result').style.display = "block";
}
</script>
</body>
</html>
