# Unidad 5
## Bitácora de proceso de aprendizaje
- La propiedad this.lifespan define el tiempo de vida de la particula. A su ves this.lifespan define el alpha de la particula o basicamente la intensidad del color haciendo que entre menos lifespan mas transparente.
- this.lifespan es una unidad (vector?) que cada segundo segun el codigo original se le resta 2, this.lifespan -=2, lo que basicamente nos dice que la particula va muriendo de forma paulatina.
```js
update() {
    this.velocity.add(this.acceleration);
    this.position.add(this.velocity);
    this.lifespan -= 2;
    this.acceleration.mult(0);
  }
```
Ese metodo Update se puede ver claramente el motion101 donde se le añade aceleracion a la particula. 

¿Qué tienen en común las subclases de partículas? ¿Qué tienen de diferente?
- Tanto Particle como Confeti tienen las mismas estructuras, esto debido a que confeti hereda de particle, pero hay un ligero cambio en el show(), esto hace que cada una difiera en el metodo run().


¿Por qué es importante que el Emitter no necesite saber qué tipo específico de partícula está gestionando? Explica esto con tus propias palabras.
- El Emmiter si gestiona que tipo de particula genera mas dentro del codigo hay un randomizer que hace que una se genere en ciertas condiciones y otra en otras. Ya dentro del metodo run(), hay un polimofirsmo en el metodo show(), aca se escoge la forma de la particula dependiendo de si es particle o si es confetti.
 
Si mañana quisieras agregar un tercer tipo de partícula, ¿Qué tendrías que crear y qué NO tendrías que modificar?
-  Depende, si quisiera solo cambiar la forma puedo heredar de particle y editar el show, ya seria heredar de particle y modificar el metodo que quiera, ya dependiendo de eso puedo modificar forma, velocidad, etc.

Compara con Example 4.2: ¿Cambió la lógica del Emitter? ¿Cambió la lógica de muerte? ¿Qué capa del sistema se modificó y cuáles permanecieron intactas?

## Bitácora de aplicación 

Ciclo de vida de la Mariposa
- Habia que escoger un ciclo de vida y me decidi por el de una mariposa, el entorno estara compuesto de 2 elementos, la mariposa en sus fases y un pequeño arbolito.
- El ciclo de vida de la mariposa lo dividire en 3, fase gusano, fase capullo y fase mariposa.
- Ciclo de vida celula enfermedad con el usuario siendo celula

