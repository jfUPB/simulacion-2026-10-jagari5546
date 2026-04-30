NOTA DEL PROFE: al revisar la bitácora el día de la entrega no encontré nada publicado.

# Unidad 3

## Bitácora de proceso de aprendizaje


## Bitácora de aplicación 
- El caos
- En esta unidad me decante por un concepto algo mas directo, que el usuario sea el que lleve el caos y el desorden a un sistema balanceado, digamos dos objetos que se orbiten entre si, el espectador o usuario seria el que lanze un objeto que destruya ese equilibrio, aun asi, el orden natural de las cosas es que ambos objetos regresen a su equilibrio, el tiempo de esto depende del daño causado por el espectador.

- La idea la saque del sistema solar binario un concepto que vi en la clase de astronomia de la universidad, en donde dos soles se orbitan a si mismos los cuales pueden estar en equilibrio o tender al caos si una fuerza externa los golpea.

- Link [https://editor.p5js.org/jagari5546/sketches/2dhsgZ4RI]
- Codigo
 ```js
let planets = [];
let meteors = [];

let aimStart = null;
let aimEnd = null;

const orbitRadius = 130;

// Movimiento más sutil
const TARGET_ORBIT_SPEED = 1.15;
const TANGENTIAL_STEER = 0.018;
const RADIAL_RETURN = 0.0028;
const DAMPING = 0.994;
const MUTUAL_G = 2600; // reducido

const MAX_POWER = 38;

function setup() {
  createCanvas(800, 800);

  let center = createVector(width / 2, height / 2);

  let p1 = center.copy().add(createVector(orbitRadius, 0));
  let p2 = center.copy().add(createVector(-orbitRadius, 0));

  planets.push(new Planet(p1.x, p1.y, color(80, 140, 255)));
  planets.push(new Planet(p2.x, p2.y, color(255, 120, 190)));
}

function draw() {
  background(0);

  drawOrbitGuide();
  drawCenter();

  applyMutualAttraction();

  for (let p of planets) {
    p.update();
    p.display();
  }

  for (let i = meteors.length - 1; i >= 0; i--) {
    meteors[i].update();
    meteors[i].display();

    if (meteors[i].hitsPlanet(planets[0]) || meteors[i].hitsPlanet(planets[1])) {
      meteors.splice(i, 1);
      continue;
    }

    if (meteors[i].isOffscreen()) {
      meteors.splice(i, 1);
    }
  }

  drawAimArrow();
}

class Planet {
  constructor(x, y, col) {
    this.pos = createVector(x, y);
    this.vel = createVector(0, 0);
    this.col = col;
    this.r = 24;
  }

  update() {
    let center = createVector(width / 2, height / 2);
    let radial = p5.Vector.sub(this.pos, center);
    let distFromCenter = max(radial.mag(), 0.0001);
    let radialDir = radial.copy().normalize();

    let tangent = createVector(-radialDir.y, radialDir.x);
    let desiredVel = tangent.mult(TARGET_ORBIT_SPEED);

    let steer = p5.Vector.sub(desiredVel, this.vel);
    steer.mult(TANGENTIAL_STEER);
    this.vel.add(steer);

    let radialError = distFromCenter - orbitRadius;
    let restore = radialDir.copy().mult(-radialError * RADIAL_RETURN);
    this.vel.add(restore);

    this.vel.mult(DAMPING);
    this.pos.add(this.vel);
  }

  display() {
    noStroke();
    fill(this.col);
    circle(this.pos.x, this.pos.y, this.r * 2);
  }
}

class Meteor {
  constructor(x, y, vx, vy) {
    this.pos = createVector(x, y);
    this.vel = createVector(vx, vy);
    this.r = 11;
  }

  update() {
    this.pos.add(this.vel);
  }

  display() {
    noStroke();
    fill(255);
    circle(this.pos.x, this.pos.y, this.r * 2);
  }

  hitsPlanet(planet) {
    let d = dist(this.pos.x, this.pos.y, planet.pos.x, planet.pos.y);

    if (d < this.r + planet.r) {
      let impulse = this.vel.copy().mult(0.22);
      planet.vel.add(impulse);
      return true;
    }

    return false;
  }

  isOffscreen() {
    return (
      this.pos.x < -80 ||
      this.pos.x > width + 80 ||
      this.pos.y < -80 ||
      this.pos.y > height + 80
    );
  }
}

function applyMutualAttraction() {
  let p1 = planets[0];
  let p2 = planets[1];

  let dir = p5.Vector.sub(p2.pos, p1.pos);
  let dSq = constrain(dir.magSq(), 12000, 120000);

  let forceMag = MUTUAL_G / dSq;
  dir.normalize();

  let force = dir.copy().mult(forceMag);

  p1.vel.add(force);
  p2.vel.sub(force);
}

function mousePressed() {
  aimStart = createVector(mouseX, mouseY);
  aimEnd = createVector(mouseX, mouseY);
}

function mouseDragged() {
  if (aimStart) {
    aimEnd.set(mouseX, mouseY);
  }
}

function mouseReleased() {
  if (aimStart && aimEnd) {
    let dir = p5.Vector.sub(aimEnd, aimStart);
    let dragAmount = constrain(dir.mag(), 0, 220);

    if (dragAmount > 5) {
      let speed = map(dragAmount, 0, 220, 0, MAX_POWER);
      dir.setMag(speed);
      meteors.push(new Meteor(aimStart.x, aimStart.y, dir.x, dir.y));
    }
  }

  aimStart = null;
  aimEnd = null;
}

function drawAimArrow() {
  if (!aimStart || !aimEnd) return;

  let dir = p5.Vector.sub(aimEnd, aimStart);
  let len = dir.mag();

  if (len < 2) return;

  stroke(255);
  strokeWeight(1.8);
  line(aimStart.x, aimStart.y, aimEnd.x, aimEnd.y);

  let arrowSize = 12;
  let angle = atan2(dir.y, dir.x);

  push();
  translate(aimEnd.x, aimEnd.y);
  rotate(angle);
  line(0, 0, -arrowSize, -arrowSize * 0.5);
  line(0, 0, -arrowSize, arrowSize * 0.5);
  pop();
}

function drawOrbitGuide() {
  noFill();
  stroke(255, 30);
  strokeWeight(1);
  circle(width / 2, height / 2, orbitRadius * 2);
}

function drawCenter() {
  noStroke();
  fill(255, 70);
  circle(width / 2, height / 2, 5);
}
```
- Screenshots
- <img width="571" height="521" alt="image" src="https://github.com/user-attachments/assets/97d58b1b-59e5-4b19-a00f-43ea921655ca" />}
- <img width="522" height="428" alt="image" src="https://github.com/user-attachments/assets/d1fb6c56-e81a-4478-808e-2f9f3f0b6e1e" />
- <img width="569" height="295" alt="image" src="https://github.com/user-attachments/assets/10a158e7-84e9-4b80-8815-3f940d767893" />




## Bitácora de reflexión
