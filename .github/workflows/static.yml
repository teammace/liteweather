<!doctype html>
<html>
<head>
<meta charset="utf-8" />
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>Weather Lite WAP</title>
<style>
body { font-family: monospace; background: #fff; color: #000; margin: 0; padding: 4px; }
h1 { font-size: 1em; margin: 0 0 4px 0; }
input, button { font-family: monospace; font-size: 1em; }
.small { font-size:0.8em; }
pre { white-space: pre-wrap; }
</style>
</head>
<body>
<h1>Weather Lite (BOM)</h1>
<form id="f">
  <input id="q" type="text" value="Melbourne" size="12">
  <button type="submit">Go</button>
</form>
<div id="status" class="small">Enter location</div>
<pre id="out"></pre>
<script>
async function bomSearch(q){
  const r = await fetch(`https://api.weather.bom.gov.au/v1/locations?search=${encodeURIComponent(q)}`);
  if(!r.ok) throw new Error('No BOM');
  const j = await r.json();
  if(!j.locations || !j.locations.length) throw new Error('No loc');
  return j.locations[0];
}
async function bomForecast(id){
  const r = await fetch(`https://api.weather.bom.gov.au/v1/locations/${id}/forecasts/daily`);
  if(!r.ok) throw new Error('No forecast');
  return r.json();
}
async function bomCurrent(id){
  const r = await fetch(`https://api.weather.bom.gov.au/v1/locations/${id}/observations`);
  if(!r.ok) throw new Error('No obs');
  return r.json();
}
async function openMeteo(lat,lon){
  const r = await fetch(`https://api.open-meteo.com/v1/forecast?latitude=${lat}&longitude=${lon}&current_weather=true&hourly=relativehumidity_2m,windspeed_10m&daily=temperature_2m_max,temperature_2m_min,weathercode&timezone=auto`);
  if(!r.ok) throw new Error('No meteo');
  return r.json();
}
async function go(e){
  if(e) e.preventDefault();
  const q = document.getElementById('q').value.trim();
  const status = document.getElementById('status');
  const out = document.getElementById('out');
  status.textContent='Loading...';
  out.textContent='';
  try {
    let loc, temp, desc, humidity, wind;
    try {
      loc = await bomSearch(q);
      const obsData = await bomCurrent(loc.id);
      const obs = obsData.observations.data[0];
      temp = obs.air_temp;
      desc = obs.wx_desc || '—';
      humidity = obs.rel_hum !== undefined ? obs.rel_hum + '%' : '—';
      wind = obs.wind_spd_kmh !== undefined ? obs.wind_spd_kmh + ' km/h ' + (obs.wind_dir || '') : '—';

      out.textContent = `${loc.name}
Temp: ${temp}°C
Cond: ${desc}
Humidity: ${humidity}
Wind: ${wind}

Forecast:`;

      const fc = await bomForecast(loc.id);
      if(fc && fc.data && fc.data.length){
        fc.data.slice(0,5).forEach(f=>{
          out.textContent += `
${f.date}: ${f.summary.text}
Min ${f.temp_min}°C / Max ${f.temp_max}°C`;
        });
      }

      status.textContent='BOM OK';
    } catch {
      status.textContent='Fallback';

      const geo = await fetch(`https://nominatim.openstreetmap.org/search?format=json&q=${encodeURIComponent(q)}&limit=1`);
      const g = await geo.json();
      if(!g[0]) throw new Error('No geo');

      const data = await openMeteo(g[0].lat, g[0].lon);
      temp = data.current_weather.temperature;
      wind = data.current_weather.windspeed + ' km/h';

      // humidity from hourly
      humidity = data.hourly?.relativehumidity_2m ? data.hourly.relativehumidity_2m[0] + '%' : '—';

      out.textContent = `${g[0].display_name.split(',')[0]}
Temp: ${temp}°C
Cond: (Est.)
Humidity: ${humidity}
Wind: ${wind}

Forecast:`;

      if(data.daily && data.daily.time){
        for(let i=0;i<Math.min(5,data.daily.time.length);i++){
          out.textContent += `
${data.daily.time[i]}: Min ${data.daily.temperature_2m_min[i]}°C / Max ${data.daily.temperature_2m_max[i]}°C`;
        }
      }
    }

  } catch(err){
    status.textContent='Err';
    out.textContent=err.message;
  }
}

document.getElementById('f').addEventListener('submit', go);
</script>
</body>
</html>
