# Introducción {#intro}

La programación de sistemas es aquella cuya finalidad es la de programar software que utilizarán otros programas. Un sistema operativo, un servidor HTTP o un motor de juegos son ejemplos de sistemas.

Este taller de Rust para absolutos novatos presenta y explica las características de Rust tratando de no compararlo con otros lenguajes. El texto, no obstante, asume que alguna vez has programado, aunque haya sido con lenguajes dinámicos o Web.

Comencemos con un sencillo programa de ejemplo para enumerar algunas características de la sintaxis de Rust.

```rust
fn main() {
    let name: String = "Salva".to_string();
    let greeting_length = greeting(name);
    println!("{}", greeting_length);
}

fn greeting(message: String) -> usize {
    println!("Hello, {}", message);
    7 + message.chars().count()
}
```

En Rust, las funciones comienzan con la palabra clave `fn` y la función llamada `main` es el punto de entrada.

Las variables se declaran con `let` acompañado de un identificador y una inicialización obligatoria. El tipo `String`, indicado entre los dos puntos `:` y el igual `=` es opcional.

Antes de continuar, no puedo dejar de enfatizar que los sistemas de tipos no existen para hacer la programación más complicada, sino que son herramientas de diseño: nos exigen coherencia en el software que construímos.

Rust es un lenguaje con un sistema de tipos modernos que permite omitir casi cualquier anotación de tipos. Rust intentará adivinar el tipo por el contexto, aplicando una técnica llamada inferencia de tipos como ocurre con `greeting_length`. Sólo en la signaturas de las funciones, Rust exige que los tipos se especifiquen explícitamente.

```rust
fn main() {
    let v = vec!();  // Rust no puede deducir de qué clase de vector se trata.
    v.push(1);       // Pero aquí, Rust sabe que v es un vector de enteros.
    v.push("mundo"); // Y esto falla: "mismatched types".
}
```

Ademas, ante el error, el compilador de Rust siempre intentará ayudarnos, emitiendo la suficiente información para **contextualizar el error, explicarlo y solucionarlo**. En Rust hay que acostumbrarse a leer los mensajes de error y colaborar con el compilador:

```
error[E0308]: mismatched types
 --> src/main.rs:5:12
  |
5 |     v.push("mundo"); // Y esto falla: "mismatched types".
  |            ^^^^^^^ expected integral variable, found reference
  |
  = note: expected type `{integer}`
             found type `&'static str`
```

Volviendo a la sintaxsis, podemos llamar métodos sobre valores usando el operador `.` como ocurre en `.to_string()` o en `.chars().count()`. La llamada a `.to_string()` es necesaria porque los literales de cadena tienen otro tipo por defecto. En los ejemplos también te encontrarás esto escrito [`format!("Literal de cadena")`](https://doc.rust-lang.org/std/macro.format.html).

El identificador `println!` no es una función al uso sino una _macro_. Una macro es un tipo de función que se ejecuta en tiempo de compilación y produce código Rust que reemplaza a la llamada.

Sin embargo, `greeting` sí que es una función más, como `main`. En la signatura de la función, los tipos son obligatorios. Debemos indicar **qué tipo de datos se esperan como parámetros y el tipo de salida**.

El **valor de retorno de cualquier función es el valor de la última expresión ejecutada**. Fíjate como la última línea de `greeting` **no termina en `;`**. En Rust, el punto y coma `;` es un operador especial, que sólo puede aparecer al final de una expresión, y que anula el resultado de dicha expresión.

## Tutorial para principiantes {#beginners}

Uno de los principales recursos de un ordenador es la memoria. La memoria es un espacio direccionable en el que se almacenan tanto los propios programas en ejecución como los datos con los que dicho programas van a trabajar.

Los programas en ejecución conviven juntos en memoria y es posible que un programa malintencionado invada el espacio de otro con el fin de controlarlo. En Rust, esta situación es imposible y por eso se dice de él que es un lenguaje de programación seguro.

Además, la memoria es un bien escaso. Los lenguajes de sistemas tratan de ocupar poco espacio para que sea el software al que dan servicio el que disfrute del grueso de la misma.

Para lograr estos objetivos, Rust posee los siguientes mecanismos, que exploraremos en esta primera parte del tutorial: propiedad, préstamo y control de la mutabilidad.

Prueba a realizar el [ejercicio 1](https://play.rust-lang.org/?code=%2F%2F+Goals%3A%0A%2F%2F+-+change+to+greet+you+by+name%0A%2F%2F+-+introduce+a+variable+%60name%60+and+insert+this+into+the+format+string%0A%2F%2F+-+try+%60println%21%28%22%7B%7D%22%2C+name%29%60+vs+%60println%21%28%22%7B%3A%3F%7D%22%2C+name%29%60+and+see+what+the+difference+is%0A%0Afn+main%28%29+%7B%0A++++println%21%28%22Hello%2C+world%21%22%29%3B%0A%7D%0A&version=nightly) para ir familiarizándote con el lenguaje.

### Propiedad {#ownership}

La propiedad (u _ownership_, en inglés) representa el **deber de gestionar un dato**, lo que incluye alterarlo y destruirlo cuando ya no sea necesario. Por defecto, en Rust, este "deber" permite leer el dato, **pero no alterarlo** y lo tiene la función que creó el mísmo.

A menos que se especifique lo contrario, cuando pasamos un dato como parámetro, transferimos la propiedad del dato a la función invocada, de forma que ya no podremos volver a acceder al mismo dato desde la función invocante. A esto se le llama "mover un valor":

```rust
fn main() {
    let name: String = "Salva".to_string();
    greeting(name);
    yell(name);      // error, "use of moved value: `name`"
}

