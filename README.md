<!doctype html>
<html lang="en">
<head>
<meta charset="utf-8">
<title>Weather Forecast Dashboard</title>
<link href="https://fonts.googleapis.com/css2?family=Pixelify+Sans&display=swap" rel="stylesheet">
<style>
body {
    font-family: 'Pixelify Sans', sans-serif;
    background-color: #7fd4e0;
    color: #4a2c1b;
    margin: 0;
    padding: 0;
    text-align: center;
}
h1 {
    margin: 20px 0;
}
input, select, button {
    margin: 5px;
    padding: 6px 10px;
    font-family: 'Pixelify Sans', sans-serif;
}
.my-table {
    border-collapse: separate;
    border-spacing: 0;
    width: 80%;
    margin: 20px auto;
    box-shadow: 0 2px 10px rgba(0,0,0,0.1);
    border-radius: 12px;
    overflow: hidden;
}
.my-table th, .my-table td {
    text-align: center;
    padding: 12px;
}
.my-table th {
    background-color: #50b0d0;
    color: #4a2c1b;
}
.my-table tr:nth-child(even) {
    background-color: #9ed8e8;
}
.my-table tr:nth-child(odd) {
    background-color: #7fd4e0; 
}
.my-table tr:hover {
    background-color: #6bc0d4;
    cursor: pointer;
}
</style>
</head>
<body>
<h1>Weather Forecast Dashboard</h1>
<label>Latitude:</label>
<input id="lat" type="text" value="44.6995">
<label>Longitude:</label>
<input id="lon" type="text" value="-73.4529">
<label>Units:</label>
<select id="units">
    <option value="metric">Celsius</option>
    <option value="imperial">Fahrenheit</option>
</select>
<label>Forecast Days:</label>
<select id="days">
    <option value="1">1</option>
    <option value="3" selected>3</option>
    <option value="5">5</option>
    <option value="7">7</option>
</select>
<button id="fetchBtn">Get Forecast</button>
<div id="results">
    <table class="my-table">
        <thead>
            <tr>
                <th>Date</th>
                <th>High Temp</th>
                <th>Low Temp</th>
                <th>Weather</th>
            </tr>
        </thead>
        <tbody id="forecastBody">
        </tbody>
    </table>
</div>
<script>
const cToF = c => (c * 9/5) + 32;
const codeToEmoji = code => {
    if (code === 0) return '☀️';
    if (code >= 1 && code <= 3) return '⛅';
    if (code >= 45 && code <= 48) return '🌫️';
    if (code >= 51 && code <= 67) return '🌧️';
    if (code >= 71 && code <= 86) return '🌨️';
    if (code >= 95 && code <= 99) return '⛈️';
    return '🔆';
};

async function fetchForecast(lat, lon, days) {
    const url = `https://api.open-meteo.com/v1/forecast?latitude=${lat}&longitude=${lon}&daily=temperature_2m_max,temperature_2m_min,weathercode&forecast_days=${days}&timezone=auto`;
    const res = await fetch(url);
    if(!res.ok) throw new Error('Network error');
    return await res.json();
}

function formatForecastData(rawData, units) {
    let tmax = rawData.daily.temperature_2m_max;
    let tmin = rawData.daily.temperature_2m_min;
    const codes = rawData.daily.weathercode;
    const dates = rawData.daily.time;
    if (units === 'imperial') {
        tmax = tmax.map(cToF);
        tmin = tmin.map(cToF);
    }
    return dates.map((date, i) => ({
        date: date,
        high: tmax[i].toFixed(1),
        low: tmin[i].toFixed(1),
        icon: codeToEmoji(codes[i])
    }));
}

function makeTable(forecast) {
    const tbody = document.getElementById('forecastBody');
    tbody.innerHTML = '';
    forecast.forEach(day => {
        const row = document.createElement('tr');
        ['date', 'high', 'low', 'icon'].forEach(key => {
            const cell = document.createElement('td');
            cell.textContent = day[key];
            row.appendChild(cell);
        });
        tbody.appendChild(row);
    });
}

document.getElementById('fetchBtn').addEventListener('click', async () => {
    const lat = parseFloat(document.getElementById('lat').value);
    const lon = parseFloat(document.getElementById('lon').value);
    const units = document.getElementById('units').value;
    const days = parseInt(document.getElementById('days').value,10);
    try {
        const rawData = await fetchForecast(lat, lon, days);
        const forecast = formatForecastData(rawData, units);
        makeTable(forecast);
    } catch(err) {
        document.getElementById('forecastBody').innerHTML = '<tr><td colspan="4">Error fetching data.</td></tr>';
        console.error(err);
    }
});
</script>
</body>
</html>
