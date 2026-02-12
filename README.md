<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>BTC Realtime Dashboard</title>

<script src="https://cdn.jsdelivr.net/npm/chart.js"></script>

<style>
body{background:#0f172a;color:#e5e7eb;font-family:Arial;padding:20px}
.container{max-width:1100px;margin:auto}
.card{background:#1e293b;padding:20px;border-radius:12px;margin-bottom:20px}
canvas{height:260px!important}
.summary{display:grid;grid-template-columns:repeat(4,1fr);gap:15px;text-align:center}
.green{color:#10b981}
.red{color:#ef4444}
</style>
</head>

<body>

<div class="container">
<h2>BTCUSDT (Binance Spot 1m)</h2>

<div class="card">
<canvas id="priceChart"></canvas>
</div>

<div class="card">
<canvas id="rsiChart"></canvas>
</div>

<div class="card">
<canvas id="macdChart"></canvas>
</div>

<div class="card summary">
<div>
<p>Price</p>
<h3 id="price">--</h3>
</div>
<div>
<p>Change</p>
<h3 id="change">--</h3>
</div>
<div>
<p>Support</p>
<h3 id="support">--</h3>
</div>
<div>
<p>Resistance</p>
<h3 id="resistance">--</h3>
</div>
</div>
</div>

<script>
let prices=[]
let labels=[]
let maxPoints=120

const priceChart=new Chart(priceChart={
type:"line",
data:{labels,datasets:[
{label:"Price",data:[],borderColor:"#3b82f6"},
{label:"MA20",data:[],borderColor:"#10b981"}
]}
})

const rsiChart=new Chart(rsiChart={
type:"line",
data:{labels,datasets:[
{label:"RSI",data:[],borderColor:"#ef4444"}
]},
options:{scales:{y:{min:0,max:100}}}
})

const macdChart=new Chart(macdChart={
data:{labels,datasets:[
{type:"bar",label:"MACD",data:[],backgroundColor:"#6366f1"},
{type:"line",label:"Signal",data:[],borderColor:"#f59e0b"}
]}
})

function MA(arr,p){
return arr.map((_,i)=>{
if(i<p-1) return null
let s=0
for(let j=i-p+1;j<=i;j++) s+=arr[j]
return s/p
})
}

function EMA(arr,p){
let k=2/(p+1)
let ema=[arr[0]]
for(let i=1;i<arr.length;i++)
ema.push(arr[i]*k+ema[i-1]*(1-k))
return ema
}

function RSI(arr,p=14){
let out=Array(p).fill(null)
for(let i=p;i<arr.length;i++){
let g=0,l=0
for(let j=i-p+1;j<=i;j++){
let d=arr[j]-arr[j-1]
d>=0?g+=d:l-=d
}
let rs=g/l
out.push(100-(100/(1+rs)))
}
return out
}

function MACD(arr){
let m=EMA(arr,12).map((v,i)=>v-EMA(arr,26)[i])
let s=EMA(m,9)
return {m,s}
}

const ws=new WebSocket("wss://stream.binance.com:9443/ws/btcusdt@kline_1m")

ws.onmessage=(e)=>{
const d=JSON.parse(e.data).k
const price=parseFloat(d.c)

prices.push(price)
labels.push("")

if(prices.length>maxPoints){
prices.shift()
labels.shift()
}

const ma=MA(prices,20)
const rsi=RSI(prices)
const macd=MACD(prices)

priceChart.data.datasets[0].data=prices
priceChart.data.datasets[1].data=ma
priceChart.update()

rsiChart.data.datasets[0].data=rsi
rsiChart.update()

macdChart.data.datasets[0].data=macd.m
macdChart.data.datasets[1].data=macd.s
macdChart.update()

document.getElementById("price").textContent="$"+price.toFixed(2)

if(prices.length>1){
let ch=((price-prices[prices.length-2])/prices[prices.length-2]*100).toFixed(2)
let el=document.getElementById("change")
el.textContent=(ch>=0?"+":"")+ch+"%"
el.className=ch>=0?"green":"red"
}

document.getElementById("support").textContent="$"+(price*0.99).toFixed(2)
document.getElementById("resistance").textContent="$"+(price*1.01).toFixed(2)
}
</script>

</body>
</html>
