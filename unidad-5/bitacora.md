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

## Bitácora de reflexión
