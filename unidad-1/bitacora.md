# Unidad 1

## Bitácora de proceso de aprendizaje
### Actividad 1
- El arte generado por codigo o generativo se basa principalmente en conceptos de aleatoriedad para generar patrones de diferentes formas, usan varios de estos patrones para el arte generativo.
### Actividad 2
- this.x esto para que cada objeto creado a partir de la clase original Walker como ejemplo new Walker tenga los atributos originales de la clase original.
### Actividad 3
- La diferencia entre la distribucion uniforme y no Uniforme es basicamente que la uniforme es completamente aleatoria entre los parametros dados y la no uniforme tiene forma de una campana de gauss.
- Para el ejercicio le pedi ayuda a chatgpt porque no habia entendido bien el tema
- // The Nature of Code
// Daniel Shiffman
// http://natureofcode.com

let walker;

function setup() {
  createCanvas(640, 240);
  walker = new Walker();
  background(255);
}

function draw() {
  walker.step();
  walker.show();
}

class Walker {
  constructor() {
    this.x = width / 2;
    this.y = height / 2;
  }

  show() {
    stroke(0);
    noFill();
    circle(this.x, this.y, 50);
  }

  step() {
    const r = random(1);

    if (r < 0.55) {
      this.x++;
    } else if (r < 0.75) {
      this.x--;
    } else if (r < 0.90) {
      this.y++;
    } else {
      this.y--;
    }

    this.x = constrain(this.x, 0, width);
    this.y = constrain(this.y, 0, height);
  }
}
- Asi quedo el codigo, lo que hace es una distribucion no, uniforme aleatoria hacia la derecha, cambiando basicamente los parametros de en ves de 4 parametros menores a uno en donde si r es menor que  0.55 osea una probabilidad del 55% se mueve a la derecha favoreciendo asi que se mueva en esa direccion
### Actividad 4
- Aca se nos pidio hacer una distribucion normal osea uniforme, el codigo quedo de la siguiente forma, para hacer que fueran graficos de barras pedi ayuda a chat ya que vi un ejemplo muy similar en la pagina donde varias columnas crecen de forma totalmente aleatoria sin tendencias hacia el centro como lo seria una campana de gauss
- let bins;
let binCount = 64;

function setup() {
  createCanvas(640, 240);
  bins = new Array(binCount).fill(0);
}

function draw() {
  let x = random(0, width);

  let index = floor(map(x, 0, width, 0, binCount));
  if (index >= 0 && index < binCount) bins[index]++;

  background(255);

  let w = width / binCount;
  let maxVal = max(bins);

  for (let i = 0; i < binCount; i++) {
    let h = map(bins[i], 0, maxVal, 0, height - 10);
    noStroke();
    fill(0);
    rect(i * w, height - h, w - 1, h);
  }
}

<img width="1783" height="803" alt="image" src="https://github.com/user-attachments/assets/d3cbd1ad-b89c-4af1-9d31-7c5b29470ad3" />
- PD: Donde solicita el link no lo pongo porque no se como guardar el sketch
### Actividad 5
- Aca el concepto de levy fligh quedo claro mas no como aplicarlo de forma correcta.
- El codigo seria el siguiente
- // The Nature of Code
// Daniel Shiffman
// http://natureofcode.com

let walker;

function setup() {
  createCanvas(640, 240);
  walker = new Walker();
  background(255);
}

function draw() {
  walker.step();
  walker.show();
}

class Walker {
  constructor() {
    this.x = width / 2;
    this.y = height / 2;

    this.minStep = 1;
    this.beta = 1.6;
    this.bigJumpProb = 0.02;
  }

  show() {
    stroke(0);
    point(this.x, this.y);
  }
  levyStepLength() {
    const u = max(1e-9, random());
    return this.minStep * pow(u, -1 / (this.beta - 1));
  }

  step() {
    const angle = random(TWO_PI);
    let stepLen = 1;
    if (random() < this.bigJumpProb) {
      stepLen = this.levyStepLength();
      stepLen = constrain(stepLen, 1, 200);
    }

    this.x += cos(angle) * stepLen;
    this.y += sin(angle) * stepLen;

    if (this.x < 0) this.x += width;
    if (this.x > width) this.x -= width;
    if (this.y < 0) this.y += height;
    if (this.y > height) this.y -= height;
  }
}

<img width="815" height="548" alt="image" src="https://github.com/user-attachments/assets/ea70b2e7-0b2a-418f-9fa5-2c1c3da83db4" />

## Bitácora de aplicación 
- Usare El concepto de Levy Flight, Caminatas Aleatorias y Ruido Perlin.
- 