fn greeting(message: String) {
    println!("Hello, {}", message);
}

fn yell(message: String) {
    println!("{}!!!", message);
}
```

El compilador de Rust es bastante claro al indicar qué ha pasado:

```
error[E0382]: use of moved value: `name`
 --> src/main.rs:4:10
  |
3 |     greeting(name);
  |               ---- value moved here
4 |     yell(name);
  |          ^^^^ value used here after move
  |
  = note: move occurs because `name` has type `std::string::String`, which does not implement the `Copy` trait
```

Para que el código anterior compile, en lugar de mover el valor en la línea 3, podemos mover una copia del valor haciendo uso del método `.clone()`:

```rust
fn main() {
    let name: String = "Salva".to_string();
    greeting(name.clone()); // Ya no movemos `name` sino una copia de `name`.
    yell(name);
}

fn greeting(message: String) {
    println!("Hello, {}", message);
}

fn yell(message: String) {
    println!("{}!!!", message);
}
```

Conviene hacer notar que igual que movemos un valor hacia el interior de una función, también es posible devolver el valor hacia el exterior, basta con devolver el valor como parte de la expresión de retorno:

```rust
fn main() {
    let name: String = "Salva".to_string();
    let former_name = greeting(name);
    yell(former_name);
}

fn greeting(message: String) -> String {
    println!("Hello, {}", message);
    message
}

fn yell(message: String) -> String {
    println!("{}!!!", message);
    message
}
```

Antes de terminar, existe una notable excepción al comportamiento por defecto de Rust. Observa como no es necesario llamar a `.clone()` sobre un número:

```rust
fn main() {
    let number = 42;
    greeting(number); // Funciona.
    yell(number);     // También funciona.
}

fn greeting(message: i32) {
    println!("Hello, {}", message);
}

fn yell(message: i32) {
    println!("{}!!!", message);
}
```

En Rust, [algunos tipos pueden marcarse como `Copy`](https://doc.rust-lang.org/std/marker/trait.Copy.html#implementors), que significa que cada vez que se pasan, se realiza una **copia implícita**.

Echa un vistazo a [las diapositivas de este tema](http://www.rust-tutorials.com/RustConf17//PDF/10-Ownership.pdf), las cuales ilustran lo que ocurren en caso de movimiento, copia (_clone_) y copia implícita (_copy_) de un valor.

Practica lo que has aprendido con el [ejercicio 2](https://play.rust-lang.org/?code=fn+main%28%29+%7B%0A++++let+%28adjective%2C+name%29+%3D+two_words%28%29%3B%0A++++let+name+%3D+format%21%28%22%7B%7D+%7B%7D%22%2C+adjective%2C+name%29%3B%0A++++print_out%28name%29%3B%0A%7D%0A%0Afn+two_words%28%29+-%3E+%28String%2C+String%29+%7B%0A++++%28format%21%28%22fellow%22%29%2C+format%21%28%22Rustaceans%22%29%29%0A%7D%0A%0Afn+remove_vowels%28name%3A+String%29+-%3E+String+%7B%0A++++%2F%2F+Goal+%231%3A+What+is+needed+here+to+make+this+compile%3F%0A++++let+output+%3D+String%3A%3Anew%28%29%3B%0A++++for+c+in+name.chars%28%29+%7B%0A++++++++match+c+%7B%0A++++++++++++%27a%27+%7C+%27e%27+%7C+%27i%27+%7C+%27o%27+%7C+%27u%27+%3D%3E+%7B%0A++++++++++++++++%2F%2F+skip+vowels%0A++++++++++++%7D%0A++++++++++++_+%3D%3E+%7B%0A++++++++++++++++output.push%28c%29%3B%0A++++++++++++%7D%0A++++++++%7D%0A++++%7D%0A++++output%0A%7D%0A%0Afn+print_out%28name%3A+String%29+%7B%0A++++let+devowelized_name+%3D+remove_vowels%28name%29%3B%0A++++println%21%28%22Removing+vowels+yields+%7B%3A%3F%7D%22%2C+devowelized_name%29%3B%0A%0A++++%2F%2F+Goal+%232%3A+What+happens+when+you+uncomment+the+%60println%60+below%3F%0A++++%2F%2F+Can+you+change+the+code+above+so+that+the+code+below+compiles%0A++++%2F%2F+successfully%3F%0A++++%2F%2F%0A++++%2F%2F+println%21%28%22Removing+vowels+from+%7B%3A%3F%7D+yields+%7B%3A%3F%7D%22%2C%0A++++%2F%2F++++++++++name%2C+devowelized_name%29%3B%0A%0A++++%2F%2F+Extra+credit%3A+Can+you+do+it+without+copying+any+data%3F%0A++++%2F%2F+%28Using+only+ownership+transfer%29%0A%7D%0A&version=nightly). Recuerda que el compilador de Rust **es tu aliado**, lee los mensajes de error con atención para obtener pistas acerca de cómo completar el ejercicio.

## Préstamo {#borrows}

Existe una forma de evitar el movimiento de un valor y aun así no tener que copiarlo, simplificando el uso de nuestras funciones. Para ello Rust permite "dar en préstamo" un valor. Puedes pensar que "dar en préstamo" es realmente "conceder acceso temporal" al valor. Esta concesión se realiza por medio de referencias y, a menos que es especifique explícitamente, el acceso resultante es de sólo lectura:

```rust
fn main() {
    let name: String = "Salva".to_string();
    greeting(&name); // Pasamos una referencia a la función.
    yell(&name);     // Pasamos otra referencia a la función.
                     // El valor en `name` NUNCA se movió.
}

