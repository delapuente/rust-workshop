# Introducción {#intro}

La programación de sistemas es aquella cuya finalidad es la de programar software que utilizarán otros programas. Un sistema operativo, un servidor HTTP o un motor de juegos son ejemplos de sistemas.

Este taller de Rust para absolutos novatos presenta y explica las características de Rust tratando de no compararlo con otros lenguajes. El texto, no obstante, asume que alguna vez has programado, aunque haya sido con lenguajes dinámicos o Web.

Comencemos con un sencillo programa de ejemplo para enumerar algunas características de la sintaxis de Rust.

```rust
fn main() {
    let name: String = "Salva".to_string();
    println!("Hola {}", emphasis(name));
}

fn emphasis(message: String) -> String {
    message.to_uppercase() + "!"
}
```

En Rust, las funciones comienzan con la palabra clave `fn` y la función llamada `main` es el punto de entrada.

Las variables se declaran con `let` acompañado de un identificador y una inicialización obligatoria. El tipo `String`, indicado entre los dos puntos `:` y el igual `=` es opcional.

Antes de continuar, no puedo dejar de enfatizar que los sistemas de tipos no existen para hacer la programación más complicada, sino que son herramientas de diseño: nos exigen coherencia en el software que construímos.

Rust es un lenguaje con un sistema de tipos modernos que permite omitir casi cualquier anotación de tipos. Rust intentará adivinar el tipo por el contexto, aplicando una técnica llamada inferencia de tipos. Sólo en la signaturas de las funciones, Rust exige que los tipos se especifiquen explícitamente.

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
5 |     v.push("mundo"); // Y esto falla.
  |            ^^^^^^^ expected integral variable, found reference
  |
  = note: expected type `{integer}`
             found type `&'static str`
```

Volviendo a la sintaxsis, podemos llamar métodos sobre valores usando el operador `.` como ocurre en `.to_string()` o `.to_uppercase()`. La llamada a `.to_string()` es necesaria porque los literales de cadena tienen otro tipo por defecto.

El identificador `println!` no es una función al uso sino una _macro_. Una macro es un tipo de función que se ejecuta en tiempo de compilación y produce código que reemplaza a la llamada en un proceso llamado macro-expansión.

Sin embargo, `emphasis` sí que es una función al uso. En la _signatura_ de la función, los tipos son obligatorios. Debemos indicar **qué tipo de datos se esperan como parámetros y el tipo de salida**.

El **valor de retorno de cualquier función es el valor de la última expresión ejecutada**. Fíjate como esta expresión **no termina en `;`**. En Rust, el punto y coma `;` es un operador especial, que sólo puede aparecer al final de una expresión, y que anula el resultado de dicha expresión.

## Tutorial para principiantes {#beginners}
