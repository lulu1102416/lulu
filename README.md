<!DOCTYPE html>
<html lang="zh-TW">
<head>
  <meta charset="UTF-8" />
  <title>金融戰略王</title>
  <style>
    body { font-family: sans-serif; padding: 2em; max-width: 800px; margin: auto; }
    .hidden { display: none; }
    input { margin: 0.3em 0; width: 40px; }
    button { margin-top: 1em; }
    table, td, th { border: 1px solid #999; border-collapse: collapse; padding: 4px; }
    #cardBox { border: 1px solid #ccc; padding: 1em; margin-top: 1em; }
  </style>
</head>
<body>

<h1>金融戰略王</h1>

<div id="startSection">
  <p>請輸入學生資料：</p>
  <input id="name" placeholder="姓名" /><br />
  <input id="id" placeholder="學號" /><br />
  <button onclick="startGame()">開始遊戲</button>
</div>

<div id="gameSection" class="hidden">
  <p>學生：<span id="studentInfo"></span></p>
  <p>目前回合：<span id="roundCount">1</span> / 5</p>
  <button onclick="drawCard()">抽卡</button>
  <div id="cardBox" class="hidden">
    <h3 id="cardTitle"></h3>
    <p id="cardDesc"></p>
    <p>請輸入 S/B/X/C/R 加權（合計最多為 5）：</p>
    <label>S: <input id="inputS" type="number" min="0" max="5" /></label>
    <label>B: <input id="inputB" type="number" min="0" max="5" /></label>
    <label>X: <input id="inputX" type="number" min="0" max="5" /></label>
    <label>C: <input id="inputC" type="number" min="0" max="5" /></label>
    <label>R: <input id="inputR" type="number" min="0" max="5" /></label><br />
    <button onclick="confirmTurn()">確定</button>
  </div>
  <p id="result"></p>
  <p>總分：<span id="totalScore">0</span></p>
  <h3>回合紀錄：</h3>
  <ul id="logList"></ul>
  <button onclick="restartGame()">重新開始</button>
  <button onclick="showAdmin()">老師後台</button>
</div>

<div id="adminSection" class="hidden">
  <h2>老師後台</h2>
  <button onclick="exportCSV()">下載 CSV</button>
  <button onclick="hideAdmin()">返回遊戲畫面</button>
  <div id="recordTable"></div>
</div>

<script>

const cards = [
  {
    title: "賓拉登發動九一一復仇恐怖攻擊",
    description: "2001年9月11日，賓拉登歷經與CIA及布希家族在阿富汗與波斯灣戰爭糾葛的恩怨情仇，終於在第二次攻擊世貿雙子星大樓的行動中展現恐怖主義的實力。",
    s: -5, b:  5, x:  5, c: -5, r: -5
  },
  {
    title: "以阿戰爭石油禁運引發能源危機",
    description: "1973年10月17日，第四波以色列與阿拉伯世界的軍事衝突迫使OPEC對支持以色列的國家實施石油禁運，導致工業國家能源供需失序。",
    s: -5, b: -4, x: -3, c:  5, r: -5
  },
  {
    title: "冷戰高峰期蘇聯發動阿富汗戰爭",
    description: "1979年12月24日，以阿戰爭剛結束，蘇聯擔心美國因以阿和平協議控制中東而發動阿富汗戰爭，卻陷入類似越戰的泥沼無法自拔。",
    s: -3, b:  3, x: -3, c:  5, r: -4
  },
  {
    title: "柏林圍牆倒塌宣告冷戰時期結束",
    description: "1989年11月9日，東歐民主開放浪潮迫使蘇聯共產體制瓦解，長達半世紀的美蘇對峙宣告結束，資本主義大獲全勝。",
    s:  5, b: -4, x:  5, c:  4, r:  5
  },
  {
    title: "尼克森震撼癱瘓布雷頓森林體系",
    description: "1973年2月13日，尼克森以單日10%美元貶值震撼全球，癱瘓布雷頓森林貨幣體系，工業大國之間自由浮動匯率體制取而代之。",
    s:  3, b: -4, x:  5, c:  4, r: -3
  },
  {
    title: "英殖民地香港九七大限回歸中國",
    description: "1997年7月1日，英國殖民地東方之珠香港回歸中國，成為中國崛起的經濟門戶與資本主義與共產主義融合的試金石。",
    s: -3, b:  4, x: -5, c: -2, r: -4
  },
  {
    title: "中國經濟崛起加入世界貿易組織",
    description: "2001年12月11日，中國改革開放後搭上高科技與製造業浪潮，以其在全球供應鏈與消費市場的地位加入WTO，躍升世界工廠。",
    s:  5, b:  4, x:  5, c:  5, r:  5
  },
  {
    title: "美俄終究為喬治亞能源利益翻臉",
    description: "2008年8月8日，能源價格飆漲提升喬治亞的戰略地位，加上冷戰舊怨，美俄因喬治亞衝突決裂，關係破裂。",
    s: -4, b:  2, x: -3, c:  5, r: -5
  },
  {
    title: "工業五國簽署廣場協議貶值美元",
    description: "1985年9月22日，工業五國簽署廣場協議，讓美元對日元與馬克貶值，以避免貿易保護主義並平衡國際收支。",
    s:  4, b: -3, x:  5, c:  5, r:  2
  },
  {
    title: "美布希發動伊拉克反恐解放戰爭",
    description: "2003年3月20日，美國布希政府以反恐為由發動伊拉克解放戰爭，誓言徹底掃蕩恐怖網絡並恢復中東秩序與穩定。",
    s:  4, b: -2, x: -3, c:  5, r:  3
  },
  {
    title: "尼克森終因水門案醜聞黯然下台",
    description: "1974年8月9日，水門案醜聞調查壓力攀升，最終迫使尼克森宣布辭職下台，留下政經亂局給副總統福特接手。",
    s: -5, b:  3, x:  5, c: -4, r: -5
  },
  {
    title: "中國央行決定人民幣與美元脫鉤",
    description: "2005年7月22日，中國人民銀行在全球貿易及資本流動壓力下，決定放棄盯住美元的匯率制度，人民幣脫鉤美元。",
    s:  3, b:  2, x:  5, c:  4, r:  3
  },
  {
    title: "雷根準備替換沃克企圖甚囂塵上",
    description: "1985年10月22日，經過沃克高壓利率政策穩定通膨後，雷根政府主導更換聯準會主席，欲加速降息並擴大貨幣供給以刺激成長。",
    s:  5, b:  3, x:  3, c:  4, r:  3
  },
  {
    title: "美發動波斯灣戰爭鞏固石油利益",
    description: "1990年8月2日，布希政府發動波斯灣戰爭，聯合沙烏地阿拉伯收復科威特，並借此鞏固美國石油戰略利益。",
    s: -3, b:  2, x: -3, c:  5, r: -3
  },
  {
    title: "美國聖海倫火山大爆發前兆地震",
    description: "1980年3月20日，美國1970年代中期地質學家研究報告所擔心的聖海倫活火山終於引發華盛頓州地震，對失控的通膨火上加油。",
    s: -4, b:  3, x:  4, c:  5, r: -5
  },
  {
    title: "亞洲SARS風暴撼動世衛防疫網",
    description: "2003年3月15日，中國經濟極速發展帶來公共衛生風險，SARS疫情蔓延整個亞洲、台灣、香港，首當其衝，WHO發布全球警戒。",
    s: -4, b:  4, x: -4, c: -4, r: -4
  },
  {
    title: "超級颶風卡崔娜重創紐奧爾良市",
    description: "2005年8月29日，第五號颶風卡崔娜發展為五級超級颶風，橫掃墨西哥灣沿岸並重創紐奧爾良及石化產業。",
    s: -2, b: -2, x:  2, c:  5, r: -3
  },
  {
    title: "安德魯颶風重創美國弗羅里達州",
    description: "1992年8月24日，安德魯颶風打亂美國石油供應鏈佈局，也成為共和黨在1992年總統大選落敗的一根稻草。",
    s: -2, b:  3, x:  5, c:  4, r: -3
  },

  {
    title: "新興市場債信暴風圈轉向俄羅斯",
    description: "1998年8月17日，俄羅斯央行在亞洲金融風暴愈演愈烈過程中試圖維繫盧布緊盯美元，加上經濟衰退導致石油需求銳減，最終以債務違約與盧布劇貶收場。",
    s: -3, b: -2, x: -5, c: -5, r: -4
  },
  {
    title: "高盛發布金磚四國投資夢想報告",
    description: "2003年10月1日，歷經網路泡沫化、企業財報醜聞及反恐戰爭，全球資金流出美國，高盛一度推動金磚四國成為世界經濟成長新引擎。",
    s:  4, b:  3, x:  5, c:  5, r:  5
  },
  {
    title: "日本央行三度加息爆破財工泡沫",
    description: "1989年12月25日，日本企業憑藉電子科技與市場氣勢，透過財務工程取得低利率融資並大舉海外併購及房地產投資，迫使央行連續加息控制貨幣供給。",
    s: -5, b: -3, x: -4, c: -4, r: -5
  },
  {
    title: "網路科技多頭司令柯恩鳴金收兵",
    description: "2000年3月28日，多年來力捧科技股、號稱多頭總司令的高盛首席策略師艾比·科恩突然轉向熊市，宣稱科技牛市已疲態盡顯。",
    s: -5, b:  4, x:  3, c: -3, r: -4
  },
  {
    title: "索羅斯黑色星期三狙擊弱勢英鎊",
    description: "1992年9月16日，德國央行高利率政策使英鎊承壓，索羅斯於黑色星期三狙擊英鎊大獲全勝，揭示匯市巨變風險。",
    s:  4, b:  2, x: -3, c:  2, r:  3
  },
  {
    title: "網際網頁搜尋公司雅虎粉末上市",
    description: "1996年4月12日，WWW時代來臨，資訊自由熱潮興起，雅虎上市正式揭開網際網路泡沫的序幕。",
    s:  5, b: -4, x: -3, c: -3, r: -3
  },
  {
    title: "能源泡沫之大陸伊利諾銀行危機",
    description: "1984年5月10日，大陸伊利諾銀行因石油與天然氣開發貸款違約引發擠兌，能源泡沫擴張形成金融系統性風險。",
    s: -3, b:  5, x: -4, c: -5, r: -3
  },
  {
    title: "避險基金天王殞落之金融啟示錄",
    description: "1998年9月23日，俄羅斯債務違約引發全球衍生性金融市場連鎖反應，使高槓桿套利基金LTCM深陷流動性危機並最終破產。",
    s: -5, b:  5, x: -5, c: -3, r: -4
  },
  {
    title: "安隆破產案之衍生性金融啟示錄",
    description: "2001年12月2日，安隆利用財務工程與能源交易建立金融帝國，卻因過度槓桿操作自掘墳墓，成為泡沫破滅的典型案例。",
    s: -5, b:  3, x:  3, c: -5, r: -3
  },
  {
    title: "信貸完美風暴夷平雷曼債券帝國",
    description: "2008年9月15日，美國財長波爾森介入雷曼危機卻最終選擇放棄紓困，釀成無法承受之黑天鵝事件，摧毀金融體系信心。",
    s: -5, b: -3, x: -4, c: -5, r: -5
  },
  {
    title: "高盛力拱第二波石油超級瘋漲期",
    description: "2008年3月7日，高盛分析指出石油價格可能突破100美元並進一步衝向200美元，煽動市場對能源超級飆漲的預期。",
    s: -4, b: -5, x:  3, c:  5, r: -5
  },
  {
    title: "網際網路龍頭思科市值超越微軟",
    description: "2000年3月24日，網路科技龍頭思科市值一舉超越微軟，象徵市場對網路經濟成長動能的高度樂觀。",
    s:  5, b: -5, x: -3, c:  2, r:  2
  },
  {
    title: "貝爾斯登次貸風暴財務工程夢魘",
    description: "2008年3月16日，貝爾斯登運用高槓桿及複雜金融結構押注次貸市場，最終因市場崩潰被迫出售並破產救助。",
    s: -5, b:  3, x:  3, c: -4, r: -5
  },
  {
    title: "金管單位忽略投資組合保險風險",
    description: "1987年7月23日，隨著投資組合保險與程式化交易興起，監管機構對系統性風險的忽視暴露市場脆弱性。",
    s: -4, b:  3, x: -3, c: -3, r: -3
  },
  {
    title: "納貝斯克槓桿融資收購野蠻戲碼",
    description: "1988年11月30日，私募巨頭 KKR 利用槓桿融資收購納貝斯克，開創企業併購的高槓桿時代。",
    s:  5, b: -4, x: -2, c:  3, r:  4
  },

  {
    title: "消費者視美房市如崩裂中的懸崖",
    description: "2007年4月13日，美國房市對經濟如即將崩裂的懸崖，加上市場炒作原物料通膨對房貸的壓力重創消費者信心指數，讓經濟陷入無可挽救的惡性循環。",
    s: -4, b:  4, x:  3, c: -4, r: -5
  },
  {
    title: "塑膠材料成為世界應用原料之王",
    description: "1976年12月31日，塑膠材料透過其在工業與民生消費的多樣化應用成為最普遍的原料，而塑化工業則驅動經濟成長、科技創新與石油通膨。",
    s:  4, b: -5, x: -3, c:  5, r:  3
  },
  {
    title: "聯準會降息至半世紀之臨界低點",
    description: "2003年6月25日，為刺激經濟成長與挽救投資市場信心，聯準會進一步降息至1958年以來最低的1%利率水準，迫使資本市場因應微利時代的到來。",
    s:  4, b:  3, x:  5, c:  4, r:  5
  },
  {
    title: "日本第一之美國政經衝擊啟示錄",
    description: "1979年5月1日，哈佛大學教授傅高義出版《日本第一》，對1970年代中期日本從科技到金融如何威脅美國霸主地位做出最佳寫照。",
    s:  4, b: -5, x:  5, c:  3, r:  4
  },
  {
    title: "葛林斯潘捍衛聯準會抗通膨立場",
    description: "1987年9月4日，雷根提名葛林斯潘為聯準會主席，新官上任三把火祭出三年來首度貼現率加息，出乎市場預料。",
    s: -5, b: -3, x: -5, c: -4, r: -4
  },
  {
    title: "卡特總統欽點沃克抗停滯性通膨",
    description: "1979年7月25日，在聯準會主席米勒無力應對停滯性通膨之際，卡特總統提名華爾街及市場眾望所歸的沃克出任聯準會主席。",
    s:  3, b: -5, x: -3, c: -4, r: -3
  },
  {
    title: "世界銀行盛讚東亞經濟奇蹟典範",
    description: "1993年9月26日，世界銀行研究報告推崇東亞經濟發展模式為新典範，宣告亞洲四小龍取代日本，成為冷戰結束後推動全球化經濟成長的新引擎。",
    s:  5, b: -3, x:  5, c:  3, r:  3
  },
  {
    title: "雷根上任沃克誓言持續打擊通膨",
    description: "1981年2月6日，美國通膨在1980年3月達到高點後，停滯性通膨讓卡特政府下台；在雷根總統信任下，新任聯準會主席沃克誓言不惜代價持續打擊通膨。",
    s: -5, b:  5, x: -4, c: -5, r: -5
  },
  {
    title: "美聯準會貨幣供給通膨失控謎團",
    description: "1973年11月6日，美聯準會主席柏恩為激進貨幣供給政策辯護，駁斥外界將通膨失控歸咎於其配合尼克森挽救失業率的作法。",
    s: -4, b: -5, x:  5, c:  5, r: -4
  },
  {
    title: "新經濟發展衍生新的經濟學迷思",
    description: "2000年2月25日，舊金山聯邦準備銀行發表「新經濟到底能夠成長多快？」專文，反映網際網路泡沫帶動的非理性繁榮及新經濟學迷思。",
    s:  5, b: -5, x: -3, c:  3, r:  4
  },
  {
    title: "福特總統的停滯性通膨政經夢魘",
    description: "1975年2月5日，尼克森下台後接手經濟爛攤的福特總統在1975年初公布1976年度財政預算，已預告對停滯性通膨的洪水猛獸無力回天。",
    s: -5, b: -5, x:  3, c:  2, r: -5
  },

  {
    title: "亞洲金融風暴威脅全球金融體系",
    description: "1997年8月14日，源自泰銖債信的亞洲金融風暴愈演愈烈，席捲整個亞洲貨幣體系，更進一步威脅全球金融體系的穩定，迫使 IMF 出手解救泰國的財政問題。",
    s: -4, b: -3, x: -5, c: -4, r: -5
  },
  {
    title: "衍生金融之波克夏奇異資本黑洞",
    description: "2009年3月6日，美國財政部與央行聯手籌備不良資產收購紓困方案，對大型銀行做壓力測試，發現 AIG 紓困背後牽連波克夏與奇異的更大系統性風險黑洞。",
    s: -5, b:  3, x:  3, c: -5, r: -5
  },
  {
    title: "世界通訊企業假帳潰散投資信心",
    description: "2002年7月21日，繼安隆之後，第二大美國公司世界通訊爆發假帳風波並宣告史上最大破產案，讓全球投資者對美國企業的信心徹底消散。",
    s: -5, b:  4, x:  5, c: -3, r: -4
  },
  {
    title: "生技泡沫之基因圖譜資產化爭議",
    description: "2000年3月14日，美國總統柯林頓與英國首相布萊爾聯合聲明，提議 HGP 人類基因圖譜計畫的研究成果應屬公共資產，並開放後續研發使用。",
    s: -5, b: -2, x: -3, c:  2, r:  3
  },
  {
    title: "一九八七黑色星期一股市大崩盤",
    description: "1987年10月19日，三巫日結算後首次全球股市同步崩盤，道瓊狂瀉508點，單日跌幅高達22.6%。",
    s: -5, b:  5, x: -3, c: -4, r: -5
  },
  {
    title: "蘋果超級盃廣告震撼推出麥金塔",
    description: "1984年1月22日，蘋果在超級盃轉播中推出震撼廣告，以麥金塔電腦挑戰 IBM 的市場龍頭地位。",
    s:  5, b: -4, x: -5, c: -2, r: -3
  },
  {
    title: "華爾街日報宣告: 股票市場已死",
    description: "1974年12月6日，道瓊多次探底下滑，年底《華爾街日報》以「市場已死」為題，形容投資大眾信心完全潰敗。",
    s:  5, b: -5, x:  3, c:  5, r:  5
  },
  {
    title: "葛林斯潘警告股市之非理性繁榮",
    description: "1996年12月5日，聯準會主席葛林斯潘在演說中對可能引發資產泡沫的非理性繁榮表達深切憂慮，強調需未雨綢繆。",
    s: -4, b: -5, x: -3, c: -4, r: -4
  }
];





let student = {};
let round = 0;
let totalScore = 0;
let logs = [];
let currentCard = null;

function startGame() {
  const name = document.getElementById("name").value.trim();
  const id = document.getElementById("id").value.trim();
  if (!name || !id) return alert("請輸入姓名與學號");
  student = { name, id };
  round = 0;
  totalScore = 0;
  logs = [];
  document.getElementById("startSection").classList.add("hidden");
  document.getElementById("gameSection").classList.remove("hidden");
  document.getElementById("studentInfo").innerText = `${name}（${id}）`;
  document.getElementById("logList").innerHTML = '';
  document.getElementById("totalScore").innerText = '0';
  document.getElementById("result").innerText = '';
  drawCard();
}

function drawCard() {
  if (round >= 5) return;
  currentCard = cards[Math.floor(Math.random() * cards.length)];
  document.getElementById("cardTitle").innerText = currentCard.title;
  document.getElementById("cardDesc").innerText = currentCard.description;
  document.getElementById("cardBox").classList.remove("hidden");
}

function confirmTurn() {
  const s = parseFloat(document.getElementById("inputS").value || 0);
  const b = parseFloat(document.getElementById("inputB").value || 0);
  const x = parseFloat(document.getElementById("inputX").value || 0);
  const c = parseFloat(document.getElementById("inputC").value || 0);
  const r = parseFloat(document.getElementById("inputR").value || 0);
  const totalWeight = s + b + x + c + r;
  if (totalWeight > 5) {
    alert("加權總合不能超過 5！");
    return;
  }

  const score =
    currentCard.s * s +
    currentCard.b * b +
    currentCard.x * x +
    currentCard.c * c +
    currentCard.r * r;

  totalScore += score;
  round += 1;

  logs.push({
    round,
    cardTitle: currentCard.title,
    input: { s, b, x, c, r },
    base: currentCard,
    score
  });

  document.getElementById("roundCount").innerText = round;
  document.getElementById("totalScore").innerText = totalScore;
  document.getElementById("result").innerText = `第 ${round} 回合得分：${score}`;
  document.getElementById("logList").innerHTML += `
    <li>
      ${currentCard.title}：${score} 分<br />
      倍數：S(${currentCard.s} * ${s}), B(${currentCard.b} * ${b}), 
      X(${currentCard.x} * ${x}), C(${currentCard.c} * ${c}), R(${currentCard.r} * ${r})
    </li>`;
  document.getElementById("inputS").value =
    document.getElementById("inputB").value =
    document.getElementById("inputX").value =
    document.getElementById("inputC").value =
    document.getElementById("inputR").value = '';

  if (round >= 5) {
    document.getElementById("result").innerText += '（遊戲結束）';
    saveToLocal();
  } else {
    drawCard();
  }
}
function saveToLocal() {
  const data = JSON.parse(localStorage.getItem("records") || "[]");
  const timestamp = new Date().toLocaleString(); // 獲取當前時間
  data.push({
    name: student.name,
    id: student.id,
    total: totalScore,
    logs,
    time: timestamp // 新增紀錄時間
  });
  localStorage.setItem("records", JSON.stringify(data));
}


function restartGame() {
  location.reload();
}

function showAdmin() {
  document.getElementById("adminSection").classList.remove("hidden");
  const data = JSON.parse(localStorage.getItem("records") || "[]");
  let html = '<table><tr><th>姓名</th><th>學號</th><th>總分</th><th>紀錄時間</th><th>紀錄</th></tr>';
  for (const rec of data) {
    html += `<tr>
      <td>${rec.name}</td>
      <td>${rec.id}</td>
      <td>${rec.total}</td>
      <td>${rec.time || "無紀錄時間"}</td>
      <td><ul>${rec.logs.map(l => `<li>${l.cardTitle}: ${l.score}分</li>`).join("")}</ul></td>
    </tr>`;
  }
  html += "</table>";
  document.getElementById("recordTable").innerHTML = html;
}

function hideAdmin() {
  document.getElementById("adminSection").classList.add("hidden");
}

function exportCSV() {
  const data = JSON.parse(localStorage.getItem("records") || "[]");
  if (!data.length) return alert("沒有資料");
  let csv = "姓名,學號,總分,每回合紀錄\\n";
  for (const rec of data) {
    const rounds = rec.logs.map(l => `${l.cardTitle}(${l.score})`).join(" / ");
    csv += `${rec.name},${rec.id},${rec.total},"${rounds}"\\n`;
  }
  const blob = new Blob([csv], { type: "text/csv;charset=utf-8;" });
  const url = URL.createObjectURL(blob);
  const a = document.createElement("a");
  a.href = url;
  a.download = "金融戰略王成績紀錄.csv";
  a.click();
}
</script>

</body>
</html>