let walkers = [];
let palettesHex;
let palettes;
let paletteIndex = 0;

let mode = 0;
let fadeOn = true;
let paused = false;
let stepsPerFrame = 6;
let walkerCount = 6;

function setup() {
  createCanvas(1000, 1000);
  pixelDensity(1);
  palettesHex = [
    ["#0B0F1A", "#1B263B", "#415A77", "#778DA9", "#E0E1DD"],
    ["#0D1B2A", "#1B263B", "#E63946", "#F1FAEE", "#A8DADC"],
    ["#0F0F0F", "#2A2A2A", "#F2E9E4", "#C9ADA7", "#9A8C98"],
    ["#0B1320", "#1C2541", "#3A506B", "#5BC0BE", "#FDE74C"]
  ];
  palettes = palettesHex.map(pal => pal.map(h => color(h)));

  resetWalkers();
  background(255);
}

function draw() {
  if (paused) {
    noStroke();
    fill(0, 90);
    textSize(12);
    text("PAUSED (P)", 10, 18);
    return;
  }

  if (fadeOn) {
    noStroke();
    fill(255, 5);
    rect(0, 0, width, height);
  }

  for (let s = 0; s < stepsPerFrame; s++) {
    for (const w of walkers) {
      w.step();
      w.show();
    }
  }

  // HUD mínimo
  noStroke();
  fill(0, 60);
  textSize(11);
  text(`walkers:${walkerCount}  mode:${mode}  fade:${fadeOn ? "ON" : "OFF"}  palette:${paletteIndex}`, 10, height - 10);
}

function resetWalkers() {
  walkers = [];
  for (let i = 0; i < walkerCount; i++) {
    walkers.push(new Walker(i));
  }
}

function keyPressed() {
  if (key === 'p' || key === 'P') {
    paused = !paused;
    return;
  }

  if (key === ' ') {
    if (keyIsDown(SHIFT)) {
      background(255);
    } else {
      fadeOn = !fadeOn;
    }
  }

  if (key === 'v' || key === 'V') {
    paletteIndex = (paletteIndex + 1) % palettes.length;

    const vibe = paletteIndex;

    for (const w of walkers) {
      if (vibe === 0) {
        w.beta = 1.8;
        w.baseJumpProb = 0.015;
        w.baseTurnSpeed = 0.010;
        w.angleSpan = TWO_PI * 1.6;
        w.maxJump = 180;
      } else if (vibe === 1) {
        w.beta = 1.45;
        w.baseJumpProb = 0.035;
        w.baseTurnSpeed = 0.013;
        w.angleSpan = TWO_PI * 2.4;
        w.maxJump = 260;
      } else if (vibe === 2) {
        w.beta = 2.2;
        w.baseJumpProb = 0.010;
        w.baseTurnSpeed = 0.007;
        w.angleSpan = TWO_PI * 1.2;
        w.maxJump = 140;
      } else if (vibe === 3) {
        w.beta = 1.6;
        w.baseJumpProb = 0.020;
        w.baseTurnSpeed = 0.016;
        w.angleSpan = TWO_PI * 3.0;
        w.maxJump = 240;
      }
    }
  }

  if (key === 'm' || key === 'M') {
    mode = (mode + 1) % 3;
  }

  if (key === 'r' || key === 'R') {
    resetWalkers();
  }

  if (key === '+') stepsPerFrame = min(30, stepsPerFrame + 1);
  if (key === '-') stepsPerFrame = max(1, stepsPerFrame - 1);
}

class Walker {
  constructor(id) {
    this.id = id;

    this.x = width / 2 + random(-30, 30);
    this.y = height / 2 + random(-30, 30);
    this.px = this.x;
    this.py = this.y;

    this.minStep = 1;
    this.beta = 1.6;
    this.maxJump = 220;

    this.baseJumpProb = 0.02;
    this.baseTurnSpeed = 0.01;

    this.t = random(1000) + id * 100;
    this.angleSpan = TWO_PI * 2;

    this.jitter = 0.12;
    this.brushBias = random(0.8, 1.25);
  }

  levyStepLength() {
    const u = max(1e-9, random());
    return this.minStep * pow(u, -1 / (this.beta - 1));
  }

