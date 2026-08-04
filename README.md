<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<title>SLOP MACHINE</title>
<style>
  body {
    background: #000a0a;
    color: #b8ff2b;
    font-family: 'Courier New', monospace;
    padding: 20px;
    max-width: 900px;
    margin: 0 auto;
  }
  h1 {
    font-size: 22px;
    letter-spacing: 2px;
    border-bottom: 2px dashed #b8ff2b;
    padding-bottom: 10px;
  }
  .sub { color: #666; font-size: 12px; margin-bottom: 20px; }
  .row { display: flex; gap: 10px; flex-wrap: wrap; margin-bottom: 12px; align-items: center; }
  label { font-size: 12px; }
  input[type=file] {
    background: #1a1a1a;
    border: 1px solid #444;
    color: #b8ff2b;
    padding: 6px;
    font-size: 11px;
  }
  button {
    background: #b8ff2b;
    color: #0a0a0a;
    border: none;
    padding: 10px 16px;
    font-weight: bold;
    font-family: inherit;
    cursor: pointer;
    letter-spacing: 1px;
  }
  button:hover { background: #d4ff6b; }
  button:active { transform: translateY(1px); }
  .ops {
    display: flex;
    flex-wrap: wrap;
    gap: 6px;
    margin: 16px 0;
  }
  .op-btn {
    background: #1a1a1a;
    color: #b8ff2b;
    border: 1px solid #b8ff2b;
    padding: 8px 10px;
    font-size: 11px;
  }
  .op-btn:hover { background: #b8ff2b; color: #0a0a0a; }
  canvas {
    border: 2px solid #333;
    max-width: 100%;
    display: block;
    margin-top: 12px;
    image-rendering: pixelated;
  }
  .log {
    font-size: 11px;
    color: #666;
    height: 60px;
    overflow-y: auto;
    border: 1px solid #222;
    padding: 8px;
    margin-top: 10px;
    background: #050505;
  }
  .status { font-size: 11px; color: #b8ff2b; margin-top: 6px; }
</style>
</head>
<body>

<h1>SLOP MACHINE</h1>
<div class="sub">no vision. no intent. no understanding. just matrix ops.</div>

<div class="row">
  <label>IMAGE A:</label>
  <input type="file" id="imgA" accept="image/*">
  <label>IMAGE B:</label>
  <input type="file" id="imgB" accept="image/*">
  <button id="loadBtn">LOAD</button>
</div>

<div class="ops">
  <button class="op-btn" data-op="xor">XOR BLEND</button>
  <button class="op-btn" data-op="channelswap">CHANNEL SWAP</button>
  <button class="op-btn" data-op="pixelsort">PIXEL SORT</button>
  <button class="op-btn" data-op="rowshift">ROW SHIFT</button>
  <button class="op-btn" data-op="noise">NOISE INJECT</button>
  <button class="op-btn" data-op="avg">AVERAGE MERGE</button>
  <button class="op-btn" data-op="tile">MOD TILE</button>
  <button class="op-btn" data-op="bitcrush">BITCRUSH</button>
  <button class="op-btn" data-op="scramble">BLOCK SCRAMBLE</button>
  <button class="op-btn" data-op="feedback">FEEDBACK SMEAR</button>
  <button class="op-btn" id="resetBtn" style="border-color:#ff4444;color:#ff4444">RESET</button>
  <button class="op-btn" id="saveBtn" style="border-color:#4488ff;color:#4488ff">SAVE PNG</button>
</div>

<canvas id="canvas" width="500" height="500"></canvas>
<div class="status" id="status">awaiting input. nothing has been chosen yet.</div>
<div class="log" id="log">&gt; system idle</div>

<script>
const canvas = document.getElementById('canvas');
const ctx = canvas.getContext('2d');
const statusEl = document.getElementById('status');
const logEl = document.getElementById('log');

let imgAData = null, imgBData = null;
let W = 500, H = 500;

function log(msg) {
  logEl.innerHTML += '<br>&gt; ' + msg;
  logEl.scrollTop = logEl.scrollHeight;
}

function loadImageFile(file) {
  return new Promise((resolve) => {
    const img = new Image();
    img.onload = () => resolve(img);
    img.src = URL.createObjectURL(file);
  });
}

document.getElementById('loadBtn').onclick = async () => {
  const fA = document.getElementById('imgA').files[0];
  const fB = document.getElementById('imgB').files[0];
  if (!fA) { log('no image A provided. machine confused.'); return; }

  const imgA = await loadImageFile(fA);
  W = 500; H = 500;
  canvas.width = W; canvas.height = H;

  ctx.drawImage(imgA, 0, 0, W, H);
  imgAData = ctx.getImageData(0, 0, W, H);

  if (fB) {
    const imgB = await loadImageFile(fB);
    const tmp = document.createElement('canvas');
    tmp.width = W; tmp.height = H;
    const tctx = tmp.getContext('2d');
    tctx.drawImage(imgB, 0, 0, W, H);
    imgBData = tctx.getImageData(0, 0, W, H);
    log('two images loaded. abomination pending.');
  } else {
    imgBData = null;
    log('one image loaded. self-mangling only.');
  }

  ctx.putImageData(imgAData, 0, 0);
  statusEl.textContent = 'raw material acquired. press buttons to violate it.';
};

function getCurrent() {
  return ctx.getImageData(0, 0, W, H);
}

const ops = {
  xor(data) {
    if (!imgBData) { log('need image B for XOR. machine refuses.'); return data; }
    const a = data.data, b = imgBData.data;
    for (let i = 0; i < a.length; i += 4) {
      a[i] ^= b[i];
      a[i+1] ^= b[i+1];
      a[i+2] ^= b[i+2];
    }
    log('xor blend executed. no meaning was considered.');
    return data;
  },
  channelswap(data) {
    const d = data.data;
    for (let i = 0; i < d.length; i += 4) {
      const r = d[i], g = d[i+1], bl = d[i+2];
      d[i] = g; d[i+1] = bl; d[i+2] = r;
    }
    log('channels rotated R->G->B->R. arbitrary.');
    return data;
  },
  pixelsort(data) {
    const d = data.data;
    for (let y = 0; y < H; y++) {
      const row = [];
      for (let x = 0; x < W; x++) {
        const i = (y * W + x) * 4;
        row.push([d[i], d[i+1], d[i+2], d[i+3]]);
      }
      row.sort((p1, p2) => (p1[0]+p1[1]+p1[2]) - (p2[0]+p2[1]+p2[2]));
      for (let x = 0; x < W; x++) {
        const i = (y * W + x) * 4;
        d[i] = row[x][0]; d[i+1] = row[x][1]; d[i+2] = row[x][2]; d[i+3] = row[x][3];
      }
    }
    log('pixels sorted by brightness. structure abolished.');
    return data;
  },
  rowshift(data) {
    const d = data.data;
    const out = new Uint8ClampedArray(d);
    for (let y = 0; y < H; y++) {
      const shift = Math.floor(Math.sin(y * 0.1) * 30);
      for (let x = 0; x < W; x++) {
        const srcX = ((x - shift) % W + W) % W;
        const si = (y * W + srcX) * 4;
        const di = (y * W + x) * 4;
        out[di] = d[si]; out[di+1] = d[si+1]; out[di+2] = d[si+2]; out[di+3] = d[si+3];
      }
    }
    log('rows shifted sinusoidally. no reason given.');
    return new ImageData(out, W, H);
  },
  noise(data) {
    const d = data.data;
    for (let i = 0; i < d.length; i += 4) {
      if (Math.random() < 0.15) {
        d[i] = Math.random() * 255;
        d[i+1] = Math.random() * 255;
        d[i+2] = Math.random() * 255;
      }
    }
    log('random noise injected. entropy increases.');
    return data;
  },
  avg(data) {
    if (!imgBData) { log('need image B for merge. machine refuses.'); return data; }
    const a = data.data, b = imgBData.data;
    for (let i = 0; i < a.length; i += 4) {
      a[i] = (a[i] + b[i]) / 2;
      a[i+1] = (a[i+1] + b[i+1]) / 2;
      a[i+2] = (a[i+2] + b[i+2]) / 2;
    }
    log('pixel-wise average taken. two things became one thing.');
    return data;
  },
  tile(data) {
    const d = data.data;
    const out = new Uint8ClampedArray(d);
    const mod = 64;
    for (let y = 0; y < H; y++) {
      for (let x = 0; x < W; x++) {
        const sx = x % mod === 0 ? W - x - 1 : x;
        const sy = y % mod === 0 ? H - y - 1 : y;
        const si = (sy * W + sx) * 4;
        const di = (y * W + x) * 4;
        out[di] = d[si]; out[di+1] = d[si+1]; out[di+2] = d[si+2]; out[di+3] = d[si+3];
      }
    }
    log('modular tiling applied. periodicity injected.');
    return new ImageData(out, W, H);
  },
  bitcrush(data) {
    const d = data.data;
    const levels = 4;
    const step = 255 / (levels - 1);
    for (let i = 0; i < d.length; i += 4) {
      d[i] = Math.round(d[i] / step) * step;
      d[i+1] = Math.round(d[i+1] / step) * step;
      d[i+2] = Math.round(d[i+2] / step) * step;
    }
    log('color depth crushed to 4 levels per channel.');
    return data;
  },
  scramble(data) {
    const d = data.data;
    const out = new Uint8ClampedArray(d);
    const block = 40;
    const cols = Math.ceil(W / block), rows = Math.ceil(H / block);
    const positions = [];
    for (let r = 0; r < rows; r++) for (let c = 0; c < cols; c++) positions.push([r, c]);
    const shuffled = [...positions].sort(() => Math.random() - 0.5);

    for (let idx = 0; idx < positions.length; idx++) {
      const [sr, sc] = positions[idx];
      const [dr, dc] = shuffled[idx];
      for (let y = 0; y < block; y++) {
        for (let x = 0; x < block; x++) {
          const srcY = sr * block + y, srcX = sc * block + x;
          const dstY = dr * block + y, dstX = dc * block + x;
          if (srcY >= H || srcX >= W || dstY >= H || dstX >= W) continue;
          const si = (srcY * W + srcX) * 4;
          const di = (dstY * W + dstX) * 4;
          out[di] = d[si]; out[di+1] = d[si+1]; out[di+2] = d[si+2]; out[di+3] = d[si+3];
        }
      }
    }
    log('image blocks shuffled randomly. spatial coherence destroyed.');
    return new ImageData(out, W, H);
  },
  feedback(data) {
    const d = data.data;
    const out = new Uint8ClampedArray(d);
    const dx = 4, dy = 2;
    for (let y = 0; y < H; y++) {
      for (let x = 0; x < W; x++) {
        const sx = Math.max(0, x - dx), sy = Math.max(0, y - dy);
        const si = (sy * W + sx) * 4;
        const di = (y * W + x) * 4;
        out[di] = (d[di] + d[si]) / 2;
        out[di+1] = (d[di+1] + d[si+1]) / 2;
        out[di+2] = (d[di+2] + d[si+2]) / 2;
        out[di+3] = 255;
      }
    }
    log('feedback smear applied. trailing ghosts of itself.');
    return new ImageData(out, W, H);
  }
};

document.querySelectorAll('.op-btn[data-op]').forEach(btn => {
  btn.onclick = () => {
    if (!imgAData) { log('nothing loaded. load an image first, dummy.'); return; }
    const current = getCurrent();
    const result = ops[btn.dataset.op](current);
    ctx.putImageData(result, 0, 0);
    statusEl.textContent = 'mutation applied. the machine does not know what it did.';
  };
});

document.getElementById('resetBtn').onclick = () => {
  if (!imgAData) return;
  ctx.putImageData(imgAData, 0, 0);
  log('reset to original. all progress toward slop erased.');
};

document.getElementById('saveBtn').onclick = () => {
  const link = document.createElement('a');
  link.download = 'slop_' + Date.now() + '.png';
  link.href = canvas.toDataURL();
  link.click();
  log('slop exported. go forth.');
};
</script>

</body>
</html>wow this is the code
