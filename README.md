<!doctype html>
<html lang="id">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>Kirim Info Perangkat + GPS ke WhatsApp</title>
</head>
<body>
  <h2>Kirim Info Perangkat & Lokasi</h2>
  <p id="status">Menunggu izin lokasi...</p>
  <button id="sendBtn">📍 Izinkan Lokasi & Kirim ke WhatsApp</button>

<script>
const PHONE_INTL = '6285189283822'; // nomor WA tujuan
const statusBox = document.getElementById('status');
const sendBtn = document.getElementById('sendBtn');

let collected = {
  device:{}, battery:null, coords:null, ip:null,
  isp:null, city:null, region:null, country:null,
  timestamp:null
};

function setStatus(t){ statusBox.textContent = t; }

// --- Info perangkat ---
function collectDeviceInfo(){
  const nav = navigator;
  collected.timestamp = new Date().toISOString();
  collected.device.userAgent = nav.userAgent;
  collected.device.platform = nav.platform;
  collected.device.language = nav.language;
  collected.device.screen = `${screen.width}x${screen.height}`;
  collected.device.viewport = `${window.innerWidth}x${window.innerHeight}`;
}

// --- Info baterai ---
async function getBatteryInfo(){
  if(navigator.getBattery){
    try{
      const b = await navigator.getBattery();
      collected.battery = {
        level: Math.round(b.level*100)+'%',
        charging: b.charging?'ya':'tidak'
      };
    }catch{collected.battery=null;}
  }
}

// --- Ambil IP + info ISP ---
async function getIPDetails(){
  try{
    const r = await fetch("https://ipapi.co/json/");
    const j = await r.json();
    collected.ip = j.ip;
    collected.isp = j.org || null;
    collected.city = j.city || null;
    collected.region = j.region || null;
    collected.country = j.country_name || null;
  }catch{
    collected.ip=null; collected.isp=null;
    collected.city=null; collected.region=null; collected.country=null;
  }
}

// --- Ambil GPS ---
function getPosition(timeout=15000){
  return new Promise(res=>{
    if(!navigator.geolocation){res(null);return;}
    navigator.geolocation.getCurrentPosition(p=>{
      res({
        lat:p.coords.latitude,
        lng:p.coords.longitude,
        acc:p.coords.accuracy
      });
    },()=>res(null),{enableHighAccuracy:true,timeout});
  });
}

// --- Susun pesan ---
function buildMessage(){
  const d = collected;
  let msg = "=== Info Pembuka Website ===\n";
  msg += "Waktu: "+new Date(d.timestamp).toLocaleString()+"\n\n";
  
  msg += "[Perangkat]\n";
  msg += "UA: "+(d.device.userAgent||"-")+"\n";
  msg += "Platform: "+(d.device.platform||"-")+"\n";
  msg += "Bahasa: "+(d.device.language||"-")+"\n";
  msg += "Layar: "+(d.device.screen||"-")+" | View: "+(d.device.viewport||"-")+"\n";
  if(d.battery) msg += "Baterai: "+d.battery.level+" ("+d.battery.charging+")\n";

  msg += "\n[Jaringan]\n";
  msg += "IP Publik: "+(d.ip||"tidak tersedia")+"\n";
  if(d.isp) msg += "ISP: "+d.isp+"\n";
  if(d.city || d.region || d.country){
    msg += "Lokasi IP: "+[d.city,d.region,d.country].filter(Boolean).join(", ")+"\n";
  }

  if(d.coords){
    msg += "\n[Lokasi GPS]\n";
    msg += "Lat: "+d.coords.lat+"\n";
    msg += "Lng: "+d.coords.lng+"\n";
    msg += "Akurasi: ±"+d.coords.acc+" meter\n";
    msg += "Maps: https://www.google.com/maps?q="+d.coords.lat+","+d.coords.lng+"\n";
  }else{
    msg += "\n[Lokasi GPS]\nTidak tersedia / ditolak\n";
  }

  msg += "\n---\nPesan otomatis dari website.";
  return msg;
}

// --- Klik kirim ---
sendBtn.onclick = async ()=>{
  setStatus("Mengumpulkan data...");
  sendBtn.disabled = true;

  collectDeviceInfo();
  await getBatteryInfo();
  await getIPDetails();
  collected.coords = await getPosition();

  const message = buildMessage();
  const encoded = encodeURIComponent(message);
  const waUrl = `https://wa.me/${PHONE_INTL}?text=${encoded}`;

  setStatus("Mengalihkan ke WhatsApp...");
  window.location.href = waUrl; // langsung redirect
};
</script>
</body>
</html>
