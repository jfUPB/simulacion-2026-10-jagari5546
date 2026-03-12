# Unidad 4

## Bitácora de proceso de aprendizaje


## Bitácora de aplicación 
### Ideación
- La idea Inicial es crear un rio con forma de onda seno, aca aplicamos los conceptos de ondas y ondas senoidales (Unidad 4) para que estas sean el rio, el rio estara suavizado con un ruido perlin (Unidad 1), este rio afecta una serie de lineas imaginemoslo como un mapa topografico, en donde el curso del rio afecta el entorno alrededor del mismo, esto se representa con una interpolacion lineal entre rojo y verde siendo rojo una altura mas elevada y donde el curso del rio tiene menos afectacion y verde una altura mas baja en donde este tiene una afectacion mas elevada. El marco de motion 101 o la unidad 3 la aplico enteramente chatGPT, debido a que tuve problemas creativos en encontrar donde aplicar este marco de fuerzas a el proyecto actual.

- Sketch.js

```js
const STEP = 6;
const RIVER_WEIGHT = 10;

const CONTOUR_COUNT = 22;
const CONTOUR_SPACING = 18;

const EFFECT_RADIUS = 70;
const DEFORM_STRENGTH = 0.95;

let time = 0;
let contourLevels = [];

function setup() {
  createCanvas(900, 600);
  noFill();
  strokeCap(ROUND);
  strokeJoin(ROUND);

  buildContourLevels();
}

function draw() {
  background(COLORS.background);

  const river = buildRiver();

  drawContours(river);
  drawRiver(river);

  time += 0.004;
}

function buildContourLevels() {
  contourLevels = [];

  const totalHeight = (CONTOUR_COUNT - 1) * CONTOUR_SPACING;
  const startY = height * 0.5 - totalHeight * 0.5;

  for (let i = 0; i < CONTOUR_COUNT; i++) {
    contourLevels.push(startY + i * CONTOUR_SPACING);
  }
}


function buildRiver() {
  let points = [];

  let posY = height * 0.5;
  let velY = 0;
  let accY = 0;

  const meander = map(mouseX, 0, width, 0.08, 2.8);

  for (let x = -20; x <= width + 20; x += STEP) {

    const s1 = sin(x * 0.0055 + time * 1.0) * 0.55;
    const s2 = sin(x * 0.0022 - time * 0.55 + 1.7) * 0.45;
    const s3 = sin(x * 0.0105 + time * 0.8 + 0.4) * 0.15;

    const organic = map(noise(x * 0.0025, time * 0.35), 0, 1, -0.5, 0.5);

    let flowForce = (s1 + s2 + s3 + organic) * meander * 0.15;

    let centerForce = (height * 0.5 - posY) * 0.0022;

    // Motion 101
    accY += flowForce;
    accY += centerForce;

    velY += accY;
    velY *= 0.99;               
    velY = constrain(velY, -4.5, 4.5);

    posY += velY;
    accY = 0;

    points.push(createVector(x, posY));
  }

  return points;
}


function drawRiver(points) {
  stroke(COLORS.river);
  strokeWeight(RIVER_WEIGHT);
  noFill();

  beginShape();
  curveVertex(points[0].x, points[0].y);

  for (let p of points) {
    curveVertex(p.x, p.y);
  }

  curveVertex(points[points.length - 1].x, points[points.length - 1].y);
  endShape();
}


function drawContours(river) {
  const maxDepth = ((CONTOUR_COUNT - 1) * CONTOUR_SPACING) * 0.5;

  for (let baseY of contourLevels) {
    const centerDist = abs(baseY - height * 0.5);
    const tColor = constrain(centerDist / maxDepth, 0, 1);
    const contourColor = lerpColor(
      color(COLORS.shallow),
      color(COLORS.deep),
      tColor
    );

    stroke(contourColor);
    strokeWeight(1.4);
    noFill();

    beginShape();

    let firstY = deformContour(baseY, river[0].y);
    curveVertex(river[0].x, firstY);

    for (let i = 0; i < river.length; i++) {
      const x = river[i].x;
      const riverY = river[i].y;

      const y = deformContour(baseY, riverY);
      curveVertex(x, y);
    }

    let lastY = deformContour(baseY, river[river.length - 1].y);
    curveVertex(river[river.length - 1].x, lastY);

    endShape();
  }
}


function deformContour(baseY, riverY) {
  const d = baseY - riverY; // distancia con signo
  const gaussian = exp(-(d * d) / (2 * EFFECT_RADIUS * EFFECT_RADIUS));
  const offset = d * gaussian * DEFORM_STRENGTH;

  return baseY + offset;
}
```
- colors.js
```js
const COLORS = {
  background: "#000000",
  river: "#1d4ed8",
  shallow: "#22c55e", 
  deep: "#ef4444"     
};
```
<img width="857" height="730" alt="image" src="https://github.com/user-attachments/assets/3da451f5-eb10-4b41-80f5-6da96d1af0c8" />

- [Link al Codigo](https://editor.p5js.org/jagari5546/sketches/c1VrT15uo)


- Conclusiones
  1. No resulto como esperaba aun asi estoy satisfecho
  2. Pudo haber resultado mejor
  3. El marco motion101 no era la mejor combinacion con la idea planteada originalmente




## Bitácora de reflexión






