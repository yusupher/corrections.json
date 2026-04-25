<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=yes">
  <title>Kwara Farm Advisor | Live Weather + Smart Advice</title>
  <!-- React & ReactDOM -->
  <script src="https://cdn.jsdelivr.net/npm/react@17.0.2/umd/react.development.js"></script>
  <script src="https://cdn.jsdelivr.net/npm/react-dom@17.0.2/umd/react-dom.development.js"></script>
  <!-- Babel for JSX -->
  <script src="https://cdn.jsdelivr.net/npm/@babel/standalone/babel.min.js"></script>
  <style>
    *{box-sizing:border-box;margin:0;padding:0;}
    body{font-family:'DM Sans',sans-serif;background:#f9f6f0;color:#1a1208;}
  </style>
</head>
<body>
  <div id="root"></div>

  <script type="text/babel">
    // ------------------------------------------------------------------
    // CSS STYLES
    // ------------------------------------------------------------------
    const css = `
      @import url('https://fonts.googleapis.com/css2?family=Playfair+Display:wght@700;900&family=DM+Sans:wght@300;400;500;600&family=DM+Mono:wght@400;500&display=swap');
      *{box-sizing:border-box;margin:0;padding:0;}
      body{font-family:'DM Sans',sans-serif;background:#f9f6f0;color:#1a1208;}
      ::-webkit-scrollbar{width:5px;}::-webkit-scrollbar-track{background:#ede4d0;}::-webkit-scrollbar-thumb{background:#c8860a;border-radius:3px;}
      @keyframes slideUp{from{opacity:0;transform:translateY(12px);}to{opacity:1;transform:translateY(0);}}
      @keyframes bounce{0%,80%,100%{transform:translateY(0);}40%{transform:translateY(-6px);}}
      .card-in{animation:slideUp .3s ease both;}
      .dot{display:inline-block;width:6px;height:6px;border-radius:50%;background:#c8860a;margin:0 2px;animation:bounce 1.1s infinite;}
      .dot:nth-child(2){animation-delay:.18s;}.dot:nth-child(3){animation-delay:.36s;}
    `;

    // ------------------------------------------------------------------
    // REAL WEATHER (OpenWeatherMap)
    // ------------------------------------------------------------------
    // Direct fetch by city name (more reliable for user)
    async function getWeatherByCity(city) {
      const API_KEY = "f11be083773d2765d183c4798bd8fdf9";
      const res = await fetch(`https://api.openweathermap.org/data/2.5/weather?q=${city},NG&units=metric&appid=${API_KEY}`);
      if (!res.ok) throw new Error("City not found");
      const data = await res.json();
      return {
        location: data.name,
        temp: Math.round(data.main.temp),
        humidity: data.main.humidity,
        wind: data.wind.speed,
        condition: data.weather?.[0]?.description || "clear",
        rain_chance: data.rain ? 80 : data.clouds?.all > 60 ? 50 : 10
      };
    }

    // ------------------------------------------------------------------
    // CROP DATABASE (full, same as original)
    // ------------------------------------------------------------------
    const CROP_DB = {
      "Ilorin East":{zone:"Peri-urban / Riverine",s1:[3,4,5],s2:[8,9],dry:["Tomato","Onion","Pepper"],crops:[
        {n:"Maize",v:"SAMMAZ15",yield:4.0,pri:"high",months:[3,4,8,9],fert:"NPK 15-15-15 (300kg/ha) + Urea (100kg/ha)",ph:"5.5–6.5",rain:"800–1500mm"},
        {n:"Rice",v:"FARO 44",yield:4.5,pri:"high",months:[5,6],fert:"NPK 20-10-10 (200kg/ha) + Urea (100kg/ha)",ph:"5.5–7.0",rain:"1000–2000mm"},
        {n:"Cassava",v:"TMS 30572",yield:18.0,pri:"high",months:[3,4,5],fert:"Organic 5t/ha + NPK (200kg/ha)",ph:"5.0–6.5",rain:"750–1400mm"},
        {n:"Cowpea",v:"IT99K-573-2-1",yield:1.5,pri:"med",months:[4,5,9],fert:"SSP only (150kg/ha)",ph:"5.5–7.0",rain:"600–1200mm"},
        {n:"Tomato",v:"F1 Hybrid",yield:15.0,pri:"high",months:[11,12,1,2],fert:"NPK (200kg/ha) + CAN + foliar",ph:"6.0–7.0",rain:"700–1200mm"},
      ]},
      "Ilorin West":{zone:"Urban Fringe",s1:[3,4,5],s2:[8,9],dry:["Leafy Vegetables","Pepper"],crops:[
        {n:"Maize",v:"SAMMAZ52",yield:4.5,pri:"high",months:[3,4,8,9],fert:"NPK 15-15-15 (300kg/ha) + Urea (100kg/ha)",ph:"5.5–6.5",rain:"800–1500mm"},
        {n:"Yam",v:"Puna White Yam",yield:15.0,pri:"high",months:[3,4],fert:"FYM 10t/ha + NPK (200kg/ha)",ph:"5.5–6.5",rain:"900–1500mm"},
        {n:"Sorghum",v:"SAMSORG 40",yield:3.0,pri:"med",months:[5,6,9],fert:"NPK 15-15-15 (150kg/ha)",ph:"5.5–7.5",rain:"500–1000mm"},
        {n:"Groundnut",v:"SAMNUT 22",yield:1.8,pri:"med",months:[5,6],fert:"SSP (200kg/ha)",ph:"5.5–7.0",rain:"600–1100mm"},
        {n:"Leafy Veg",v:"Amaranth/Tete",yield:8.0,pri:"high",months:[1,2,11,12],fert:"NPK (150kg/ha) + Poultry manure 3t/ha",ph:"6.0–7.0",rain:"600–1200mm"},
      ]},
      "Ilorin South":{zone:"Southern Guinea Savanna",s1:[3,4,5],s2:[8,9],dry:["Sweet Potato","Vegetables"],crops:[
        {n:"Maize",v:"SAMMAZ29",yield:3.8,pri:"high",months:[3,4,8,9],fert:"NPK 15-15-15 (300kg/ha) + Urea (100kg/ha)",ph:"5.5–6.5",rain:"850–1400mm"},
        {n:"Cassava",v:"TMS I 92/0067",yield:20.0,pri:"high",months:[3,4,5],fert:"Organic 5t/ha + NPK (200kg/ha)",ph:"5.0–6.5",rain:"700–1300mm"},
        {n:"Soybean",v:"TGX 1448-2E",yield:2.5,pri:"high",months:[5,6],fert:"Rhizobium + SSP (100kg/ha)",ph:"6.0–7.0",rain:"700–1200mm"},
        {n:"Yam",v:"Yellow Yam",yield:12.0,pri:"med",months:[3,4],fert:"FYM 10t/ha",ph:"5.5–6.5",rain:"900–1500mm"},
        {n:"Sweet Potato",v:"Ex-Igbariam",yield:10.0,pri:"med",months:[9,10],fert:"NPK 15-15-15 (200kg/ha)",ph:"5.5–6.5",rain:"750–1200mm"},
      ]},
      "Moro":{zone:"Northern Guinea Savanna",s1:[4,5],s2:[8,9],dry:["Onion","Tomato"],crops:[
        {n:"Maize",v:"SAMMAZ15",yield:3.5,pri:"high",months:[4,5],fert:"NPK 15-15-15 (300kg/ha) + Urea (100kg/ha)",ph:"5.5–6.5",rain:"800–1500mm"},
        {n:"Rice",v:"NERICA L-34",yield:4.0,pri:"high",months:[5,6],fert:"NPK 20-10-10 (200kg/ha) + Urea (100kg/ha)",ph:"5.5–7.0",rain:"1000–1800mm"},
        {n:"Sorghum",v:"SAMSORG 14",yield:2.8,pri:"med",months:[5,6],fert:"NPK 15-15-15 (150kg/ha)",ph:"5.5–7.5",rain:"450–900mm"},
        {n:"Cowpea",v:"SAMPEA 7",yield:1.5,pri:"med",months:[6,7],fert:"SSP (150kg/ha)",ph:"5.5–7.0",rain:"500–1000mm"},
        {n:"Cassava",v:"TMS 98/0581",yield:18.0,pri:"high",months:[4,5],fert:"Organic 5t/ha + NPK (200kg/ha)",ph:"5.0–6.5",rain:"700–1300mm"},
      ]},
      "Asa":{zone:"Northern Guinea Savanna",s1:[4,5],s2:[8,9],dry:[],crops:[
        {n:"Maize",v:"SAMMAZ52",yield:3.5,pri:"high",months:[4,5],fert:"NPK 15-15-15 (300kg/ha) + Urea (100kg/ha)",ph:"5.5–6.5",rain:"800–1400mm"},
        {n:"Groundnut",v:"SAMNUT 24",yield:2.0,pri:"high",months:[5,6],fert:"SSP (200kg/ha)",ph:"5.5–7.0",rain:"600–1100mm"},
        {n:"Sorghum",v:"SAMSORG 40",yield:3.0,pri:"med",months:[5,6,9],fert:"NPK 15-15-15 (150kg/ha)",ph:"5.5–7.5",rain:"450–900mm"},
        {n:"Yam",v:"Water Yam",yield:10.0,pri:"med",months:[3,4],fert:"Organic 8t/ha + NPK (200kg/ha)",ph:"5.5–6.5",rain:"900–1500mm"},
        {n:"Millet",v:"Ex-Borno",yield:1.5,pri:"low",months:[5,6],fert:"NPK 15-15-15 (100kg/ha)",ph:"5.0–7.0",rain:"400–800mm"},
      ]},
      "Kaiama":{zone:"Sudan / N. Guinea Savanna",s1:[5,6],s2:[],dry:[],crops:[
        {n:"Sorghum",v:"SAMSORG 17",yield:2.5,pri:"high",months:[5,6],fert:"NPK 15-15-15 (100kg/ha)",ph:"5.5–7.5",rain:"400–900mm"},
        {n:"Millet",v:"SOSAT-C88",yield:1.2,pri:"high",months:[5,6],fert:"Urea microdose (50kg/ha)",ph:"5.0–7.0",rain:"350–750mm"},
        {n:"Cowpea",v:"IT97K-499-35",yield:1.2,pri:"high",months:[6,7],fert:"SSP (100kg/ha)",ph:"5.5–7.0",rain:"400–850mm"},
        {n:"Groundnut",v:"SAMNUT 22",yield:1.5,pri:"med",months:[5,6],fert:"SSP (150kg/ha)",ph:"5.5–7.0",rain:"500–1000mm"},
        {n:"Maize",v:"SAMMAZ15",yield:2.5,pri:"med",months:[5,6],fert:"NPK 15-15-15 (200kg/ha) + Urea",ph:"5.5–6.5",rain:"700–1200mm"},
      ]},
      "Baruten":{zone:"Sudan Savanna",s1:[5,6],s2:[],dry:[],crops:[
        {n:"Millet",v:"SOSAT-C88",yield:1.0,pri:"high",months:[5,6],fert:"Urea microdose (25–50kg/ha)",ph:"5.0–7.0",rain:"350–750mm"},
        {n:"Sorghum",v:"SK 5912",yield:2.0,pri:"high",months:[5,6],fert:"NPK 15-15-15 (100kg/ha)",ph:"5.5–7.5",rain:"400–850mm"},
        {n:"Cowpea",v:"IT99K-573-1",yield:1.0,pri:"med",months:[6,7],fert:"SSP (100kg/ha)",ph:"5.5–7.0",rain:"400–850mm"},
        {n:"Groundnut",v:"SAMNUT 26",yield:1.3,pri:"med",months:[5,6],fert:"SSP (150kg/ha)",ph:"5.5–7.0",rain:"500–1000mm"},
        {n:"Sesame",v:"E8",yield:0.8,pri:"med",months:[6,7],fert:"NPK 15-15-15 (100kg/ha)",ph:"5.5–7.5",rain:"500–1000mm"},
      ]},
      "Ekiti (Kwara)":{zone:"Northern Guinea Savanna",s1:[3,4,5],s2:[8,9],dry:["Vegetables"],crops:[
        {n:"Yam",v:"White Yam (Efuru)",yield:12.0,pri:"high",months:[3,4],fert:"FYM 10t/ha + NPK (200kg/ha)",ph:"5.5–6.5",rain:"900–1500mm"},
        {n:"Cassava",v:"TMS 30572",yield:18.0,pri:"high",months:[3,4,5],fert:"Organic 5t/ha + NPK (200kg/ha)",ph:"5.0–6.5",rain:"700–1300mm"},
        {n:"Maize",v:"SAMMAZ15",yield:3.5,pri:"high",months:[4,5],fert:"NPK 15-15-15 (300kg/ha) + Urea (100kg/ha)",ph:"5.5–6.5",rain:"800–1400mm"},
        {n:"Cowpea",v:"SAMPEA 11",yield:1.5,pri:"med",months:[6,7],fert:"SSP (150kg/ha)",ph:"5.5–7.0",rain:"600–1100mm"},
        {n:"Sorghum",v:"SAMSORG 40",yield:2.8,pri:"med",months:[5,6],fert:"NPK 15-15-15 (150kg/ha)",ph:"5.5–7.5",rain:"500–1000mm"},
      ]},
      "Irepodun":{zone:"Northern Guinea Savanna",s1:[4,5],s2:[8,9],dry:["Onion","Tomato"],crops:[
        {n:"Maize",v:"SAMMAZ52",yield:4.0,pri:"high",months:[4,5],fert:"NPK 15-15-15 (300kg/ha) + Urea (100kg/ha)",ph:"5.5–6.5",rain:"800–1500mm"},
        {n:"Rice",v:"FARO 52",yield:4.5,pri:"high",months:[5,6],fert:"NPK 20-10-10 (200kg/ha) + Urea (100kg/ha)",ph:"5.5–7.0",rain:"1000–2000mm"},
        {n:"Sorghum",v:"SAMSORG 14",yield:2.8,pri:"med",months:[5,6],fert:"NPK 15-15-15 (150kg/ha)",ph:"5.5–7.5",rain:"450–900mm"},
        {n:"Yam",v:"Yellow Yam",yield:12.0,pri:"med",months:[3,4],fert:"Organic 10t/ha + NPK (200kg/ha)",ph:"5.5–6.5",rain:"900–1500mm"},
        {n:"Groundnut",v:"SAMNUT 22",yield:1.8,pri:"med",months:[5,6],fert:"SSP (200kg/ha)",ph:"5.5–7.0",rain:"600–1100mm"},
      ]},
      "Isin":{zone:"Northern Guinea Savanna",s1:[3,4,5],s2:[8,9],dry:["Vegetables"],crops:[
        {n:"Yam",v:"White Yam (Puna)",yield:12.0,pri:"high",months:[3,4],fert:"FYM 10t/ha",ph:"5.5–6.5",rain:"900–1500mm"},
        {n:"Cassava",v:"TME 419",yield:20.0,pri:"high",months:[3,4,5],fert:"Organic 5t/ha + NPK (200kg/ha)",ph:"5.0–6.5",rain:"700–1300mm"},
        {n:"Maize",v:"SAMMAZ29",yield:3.5,pri:"high",months:[4,5],fert:"NPK 15-15-15 (300kg/ha) + Urea (100kg/ha)",ph:"5.5–6.5",rain:"800–1400mm"},
        {n:"Cowpea",v:"IT99K-573-2-1",yield:1.5,pri:"med",months:[6,7],fert:"SSP (150kg/ha)",ph:"5.5–7.0",rain:"600–1100mm"},
        {n:"Sorghum",v:"SAMSORG 40",yield:2.5,pri:"med",months:[5,6],fert:"NPK 15-15-15 (150kg/ha)",ph:"5.5–7.5",rain:"500–900mm"},
      ]},
      "Oke-Ero":{zone:"Northern Guinea Savanna",s1:[4,5],s2:[8,9],dry:["Vegetables"],crops:[
        {n:"Maize",v:"SAMMAZ15",yield:3.5,pri:"high",months:[4,5],fert:"NPK 15-15-15 (300kg/ha) + Urea (100kg/ha)",ph:"5.5–6.5",rain:"800–1400mm"},
        {n:"Yam",v:"Yellow Yam",yield:10.0,pri:"high",months:[3,4],fert:"Organic 8t/ha + NPK (200kg/ha)",ph:"5.5–6.5",rain:"900–1500mm"},
        {n:"Cassava",v:"TMS 30572",yield:16.0,pri:"med",months:[4,5],fert:"Organic 5t/ha + NPK (200kg/ha)",ph:"5.0–6.5",rain:"700–1300mm"},
        {n:"Cowpea",v:"SAMPEA 7",yield:1.5,pri:"med",months:[6,7],fert:"SSP (150kg/ha)",ph:"5.5–7.0",rain:"600–1100mm"},
        {n:"Soybean",v:"TGX 1740-2F",yield:2.0,pri:"med",months:[5,6],fert:"Rhizobium + SSP (100kg/ha)",ph:"6.0–7.0",rain:"700–1200mm"},
      ]},
      "Offa":{zone:"Southern Guinea Savanna",s1:[3,4,5],s2:[8,9],dry:["Sweet Potato","Vegetables"],crops:[
        {n:"Maize",v:"SAMMAZ52",yield:4.0,pri:"high",months:[3,4,8,9],fert:"NPK 15-15-15 (300kg/ha) + Urea (100kg/ha)",ph:"5.5–6.5",rain:"900–1500mm"},
        {n:"Cassava",v:"TMS I 92/0067",yield:20.0,pri:"high",months:[3,4,5],fert:"Organic 5t/ha + NPK (200kg/ha)",ph:"5.0–6.5",rain:"800–1400mm"},
        {n:"Yam",v:"Water Yam",yield:12.0,pri:"high",months:[3,4],fert:"Organic 10t/ha + NPK (200kg/ha)",ph:"5.5–6.5",rain:"900–1500mm"},
        {n:"Cowpea",v:"IT97K-499-35",yield:1.8,pri:"med",months:[5,9],fert:"SSP (150kg/ha)",ph:"5.5–7.0",rain:"700–1200mm"},
        {n:"Soybean",v:"TGX 1448-2E",yield:2.5,pri:"high",months:[5,6],fert:"Rhizobium + SSP (150kg/ha)",ph:"6.0–7.0",rain:"800–1300mm"},
      ]},
      "Oyun":{zone:"Southern Guinea Savanna",s1:[3,4,5],s2:[8,9],dry:["Vegetables","Sweet Potato"],crops:[
        {n:"Maize",v:"SAMMAZ29",yield:3.8,pri:"high",months:[3,4,8,9],fert:"NPK 15-15-15 (300kg/ha) + Urea (100kg/ha)",ph:"5.5–6.5",rain:"850–1500mm"},
        {n:"Rice",v:"FARO 44",yield:4.0,pri:"high",months:[5,6],fert:"NPK 20-10-10 (200kg/ha) + Urea (100kg/ha)",ph:"5.5–7.0",rain:"1000–1800mm"},
        {n:"Cassava",v:"TME 419",yield:20.0,pri:"high",months:[3,4,5],fert:"Organic 5t/ha + NPK (200kg/ha)",ph:"5.0–6.5",rain:"800–1400mm"},
        {n:"Yam",v:"White Yam",yield:12.0,pri:"med",months:[3,4],fert:"FYM 10t/ha",ph:"5.5–6.5",rain:"900–1500mm"},
        {n:"Soybean",v:"TGX 1740-2F",yield:2.5,pri:"high",months:[5,6],fert:"Rhizobium + SSP (150kg/ha)",ph:"6.0–7.0",rain:"800–1300mm"},
      ]},
      "Patigi":{zone:"Riverine / N. Guinea Savanna",s1:[4,5],s2:[8,9],dry:["Tomato","Onion","Pepper","Vegetables"],crops:[
        {n:"Rice",v:"FARO 44",yield:5.0,pri:"high",months:[5,6],fert:"NPK 20-10-10 (200kg/ha) + Urea (100kg/ha)",ph:"5.5–7.0",rain:"1000–2000mm"},
        {n:"Tomato",v:"Tropimech",yield:20.0,pri:"high",months:[11,12,1,2],fert:"NPK (200kg/ha) + CAN (150kg/ha)",ph:"6.0–7.0",rain:"700–1200mm"},
        {n:"Onion",v:"Red Creole",yield:15.0,pri:"high",months:[11,12,1],fert:"NPK (300kg/ha) + CAN (200kg/ha)",ph:"6.0–7.0",rain:"500–1000mm"},
        {n:"Maize",v:"SAMMAZ15",yield:4.5,pri:"high",months:[4,5],fert:"NPK 15-15-15 (300kg/ha) + Urea (100kg/ha)",ph:"5.5–6.5",rain:"800–1500mm"},
        {n:"Sorghum",v:"SAMSORG 17",yield:2.8,pri:"med",months:[5,6],fert:"NPK 15-15-15 (150kg/ha)",ph:"5.5–7.5",rain:"450–900mm"},
      ]},
      "Edu":{zone:"Riverine / N. Guinea Savanna",s1:[4,5],s2:[8,9],dry:["Tomato","Vegetables","Onion"],crops:[
        {n:"Rice",v:"FARO 52",yield:5.0,pri:"high",months:[5,6],fert:"NPK 20-10-10 (200kg/ha) + Urea (100kg/ha)",ph:"5.5–7.0",rain:"1000–2000mm"},
        {n:"Sugarcane",v:"NCO 376",yield:60.0,pri:"high",months:[3,4],fert:"NPK 20-10-10 (500kg/ha) + Urea (200kg/ha)",ph:"6.0–7.5",rain:"1500–2500mm"},
        {n:"Maize",v:"SAMMAZ52",yield:4.5,pri:"high",months:[4,5],fert:"NPK 15-15-15 (300kg/ha) + Urea (100kg/ha)",ph:"5.5–6.5",rain:"800–1500mm"},
        {n:"Sorghum",v:"SAMSORG 14",yield:2.8,pri:"med",months:[5,6],fert:"NPK 15-15-15 (150kg/ha)",ph:"5.5–7.5",rain:"450–900mm"},
        {n:"Cowpea",v:"SAMPEA 11",yield:1.5,pri:"med",months:[8,9],fert:"SSP (150kg/ha)",ph:"5.5–7.0",rain:"500–1000mm"},
      ]},
      "Kaima":{zone:"Sudan / N. Guinea Savanna",s1:[5,6],s2:[],dry:[],crops:[
        {n:"Cotton",v:"SAMCOT 9",yield:1.5,pri:"high",months:[5,6],fert:"NPK 15-15-15 (200kg/ha) + Urea (100kg/ha)",ph:"6.0–7.5",rain:"700–1200mm"},
        {n:"Sorghum",v:"SAMSORG 40",yield:2.5,pri:"high",months:[5,6],fert:"NPK 15-15-15 (100kg/ha)",ph:"5.5–7.5",rain:"400–850mm"},
        {n:"Millet",v:"SOSAT-C88",yield:1.2,pri:"high",months:[5,6],fert:"Urea microdose (25–50kg/ha)",ph:"5.0–7.0",rain:"350–750mm"},
        {n:"Groundnut",v:"SAMNUT 26",yield:1.5,pri:"med",months:[5,6],fert:"SSP (150kg/ha)",ph:"5.5–7.0",rain:"500–1000mm"},
        {n:"Sesame",v:"Yandev 55",yield:0.8,pri:"med",months:[6,7],fert:"NPK 15-15-15 (100kg/ha)",ph:"5.5–7.5",rain:"500–1000mm"},
      ]},
    };
    const LGAS = Object.keys(CROP_DB);
    const MS = ["Jan","Feb","Mar","Apr","May","Jun","Jul","Aug","Sep","Oct","Nov","Dec"];
    const MF = ["January","February","March","April","May","June","July","August","September","October","November","December"];
    const RAIN = {
      "Sudan Savanna":[2,3,10,30,70,100,130,140,100,50,10,3],
      "Sudan / N. Guinea Savanna":[3,5,15,45,90,120,140,150,110,60,15,4],
      "Northern Guinea Savanna":[5,10,25,65,120,155,165,180,150,80,20,8],
      "Southern Guinea Savanna":[8,15,40,90,145,175,180,195,175,110,35,12],
      "Peri-urban / Riverine":[10,15,45,95,150,175,185,200,180,115,40,15],
      "Riverine / N. Guinea Savanna":[8,12,35,80,140,165,170,185,160,95,30,10],
      "Urban Fringe":[8,12,30,80,130,160,165,175,160,90,28,10],
    };

    function pStatus(d, m) {
      if (d.s1.includes(m)) return { label:"🟢 Season 1 open — plant now", bg:"#dcfce7", tc:"#166534" };
      if (d.s2.includes(m)) return { label:"🟢 Season 2 open — plant now", bg:"#dcfce7", tc:"#166534" };
      const p1 = d.s1.map(x=>x===1?12:x-1), p2 = d.s2.map(x=>x===1?12:x-1);
      if (p1.includes(m)||p2.includes(m)) return { label:"🟡 Prepare now — planting next month", bg:"#fef9c3", tc:"#854d0e" };
      if (d.dry.length&&(m>=11||m<=2)) return { label:"🔵 Dry season — irrigated crops only", bg:"#dbeafe", tc:"#1e3a5f" };
      return { label:"🔴 Off-season — land prep & soil amendment", bg:"#fee2e2", tc:"#991b1b" };
    }

    function Dots() { return <span><span className="dot"/><span className="dot"/><span className="dot"/></span>; }

    function Card({ icon, bg, title, sub, children, onClose, delay=0 }) {
      const ref = React.useRef();
      const close = () => {
        if (!ref.current) return;
        ref.current.style.transition = "all .22s ease";
        ref.current.style.opacity = "0";
        ref.current.style.transform = "translateY(8px)";
        setTimeout(() => { if (ref.current) ref.current.style.display = "none"; }, 230);
      };
      return (
        <div ref={ref} className="card-in" style={{background:"#fff",borderRadius:12,border:"1px solid #d4c4a0",boxShadow:"0 4px 20px rgba(26,18,8,.1)",overflow:"hidden",animationDelay:`${delay}s`}}>
          <div style={{display:"flex",alignItems:"center",gap:12,padding:"13px 15px 13px 18px",borderBottom:"1px solid #ede4d0"}}>
            <div style={{width:38,height:38,borderRadius:9,background:bg,display:"flex",alignItems:"center",justifyContent:"center",fontSize:"1.2rem",flexShrink:0}}>{icon}</div>
            <div style={{flex:1}}>
              <div style={{fontFamily:"'Playfair Display',serif",fontSize:".98rem",fontWeight:700}}>{title}</div>
              {sub&&<div style={{fontSize:".74rem",color:"#5c4a2a",marginTop:1}}>{sub}</div>}
            </div>
            <button onClick={close} title="Close" style={{width:26,height:26,borderRadius:"50%",border:"1.5px solid #d4c4a0",background:"#f9f6f0",cursor:"pointer",fontSize:".7rem",color:"#5c4a2a",display:"flex",alignItems:"center",justifyContent:"center",flexShrink:0}}
              onMouseEnter={e=>{e.currentTarget.style.background="#fee2e2";e.currentTarget.style.color="#991b1b";}}
              onMouseLeave={e=>{e.currentTarget.style.background="#f9f6f0";e.currentTarget.style.color="#5c4a2a";}}>✕</button>
          </div>
          <div style={{padding:18}}>{children}</div>
        </div>
      );
    }

    function WeatherPanel({ weather, status, onRefresh }) {
      const bar = {background:"linear-gradient(135deg,#0f2d4a,#1a4a6e 50%,#0f3320)",borderRadius:12,border:"1px solid rgba(74,158,202,.3)",marginBottom:20,overflow:"hidden"};
      if (status==="loading") return (
        <div style={{...bar,padding:22,color:"rgba(255,255,255,.55)",fontSize:".86rem",display:"flex",alignItems:"center",gap:10}}>
          <Dots/> Loading weather…
        </div>
      );
      if (status==="error"||!weather) return (
        <div style={{...bar,padding:18,color:"rgba(255,180,100,.85)",fontSize:".84rem",textAlign:"center"}}>
          ⚠️ Weather unavailable. Try a different city.
        </div>
      );
      const cond = weather.condition||"";
      const icon = cond.toLowerCase().includes("rain")||cond.toLowerCase().includes("shower")?"🌧️":cond.toLowerCase().includes("thunder")?"⛈️":cond.toLowerCase().includes("cloud")?"⛅":cond.toLowerCase().includes("fog")?"🌫️":"☀️";
      return (
        <div style={bar}>
          <div style={{padding:"11px 18px 9px",display:"flex",justifyContent:"space-between",alignItems:"center",borderBottom:"1px solid rgba(255,255,255,.08)"}}>
            <span style={{fontSize:".84rem",fontWeight:700,color:"#7dd3fc"}}>🌍 {weather.location}</span>
            <button onClick={onRefresh} style={{background:"none",border:"none",color:"#7dd3fc",cursor:"pointer",fontSize:".7rem"}}>⟳ Refresh</button>
          </div>
          <div style={{display:"grid",gridTemplateColumns:"repeat(4,1fr)"}}>
            {[
              {icon, val:`${weather.temp}°C`, label:"Temperature", sub:cond},
              {icon:"💧", val:`${weather.humidity}%`, label:"Humidity", sub:weather.humidity>70?"High":weather.humidity<40?"Low":"Moderate"},
              {icon:"🌧️", val:`${weather.rain_chance}%`, label:"Rain Today", sub:"probability"},
              {icon:"🌬️", val:`${weather.wind}km/h`, label:"Wind Speed", sub:""},
            ].map((m,i)=>(
              <div key={i} style={{padding:"11px 8px",textAlign:"center",borderRight:i<3?"1px solid rgba(255,255,255,.07)":"none"}}>
                <div style={{fontSize:"1.25rem",marginBottom:2}}>{m.icon}</div>
                <div style={{fontFamily:"'DM Mono',monospace",fontSize:"1.05rem",color:"#fff",fontWeight:500}}>{m.val}</div>
                <div style={{fontSize:".62rem",color:"rgba(255,255,255,.42)",textTransform:"uppercase",letterSpacing:".04em",marginTop:2}}>{m.label}</div>
                <div style={{fontSize:".62rem",color:"#7dd3fc",marginTop:1}}>{m.sub}</div>
              </div>
            ))}
          </div>
        </div>
      );
    }

    // ------------------------------------------------------------------
    // MAIN COMPONENT
    // ------------------------------------------------------------------
    function KwaraFarm() {
      const month = new Date().getMonth()+1;
      const [lga, setLga] = React.useState("");
      const [search, setSearch] = React.useState("");
      const [sugg, setSugg] = React.useState([]);
      const [farmSize, setFarmSize] = React.useState(2);
      const [soil, setSoil] = React.useState("unknown");
      const [water, setWater] = React.useState("rainfed");
      const [goal, setGoal] = React.useState("food_security");
      const [concern, setConcern] = React.useState("none");
      const [checks, setChecks] = React.useState({plant:true,crop:true,fert:true,risk:true,yield:false});
      const [cards, setCards] = React.useState([]);
      const [busy, setBusy] = React.useState(false);
      const [weather, setWeather] = React.useState(null);
      const [wxStatus, setWxStatus] = React.useState("loading");
      const [weatherCity, setWeatherCity] = React.useState("Ilorin");
      const cardId = React.useRef(0);

      // Function to fetch weather by city name
      const fetchWeather = async (city) => {
        setWxStatus("loading");
        try {
          const data = await getWeatherByCity(city);
          setWeather(data);
          setWxStatus("ok");
        } catch (e) {
          setWxStatus("error");
          setWeather(null);
        }
      };

      // When LGA changes, set a sensible default city (e.g., "Ilorin" for Ilorin LGAs)
      React.useEffect(() => {
        if (!lga) return;
        let baseCity = lga.split(" ")[0]; // "Ilorin" from "Ilorin East"
        if (baseCity === "Ekiti") baseCity = "Ilorin"; // fallback
        setWeatherCity(baseCity);
        fetchWeather(baseCity);
      }, [lga]);

      // Initial load: default Ilorin weather
      React.useEffect(() => {
        fetchWeather("Ilorin");
      }, []);

      const selectLGA = (name) => { setLga(name); setSearch(name); setSugg([]); };
      const onSearch = (v) => { setSearch(v); setSugg(v.length<1?[]:LGAS.filter(l=>l.toLowerCase().includes(v.toLowerCase())).slice(0,6)); };

      // Local rule‑based advice generator
      const generateLocalAdvice = () => {
        const d = CROP_DB[lga];
        const rain = RAIN[d.zone]||RAIN["Northern Guinea Savanna"];
        const curRain = rain[month-1];
        const ps = pStatus(d, month);
        
        let climateMsg = "";
        if (ps.label.includes("plant now")) {
          climateMsg = `✅ Excellent planting window in ${MF[month-1]}. Soil moisture is building (${curRain}mm this month). Plant early to maximise the season.`;
        } else if (ps.label.includes("Prepare")) {
          climateMsg = `⏳ Prepare land now – planting starts next month (${d.s1.length ? MF[d.s1[0]-1] : MF[d.s2[0]-1]}). Focus on clearing and compost.`;
        } else if (ps.label.includes("Dry season")) {
          climateMsg = `💧 Dry season – only feasible with irrigation. Grow ${d.dry.join(", ")} using water‑saving methods (drip, mulching).`;
        } else {
          climateMsg = `🔄 Off‑season – ideal for soil improvement: apply lime, plant cover crops (cowpea), and plan crop rotation.`;
        }

        let actions = [];
        if (ps.label.includes("plant now")) {
          actions = [
            `Prepare seedbeds and plant ${d.crops[0].n} (${d.crops[0].v}) – sow at 2‑3cm depth.`,
            `Apply ${d.crops[0].fert.split("(")[0]} at planting – adjust for your ${farmSize}ha.`,
            `Install light mulch to reduce evaporation and control weeds.`
          ];
        } else if (ps.label.includes("Prepare")) {
          actions = [
            `Clear land and incorporate 5t/ha of compost or poultry manure.`,
            `Test soil pH – if below 5.5, apply 1‑2t/ha of lime.`,
            `Order certified seeds: ${d.crops[0].n}, ${d.crops[1]?.n || d.crops[0].n}.`
          ];
        } else if (ps.label.includes("Dry season")) {
          actions = [
            `Set up drip irrigation or install water‑conserving systems.`,
            `Sow ${d.dry[0]} and ${d.dry[1] || "leafy vegetables"} in nursery trays.`,
            `Apply thick mulch (straw, grass) to keep soil temperature down.`
          ];
        } else {
          actions = [
            `Plant fast‑growing cover crops like cowpea or lablab to fix nitrogen.`,
            `Apply rock phosphate or SSP (150kg/ha) to build phosphorus for next season.`,
            `Plan crop rotation – avoid planting same family two years in a row.`
          ];
        }

        let riskMsg = "";
        const isDryZone = ["Baruten","Kaiama","Kaima"].includes(lga);
        const isFloodZone = ["Patigi","Edu"].includes(lga);
        if (isDryZone && (month >= 11 || month <= 2)) {
          riskMsg = "⚠️ High drought risk – use early‑maturing varieties (e.g., SAMMAZ 15 for maize). Install rainwater harvesting structures.";
        } else if (isFloodZone && month >= 6 && month <= 9) {
          riskMsg = "🌊 Flood risk high – avoid low‑lying fields, dig drainage channels, plant on raised beds.";
        } else if (month >= 5 && month <= 9) {
          riskMsg = "🐛 Pest pressure (fall armyworm, aphids) increases – scout twice weekly, use neem or recommended pesticides if threshold exceeded.";
        } else if (concern === "drought") {
          riskMsg = "🌵 You selected drought as a concern – adopt drought‑tolerant varieties (e.g., SAMMAZ 52) and practice mulching.";
        } else if (concern === "flooding") {
          riskMsg = "💧 Flooding concern – ensure proper drainage and plant on ridges or raised beds.";
        } else if (concern === "pest") {
          riskMsg = "🦗 Pest concern – implement integrated pest management: crop rotation, pheromone traps, and neem sprays.";
        } else {
          riskMsg = "⚠️ Moderate risk – maintain regular field inspection and keep equipment ready for sudden weather changes.";
        }

        let soilTip = "";
        if (soil === "Ultisol") {
          soilTip = "Ultisols are acidic – apply lime (1‑2 t/ha) 3 weeks before planting. Intercrop with pigeon pea to recycle nutrients.";
        } else if (soil === "Fluvisol") {
          soilTip = "Fluvisols are fertile but prone to flooding – drain excess water and avoid prolonged waterlogging. Use early maturing varieties.";
        } else if (soil === "Entisol") {
          soilTip = "Sandy soils need organic matter – apply 5t/ha of poultry manure or compost, and split fertiliser into 3 applications.";
        } else if (concern === "soil_acidity") {
          soilTip = "Soil acidity concern: broadcast agricultural lime (1‑2 t/ha) and incorporate well. Use acid‑tolerant crops like cassava.";
        } else {
          soilTip = "Alfisols respond well to balanced NPK. Add 3t/ha of compost to improve water holding capacity and reduce fertiliser need.";
        }

        return { climateMsg, actions, riskMsg, soilTip };
      };

      // Main getAdvice (no external API)
      const getAdvice = () => {
        if (!lga) return;
        const d = CROP_DB[lga]; if (!d) return;
        setBusy(true); setCards([]);

        const rain = RAIN[d.zone]||RAIN["Northern Guinea Savanna"];
        const annualRain = rain.reduce((a,b)=>a+b,0);
        const curRain = rain[month-1];
        const ps = pStatus(d, month);
        const barMax = Math.max(...rain);
        const newCards = [];
        const nid = () => ++cardId.current;

        // Planting card
        newCards.push({ id:nid(), icon:"📅", bg:ps.bg, title:`Planting — ${MF[month-1]}`, sub:`${lga} · ${d.zone}`,
          body:(
            <div>
              <div style={{borderRadius:9,padding:"12px 14px",background:ps.bg,color:ps.tc,fontWeight:600,marginBottom:checks.plant?14:0,fontSize:".88rem"}}>{ps.label}</div>
              {checks.plant&&<>
                <div style={{fontSize:".67rem",fontWeight:700,textTransform:"uppercase",letterSpacing:".09em",color:"#5c4a2a",marginBottom:8}}>Monthly Rainfall (mm)</div>
                <div style={{display:"grid",gridTemplateColumns:"repeat(12,1fr)",gap:2}}>
                  {MS.map(m=><div key={m} style={{textAlign:"center",fontSize:".58rem",color:"#5c4a2a",paddingBottom:2}}>{m}</div>)}
                  {rain.map((r,i)=>{
                    const h=Math.max(Math.round((r/barMax)*56),4);
                    const isNow=(i+1)===month,inS1=d.s1.includes(i+1),inS2=d.s2.includes(i+1),isDry=d.dry.length&&(i>=10||i<=1);
                    const bg=isNow?"#c8860a":inS1?"#2d6a1f":inS2?"#4a9eca":isDry?"#7ab648":"#e5e7eb";
                    return <div key={i} title={`${MS[i]}: ~${r}mm`} style={{height:h,background:bg,borderRadius:3,alignSelf:"end"}}/>;
                  })}
                </div>
                <div style={{display:"flex",gap:12,flexWrap:"wrap",marginTop:9}}>
                  {[["#c8860a","Now"],["#2d6a1f","Season 1"],...(d.s2.length?[["#4a9eca","Season 2"]]:[]),(d.dry.length?[["#7ab648","Dry Season"]]:null)].filter(Boolean).map(([c,l])=>(
                    <span key={l} style={{display:"flex",alignItems:"center",gap:5,fontSize:".7rem",color:"#5c4a2a"}}>
                      <span style={{width:10,height:10,borderRadius:2,background:c,display:"inline-block"}}/>{l}
                    </span>
                  ))}
                </div>
                <div style={{marginTop:8,fontSize:".76rem",color:"#5c4a2a"}}>Annual: ~{annualRain}mm · This month: ~{curRain}mm</div>
              </>}
            </div>
          )
        });

        // Crops card
        if (checks.crop) newCards.push({ id:nid(), icon:"🌱", bg:"#dcfce7", title:"Recommended Crops", sub:lga,
          body:(
            <div>
              <div style={{display:"grid",gridTemplateColumns:"repeat(auto-fill,minmax(175px,1fr))",gap:10}}>
                {d.crops.map((c,i)=>{
                  const p=c.pri==="high"?{bg:"#dcfce7",tc:"#166534",lbl:"✓ Best choice"}:c.pri==="med"?{bg:"#fef9c3",tc:"#854d0e",lbl:"◎ Suitable"}:{bg:"#fee2e2",tc:"#991b1b",lbl:"◇ Marginal"};
                  return (
                    <div key={i} style={{border:"1.5px solid #d4c4a0",borderRadius:9,padding:12,background:"#f9f6f0"}}>
                      <span style={{display:"inline-block",fontSize:".6rem",fontWeight:700,textTransform:"uppercase",letterSpacing:".04em",padding:"2px 7px",borderRadius:50,background:p.bg,color:p.tc,marginBottom:5}}>{p.lbl}</span>
                      <div style={{fontWeight:700,fontSize:".9rem"}}>{c.n}</div>
                      <div style={{fontSize:".72rem",color:"#5c4a2a",fontStyle:"italic",marginBottom:6}}>{c.v}</div>
                      <div style={{fontFamily:"'DM Mono',monospace",fontSize:".78rem",color:"#2d6a1f",fontWeight:600}}>↑ {c.yield} t/ha</div>
                      <div style={{fontSize:".68rem",color:"#6b3a1f",marginTop:3}}>🧪 {c.fert.split("(")[0].trim()}</div>
                      <div style={{fontSize:".66rem",color:"#5c4a2a",marginTop:3}}>pH {c.ph} · {c.rain}</div>
                      <div style={{fontSize:".66rem",color:"#4a9eca",marginTop:2}}>🗓 {c.months.map(m=>MS[m-1]).join(", ")}</div>
                    </div>
                  );
                })}
              </div>
              {d.dry.length>0&&water!=="rainfed"&&<div style={{marginTop:12,borderRadius:9,padding:"11px 13px",background:"#dbeafe",border:"1px solid #60a5fa",color:"#1e3a5f",fontSize:".83rem"}}>💧 <strong>Dry season (with irrigation):</strong> {d.dry.join(", ")} — November to February</div>}
            </div>
          )
        });

        // Fertilizer card
        if (checks.fert) {
          const tip = soil==="Ultisol"?"⚠️ Ultisol soils are acidic — apply lime (1–2 t/ha) before fertilizer.":soil==="Fluvisol"?"✅ Fluvisol soils are fertile — reduce NPK by 20%.":soil==="Entisol"?"⚠️ Sandy soils — split fertilizer into 3 applications.":"ℹ️ Alfisol responds well to balanced NPK. Add organic matter.";
          newCards.push({ id:nid(), icon:"🧪", bg:"#fef3c7", title:"Fertilizer Plan", sub:`Per hectare · ${lga}`,
            body:(
              <div>
                {d.crops.slice(0,3).map((c,i)=>(
                  <div key={i} style={{padding:"9px 0",borderBottom:i<2?"1px solid #ede4d0":"none"}}>
                    <div style={{fontWeight:700,fontSize:".87rem"}}>{c.n} <span style={{fontWeight:400,color:"#5c4a2a",fontSize:".76rem"}}>({c.v})</span></div>
                    <div style={{fontSize:".81rem",marginTop:3,color:"#6b3a1f"}}>📦 {c.fert}</div>
                    <div style={{fontSize:".72rem",color:"#5c4a2a",marginTop:2}}>For {farmSize} ha · pH: {c.ph}</div>
                  </div>
                ))}
                <div style={{marginTop:11,borderRadius:9,padding:"11px 13px",background:"#fef3c7",border:"1px solid #fbbf24",color:"#78350f",fontSize:".83rem"}}>🌍 {tip}</div>
              </div>
            )
          });
        }

        // Risk card
        if (checks.risk) {
          const isN=["Baruten","Kaiama","Kaima"].includes(lga), isR=["Patigi","Edu"].includes(lga);
          const risks=[{l:"Drought Risk",v:isN?85:isR?30:45},{l:"Flood Risk",v:isR?80:isN?10:30},{l:"Soil Degradation",v:["Oke-Ero","Isin"].includes(lga)?65:40},{l:"Pest Pressure",v:month>=5&&month<=9?70:35}];
          newCards.push({ id:nid(), icon:"⚠️", bg:"#fee2e2", title:"Climate Risk", sub:`${lga} · ${MF[month-1]}`,
            body:(
              <div>
                {risks.map((r,i)=>{
                  const col=r.v>60?"#ef4444":r.v>35?"#f59e0b":"#4ade80";
                  return (
                    <div key={i} style={{display:"flex",alignItems:"center",gap:10,marginBottom:9}}>
                      <div style={{width:112,fontSize:".78rem",color:"#5c4a2a",flexShrink:0}}>{r.l}</div>
                      <div style={{flex:1,background:"#ede4d0",borderRadius:50,height:8,overflow:"hidden"}}>
                        <div style={{width:`${r.v}%`,height:"100%",background:col,borderRadius:50}}/>
                      </div>
                      <div style={{width:34,fontFamily:"'DM Mono',monospace",fontSize:".74rem",fontWeight:600,textAlign:"right",color:col}}>{r.v}%</div>
                    </div>
                  );
                })}
                {isR&&<div style={{borderRadius:9,padding:"10px 13px",background:"#fef3c7",border:"1px solid #fbbf24",color:"#78350f",fontSize:".82rem",marginTop:8}}>🌊 Flood plain — avoid low-lying plots June–September.</div>}
                {isN&&<div style={{borderRadius:9,padding:"10px 13px",background:"#fef3c7",border:"1px solid #fbbf24",color:"#78350f",fontSize:".82rem",marginTop:8}}>☀️ Drought zone — use early-maturing varieties and mulch.</div>}
              </div>
            )
          });
        }

        // Yield card
        if (checks.yield) {
          const prices={Rice:650000,Maize:280000,Cassava:90000,Yam:250000,Soybean:450000,Tomato:200000,Onion:300000,Groundnut:500000,Sugarcane:30000,Cotton:600000,Sorghum:200000,Cowpea:700000,Millet:180000,Sesame:800000};
          newCards.push({ id:nid(), icon:"📈", bg:"#dcfce7", title:"Yield & Revenue", sub:`${farmSize}ha · ${lga}`,
            body:(
              <div style={{overflowX:"auto"}}>
                <table style={{width:"100%",borderCollapse:"collapse",fontSize:".82rem"}}>
                  <thead><tr style={{background:"#1a1208",color:"#fff"}}>{["Crop","Yield/ha","Total","Revenue (₦)"].map(h=><th key={h} style={{padding:"8px 11px",textAlign:h==="Crop"?"left":"right"}}>{h}</th>)}</table></thead>
                  <tbody>{d.crops.slice(0,3).map((c,i)=>{
                    const pk=Object.keys(prices).find(p=>c.n.toLowerCase().includes(p.toLowerCase()))||"Maize";
                    const rev=Math.round(c.yield*farmSize*(prices[pk]||200000));
                    return <tr key={i} style={{background:i%2?"#fff":"#f9f6f0"}}>
                      <td style={{padding:"7px 11px",fontWeight:600}}>{c.n}</td>
                      <td style={{padding:"7px 11px",textAlign:"right",fontFamily:"'DM Mono',monospace"}}>{c.yield}t</td>
                      <td style={{padding:"7px 11px",textAlign:"right",fontFamily:"'DM Mono',monospace"}}>{(c.yield*farmSize).toFixed(1)}t</td>
                      <td style={{padding:"7px 11px",textAlign:"right",fontFamily:"'DM Mono',monospace",color:"#2d6a1f",fontWeight:700}}>₦{rev.toLocaleString()}</td>
                    </tr>;
                  })}</tbody>
                </table>
                <div style={{fontSize:".7rem",color:"#5c4a2a",marginTop:7}}>* Average Kwara market prices.</div>
              </div>
            )
          });
        }

        // Add local advice card
        const localAdvice = generateLocalAdvice();
        newCards.push({ id:nid(), icon:"🤖", bg:"linear-gradient(135deg,#dbeafe,#dcfce7)", title:"Smart Farm Advisor", sub:`${lga} · ${MF[month-1]}`,
          body: (
            <div style={{fontSize:".89rem",lineHeight:1.78,color:"#1a1208"}}>
              <p><strong>🌡️ Climate note:</strong> {localAdvice.climateMsg}</p>
              <p style={{marginTop:12}}><strong>⚡ Top 3 actions (next 2 weeks):</strong></p>
              <ol style={{marginLeft:20,marginTop:6}}>{localAdvice.actions.map((a,i)=><li key={i} style={{marginBottom:6}}>{a}</li>)}</ol>
              <p style={{marginTop:12}}><strong>⚠️ Main risk & handling:</strong> {localAdvice.riskMsg}</p>
              <p style={{marginTop:12}}><strong>🌱 Soil tip:</strong> {localAdvice.soilTip}</p>
            </div>
          )
        });

        setCards(newCards);
        setBusy(false);
      };

      const inp = {width:"100%",padding:"9px 12px",borderRadius:8,border:"1.5px solid #d4c4a0",fontFamily:"'DM Sans',sans-serif",fontSize:".9rem",color:"#1a1208",background:"#fff",outline:"none"};
      const sel = {...inp,appearance:"none",background:"#f9f6f0",backgroundImage:`url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' width='14' height='14' viewBox='0 0 24 24' fill='none' stroke='%235c4a2a' stroke-width='2'%3E%3Cpath d='M6 9l6 6 6-6'/%3E%3C/svg%3E")`,backgroundRepeat:"no-repeat",backgroundPosition:"right 10px center"};
      const lbl = {display:"block",fontSize:".76rem",fontWeight:600,color:"#5c4a2a",textTransform:"uppercase",letterSpacing:".06em",marginBottom:5,marginTop:13};

      return (
        <>
          <style>{css}</style>
          <div style={{background:"linear-gradient(135deg,#1a1208,#2d1f0a 40%,#2d6a1f)",padding:"40px 22px 48px",position:"relative",overflow:"hidden"}}>
            <div style={{position:"absolute",inset:0,opacity:.04,backgroundImage:`url("data:image/svg+xml,%3Csvg width='60' height='60' viewBox='0 0 60 60' xmlns='http://www.w3.org/2000/svg'%3E%3Cg fill='%23fff'%3E%3Cpath d='M36 34v-4h-2v4h-4v2h4v4h2v-4h4v-2h-4zm0-30V0h-2v4h-4v2h4v4h2V6h4V4h-4zM6 34v-4H4v4H0v2h4v4h2v-4h4v-2H6zM6 4V0H4v4H0v2h4v4h2V6h4V4H6z'/%3E%3C/g%3E%3C/svg%3E")`}}/>
            <div style={{maxWidth:1000,margin:"0 auto",position:"relative"}}>
              <div style={{display:"inline-flex",alignItems:"center",gap:7,background:"rgba(232,168,32,.15)",border:"1px solid rgba(232,168,32,.35)",color:"#e8a820",fontSize:11,fontWeight:600,letterSpacing:".08em",textTransform:"uppercase",padding:"5px 13px",borderRadius:50,marginBottom:18}}>🌾 Kwara State Agricultural Advisory</div>
              <h1 style={{fontFamily:"'Playfair Display',serif",fontSize:"clamp(1.9rem,4.5vw,3rem)",fontWeight:900,color:"#fff",lineHeight:1.1,marginBottom:11}}>Farm Smarter with <span style={{color:"#e8a820"}}>Climate Intelligence</span></h1>
              <p style={{color:"rgba(255,255,255,.6)",fontSize:".98rem",maxWidth:520,lineHeight:1.6,marginBottom:24}}>Select your LGA, then get personalised planting, fertiliser, and risk advice.</p>
              <div style={{display:"flex",gap:26,flexWrap:"wrap"}}>
                {[["16","LGAs"],["Auto","Location"],["Live","Weather"],["AI","Advice"]].map(([v,l])=>(
                  <div key={l}><strong style={{display:"block",fontFamily:"'DM Mono',monospace",fontSize:"1.35rem",color:"#e8a820"}}>{v}</strong><span style={{fontSize:".76rem",color:"rgba(255,255,255,.5)",textTransform:"uppercase",letterSpacing:".06em"}}>{l}</span></div>
                ))}
              </div>
            </div>
          </div>

          <div style={{maxWidth:1000,margin:"0 auto",padding:"32px 18px"}}>
            <div style={{display:"grid",gridTemplateColumns:"310px 1fr",gap:22,alignItems:"start"}}>

              {/* LEFT PANEL */}
              <div style={{background:"#fff",borderRadius:12,border:"1px solid #d4c4a0",boxShadow:"0 4px 20px rgba(26,18,8,.1)",overflow:"hidden"}}>
                <div style={{background:"#1a1208",color:"#fff",padding:"15px 18px",display:"flex",alignItems:"center",gap:9}}>
                  <span>🗺️</span><span style={{fontFamily:"'Playfair Display',serif",fontSize:"1.05rem"}}>Your Farm Profile</span>
                </div>
                <div style={{padding:18}}>

                  {/* Manual weather search */}
                  <div style={{background:"#e0f2fe",borderRadius:9,padding:13,marginBottom:12,border:"1px solid #7dd3fc"}}>
                    <div style={{fontSize:".7rem",fontWeight:700,textTransform:"uppercase",letterSpacing:".08em",color:"#0f2d4a",marginBottom:8}}>🌤️ Weather Location</div>
                    <div style={{display:"flex",gap:6}}>
                      <input style={{...inp,background:"#fff"}} placeholder="City, e.g. Ilorin" value={weatherCity} onChange={e=>setWeatherCity(e.target.value)} onKeyPress={e=>e.key==='Enter'&&fetchWeather(weatherCity)}/>
                      <button onClick={()=>fetchWeather(weatherCity)} style={{padding:"9px 11px",borderRadius:8,border:"none",background:"#0284c7",color:"#fff",fontWeight:700,cursor:"pointer",fontSize:".8rem"}}>Get</button>
                    </div>
                    <div style={{fontSize:".68rem",color:"#0f2d4a",marginTop:4}}>Type your city (e.g., Ilorin) and click Get.</div>
                  </div>

                  {/* LGA SEARCH */}
                  <div style={{background:"#ede4d0",borderRadius:9,padding:13,marginBottom:2,border:"1px solid #d4c4a0"}}>
                    <div style={{fontSize:".7rem",fontWeight:700,textTransform:"uppercase",letterSpacing:".08em",color:"#5c4a2a",marginBottom:8}}>📍 Search LGA</div>
                    <div style={{position:"relative"}}>
                      <div style={{display:"flex",gap:6}}>
                        <input style={{...inp,flex:1}} placeholder="Type LGA name…" value={search} onChange={e=>onSearch(e.target.value)} onKeyDown={e=>e.key==="Escape"&&setSugg([])}/>
                        <button onClick={()=>{const m=LGAS.find(l=>l.toLowerCase().includes(search.toLowerCase()));if(m)selectLGA(m);}} style={{padding:"9px 11px",borderRadius:8,border:"none",background:"#c8860a",color:"#fff",fontWeight:700,cursor:"pointer",fontSize:".8rem"}}>Go</button>
                      </div>
                      {sugg.length>0&&<div style={{position:"absolute",zIndex:200,width:"100%",background:"#fff",border:"1.5px solid #d4c4a0",borderRadius:8,boxShadow:"0 4px 20px rgba(26,18,8,.12)",maxHeight:180,overflowY:"auto"}}>
                        {sugg.map((l,i)=><div key={i} onClick={()=>selectLGA(l)} style={{padding:"8px 12px",cursor:"pointer",borderBottom:"1px solid #f5f0e8",fontSize:".86rem",fontWeight:600}} onMouseEnter={e=>e.currentTarget.style.background="#f5f0e8"} onMouseLeave={e=>e.currentTarget.style.background=""}>{l}</div>)}
                      </div>}
                    </div>
                  </div>

                  <label style={lbl}>Or Select LGA</label>
                  <select style={sel} value={lga} onChange={e=>selectLGA(e.target.value)}>
                    <option value="">— Select LGA —</option>
                    {LGAS.map(l=><option key={l}>{l}</option>)}
                  </select>

                  <label style={lbl}>Farm Size: <span style={{fontFamily:"'DM Mono',monospace",color:"#c8860a"}}>{farmSize} ha</span></label>
                  <input type="range" min="0.5" max="50" step="0.5" value={farmSize} onChange={e=>setFarmSize(+e.target.value)} style={{width:"100%",marginTop:3}}/>

                  <label style={lbl}>Soil Type</label>
                  <select style={sel} value={soil} onChange={e=>setSoil(e.target.value)}>
                    <option value="unknown">Not Sure / Mixed</option>
                    <option value="Alfisol">Alfisol (Sandy-Loam)</option>
                    <option value="Ultisol">Ultisol (Leached/Acidic)</option>
                    <option value="Fluvisol">Fluvisol (Riverine)</option>
                    <option value="Entisol">Entisol (Sandy)</option>
                    <option value="Vertisol">Vertisol (Heavy Clay)</option>
                  </select>

                  <label style={lbl}>Water Source</label>
                  <select style={sel} value={water} onChange={e=>setWater(e.target.value)}>
                    <option value="rainfed">Rain-fed Only</option>
                    <option value="irrigation">Irrigation Available</option>
                    <option value="riverine">Riverine/Flood Plain</option>
                  </select>

                  <label style={lbl}>Goal</label>
                  <select style={sel} value={goal} onChange={e=>setGoal(e.target.value)}>
                    <option value="food_security">Food Security</option>
                    <option value="commercial">Commercial / Market</option>
                    <option value="mixed">Mixed</option>
                    <option value="export">Export / Processing</option>
                  </select>

                  <label style={lbl}>Advice Needed</label>
                  {[["plant","🗓️ When to plant"],["crop","🌱 What to grow"],["fert","🧪 Fertilizer plan"],["risk","⚠️ Climate risk"],["yield","📈 Yield estimate"]].map(([k,l])=>(
                    <label key={k} style={{display:"flex",alignItems:"center",gap:8,marginTop:5,fontSize:".84rem",cursor:"pointer",padding:"5px 7px",borderRadius:7}} onMouseEnter={e=>e.currentTarget.style.background="#f5f0e8"} onMouseLeave={e=>e.currentTarget.style.background=""}>
                      <input type="checkbox" checked={checks[k]} onChange={e=>setChecks(p=>({...p,[k]:e.target.checked}))} style={{accentColor:"#c8860a",width:14,height:14}}/>{l}
                    </label>
                  ))}

                  <label style={lbl}>Concern</label>
                  <select style={sel} value={concern} onChange={e=>setConcern(e.target.value)}>
                    <option value="none">None</option>
                    <option value="drought">Drought / Dry spell</option>
                    <option value="flooding">Flooding</option>
                    <option value="low_yield">Low yields</option>
                    <option value="pest">Pest & disease</option>
                    <option value="soil_acidity">Soil acidity</option>
                    <option value="market">Market / price</option>
                  </select>

                  <button onClick={getAdvice} disabled={busy||!lga} style={{width:"100%",marginTop:16,padding:13,background:busy||!lga?"rgba(200,134,10,.35)":"linear-gradient(135deg,#c8860a,#e8a820)",border:"none",borderRadius:10,color:"#1a1208",fontFamily:"'DM Sans',sans-serif",fontSize:".97rem",fontWeight:700,cursor:busy||!lga?"not-allowed":"pointer",boxShadow:busy||!lga?"none":"0 4px 14px rgba(200,134,10,.35)"}}>
                    {busy?<span style={{display:"flex",alignItems:"center",justifyContent:"center",gap:8}}><Dots/> Generating…</span>:"🤖 Get AI Farming Advice"}
                  </button>
                </div>
              </div>

              {/* RIGHT PANEL */}
              <div>
                <WeatherPanel weather={weather} status={wxStatus} onRefresh={()=>fetchWeather(weatherCity)}/>

                {cards.length>1&&<button onClick={()=>setCards([])} style={{display:"flex",alignItems:"center",gap:7,padding:"8px 13px",background:"#fff",border:"1.5px solid #d4c4a0",borderRadius:8,fontSize:".78rem",fontWeight:600,color:"#5c4a2a",cursor:"pointer",marginBottom:12}} onMouseEnter={e=>{e.currentTarget.style.borderColor="#fca5a5";e.currentTarget.style.color="#991b1b";}} onMouseLeave={e=>{e.currentTarget.style.borderColor="#d4c4a0";e.currentTarget.style.color="#5c4a2a";}}>✕ Close All</button>}

                <div style={{display:"flex",flexDirection:"column",gap:16}}>
                  {cards.length===0&&!busy&&(
                    <div style={{border:"2px dashed #d4c4a0",borderRadius:12,padding:"52px 20px",textAlign:"center",color:"#5c4a2a"}}>
                      <div style={{fontSize:"2.8rem",marginBottom:12,opacity:.3}}>🌾</div>
                      <p style={{fontSize:".9rem",maxWidth:340,margin:"0 auto",lineHeight:1.6}}>{lga?`Ready for ${lga}. Click "Get AI Farming Advice".`:"Select your LGA and click Get AI Farming Advice."}</p>
                    </div>
                  )}
                  {busy&&cards.length===0&&<div style={{background:"#fff",borderRadius:12,border:"1px solid #d4c4a0",padding:24,display:"flex",alignItems:"center",gap:11,color:"#5c4a2a",fontSize:".88rem"}}><Dots/> Generating advice for {lga}…</div>}
                  {cards.map((card,i)=>(
                    <Card key={card.id} icon={card.icon} bg={card.bg} title={card.title} sub={card.sub} delay={i*.05}>{card.body}</Card>
                  ))}
                </div>
              </div>
            </div>
          </div>
        </>
      );
    }

    ReactDOM.render(<KwaraFarm />, document.getElementById("root"));
  </script>
</body>
</html>
