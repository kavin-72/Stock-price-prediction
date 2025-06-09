<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
  <title>Stock Price Predictor</title>
  <style>
    * {
      box-sizing: border-box;
      margin: 0;
      padding: 0;
    }

    body {
      font-family: 'Arial', sans-serif;
      background-color: #f1f2f6;
      display: flex;
      justify-content: center;
      align-items: center;
      height: 100vh;
      margin: 0;
      color: #333;
    }

    .container {
      background: #ffffff;
      padding: 30px;
      border-radius: 8px;
      box-shadow: 0 4px 10px rgba(0, 0, 0, 0.1);
      width: 100%;
      max-width: 600px;
    }

    h1 {
      font-size: 30px;
      color: #2d87f0;
      text-align: center;
      margin-bottom: 20px;
    }

    label {
      font-size: 16px;
      margin-bottom: 10px;
      display: block;
    }

    input[type="file"], input[type="date"], button {
      width: 100%;
      padding: 12px;
      margin: 10px 0;
      font-size: 16px;
      border: 1px solid #ddd;
      border-radius: 5px;
      background-color: #f7f7f7;
      transition: all 0.3s ease;
    }

    input[type="file"] {
      font-size: 14px;
    }

    button {
      background-color: #2d87f0;
      color: white;
      cursor: pointer;
      transition: background-color 0.3s ease;
    }

    button:hover {
      background-color: #1c6ec0;
    }

    .output {
      background: #f7f7f7;
      padding: 15px;
      border-radius: 5px;
      margin-top: 20px;
      font-size: 16px;
      line-height: 1.6;
    }

    .output strong {
      color: #2d87f0;
    }

    .output hr {
      margin: 10px 0;
      border: 0;
      border-top: 1px solid #ddd;
    }

    .alert {
      background: #ff4d4d;
      color: white;
      padding: 15px;
      border-radius: 5px;
      margin-top: 10px;
    }

  </style>
</head>
<body>
  <div class="container">
    <h1>📈 Stock Price Predictor</h1>

    <input type="file" id="fileInput" accept=".csv">
    <label for="dateInput">Enter Date to Predict (YYYY-MM-DD):</label>
    <input type="date" id="dateInput">
    <button onclick="predictPrice()">Predict</button>

    <div class="output" id="output"></div>
  </div>

  <script>
    let dates = [], prices = [], parsedDates = [];

    document.getElementById('fileInput').addEventListener('change', function(e) {
      const file = e.target.files[0];
      const reader = new FileReader();

      reader.onload = function(event) {
        const lines = event.target.result.trim().split(/\r?\n/).slice(1);
        dates = [];
        prices = [];
        parsedDates = [];

        lines.forEach(line => {
          const [dateStrRaw, priceStrRaw] = line.split(',');
          const dateStr = dateStrRaw.trim();
          const priceStr = priceStrRaw.trim();
          const timestamp = new Date(dateStr).getTime();
          const price = parseFloat(priceStr);

          if (!isNaN(timestamp) && !isNaN(price)) {
            dates.push(dateStr);
            parsedDates.push(timestamp);
            prices.push(price);
          }
        });

        if (dates.length > 0) {
          const latestDate = dates[dates.length - 1];
          const latestPrice = prices[prices.length - 1];

          document.getElementById("output").innerHTML = `
            ✅ <strong>Latest Data:</strong><br/>
            Date: <strong>${latestDate}</strong><br/>
            Price: <strong>${latestPrice}</strong><br/>
          `;
        } else {
          document.getElementById("output").innerHTML = `
            <div class="alert">❌ Invalid CSV file format.</div>
          `;
        }
      };

      reader.readAsText(file);
    });

    function predictPrice() {
      const dateStr = document.getElementById("dateInput").value;
      const targetTime = new Date(dateStr).getTime();

      if (!dateStr || isNaN(targetTime) || parsedDates.length === 0) {
        alert("Please select a valid CSV file and enter a valid date.");
        return;
      }

      // Linear regression: y = mx + c
      const n = parsedDates.length;
      const sumX = parsedDates.reduce((a, b) => a + b, 0);
      const sumY = prices.reduce((a, b) => a + b, 0);
      const sumXY = parsedDates.reduce((sum, x, i) => sum + x * prices[i], 0);
      const sumX2 = parsedDates.reduce((sum, x) => sum + x * x, 0);

      const denominator = (n * sumX2 - sumX * sumX);
      if (denominator === 0) {
        alert("Prediction not possible with current data.");
        return;
      }

      const m = (n * sumXY - sumX * sumY) / denominator;
      const c = (sumY - m * sumX) / n;
      const predicted = m * targetTime + c;

      document.getElementById("output").innerHTML += `
        <hr>
        📅 <strong>Prediction for ${dateStr}</strong><br/>
        Predicted Price: <strong>${predicted.toFixed(2)}</strong>
      `;
    }
  </script>
</body>
</html>