- Al final me decidi por una recomendación personal de chatGPT, buscando ideas encontre un ciclo mas abstracto y mas facil de hacer interactivo, el ciclo seria, Recuerdo -> Nublado -> Olvido
- Me decante por esta opcion ya que de forma personal hay recuerdos y que se recuerdan valga la redundacia y hay otros que se tienen a olvidar, no por completo pero quedan digamos algunas pequeñas partes del mismo mas no el recuerdo completo, me lo imagino como un gran circulo emitiendo luces que poco a poco va perdiendo tanto como su brillo como su luz, el usuario seria basicamente el encargado de mantener el recuerdo con vida, alimentandolo por decirlo de alguna forma, si el usuario no lo alimenta el recuerdo muere y otro ocupa su lugar.
- [Link al Codigo](https://editor.p5js.org/jagari5546/sketches/lKXwBvsjn)
- Codigo
Particle:
```js
class Particle {
  constructor(x, y, options = {}) {
    this.position = createVector(x, y);
    this.velocity = options.velocity
      ? options.velocity.copy()
      : p5.Vector.random2D().mult(random(0.2, 1));

    this.acceleration = createVector(0, 0);
    this.lifespan = options.lifespan ?? 255;
    this.decay = options.decay ?? 2;
    this.size = options.size ?? 10;
    this.baseSize = this.size;
  }

  applyForce(force) {
    this.acceleration.add(force);
  }

  behave(core, feeding) {}

  update() {
    this.velocity.add(this.acceleration);
    this.position.add(this.velocity);
    this.acceleration.mult(0);
    this.lifespan -= this.decay;
  }

  show() {}

  run() {
    this.update();
    this.show();
  }

  onDeath() {
    return [];
  }

  isDead() {
    return this.lifespan <= 0;
  }
}
```
- MistParticle
```js
class MistParticle extends Particle {
  constructor(x, y, state = "nublado") {
    const velocity = p5.Vector.random2D().mult(random(0.1, 0.5));
    velocity.y += random(-0.5, -0.05);

    super(x, y, {
      velocity,
      lifespan: random(150, 230),
      decay: random(0.6, 1.0),
      size: random(16, 36)
    });

    this.state = state;
    this.blend = { recuerdo: 0, nublado: 1, olvido: 0 };
    this.noiseOffset = random(1000);
    this.growth = random(0.03, 0.08);
  }

  behave(core, feeding) {
    this.blend = core.getBlend();
    this.state = core.getDominantState(this.blend);

    const drift = createVector(
      map(noise(this.noiseOffset + frameCount * 0.01), 0, 1, -0.03, 0.03),
      map(noise(this.noiseOffset + 500 + frameCount * 0.01), 0, 1, -0.015, 0.005)
    );

    this.applyForce(drift);

    // mientras más olvido, más la niebla se dispersa hacia afuera y sube
    const up = createVector(0, -0.002 - this.blend.olvido * 0.003);
    this.applyForce(up);

    const fromCore = p5.Vector.sub(this.position, core.position);
    if (fromCore.mag() > 0) {
      fromCore.setMag(this.blend.olvido * 0.006);
      this.applyForce(fromCore);
    }

    if (feeding && !core.forgotten) {
      const pull = p5.Vector.sub(core.position, this.position);
      pull.setMag(0.004 + this.blend.nublado * 0.002);
      this.applyForce(pull);
    }
  }

  update() {
    super.update();
    this.velocity.mult(0.985);
    this.size += this.growth;
  }

  mix(a, b, c) {
    return (
      a * this.blend.recuerdo +
      b * this.blend.nublado +
      c * this.blend.olvido
    );
  }

  show() {
    const alpha = map(this.lifespan, 0, 230, 0, 110, true);

    noStroke();

    fill(
      this.mix(235, 185, 115),
      this.mix(240, 200, 125),
      this.mix(210, 235, 145),
      alpha * this.mix(0.08, 0.22, 0.22)
    );
    circle(this.position.x - this.size * 0.15, this.position.y, this.size * 1.15);

    fill(
      this.mix(250, 220, 165),
      this.mix(250, 230, 175),
      this.mix(225, 245, 190),
      alpha * this.mix(0.05, 0.14, 0.12)
    );
    circle(this.position.x + this.size * 0.2, this.position.y - this.size * 0.08, this.size * 0.95);

    fill(
      this.mix(255, 235, 220),
      this.mix(255, 240, 225),
      this.mix(240, 250, 235),
      alpha * this.mix(0.03, 0.08, 0.05)
    );
    circle(this.position.x, this.position.y, this.size * 1.4);
  }
}
```
- MemorySystem
```js
class MemorySystem {
  constructor(x, y) {
    this.origin = createVector(x, y);
    this.core = new MemoryCore(x, y);
    this.particles = [];
    this.hasDeathBurst = false;
    this.finished = false;
  }

  run(feeding) {
    this.core.update(feeding);
    this.emit();

    for (let i = this.particles.length - 1; i >= 0; i--) {
      const particle = this.particles[i];
      particle.behave(this.core, feeding);
      particle.run();

      if (particle.isDead()) {
        const replacements = particle.onDeath();
        this.particles.push(...replacements);
        this.particles.splice(i, 1);
      }
    }

    if (this.core.forgotten && !this.hasDeathBurst && this.core.visualEnergy < 3) {
      this.releaseIntoForgetting();
      this.hasDeathBurst = true;
      this.finished = true;
    }

    this.core.show();
  }

  emit() {
    if (this.hasDeathBurst) return;

    const blend = this.core.getBlend();

    const shardChance =
      blend.recuerdo * 0.22 +
      blend.nublado * 0.12 +
      blend.olvido * 0.05;

    const mistChance =
      blend.recuerdo * 0.01 +
      blend.nublado * 0.09 +
      blend.olvido * 0.14;

    if (!this.core.forgotten && random(1) < shardChance) {
      this.particles.push(
        new MemoryShard(this.core.position.x, this.core.position.y, this.core.state)
      );

      const shardCost =
        blend.recuerdo * 0.34 +
        blend.nublado * 0.18 +
        blend.olvido * 0.08;

      this.core.consume(shardCost);
    }

    if (!this.core.forgotten && random(1) < mistChance) {
      this.particles.push(
        new MistParticle(
          this.core.position.x + random(-14, 14),
          this.core.position.y + random(-14, 14),
          this.core.state
        )
      );

      const mistCost =
        blend.recuerdo * 0.01 +
        blend.nublado * 0.05 +
        blend.olvido * 0.05;

      this.core.consume(mistCost);
    }
  }

  releaseIntoForgetting() {
    for (let i = 0; i < 20; i++) {
      this.particles.push(
        new MemoryShard(this.core.position.x, this.core.position.y, "olvido")
      );
    }

    for (let i = 0; i < 36; i++) {
      this.particles.push(
        new MistParticle(
          this.core.position.x + random(-18, 18),
          this.core.position.y + random(-18, 18),
          "olvido"
        )
      );
    }
  }

  restart() {
    this.core = new MemoryCore(this.origin.x, this.origin.y);
    this.particles = [];
    this.hasDeathBurst = false;
    this.finished = false;
  }
}
```
- MemoryShard 
```js
class MemoryShard extends Particle {
  constructor(x, y, state = "recuerdo") {
    const angle = random(TWO_PI);
    const offset = p5.Vector.fromAngle(angle).mult(random(18, 34));
    const start = createVector(x, y).add(offset);

    const velocity = p5.Vector.fromAngle(angle).mult(random(0.4, 1.4));
    velocity.y += random(-0.4, 0.15);

    super(start.x, start.y, {
      velocity,
      lifespan: random(130, 190),
      decay: random(1.2, 1.8),
      size: random(5, 10)
    });

    this.state = state;
    this.blend = { recuerdo: 1, nublado: 0, olvido: 0 };
    this.wanderSeed = random(1000);
    this.rotation = random(TWO_PI);
    this.spin = random(-0.03, 0.03);
  }

  behave(core, feeding) {
    this.blend = core.getBlend();
    this.state = core.getDominantState(this.blend);

    const drift = createVector(
      map(noise(this.wanderSeed + frameCount * 0.01), 0, 1, -0.02, 0.02),
      map(noise(this.wanderSeed + 300 + frameCount * 0.01), 0, 1, -0.01, 0.01)
    );
    this.applyForce(drift);

    const toCore = p5.Vector.sub(core.position, this.position);

    // fuerza mezclada suavemente según el estado visual
    let pullStrength =
      this.blend.recuerdo * 0.018 +
      this.blend.nublado * 0.007 -
      this.blend.olvido * 0.016;

    let pull = toCore.copy();
    if (pullStrength >= 0) {
      pull.setMag(pullStrength);
    } else {
      pull.mult(-1);
      pull.setMag(abs(pullStrength));
    }
    this.applyForce(pull);

    const sideways = createVector(-toCore.y, toCore.x);
    if (sideways.mag() > 0) {
      sideways.setMag(this.blend.nublado * 0.01);
      this.applyForce(sideways);
    }

    if (feeding) {
      const restoration = p5.Vector.sub(core.position, this.position);
      restoration.setMag(0.028);
      this.applyForce(restoration);
    }
  }

  update() {
    super.update();
    this.size *= 0.995;
    this.rotation += this.spin;
  }

  mix(a, b, c) {
    return (
      a * this.blend.recuerdo +
      b * this.blend.nublado +
      c * this.blend.olvido
    );
  }

  show() {
    const alpha = map(this.lifespan, 0, 190, 0, 255, true);

    noStroke();
    push();
    translate(this.position.x, this.position.y);
    rotate(this.rotation);

    fill(
      this.mix(255, 195, 140),
      this.mix(235, 205, 150),
      this.mix(170, 210, 180),
      alpha * this.mix(0.9, 0.7, 0.45)
    );
    ellipse(0, 0, this.size * this.mix(1.5, 1.7, 1.8), this.size * this.mix(1.0, 1.1, 1.2));

    fill(
      this.mix(255, 230, 230),
      this.mix(250, 235, 235),
      this.mix(220, 245, 230),
      alpha * this.mix(0.5, 0.25, 0.12)
    );
    circle(0, 0, this.size * this.mix(0.7, 1.4, 1.7));

    pop();
  }

  onDeath() {
    const next = [];
    const amount = this.blend.olvido > 0.45 ? 2 : 1;

    for (let i = 0; i < amount; i++) {
      next.push(new MistParticle(this.position.x, this.position.y, this.state));
    }

    return next;
  }
}
```
- MemoryCore
```js 
class MemoryCore {
  constructor(x, y) {
    this.position = createVector(x, y);

    this.energy = 100;        
    this.visualEnergy = 100;  

    this.state = "recuerdo";
    this.radius = 54;
    this.haloRadius = 86;
    this.interactionRadius = 120;
    this.forgotten = false;
  }

  update(feeding) {
    if (!this.forgotten) {
      this.energy += feeding ? 0.45 : -0.03;
      this.energy = constrain(this.energy, 0, 100);

      if (this.energy <= 0) {
        this.forgotten = true;
      }
    } else {
      this.energy = 0;
    }

    this.visualEnergy = lerp(this.visualEnergy, this.energy, feeding ? 0.1 : 0.05);

    const targetRadius = map(this.visualEnergy, 0, 100, 16, 54);
    const targetHalo = map(this.visualEnergy, 0, 100, 28, 86);

    this.radius = lerp(this.radius, targetRadius, 0.08);
    this.haloRadius = lerp(this.haloRadius, targetHalo, 0.08);

    const blend = this.getBlend();
    this.state = this.getDominantState(blend);
  }

  consume(amount) {
    if (this.forgotten) return;
    this.energy = max(0, this.energy - amount);

    if (this.energy <= 0) {
      this.forgotten = true;
    }
  }

  smoothStep(edge0, edge1, x) {
    let t = constrain((x - edge0) / (edge1 - edge0), 0, 1);
    return t * t * (3 - 2 * t);
  }

  getBlend() {
    const e = this.visualEnergy;

    let recuerdo = this.smoothStep(38, 82, e);
    let olvido = 1 - this.smoothStep(18, 62, e);

    let nublado = 1 - abs(e - 45) / 28;
    nublado = constrain(nublado, 0, 1);

    const total = recuerdo + nublado + olvido;

    return {
      recuerdo: recuerdo / total,
      nublado: nublado / total,
      olvido: olvido / total
    };
  }

  getDominantState(blend = this.getBlend()) {
    if (blend.recuerdo >= blend.nublado && blend.recuerdo >= blend.olvido) {
      return "recuerdo";
    }
    if (blend.nublado >= blend.recuerdo && blend.nublado >= blend.olvido) {
      return "nublado";
    }
    return "olvido";
  }

  mix(a, b, c, blend) {
    return a * blend.recuerdo + b * blend.nublado + c * blend.olvido;
  }

  show() {
    const blend = this.getBlend();
    const pulse = sin(frameCount * 0.05) * 3;

    push();
    translate(this.position.x, this.position.y);
    noStroke();

    // halo exterior
    fill(
      this.mix(255, 180, 120, blend),
      this.mix(225, 195, 130, blend),
      this.mix(150, 235, 160, blend),
      this.mix(18, 14, 10, blend)
    );
    circle(0, 0, (this.haloRadius + 26 + pulse) * 2);

    // halo medio
    fill(
      this.mix(255, 205, 150, blend),
      this.mix(235, 215, 160, blend),
      this.mix(180, 240, 190, blend),
      this.mix(36, 24, 16, blend)
    );
    circle(0, 0, (this.haloRadius + 8 + pulse) * 2);

    // aro luminoso
    noFill();
    stroke(
      this.mix(255, 220, 185, blend),
      this.mix(246, 230, 195, blend),
      this.mix(215, 255, 220, blend),
      this.mix(185, 110, 55, blend)
    );
    strokeWeight(this.mix(3, 2.5, 2, blend));
    circle(0, 0, (this.radius + 8 + pulse) * 2);

    // cuerpo difuso
    noStroke();
    fill(
      this.mix(255, 195, 130, blend),
      this.mix(240, 205, 140, blend),
      this.mix(190, 230, 180, blend),
      this.mix(85, 55, 28, blend)
    );
    circle(0, 0, this.radius * this.mix(2.05, 2.15, 2.2, blend));

    // centro
    fill(
      this.mix(255, 230, 220, blend),
      this.mix(252, 235, 225, blend),
      this.mix(235, 250, 240, blend),
      this.mix(220, 120, 55, blend)
    );
    circle(0, 0, this.radius * this.mix(1.15, 1.1, 1.0, blend));

    pop();
  }
}
```
<img width="751" height="601" alt="image" src="https://github.com/user-attachments/assets/6d83c025-747e-41b2-9768-d845da06e86a" />
<img width="650" height="458" alt="image" src="https://github.com/user-attachments/assets/0806c929-1290-435c-8e38-e0d92d6f9bfb" />
<img width="664" height="478" alt="image" src="https://github.com/user-attachments/assets/fbbd29b4-a3ae-418c-beb9-854ef54de96d" />




## Bitácora de reflexión
