<!DOCTYPE html>
<html lang="zh-Hant">
<head>
    <meta charset="UTF-8">
    <title>材料領用借用表</title>
    <link href="https://fonts.googleapis.com/css2?family=Roboto:wght@400;700&display=swap" rel="stylesheet">
    <style>
        body {
            font-family: 'Roboto', '微軟正黑體', sans-serif;
            background: linear-gradient(to right, #f8fafc, #e2e8f0);
            margin: 0;
            padding: 0;
        }
        header {
            display: flex;
            align-items: center;
            background: #0053A6;
            color: #fff;
            padding: 12px 24px;
            border-bottom: 5px solid #008fda;
        }
        header img {
            height: 56px;
            margin-right: 20px;
            filter: drop-shadow(0 2px 2px #222);
        }
        h1 {
            letter-spacing: 2px;
            font-size: 2rem;
            margin: 0;
        }
        form {
            background: #fff;
            border-radius: 12px;
            max-width: 570px;
            margin: 32px auto 0 auto;
            box-shadow: 0 4px 30px rgba(0,40,60,0.10);
            padding: 24px;
        }
        label {
            display: inline-block;
            margin-top: 14px;
            font-weight: 700;
        }
        input[type="text"], input[type="number"], select {
            font-size: 1rem;
            margin-top: 8px;
            margin-bottom: 10px;
            width: 92%;
            padding: 6px 2%;
            border-radius: 6px;
            border: 1px solid #b6bbc6;
            background: #f8fafc;
        }
        input[type="number"]:invalid {
            border: 1.5px solid #E63946;
        }
        .error {
            color: #E63946;
            font-size: 0.97em;
            margin: 0 0 8px 0;
        }
        button {
            background: linear-gradient(90deg,#0053A6,#008fda);
            color: #fff;
            font-size: 1.12em;
            margin-top: 15px;
            border: none;
            border-radius: 6px;
            padding: 8px 22px;
            cursor: pointer;
            box-shadow: 0 2px 8px rgba(0,40,90,0.09);
            transition: background 0.3s;
        }
        button:hover { background: #008fda; }
        .summary {
            background: #0053A6;
            color: #fff;
            border-radius: 10px;
            margin: 21px 0 0 0;
            padding: 12px 14px;
            font-weight: 700;
        }
        .select-material {
            margin-bottom: 8px;
        }
    </style>
</head>
<body>
    <header>
        <img src="https://www.taipower.com.tw/themes/taipower/images/logo.svg" alt="台灣電力公司logo">
        <h1>材料領用借用表</h1>
    </header>

    <form id="materialForm" autocomplete="off">
        <label for="person">領料人：</label>
        <input type="text" id="person" placeholder="請輸入領料人姓名" required>

        <label for="category">材料類別：</label>
        <select id="category" onchange="updateMaterials()" required>
            <option value="">請選擇材料類別</option>
            <option value="A">A 類</option>
            <option value="B">B 類</option>
            <option value="C">C 類</option>
            <option value="F">F 類</option>
            <option value="H">H 類</option>
        </select>

        <label for="material">材料名稱：</label>
        <select id="material" class="select-material" required>
            <option value="">請先選擇類別</option>
        </select>

        <label for="rating">評價類型：</label>
        <input type="text" id="rating" placeholder="請輸入評價類型">

        <label for="unit">單位：</label>
        <input type="text" id="unit" placeholder="請輸入單位">

        <label for="quantity">領出數量：</label>
        <input type="number" id="quantity" min="1" placeholder="請輸入正整數" required>
        <div id="errorQty" class="error" style="display:none;">領出數量必須為正整數</div>

        <button type="submit">儲存並重新填寫</button>
    </form>
    <div class="summary" id="summary">
        今日各材料領用total：
        <ul id="totals"></ul>
    </div>
    <script>
    // 材料資料：完全展開所有分類與材料
    const materialDict = {
        "A": [
            "聚合柱型礙子 11.4kV",
            "懸垂礙子 C型 15000磅 φ10\" 灰色",
            "懸垂礙子 C型 10000磅 6\"φ 灰色",
            "防盬裝腳礙子 11.4KV 無腳",
            "軸型礙子 低壓 軸型",
            "拉線礙子 低壓 大型 L139.7㎜",
            "聚合柱型礙子 22.8KV",
            "聚合裝腳礙子 11.4KV 夾線型"
        ],
        "B": [
            "互連器 間接型A / 477MCM XLPE",
            "互連器 間接型B / AWG #2 XLPE",
            "互連繩 B型/ 112㎝",
            "互連繩 A型 / 52㎝",
            "互連器 夾條型A / 477MCM XLPE",
            "互連器 夾條型B / AWG #2 XLPE",
            "配電線路電桿反光貼紙 非鍍層立方角錐之微稜鏡反射材料 90x100㎝ 黃黑相間色",
            "配電線路電桿反光貼紙 非鍍層立方角錐之微稜鏡反射材料 90x60㎝ 黃黑相間色",
            "拉線夾板 銅線 100-150㎜²",
            "拉線夾板 鋁線 AWG 2/0-#2",
            "拉線夾板 鋁線 477MCM-300MCM-AWG 4/0",
            "V型掛鐵 A型無縫 SS400",
            "活線線夾 銅線 M8-60㎜2,T8-50㎜2",
            "H型壓接套管 鋁製 A#2,C60-38×A#2-#6,C38-14",
            "碳鋼擴張螺栓 S45C H內牙外迫式 5/8\"φx2-1/2\"",
            "電力保險絲組消音器 25KV 200A",
            "鉻酸鋅糊 1磅裝 材規Y413",
            "限流熔絲 15.5kV 40A 25kA 附抑弧開關",
            "電力保險絲 25kV 65E  12.5kA",
            "電力保險絲 25kV 125E  12.5kA",
            "開關箱 屋外型 不鏽鋼 600x400x230㎜ T2㎜",
            "壓接套管 銅 22㎜² 銅製",
            "壓接套管 銅 60㎜² 銅製",
            "壓接套管 銅 100㎜2 銅製",
            "C型壓接套管 銅製 銅14-22×8 α型",
            "C型壓接套管 銅製 銅14-38×14-38 β型",
            "電桿綁帶 複合電桿用B型 φ235×75×6㎜",
            "配電線路電桿反光標語片 CNS 4345感壓型第1級 33x12㎝自黏式Y193",
            "電桿牌 鋁 220×90×T0.511㎜ 藍底白字",
            "路燈開關 1P 240Vac 135A 非水銀式",
            "路燈開關 1P 240Vac 135A  電磁式",
            "成型支線夾條 GSW 55㎜2 襯圈型 L890㎜",
            "角型橫擔押 L1420㎜",
            "角型橫擔押 L1745㎜",
            "頂梢 11kV 防盬裝腳礙子 SS400",
            "低壓線架 三線式 鍍鋅鋼 附鐵閂",
            "低壓線架 五線式 鍍鋅鋼 附鐵閂",
            "鐵閂 單眼 φ5/8\"×1800㎜ 鐵鍍鋅",
            "立雕型線路指示器 單相 15kV 指示燈型",
            "支架 #1電纜固定帶用 L型 135×92.5×72×T3㎜ 304SST",
            "支架 500MCM電纜固定帶用 L型140x92.5x87xT3㎜ 304SST",
            "壓接端子 銅 250MCM 閉口型 短型 2孔 13㎜φ",
            "壓接端子 銅 500MCM 閉口型 長型 2孔 15㎜φ 屋外終端用",
            "螺栓鎖 鎖鉤:5㎜φ 鎖鉤:鋁 螺栓:SS400 栓涵:鎂鋁矽合金",
            "孔蓋定位標示器 66.3KHz 測深50㎝",
            "孔蓋定位標示器 169.8KHz 測深50㎝",
            "伸出鐵架 標準式 鍍鋅鋼",
            "被覆瓷罩 陶瓷 11.4KV桿上變壓器用 90φx95φxH95㎜",
            "鍍鋅方墊圈 SS400 M16 18㎜φx4.5㎜ L52xW52㎜",
            "鍍鋅方頭螺栓 SGD30 M16×P2.0×240㎜ 附單螺帽",
            "鍍鋅方頭螺栓 SGD30 M16×P2.0×300㎜ 附單螺帽",
            "低壓線架 單線式 鍍鋅鋼 附鐵閂",
            "低壓線架 單線式 鍍鋅鋼 附鐵閂",
            "拉線夾板 銅線 22-38㎜²",
            "橫擔梢 11kV 防盬裝腳礙子  SS400",
            "鍍鋅方頭螺栓 SGD290-D M16×P2.0×270㎜ 附單螺帽",
            "鍍鋅方頭螺栓 SGD30 M16×P2.0×350㎜ 附單螺帽",
            "鍍鋅方頭螺栓 SGD290-D M16×P2.0×450㎜ 附單螺帽",
            "棉紗手套 棉紗含棉量100％ L250㎜ 電工用",
            "拉線夾板 銅線 50-80㎜2",
            "鍍鋅方頭螺栓 SGD30 M16×P2.0×140㎜ 附單螺帽",
            "鍍鋅方頭螺栓 SGD30 M16×P2.0×210㎜ 附單螺帽"

            // ...如需更多可自行加入
        ],
        
            
            "C": [
            "25KV交連PE引接線 1C 銅 22㎜2 交連PE被覆 被覆顏色黑色",
            "600V PVC鋁風雨線 1C 鋁 AWG #2 藍色",
            "600V PVC鋁風雨線 1C 鋁 AWG #2 灰色",
            "600V PVC風雨線 1C 銅 8㎜2 黑色",
            "600V PVC風雨線 1C 銅 22㎜2 黑色",
            "600V PVC鋁風雨線 1C 鋁 AWG 4/0 藍色",
            "鋼心鋁線 AWG #2 6/1",
            "全鋁線 300MCM 19股",
            "硬銅絞線(一般用) 100㎜²",
            "600V PVC電線 1C 銅 60㎜2 黑色",
            "600V PVC電線 1C 銅 22㎜2 黑色",
            "600V PVC電線 1C 銅 8㎜2 黑色",
            "硬銅絞線(一般用) 60㎜²",
            "硬銅絞線(一般用) 22㎜²",
            "素材木橫擔 L1800×90×90㎜",
            "素材木橫擔 L1200×90×90㎜",
            "600V PVC風雨線 1C 銅 60㎜2 黑色",
            "600V PVC風雨線 1C 銅 125㎜2 黑色",
            "鍍鋅鋼絞線 55㎜² (φ3.2㎜×7股) 支線用 S撚",
            "鍍鋅鐵線 SWM BWG#8 (φ4.19㎜)",
            "輕鋼橫擔 L2400×75×75×T2.3㎜",
            "鍍鋅等邊角鋼 SS400 90×90×T10㎜",
            "管路封塞 RP-03型 80㎜φ 一般用",
            "管路封塞 RP-05型 125㎜φ 一般用",
            "管路封塞 RP-06型 150㎜φ 一般用"
            "電纜用絕緣膠膏帶 T3.175㎜xW38.1㎜xL152㎝",
            "自融性防水膠帶 T1.7㎜xW38㎜xL305㎝",
            "預張式電纜終端接頭 屋外型 25kV 1C 銅 500MCM",
            "預張式電纜終端接頭 屋外型 25kV 1C 銅 AWG #1",
            "套管插頭 25kV 200A AWG #1",
            "預鑄型電纜直線接頭 25kV 1C 銅 AWG #1",
            "預鑄型電纜直線接頭 25kV 1C 銅 500MCM",
            "插頭封套 25kV 200A AWG #1",
            "負載可切肘型端頭 25kV 200A AWG #1 有電壓測試點",
            "四路分歧插頭 25kV 200A AWG #1",
            "負載不可切肘型端頭 25kV 600A 500MCM 有電壓測試點",
            "四路分歧插頭 25kV 200A AWG #1 A類配電箱用",
            "雙通套管插頭 25kV 200A AWG #1",
            "雙通插頭 25kV 200A AWG #1",
            "灌膠式電纜分歧接頭 600V 1C 銅 AWG 2/0~250MCM",
            "接戶橫木 60×60㎜"
    





            // ...如需更多可自行加入
        ],
        
            "F": [
            "亭置式開關 3相 24kV 四路 2-600A 2-200A SF6絕緣低碳鋼型",
            "非晶質鐵心桿上變壓器 單相 6.9kV 240/120V 25kVA",
            "改良套管型桿上變壓器 單相 6.9kV 240/120V 50kVA",
            "非晶質鐵心桿上變壓器 單相 6.9kV 240/120V 50kVA",
            "非晶質鐵心桿上變壓器 單相 6.9kV 240/120V 100kVA",
            "密封型桿上變壓器 單相 6.9kV 240/120V 100kVA 不鏽鋼",
            "密封型桿上變壓器 單相 6.9kV 240/120V 50kVA 不鏽鋼",
            "桿上變壓器 單相 6.9kV 240/120V 50kVA",
            "桿上變壓器 單相 6.9kV 240/120V 25kVA",
            "密封型桿上變壓器 單相 6.9kV 240/120V 25kVA 不鏽鋼",
            "電容器用油開關 單相 14.4kV 200A 手動/電動 115Vac",
            "高壓電容器 單相 6.9kV 200kVAR 屋外型",
            "高壓電容器 單相 6.9kV 100kVAR 屋外型",
            "改良套管型桿上變壓器 單相 6.9kV 240/120V 167kVA",
            "桿上變壓器 單相 6.9/13.8kV 240/120V 25kVA 不銹鋼",
            "桿上變壓器 單相 6.9kV 240/120V 100kVA 不鏽鋼",
            "桿上變壓器 單相 6.9/13.8kV 240/120V 100kVA 不銹鋼",
            "亭置式變壓器 單相 6.9/13.8kV 25kVA 直式高燃點一般型附分接頭",
            "亭置式變壓器 單相 6.9/13.8kV 50kVA 直式高燃點一般型附分接頭",
            "改良套管型桿上變壓器 單相 6.9kV 240/120V 100kVA",
            "亭置式變壓器 單相 6.9/13.8kV 167kVA直式高燃點一般型附分接頭",
            "亭置式變壓器 單相 6.9/13.8kV 167kVA直式高燃點一般型附分接頭",
            "亭置式氣封開關 三相 24kV 四路 3-600A 1-200A SF6絕緣",
            "亭置式開關 3相 24kV 二路 2-200A SF6絕緣 低碳鋼型",
            "地下四路自動線路開關 三相 24kV 600A 2S2F 24V簡易型(I量測)",
            "亭置式氣封開關 三相 24kV 四路 2-600A 2-200A SF6絕緣",
            "地下四路自動線路開關 三相24kV600A 3S1F 24V標準型(V、I量測)",
            "高壓配電箱 屋外型 鐵 A類x1 單具組",
            "亭置式變壓器 單相 6.9/13.8kV 25kVA 直式附分接頭",
            "亭置式變壓器 單相 6.9/13.8kV 100kVA直式高燃點一般型附分接頭",
            "亭置式變壓器 單相 6.9/13.8kV 100kVA直式高燃點一般型附分接頭",
            "亭置式變壓器 三相 12/24kV 500kVA 高燃點一般型附分接頭",
            "高壓配電箱 屋外型 不鏽鋼 A類x1 單具組",
            "高壓配電箱 屋外型 不鏽鋼 A類x1 單具組",
            "預力混凝土基樁 預力電桿用 4000x200x40㎜",
            "高壓配電箱 屋外型 不鏽鋼 B類J型",
            "高壓配電箱 屋外型 不鏽鋼 B類P型",
            "高壓配電箱 屋外型 不鏽鋼 D類x1 單具組",
            "預力電桿 9m 配電 圓型 B級",
            "預力電桿 10.5m 配電 圓型 B級",
            "預力混凝土基樁 6000x250x50㎜",
            "預力混凝土基樁 6000x250x50㎜",
            "人孔蓋(含外蓋及蓋座)MC-150型外蓋φ908×80㎜蓋座φ1200×150㎜",
            "水泥腳木 空心圓形G3 φ150×1000㎜ 支線用",
            "水泥腳木 空心圓形G3 φ150×1000㎜ 支線用",
            "預力電桿 14m 配電 圓型 C級",
            "預力電桿 12m 配電 圓型 C級",
            "預力電桿 12m 配電 圓型 E級",
            "手孔蓋HR150外蓋1068×668×80㎜座1210×810×150㎜",
            "手孔蓋墊座 方型 1210x810x200㎜ 混凝土"
   






            // ...如需更多可自行加入
        ],
        "H": [
            "電力保險絲座 25KV 200A 端子式",
            "電力保險絲座 25KV 200A 套管式",
            "限流熔絲座 15.5KV 200A 套管式",
            "限流熔絲座 15.5KV 200A 端子式",
            "線路故障指示器 單相 280A(600A) 1a 2指示燈 電池",
            "線路故障指示器 單相 800A(1000A) 1a 2指示燈 電池",
            "線路故障指示器 單相 800A 1指示燈 電池 FIC OV-A 架空線路",
            "聚合型熔絲鏈開關 15/27kV 100A 12kA 屋外型  附被覆罩",
            "配電級瓷套管避雷器 9KV 10KA 氧化鋅 無雷擊計數器 防鹽型",
            "配電級瓷套管避雷器 9kV 10kA 氧化鋅 無雷擊計數器 一般型",
            "配電級聚合套管避雷器 18KV 10KA 氧化鋅 無雷擊計數器",
            "高壓被覆罩 PE 11.4kV #2 H壓接型接線環用",
            "高壓被覆罩 PE 11.4kV #2 拉線夾板用(圓形開口)",
            "高壓被覆罩 PE 11.4kV 477MCM H壓接型接線環用",
            "高壓被覆罩 PE 22.8kV 477C圓形開口 477MCM導線、拉線夾板用",
            "高壓被覆罩 PE 22.8kV 477H 477MCM導線、H壓接型接線環用",
            "高壓被覆罩 PE 22.8kV #2圓形開口 #2AWG導線、拉線夾板用",
            "高壓被覆罩 PE 22.8kV #2H #2AWG線、H壓接型接線環用",
            "被覆罩 矽橡膠 11.4/22.8kV 避雷器用",
            "手動型線路負載啟斷開關 三相 15kV 600A 屋外型 SF6",
            "架空自動線路開關 三相 25.8kV 600A 24Vdc 標準型閂鎖型無控制箱",
            "負載啟斷開關 三相 25.8kV 600A 屋外型 SF6 手動型",
            "熔絲鏈開關 7.8/15kV 100A 10kA 屋外型 不鏽鋼固定架附被覆罩型",
            "配電級聚合套管避雷器 9KV 10KA 氧化鋅 無雷擊計數器",
            "U型軋頭 UC-501",
            "碳鋼單眼螺帽 SGD290-D M16-N/A",
            "架空自動線路開關 三相 15kV 600A 24Vdc 標準型 閂鎖型無控制箱",
            "接地銅棒 φ5/8\"×L8' 附線夾",
            "熔絲鏈開關 7.8/15kV 100A 10kA 屋外型 開型"
            "U型軋頭 UC-501",
            "碳鋼單眼螺帽 SGD290-D M16-N/A",
            "架空自動線路開關 三相 15kV 600A 24Vdc 標準型 閂鎖型無控制箱",
            "架空自動線路開關 三相 15kV 600A 24Vdc 標準型 閂鎖型無控制箱",
            "接地銅棒 φ5/8"×L8' 附線夾",
            "接地銅棒 φ5/8"×L8' 附線夾",
            "熔絲鏈開關 7.8/15kV 100A 10kA 屋外型 開型",



            // ...如需更多可自行加入
        ]
    };

    let dailyRecords = {};

    function updateMaterials() {
        const cat = document.getElementById("category").value;
        const matSel = document.getElementById("material");
        matSel.innerHTML = '<option value="">請選擇材料</option>';
        if (cat && materialDict[cat]) {
            materialDict[cat].forEach((item, idx) => {
                let opt = document.createElement("option");
                opt.value = item;
                opt.text = item;
                matSel.appendChild(opt);
            });
        }
    }

    document.getElementById("materialForm").onsubmit = function(e) {
        e.preventDefault();

        const qty = document.getElementById("quantity").value;
        if (!(qty && !isNaN(qty) && Number(qty) > 0)) {
            document.getElementById("errorQty").style.display = "block";
            return false;
        }
        document.getElementById("errorQty").style.display = "none";

        const person = document.getElementById("person").value.trim();
        const category = document.getElementById("category").value;
        const material = document.getElementById("material").value;
        const rating = document.getElementById("rating").value.trim();
        const unit = document.getElementById("unit").value.trim();
        const quantity = Number(qty);

        const key = category + ":" + material;

        if (!dailyRecords[key]) dailyRecords[key] = 0;
        dailyRecords[key] += quantity;

        let lis = '';
        for (const k in dailyRecords) {
            lis += `<li>${k}：${dailyRecords[k]}</li>`;
        }
        document.getElementById("totals").innerHTML = lis;

        this.reset();
        document.getElementById("material").innerHTML = '<option value="">請先選擇類別</option>';
    }
    </script>
</body>
</html>