fn greeting(message: &String) {
    println!("Hello, {}", message);
}

fn yell(message: &String) {
    println!("{}!!!", message);
}
```

Fíjate en la sintáxis: el tipo `&String` significa "referencia a String" y el mismo ampersand `&` acompañado de un valor como en `&name` significa "tomar una referencia al valor". Al pasar, devolver o asignar referencias estamos "prestando el valor".

Los préstamos evitan el uso de copias sin tener que mover el valor _de vuelta_, lo que resulta en usos más naturales de las funciones.

Además **los préstamos no transfieren la propiedad**. El encargado de destruir el dato creado lo mantiene la función desde la que se tomó el préstamo.

Trata de adaptar tú el [siguiente ejemplo](http://is.gd/no0tTH).

Es importante recordar que el acceso por defecto de Rust es de **sólo lectura**. Esto quiere decir que, a menos que se especifique explícitamente, no podemos alterar el valor de una variable...

```rust
fn main() {
    let name = "Salva".to_string();
    name.push('!'); // error, "cannot borrow immutable local variable `name` as mutable"
}
```

...ni de una referencia:

```rust
fn main() {
    let name = "Salva".to_string();
    emphasis(&name);
}

fn emphasis(message: &String) {
    message.push('!'); // error, cannot borrow immutable borrowed content `*message` as mutable
}
```

### Sintáxis de alto nivel y eficiencia de bajo nivel

Además de un lenguaje de sistemas, Rust es un **lenguaje moderno, de alto nivel** por lo que ofrece estructuras sintácticas avanzadas pero evitando copias o reservas de memoria.

Es el caso con las porciones (comunmente conocidas como _slices_), en las que nos referimos a un subrango de un vector:

```rust
fn main() {
    let name: String = "Salva".to_string();
    greeting(&name);
    greeting(&name[1..]);    // Esto es una porción o slice, desde el segundo
                             // elemento en adelante.
}

fn greeting(message: &str) { // Hemos cambiado la signatura de la función aquí.
    println!("Hello, {}", message);
}
```

O también de las iteraciones:

```rust
fn main() {
    let names: String = "Salva Fernando Juan".to_string();
    greeting(&names);
}

fn greeting(message: &str) {
    for name in message.split(' ') { // No hay copia alguna de los datos.
      println!("Hello, {}", name);
    }
}
```

Ninguno de los ejemplos anteriores involucra copias de ningún modo. Puedes repasar las [diapositivas de este tema](http://www.rust-tutorials.com/RustConf17//PDF/20-Sharing.pdf) que incluyen representaciones de la memoria en estos casos para comprobar que las copias no son necesarias.

Cuando creas conveniente, practica con el [siguiente ejercicio](https://play.rust-lang.org/?code=pub+fn+main%28%29+%7B%0A++++let+string+%3D+format%21%28%22my+friend%22%29%3B%0A++++greet%28string.clone%28%29%29%3B%0A++++greet%28string%29%3B%0A%7D%0A%0Afn+greet%28name%3A+String%29+%7B%0A++++println%21%28%22Hello%2C+%7B%7D%21%22%2C+name%29%3B%0A%7D%0A%0A%2F%2F+Goal+%231%3A+Convert+%60greet%60+to+use+borrowing%2C+not+ownership%2C+so+that%0A%2F%2F+this+program+executes+without+doing+any+cloning.%0A%2F%2F%0A%2F%2F+Goal+%232%3A+Use+a+subslice+so+that+it+prints+%22Hello%2C+friend%22+instead+of%0A%2F%2F+%22Hello%2C+my+friend%22.%0A&version=nightly).