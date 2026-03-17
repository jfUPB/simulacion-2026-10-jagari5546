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

## Bitácora de aplicación 


## Bitácora de reflexión
