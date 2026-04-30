# Unidad 2

## Bitácora de proceso de aprendizaje
### Vectores
- mag() = Magnitud del vector, raiz de la suma de los cuadrados de sus componentes. 
- magSq() = ambos componentes del vector al cuadrado.
- normalize() = sirve para dejar un vector con longitud (magnitud) = 1 sin cambiar su dirección.
- metodo dot() = producto punto, sirve para medir qué tan alineados están dos vectores. Devuelve un número (no un vector).
- producto cruz, El producto cruz (cross) en p5.Vector te da un vector (en 3D) que es perpendicular a los dos vectores originales. Sirve para saber orientación (derecha/izquierda), área, torque, normales, etc.
- dist() = El método dist() sirve para calcular la distancia entre dos puntos (o entre dos vectores). En otras palabras: “¿qué tan lejos está A de B?”, basicamente si tenemos un dos vectores perpendiculares seria la distancia entre ellos formando un triangulo rectangulo en donde la hipotenusa seria el vector calculado por el metodo dist().
- limit() = Sirve para darle un valor maximo a un vector que no supere esa mangitud en videojuegos sirve para temas de velocidad.
- <img width="1005" height="498" alt="image" src="https://github.com/user-attachments/assets/25ae6eee-c9a5-416b-8ff7-160fc4891ccb" />
- <img width="1062" height="448" alt="image" src="https://github.com/user-attachments/assets/86f8cd10-f30c-427a-8aeb-7c1701c15494" />





## Bitácora de aplicación 
- Mi obra de arte representa una lluvia de estrellas, lo unico que se modifica y se interactua es con el limite al que caen las mismas siendo mouse izq menos velocidad y mouse der mas velocidad, me gusta la idea de un cielo nocturno mientras ves las estrellas pasar como por ejemplo en los documentales cuando aceleran la velocidad de reproduccion de una camara estacionaria, el concepto inicial era una lluvia normal pero el resultado me llevo a una lluvia de estrellas.
- Codigo de la aplicacion
mover.js
```js
// The Nature of Code
// Daniel Shiffman
// http://natureofcode.com

class Mover {
  constructor() {
    this.position = createVector(random(width), height - 2);
    this.velocity = createVector();
    this.acceleration = createVector(-0.001, 0.01);
    this.topSpeed = 10;
  }

  update() {
    
    this.topSpeed = map(constrain(mouseX, 0, width), 0, width, 1, 20)
    
    this.velocity.add(this.acceleration);
    this.velocity.limit(this.topSpeed);
    this.position.add(this.velocity);
  }

  show() {
    stroke(0);
    strokeWeight(1);
    fill(255);
    rect(this.position.x, this.position.y, 2, 8);
  }

  checkEdges() {
    if (this.position.x > width) {
      this.position.x = 0;
    } else if (this.position.x < 0) {
      this.position.x = width;
    }

    if (this.position.y > height) {
      this.position.y = 0;
    } else if (this.position.y < 0) {
      this.position.y = height;
    }
  }
}

```
scketh.js
```js
// The Nature of Code
// Daniel Shiffman
// http://natureofcode.com

let movers=[];
let numMovers=300;

function setup() {
  createCanvas(500, 500);
  for(let i = 0; i < numMovers; i++){
    movers.push(new Mover());
    
    movers[i].position.y = random(height);
  }
  background(0)
}

function draw() {
  background(0, 40);
for (let m of movers){
  
  m.show();
  m.update();
  m.checkEdges();
}
}
```
- [Link al Codigo](https://editor.p5js.org/jagari5546/sketches/DztPkbhBr)
  
- Imagen Representativa <img width="652" height="635" alt="image" src="https://github.com/user-attachments/assets/a46240c8-06e9-4cd5-9319-a0a3cc0bbbd5" />


## Bitácora de reflexión