  step() {
    this.px = this.x;
    this.py = this.y;

    const mx = constrain(mouseX / width, 0, 1);
    const my = constrain(mouseY / height, 0, 1);

    const turnSpeed = this.baseTurnSpeed * lerp(0.5, 2.5, mx);
    const angleSpan = this.angleSpan * lerp(0.8, 1.4, mx);
    const jumpProb  = this.baseJumpProb * lerp(0.5, 3.0, 1 - my);

    const n = noise(this.t);
    let angle = n * angleSpan;
    angle += random(-this.jitter, this.jitter);

    let stepLen = 1;
    const jumped = (random() < jumpProb);
    if (jumped) {
      stepLen = this.levyStepLength();
      stepLen = constrain(stepLen, 1, this.maxJump);
    }

    this.x += cos(angle) * stepLen;
    this.y += sin(angle) * stepLen;

    if (this.x < 0) { this.x += width; this.px += width; }
    if (this.x > width) { this.x -= width; this.px -= width; }
    if (this.y < 0) { this.y += height; this.py += height; }
    if (this.y > height) { this.y -= height; this.py -= height; }

    this.t += turnSpeed * (jumped ? 2.2 : 1.0);
  }

  show() {
    const pal = palettes[paletteIndex];

    const cIndex = floor(map(noise(this.t + 500 + this.id * 10), 0, 1, 0, pal.length));
    const col = pal[constrain(cIndex, 0, pal.length - 1)];

    const d = dist(this.px, this.py, this.x, this.y);
    const w = map(noise(this.t + 1000 + this.id * 20), 0, 1, 0.6, 3.2) * this.brushBias;

    if (mode === 0) {
      strokeWeight(w);
      stroke(red(col), green(col), blue(col), 95);
      line(this.px, this.py, this.x, this.y);

    } else if (mode === 1) {

      noStroke();
      const a = map(noise(this.t + 2000 + this.id * 30), 0, 1, 35, 160);
      fill(red(col), green(col), blue(col), a);

      const r = map(constrain(d, 0, 140), 0, 140, 1, 9) * this.brushBias;
      circle(this.x, this.y, r);

    } else if (mode === 2) {
      // Ribbon
      const dx = this.x - this.px;
      const dy = this.y - this.py;
      const len = max(1e-6, sqrt(dx * dx + dy * dy));
      const nx = -dy / len;
      const ny = dx / len;

      const ribbonW = map(noise(this.t + 3000 + this.id * 40), 0, 1, 1, 9) * this.brushBias;

      strokeWeight(1.1);
      stroke(red(col), green(col), blue(col), 70);
      line(this.px + nx * ribbonW, this.py + ny * ribbonW, this.x + nx * ribbonW, this.y + ny * ribbonW);
      line(this.px - nx * ribbonW, this.py - ny * ribbonW, this.x - nx * ribbonW, this.y - ny * ribbonW);

     
      noStroke();
      fill(red(col), green(col), blue(col), 14);
      circle(this.x, this.y, map(noise(this.t + 4000 + this.id * 50), 0, 1, 1, 3));
    }
  }
}


- Donde se aplican los conceptos
- Las caminatas aleatorias, estas se ubican en el metodo step() del Walker, se aplica cada freme y hace 2 cosas, toma direccion y distancia de paso, en el codigo estan en las lineas 175 y 176,
this.x += cos(angle) * stepLen;
this.y += sin(angle) * stepLen;

Mas detalladamente seria, en el metodo draw() este se ejecuta 60 veces por segundo, permitiendo la repeticion y el trazado constante, 
step() {
  this.px = this.x;
  this.py = this.y;
  ...
  this.x += cos(angle) * stepLen;
  this.y += sin(angle) * stepLen;
este codigo es el que permite la caminata donde se guarda el estado anterior y se dibuja el segmento, despues se aplican las dos lineas de this.x.... que son las que dan la aleatoriedad del paso. Se uso el random walker existente en la pagina como base para toda la obra de arte generativa. 

- El ruido Perlin se usa para suavizar la direccion del movimiento, segun la explicacion de la pagina permite que sistemas aleatorios tengan mas continuidad entonces tome el concepto para que al aplicarlo junto a la caminata aleatoria y el levy flight los saltos no se sientan como disonantes ante lo demas, dentro del codigo ocurre aca

const n = noise(this.t);
let angle = n * angleSpan;
angle += random(-this.jitter, this.jitter);
...
this.t += turnSpeed * (jumped ? 2.2 : 1.0);

lo importante ocurre aca, const n = noise(this.t); noise devuelve un valor entre 0 y 1, entonces si this.t cambia noise(this.t) cambia tambien y suaviza el proceso, osea basicamente que la direccion en la que gira el random walker no se sienta como giros bruscos sino de forma mas "natural". 

- Levy flight, basicamente hace que aunque la caminata sea paso a paso hay momentos donde la longitud del paso es mucho mayor, en el codigo esta representado de la siguiente manera.
levyStepLength() {
  const u = max(1e-9, random());
  return this.minStep * pow(u, -1 / (this.beta - 1));
}
 




## Bitácora de reflexión



