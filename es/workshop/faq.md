# Preguntas en clase

Esta sección es un compendio de preguntas hechas en clase sin ningún orden en particular. Las respuestas contestan la pregunta y expanden el alcance de la misma según convenga.

## ¿Se puede reutilizar una variable? {#reuse-names}

Sí, se puede redeclarar una variable con `let` y utilizar el mismo identificador en más de una ocasión. Esto no sobreescribe el valor anterior, la nueva declaración corresponde a **un lugar diferente en la memoria** aunque con el mismo nombre:

```rust
fn main() {
    let answer = 24;
    println!("La dirección de memoria de `answer` es {:p}", &answer);
    let answer = 42;
    println!("The answer to life the universe and everything: {}", answer);
    // Esta dirección será distinta a la anterior.
    println!("La dirección de memoria de `answer` es {:p}", &answer);
}
```

Asignar por segunda vez a un nombre es tratar de ocupar el mismo espacio de memoria con otros datos. Eso es lo que está prohibido por defecto en Rust:

```rust
fn main() {
    let answer = 24;
    println!("La dirección de memoria de `answer` es {:p}", &answer);
    answer = 42; // error, cannot assign twice to immutable variable `answer`
    println!("The answer to life the universe and everything: {}", answer);
    // Aquí no llega porque fallará al intentar asignar dos veces a la misma variable.
    println!("La dirección de memoria de `answer` es {:p}", &answer);
}
```

Para que el código anterior compilase, tendríamos que declarar **explícitamente** la variable `answer` como mutable. Ahora el espacio de memoria puede cambiarse:

```rust
fn main() {
    let mut answer = 24;
    println!("La dirección de memoria de `answer` es {:p}", &answer);
    answer = 42;
    println!("The answer to life the universe and everything: {}", answer);
    // Esta dirección será la misma que la anterior.
    println!("La dirección de memoria de `answer` es {:p}", &answer);
}
```

## ¿Los datos se pasan por valor o por referencia? {#value-or-reference}

Estrictamente, los datos se pasan por valor a menos que indiquemos que queremos pasarlos como referencia y, aun así, se debe hacer **siempre de forma explícita**:

```rust
fn main() {
  let name = "Salva".to_string();
  pass_by_ref(&name);  // Tomamos una referencia explícitamente.
  pass_by_value(name);
}

fn pass_by_ref(n: &String) {}
fn pass_by_value(n: String) {}
```

## ¿Por qué "mover un valor"? ¿Para qué sirve? {#why-movement-semantics}

Hemos dicho que mover un valor es equivalente a transferir la propiedad y hemos definido también que la propiedad es el "deber de gestionar un valor".

La parte más importante de esta gestión es la destrucción del valor y es importante que, tras destruir un valor, nadie acceda a esa región de memoria pensando que está accediendo a un dato válido. Esta región queda a disposición del compilador para reusarla como convenga pero el desarrollador no puede asumir qué contendrá después de destruir el dato que había en ella.

Para que no se produzcan usos no deseados de datos destruidos, Rust impone dos reglas:

1. Un sólo propietario.
2. El propietario se encarga de destruir el dato al terminar su ejecución.

De esta forma, saber dónde se va a destruir un dato es predecible y detectar los usos posteriores a la destrucción es posible.

Este comportamiento "agresivo" de liberar rápido permite reutilizar grandes cantidades de memoria.

## ¿Qué diferencia hay entre `"{}"` y `"{:?}"` cuando se usa con `println!`? {#display-and-debug}

Los especificadores `{}` y `{:?}` utilizados en una cadena de formato difieren en que el segundo está orientado a la depuración del programa. En Rust, esto se traduce a una representación que expone el estado interno del valor y, comunmente, es código Rust válido que podría utilizarse para generar otro objeto idéntico. Por ejemplo, observa las diferencias entre:

```rust
fn main() {
  let message = "Hello world";
  println!("Displaying \"Hello world\": {}", message);
  println!("Debugging \"Hello world\": {:?}", message);
}
```

El especificador `{}` invoca a la función `fmt` del aspecto (_trait_) [`Display`](https://doc.rust-lang.org/std/fmt/trait.Display.html) del valor mientras que `{:?}` invoca la función homónima pero del aspecto [`Debug`](https://doc.rust-lang.org/std/fmt/trait.Debug.html).

Las estructuras personalizadas no implementan ninguno de estos aspectos pero podemos pedir al compilador de Rust que implemente la representación de depuración por nosotros. Compara la salida con el valor en el `let`:

```rust
#[derive(Debug)]
struct S { a: i32, b: i32 }

fn main() {
  let s = S { a: 1, b: 2 };
  println!("Debugging s: {:?}", s);
}
```

## ¿Qué diferencia hay entre `String`, `&String` y `&str`? {#string-refstring-refstr}

Los tipos `String`, `&String` y `&str` se leen "cadena", "referencia a cadena" y "porción (_slice_) de una cadena" respectivamente. La diferencia fundamental se da entre `String` y `&str`. Semánticamente el primer tipo representa un texto en memoria y el segundo una porción de un texto en memoria. De manera práctica, el primero require la reserva de memoria y el segundo no.

A la hora de implementar procedimientos en los que el resultado es una porción de un vector de algún tipo &mdash;las cadenas de texto son vectores de bytes&mdash;, se prefiere que el resultado quede expresado como una porción (_slice_) del espacio de búsqueda.

Considera el problema de escribir una función que devuelva una porción de una cadena. Por ejemplo, la primera palabra o todo el texto si no hay espacios:

```rust
// Utilizamos referencias para evitar tomar posesión del parámetro.
fn first_word(target: &String) -> ??? {

}
```

¿Qué devuelve esta función? Una opción es devolver una nueva cadena:

```rust
fn first_word(target: &String) -> String {
  let mut word = "".to_string();
  for c in target.chars() {
    if !c.is_whitespace() {
      word.push(c);
    }
    else {
      break;
    }
  }
  word
}
```

Otra opción es devolver una vista de la cadena original que revele sólo la primera palabra:

```rust
fn first_word(target: &String) -> &str {
  match target.find(char::is_whitespace) {
    Some(index) => { &target[0..index] }
    _ => { &target }
  }
}
```

En Rust, los literales de cadena tienen tipo "porción de `String`" y no `String`. Esto hace imposible llamar a la función anterior así:

```rust
fn main() {
    println!("{}", first_word("Hola mundo")); // error, mismatched types
}


fn first_word(target: &String) -> &str {
  match target.find(char::is_whitespace) {
    Some(index) => { &target[0..index] }
    _ => { &target }
  }
}
```

Pero en Rust, un tipo `&String` es automáticamente convertido a un tipo `&str` por lo que podemos modificar la signatura de nuestra función para que sea más cómoda de utilizar:

```rust
fn main() {
    println!("{}", first_word("Hola mundo"));
    println!("{}", first_word(&"Hola mundo".to_string()));
}


fn first_word(target: &str) -> &str {
  match target.find(char::is_whitespace) {
    Some(index) => { &target[0..index] }
    _ => { &target }
  }
}
```

Este ejemplo está inspirado en el capítulo [_The Slices Type_](https://doc.rust-lang.org/book/second-edition/ch04-03-slices.html) del libro online [_The Rust Programming Language_](https://doc.rust-lang.org/book/second-edition) (segunda edición).