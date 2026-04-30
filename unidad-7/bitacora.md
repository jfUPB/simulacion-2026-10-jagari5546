# Unidad 7

## Bitácora de proceso de aprendizaje


## Bitácora de aplicación 
### APOCALIPSIS
- Decidi usar la palabra apocalipsis y que en la primera i de la palabra el punto de la misma tenga forma de bomba y pueda ser usada, movida y que si toca algun otro objeto explote y se cree una explosion nuclear devastando todo junto con las letras.
- Se uso un objeto en la mitad (Invisible) para sostener las letras dentro del sistema, FALLO TOTAL
- Se cambio la dinamica TOTALMENTE
### Cambios Finales
- Se decidio cambiar en ves de la I la letra O por una bomba, esta misma cae al suelo al ser interactuada por el usuario, al hacer click suena un sonido de bomba lo cual hace que las letras salgan disparadas en multiples direcciones, la idea inicial era que se pudiera interactuar con la bomba y moverla a libertad pero por problemas de diseño se llego a un punto limite.
- IA: La IA no hizo participacion alguna en el proceso creativo, todo es de autoria y cada cambio en el diseño fue propuesto por mi, la IA ayudo a conceptualizar y llevar al codigo las ideas iniciales y a ayudar a debbugear y encontrar errores criticos en el sistema a su ves soluciones para el mismo fueron propuestas por mi persona y la IA las implemento dentro del mismo. 
- Codigo
animation.js
```js
// animation.js
const Engine = Matter.Engine;
const Bodies = Matter.Bodies;
const Body = Matter.Body;
const Composite = Matter.Composite;
const Events = Matter.Events;
const Mouse = Matter.Mouse;
const MouseConstraint = Matter.MouseConstraint;

let engine, ground;
let wallLeft, wallRight, wallTop;
let letters = []; 
let letterBodies = []; 
let bombBody = null;
let bombActivated = false;
let mouseConstraint = null;
let exploded = false;

let explosionActive = false;
let explosionX = 0;
let explosionY = 0;
let explosionProgress = 0;
const EXPLOSION_DURATION = 60;

const LETTER_LIBRARY = {
  a: { w: 42, h: 60 },
  p: { w: 40, h: 60 },
  o: { w: 44, h: 60 },
  c: { w: 40, h: 60 },
  l: { w: 22, h: 60 },
  i: { w: 20, h: 60 },
  s: { w: 38, h: 60 }
};

const WORD = "apocalipsis";
const FONT_SIZE = 52;
const GAP = 10;

function totalWordWidth(text) {
  let total = 0;
  for (let i = 0; i < text.length; i++) {
    const ch = text[i].toLowerCase();
    total += LETTER_LIBRARY[ch].w + (i < text.length - 1 ? GAP : 0);
  }
  return total;
}

function initPhysics() {
  engine = Engine.create();
  engine.gravity.x = 0;
  engine.gravity.y = 1;
  engine.gravity.scale = 0.001;

  const W = width;
  const H = height;
  const T = 60;

  ground = Bodies.rectangle(W / 2, H + T / 2, W * 2, T, {
    isStatic: true, restitution: 0.9, label: 'ground'
  });
  wallLeft = Bodies.rectangle(-T / 2, H / 2, T, H * 2, {
    isStatic: true, restitution: 0.9, label: 'wall'
  });
  wallRight = Bodies.rectangle(W + T / 2, H / 2, T, H * 2, {
    isStatic: true, restitution: 0.9, label: 'wall'
  });
  wallTop = Bodies.rectangle(W / 2, -T / 2, W * 2, T, {
    isStatic: true, restitution: 0.9, label: 'wall'
  });

  Composite.add(engine.world, [ground, wallLeft, wallRight, wallTop]);

  const wordW = totalWordWidth(WORD);
  const startX = (W - wordW) / 2;
  const startY = H / 2 - 30;

  buildLetterData(WORD, startX, startY);
  setupBomb();
  setupMouseConstraint();
  setupCollisionEvents();
}

function buildLetterData(text, startX, startY) {
  letters = [];
  let cursorX = startX;
  let firstO = true;

  for (let i = 0; i < text.length; i++) {
    const ch = text[i].toLowerCase();
    const model = LETTER_LIBRARY[ch];
    const cx = cursorX + model.w / 2;
    const cy = startY + model.h / 2;

    if (ch === 'o' && firstO) {
      firstO = false;
    } else {
      letters.push({
        char: ch,
        x: cx,
        y: cy,
        width: model.w,
        height: model.h,
        body: null,
        angle: 0
      });
    }

    cursorX += model.w + GAP;
  }
}

function setupBomb() {
  const W = width;
  const H = height;
  const wordW = totalWordWidth(WORD);
  const startX = (W - wordW) / 2;
  const startY = H / 2 - 30;

  let cursorX = startX;
  let firstO = true;
  for (let i = 0; i < WORD.length; i++) {
    const ch = WORD[i].toLowerCase();
    const model = LETTER_LIBRARY[ch];
    if (ch === 'o' && firstO) {
      const cx = cursorX + model.w / 2;
      const cy = startY + model.h / 2;
      bombBody = Bodies.circle(cx, cy, model.w * 0.4, {
        isStatic: false,
        restitution: 0.5,
        friction: 0.1,
        density: 0.004,
        frictionAir: 0.99,
        label: 'bomb',
        collisionFilter: { category: 0x0002, mask: 0x0001 }
      });
      Composite.add(engine.world, bombBody);
      firstO = false;
      break;
    }
    cursorX += model.w + GAP;
  }
}

function setupMouseConstraint() {
  const canvas = document.querySelector('canvas');
  const mouse = Mouse.create(canvas);

  mouseConstraint = MouseConstraint.create(engine, {
    mouse: mouse,
    constraint: { stiffness: 0.2, render: { visible: false } },
    collisionFilter: { mask: 0x0002 }
  });

  Composite.add(engine.world, mouseConstraint);

  Events.on(mouseConstraint, 'startdrag', function(event) {
    if (event.body === bombBody && !bombActivated) {
      bombActivated = true;
      bombBody.frictionAir = 0.08;
    }
  });
}

function setupCollisionEvents() {
  Events.on(engine, 'collisionStart', function(event) {
    if (!bombActivated || exploded) return;

    for (let pair of event.pairs) {
      const { bodyA, bodyB } = pair;
      const involvesBomb = bodyA === bombBody || bodyB === bombBody;
      if (!involvesBomb) continue;

      const other = bodyA === bombBody ? bodyB : bodyA;
      if (other.label === 'ground' || other.label === 'wall') {
        triggerExplosion(bombBody.position.x, bombBody.position.y);
        return;
      }
    }
  });
}

function triggerExplosion(x, y) {
  if (exploded) return;
  exploded = true;
  explosionActive = true;
  explosionX = x;
  explosionY = y;
  explosionProgress = 0;

  if (typeof explosionSound !== 'undefined' &&
      explosionSound && !explosionSound.isPlaying()) {
    explosionSound.play();
  }

  const forceMag = 30;

  letterBodies = [];
  for (let letter of letters) {
    const dx = letter.x - x;
    const dy = letter.y - y;
    const d = Math.sqrt(dx * dx + dy * dy) || 1;
    const nx = dx / d;
    const ny = dy / d;
    const falloff = Math.min(2, 500 / d);

    const body = Bodies.rectangle(letter.x, letter.y, letter.width, letter.height, {
      isStatic: false,
      restitution: 0.85,
      friction: 0.02,
      frictionAir: 0.01,
      collisionFilter: { category: 0x0001, mask: 0x0001 }
    });

    Body.setVelocity(body, {
      x: nx * forceMag * falloff * (0.8 + Math.random() * 0.4),
      y: ny * forceMag * falloff * (0.8 + Math.random() * 0.4)
    });
    Body.setAngularVelocity(body, (Math.random() - 0.5) * 0.5);

    letter.body = body;
    letterBodies.push(body);
  }

  Composite.add(engine.world, letterBodies);
}

function updatePhysics() {
  Engine.update(engine, 1000 / 60);

  // sincronizar posición/ángulo post-explosión
  for (let letter of letters) {
    if (letter.body) {
      letter.x = letter.body.position.x;
      letter.y = letter.body.position.y;
      letter.angle = letter.body.angle;
    }
  }

  if (explosionActive) {
    explosionProgress++;
    if (explosionProgress >= EXPLOSION_DURATION) {
      explosionActive = false;
    }
  }
}

function drawExplosion() {
  if (!explosionActive) return;

  const t = explosionProgress / EXPLOSION_DURATION;
  const eased = 1 - pow(1 - t, 3);

  push();

  if (t < 0.12) {
    const flashAlpha = map(t, 0, 0.12, 255, 0);
    noStroke(); fill(255, flashAlpha);
    rect(0, 0, width, height);
  }

  const maxR = max(width, height) * 1.3;
  const shockR = eased * maxR;
  const shockAlpha = map(t, 0, 1, 220, 0);
  noFill();
  stroke(255, shockAlpha);
  strokeWeight(map(t, 0, 0.2, 14, 1));
  circle(explosionX, explosionY, shockR * 2);
  stroke(160, shockAlpha * 0.6);
  strokeWeight(map(t, 0, 0.2, 7, 1));
  circle(explosionX, explosionY, shockR * 0.65 * 2);

  const fireR = (1 - eased) * 200 + 10;
  const fireAlpha = map(t, 0, 1, 255, 0);
  noStroke();
  const layers = [255, 220, 160, 100, 60];
  for (let i = 0; i < layers.length; i++) {
    const lr = fireR * (1 - i * 0.16);
    fill(layers[i], fireAlpha * (1 - i * 0.15));
    circle(explosionX, explosionY, lr * 2);
  }

  randomSeed(99);
  noStroke();
  for (let i = 0; i < 30; i++) {
    const angle = random(TWO_PI);
    const speed = random(0.2, 1.0);
    const px = explosionX + cos(angle) * shockR * 0.45 * speed;
    const py = explosionY + sin(angle) * shockR * 0.45 * speed;
    fill(random(150, 255), fireAlpha * speed);
    circle(px, py, random(3, 12) * (1 - t));
  }

  if (t > 0.25) {
    const st = map(t, 0.25, 1, 0, 1);
    randomSeed(13);
    noStroke();
    for (let i = 0; i < 10; i++) {
      const angle = random(TWO_PI);
      const d = st * 140 + random(10, 50);
      const cx = explosionX + cos(angle) * d;
      const cy = explosionY + sin(angle) * d - st * 100;
      const sz = random(25, 90) * st;
      const sa = map(st, 0, 0.4, 0, 100) * (1 - st * 0.6);
      fill(120, sa);
      circle(cx, cy, sz);
    }
  }

  pop();
}

function drawApocalipsisWord() {
  for (let letter of letters) {
    push();
    translate(letter.x, letter.y);
    rotate(letter.angle);
    noStroke();
    fill(255);
    textAlign(CENTER, CENTER);
    textSize(FONT_SIZE);
    text(letter.char.toUpperCase(), 0, 0);
    pop();
  }

  if (bombBody) {
    const pos = bombBody.position;
    const angle = bombBody.angle;
    push();
    translate(pos.x, pos.y);
    rotate(angle);
    drawNuclearBomb(LETTER_LIBRARY['o'].w * 0.4);
    pop();
  }
}

function drawNuclearBomb(r) {
  push();
  rotate(PI);

  // cuerpo
  fill(220); stroke(255); strokeWeight(1.5);
  ellipse(0, r * 0.3, r * 2.2, r * 2.8);

  // nariz (ahora abajo)
  fill(200); stroke(255); strokeWeight(1);
  beginShape();
  vertex(-r * 0.4, -r * 0.9);
  vertex(r * 0.4, -r * 0.9);
  vertex(0, -r * 2.0);
  endShape(CLOSE);

  fill(180); stroke(255); strokeWeight(1);
  beginShape();
  vertex(-r * 0.3, r * 1.6); vertex(-r * 1.1, r * 2.4);
  vertex(-r * 0.8, r * 1.9); vertex(-r * 0.1, r * 1.4);
  endShape(CLOSE);
  beginShape();
  vertex(r * 0.3, r * 1.6); vertex(r * 1.1, r * 2.4);
  vertex(r * 0.8, r * 1.9); vertex(r * 0.1, r * 1.4);
  endShape(CLOSE);
  beginShape();
  vertex(-r * 0.15, r * 1.5); vertex(r * 0.15, r * 1.5);
  vertex(r * 0.1, r * 2.5);  vertex(-r * 0.1, r * 2.5);
  endShape(CLOSE);

  stroke(100); strokeWeight(0.8); noFill();
  line(-r * 0.9, r * 0.3, r * 0.9, r * 0.3);
  line(-r * 0.7, -r * 0.3, r * 0.7, -r * 0.3);

  fill(50); noStroke();
  circle(0, r * 0.3, r * 0.35);
  stroke(50); strokeWeight(2.5); noFill();
  for (let i = 0; i < 3; i++) {
    push();
    translate(0, r * 0.3);
    rotate(TWO_PI / 3 * i);
    arc(0, 0, r * 0.9, r * 0.9, -PI / 6, PI / 6);
    pop();
  }
  pop();
}
```
sketch.js
```js
let explosionSound;

function preload() {
  explosionSound = loadSound('ExplotionSound.mp3');
}

function setup() {
  createCanvas(1920, 1080);
  initPhysics();
  textFont('serif');
}

function draw() {
  background(0);
  updatePhysics();
  drawExplosion();
  drawApocalipsisWord();
}

function keyPressed() {
  if (key === ' ') {
    let fs = fullscreen();
    fullscreen(!fs);
  }
}
```
[Link Al Codigo](https://editor.p5js.org/jagari5546/sketches/DUVCzZg8H)
<img width="1908" height="1076" alt="image" src="https://github.com/user-attachments/assets/909fc248-c271-41b3-b3f6-626fa55d17ec" />
<img width="1917" height="1066" alt="image" src="https://github.com/user-attachments/assets/7e924372-5ee2-4689-8f60-ffd7732a2397" />


## Bitácora de reflexión
