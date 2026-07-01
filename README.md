# travel-fashion
<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>수학여행 AI 코디 추천 시스템</title>
    <style>
        body { font-family: 'Malgun Gothic', sans-serif; background-color: #f0f2f5; margin: 0; padding: 20px; text-align: center; color: #333; display: flex; justify-content: center; align-items: center; min-height: 100vh; }
        .wrapper { background-color: #fff; max-width: 500px; width: 100%; border-radius: 20px; box-shadow: 0 10px 25px rgba(0,0,0,0.05); overflow: hidden; }
        .page { padding: 30px; background: #fff; }
        .hidden { display: none !important; }
        h1 { color: #ff6b6b; font-size: 26px; margin: 0 0 10px; }
        h2 { color: #333; font-size: 18px; margin: 0 0 25px; }
        label { display: block; text-align: left; font-weight: bold; margin: 15px 0 5px; color: #444; }
        select { width: 100%; padding: 12px; border-radius: 10px; border: 1px solid #ddd; font-size: 15px; background-color: #fff; cursor: pointer; }
        .color-picker-container { display: flex; justify-content: space-between; align-items: center; border: 1px solid #ddd; padding: 10px; border-radius: 10px; margin-top: 5px; }
        input[type="color"] { border: none; width: 50px; height: 50px; border-radius: 10px; cursor: pointer; padding: 0; background: none; }
        button { width: 100%; padding: 15px; margin-top: 30px; border-radius: 10px; background-color: #ff6b6b; color: white; border: none; font-size: 16px; font-weight: bold; cursor: pointer; }
        button:hover { background-color: #ee5b5b; }
        .result-summary { background-color: #f9f9f9; padding: 20px; border-radius: 12px; margin-top: 10px; }
        .score { font-size: 24px; font-weight: bold; margin-bottom: 10px; }
        .matching-info { text-align: left; font-size: 14px; color: #666; display: flex; gap: 10px; margin-top: 10px; flex-direction: column; }
        .color-circle { width: 25px; height: 25px; border-radius: 5px; border: 1px solid #eee; display: inline-block; vertical-align: middle; margin-left: 5px; }
        .tip-box { background-color: #f4f6f9; border: 1px solid #e2e8f0; border-radius: 12px; padding: 20px; text-align: left; margin: 20px 0; font-size: 14px; line-height: 1.6; color: #333; }
        .nav-btn { background-color: #eee; color: #555; margin-top: 20px; }
    </style>
</head>
<body>

<div class="wrapper">
    <div id="input-page" class="page">
        <h1>👗 수학여행 코디 추천</h1>
        <h2>퍼스널 컬러 & 과학적 색채 매칭</h2>
        <hr style="border: 0; height: 1px; background: #eee; margin-bottom: 25px;">
        
        <label>1. 당신의 퍼스널 컬러는?</label>
        <select id="personal">
            <option value="">-- 선택하세요 --</option>
            <option value="봄웜">봄 웜톤 (Spring Warm)</option>
            <option value="여름쿨">여름 쿨톤 (Summer Cool)</option>
            <option value="가을웜">가을 웜톤 (Autumn Warm)</option>
            <option value="겨울쿨">겨울 쿨톤 (Winter Cool)</option>
        </select>

        <label>2. 가져갈 상의 색상을 골라주세요!</label>
        <div class="color-picker-container">
            <span style="color:#888; font-size:14px;">원하는 상의 색상 선택 ➡️</span>
            <input type="color" id="top-color" value="#ffffff">
        </div>

        <label>3. 매치할 하의 색상을 골라주세요!</label>
        <div class="color-picker-container">
            <span style="color:#888; font-size:14px;">원하는 하의 색상 선택 ➡️</span>
            <input type="color" id="bottom-color" value="#224488">
        </div>

        <button onclick="calculateAndNavigate()">✨ 과학적 색채 매칭 시작</button>
    </div>

    <div id="result-page" class="page hidden">
        <h1>🛍️ AI 코디 큐레이션 결과</h1>
        <hr style="border: 0; height: 1px; background: #eee; margin-bottom: 25px;">
        <div class="result-summary">
            <div id="resultScore" class="score"></div>
            <div class="matching-info">
                <div><b>선택한 톤:</b> <span id="personal-text"></span></div>
                <div><b>상의색:</b> <span id="resultTop" class="color-circle"></span></div>
                <div><b>하의색:</b> <span id="resultBottom" class="color-circle"></span></div>
            </div>
        </div>
        <div id="tip-wrapper" class="tip-box">
            <div id="resultTip"></div>
        </div>
        <button class="nav-btn" onclick="backToInput()">🔄 다시 매치하기</button>
    </div>
</div>

<script>
    const personalMap = { '봄웜': '봄 웜톤', '여름쿨': '여름 쿨톤', '가을웜': '가을 웜톤', '겨울쿨': '겨울 쿨톤' };

    function rgbToHsv(hex) {
        hex = hex.replace('#', '');
        let r = parseInt(hex.substring(0, 2), 16) / 255;
        let g = parseInt(hex.substring(2, 4), 16) / 255;
        let b = parseInt(hex.substring(4, 6), 16) / 255;
        let max = Math.max(r, g, b), min = Math.min(r, g, b);
        let h, s, v = max;
        let d = max - min;
        s = max === 0 ? 0 : d / max;
        if (max === min) { h = 0; } 
        else {
            switch (max) {
                case r: h = (g - b) / d + (g < b ? 6 : 0); break;
                case g: h = (b - r) / d + 2; break;
                case b: h = (r - g) / d + 4; break;
            }
            h /= 6;
        }
        return { h: Math.round(h * 360), s: Math.round(s * 100), v: Math.round(v * 100) };
    }

    function calculateAndNavigate() {
        const personal = document.getElementById('personal').value;
        const topHex = document.getElementById('top-color').value;
        const bottomHex = document.getElementById('bottom-color').value;

        if(!personal) { alert("⚠️ 퍼스널 컬러를 먼저 선택해 주세요!"); return; }

        document.getElementById('personal-text').innerText = personalMap[personal];
        document.getElementById('resultTop').style.backgroundColor = topHex;
        document.getElementById('resultBottom').style.backgroundColor = bottomHex;

        const topHsv = rgbToHsv(topHex);
        const bottomHsv = rgbToHsv(bottomHex);

        let hueDiff = Math.abs(topHsv.h - bottomHsv.h);
        if (hueDiff > 180) hueDiff = 360 - hueDiff;

        let baseScore = 75;
        let detailReason = "";

        const isTopAchromatic = topHsv.s < 12 || topHsv.v < 15 || (topHsv.s < 15 && topHsv.v > 85);
        const isBottomAchromatic = bottomHsv.s < 12 || bottomHsv.v < 15 || (bottomHsv.s < 15 && bottomHsv.v > 85);

        if (isTopAchromatic || isBottomAchromatic) {
            baseScore += 15;
            detailReason = "👗 **안정적인 베이직 코디!**<br>실패가 없는 무채색 치트키 조합입니다! 베이직하면서도 깔끔한 실루엣을 연출하여 활동성이 많은 수학여행 TPO에 가장 안정적이고 세련된 인상을 줍니다.";
        } else if (hueDiff <= 30) {
            baseScore += 18;
            detailReason = "✨ **트렌디한 톤온톤 스타일!**<br>유사한 색상 계열을 매치한 시밀러 스타일입니다! 전체적인 통일감을 주어 키가 더 커 보이고 정돈된 느낌을 주는 트렌디한 패션 코디네이션입니다.";
        } else if (hueDiff >= 150 && hueDiff <= 180) {
            baseScore -= 20;
            detailReason = "🚨 **강렬한 보색 대비 스타일!**<br>상·하의가 색상환의 정반대에 위치한 과감한 조합입니다. 색상 대비가 너무 강해 시선이 분산될 수 있으니 흰색 티셔츠나 아우터를 걸쳐 대비를 중화시켜 보세요.";
        } else {
            baseScore += 2;
            detailReason = "👍 **무난하고 은은한 스타일!**<br>색상의 대비가 자연스럽게 이루어지는 평범하고 대중적인 스타일링입니다. 일상에서 무난하게 소화 가능한 매치입니다.";
        }

        if (personal === "봄웜" || personal === "가을웜") {
            if ((topHsv.h >= 20 && topHsv.h <= 90) && !isTopAchromatic) baseScore += 6;
            if ((topHsv.h >= 160 && topHsv.h <= 260) && !isTopAchromatic) baseScore -= 10;
        } else {
            if ((topHsv.h >= 160 && topHsv.h <= 260) && !isTopAchromatic) baseScore += 6;
            if ((topHsv.h >= 20 && topHsv.h <= 90) && !isTopAchromatic) baseScore -= 10;
        }

        let finalScore = Math.max(40, Math.min(99, Math.round(baseScore)));
        const tipWrapper = document.getElementById('tip-wrapper');
        const resultScore = document.getElementById('resultScore');

        if (finalScore >= 90) {
            resultScore.style.color = "#1ea952";
            tipWrapper.style.backgroundColor = "#eef9f3";
            tipWrapper.style.borderColor = "#cdf0da";
        } else if (finalScore >= 70) {
            resultScore.style.color = "#3182ce";
            tipWrapper.style.backgroundColor = "#ebf8ff";
            tipWrapper.style.borderColor = "#bee3f8";
        } else {
            resultScore.style.color = "#e53e3e";
            tipWrapper.style.backgroundColor = "#fff5f5";
            tipWrapper.style.borderColor = "#fed7d7";
        }

        resultScore.innerText = `📊 패션 조화도 점수: ${finalScore}점`;
        document.getElementById('resultTip').innerHTML = detailReason;

        document.getElementById('input-page').classList.add('hidden');
        document.getElementById('result-page').classList.remove('hidden');
    }

    function backToInput() {
        document.getElementById('input-page').classList.remove('hidden');
        document.getElementById('result-page').classList.add('hidden');
    }
</script>
</body>
</html>