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
## Bitácora de aplicación 



## Bitácora de reflexión

