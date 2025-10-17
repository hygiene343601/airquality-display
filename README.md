<!DOCTYPE html>
<html lang="zh-Hant">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>中山區空氣品質即時看板</title>
  <style>
    body {
      background-color: #2e7d32; /* 綠底 */
      color: white;
      font-family: "Noto Sans TC", sans-serif;
      display: flex;
      flex-direction: column;
      align-items: center;
      justify-content: center;
      height: 100vh;
      margin: 0;
      text-align: center;
    }
    h1 {
      font-size: 3rem;
      margin-bottom: 1rem;
    }
    .data {
      font-size: 2rem;
      margin: 0.5rem 0;
    }
    .status {
      font-size: 2.5rem;
      font-weight: bold;
      margin-top: 1rem;
    }
    .time {
      margin-top: 2rem;
      font-size: 1.2rem;
      opacity: 0.8;
    }
  </style>
</head>
<body>
  <h1>中山區空氣品質即時看板</h1>
  <div class="data" id="aqi">AQI：--</div>
  <div class="data" id="pm25">PM2.5：-- μg/m³</div>
  <div class="data" id="temp">溫度：-- °C</div>
  <div class="data" id="humi">濕度：-- %</div>
  <div class="status" id="status">讀取中...</div>
  <div class="time" id="update">更新時間：--</div>

  <script>
    async function loadData() {
      try {
        // 空氣品質 API
        const airRes = await fetch('https://data.epa.gov.tw/api/v1/aqx_p_432?api_key=9be8f185-8c2b-4c33-bf02-3c92b3c7e1ef&format=json');
        const airData = await airRes.json();
        const zhongshan = airData.records.find(r => r.sitename.includes('中山'));

        if (zhongshan) {
          document.getElementById('aqi').textContent = `AQI：${zhongshan.aqi}`;
          document.getElementById('pm25').textContent = `PM2.5：${zhongshan["pm2.5"]} μg/m³`;
          document.getElementById('status').textContent = zhongshan.status;
          document.getElementById('update').textContent = `更新時間：${zhongshan.publish_time}`;
        }

        // 氣象署即時觀測資料（臺北市平均）
        const weatherRes = await fetch('https://opendata.cwb.gov.tw/api/v1/rest/datastore/O-A0001-001?Authorization=CWB-3A932F2E-0C31-4A9A-B302-F01DF512F6C0&format=JSON');
        const weatherData = await weatherRes.json();
        const taipei = weatherData.records.location.find(l => l.locationName === '臺北');

        if (taipei) {
          const temp = taipei.weatherElement.find(e => e.elementName === 'TEMP').elementValue;
          const humi = taipei.weatherElement.find(e => e.elementName === 'HUMD').elementValue;
          document.getElementById('temp').textContent = `溫度：${parseFloat(temp).toFixed(1)} °C`;
          document.getElementById('humi').textContent = `濕度：${(parseFloat(humi) * 100).toFixed(0)} %`;
        }

      } catch (error) {
        console.error('資料載入錯誤：', error);
        document.getElementById('status').textContent = '資料讀取失敗';
      }
    }

    loadData();
    setInterval(loadData, 300000); // 每 5 分鐘更新一次
  </script>
</body>
</html>
