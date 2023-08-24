# Preguntas frecuentes sobre Go

Por Jorge Fuertes Alfranca AKA Queru.

Libre distribución con licencia GPL v3 o superior.

## Beneficios de usar Go frente a otros lenguajes

- Go se diseño de forma pragmática, no es un experimento académico. Cada característica y decisión sobre su sintáxis se pensó para hacer más fácil la vida del programador.
- Es más amigable para el programador que C++ o Java.
- Go está optimizado para concurrencia y escala muy bien.
- Se lee mejor que otros lenguajes gracias a que tiene un formato estándar de código.
- El _recolector de basura_ es mucho más eficiente, por diseño y porque funciona concurrente con el resto del programa.

## ¿Qué tipos de datos usa Go?

- Method _(método)_
- Boolean _(boleano)_
- Numeric _(numérico)_
- String _(cadena)_
- Array _(matríz)_
- Slice _(matríz dinámica)_
- Pointer _(puntero)_
- Function _(función)_
- Interface _(interfaz)_
- Channel _(canal)_

## ¿Qué tipo de conversión soporta Go?

Conversión explícita para satisfacer los requisitos del tipado estricto o tipado duro _(strong typing)_.

```go
i := 55 // int
j := 60.1 // float64
sum1 := float(i) + j
```

## ¿Qué es una gorutina y como la puedes detener?

Es una función o un método que se ejecuta concurrentemente con otras _gorutinas_ usando un hilo especial para _gorutina_. Los hilos de _gorutinas_ son mucho más ligeros que un hilo estándar, y muchos programas __Go__ utilizan cientos de _gorutinas_.

```text
                   channel
+------------+  send    receive  +------------+
|            | ----------------> |            |
| gorutina 1 |  receive    send  | gorutina 2 |
|            | <---------------- |            |
+------------+                   +------------+
```

Para crear una _gorutina_ usamos la palabra reservada `go` antes de la llamada la función:

```go
package main

import (
 "fmt"
 "time"
)

func main() {
 start := time.Now()
 go func() {
  for i := 0; i < 3; i++ {
   fmt.Println(i)
  }
 }()
}
```

Podemos detener una _gorutina_ enviando una señal a su canal. Las _gorutinas_ sólo responden a señales si están programadas para comprobarlas, así que tenemos que incluir comprobaciones en los puntos correctos, como en la parte superior del bucle `for`:

```go
package main

import (
 "fmt"
 "time"
)

func main() {
 start := time.Now()
 quit := make(chan bool)
 go func() {
  for i := 0; i < 3; i++ {
   select {
   case <- quit:
    return
   }
   fmt.Println(i)
  }
 }()

 // cerrar la gorutina
 quit <- true
}
```

Por otro lado esta _gorutina_ se detendrá al terminar el bucle `for` o bien cuando termine todo el programa, ya que todas las _gorutinas_ serán detenidas.

Otro ejemplo:

```go
package main

import "sync"
func main() {
    var wg sync.WaitGroup
    wg.Add(1)

    ch := make(chan int)
    go func() {
        for {
            foo, ok := <- ch
            if !ok {
                println("done")
                wg.Done()
                return
            }
            println(foo)
        }
    }()
    ch <- 1
    ch <- 2
    ch <- 3
    close(ch)

    wg.Wait()
}
```

Otro, con `struct`:

```go
package main

import "fmt"
import "time"

func do_stuff() int {
    return 1
}

func main() {

    ch := make(chan int, 100)
    done := make(chan struct{})
    go func() {
        for {
            select {
            case ch <- do_stuff():
            case <-done:
                close(ch)
                return
            }
            time.Sleep(100 * time.Millisecond)
        }
    }()

    go func() {
        time.Sleep(3 * time.Second)
        done <- struct{}{}
    }()

    for i := range ch {
        fmt.Println("receive value: ", i)
    }

    fmt.Println("finish")
}
```

Por último podríamos detener la _gorutina_ utilizando un _contexto_:

```go
package main

import (
    "context"
    "fmt"
    "time"
)

func main() {
    forever := make(chan struct{})
    ctx, cancel := context.WithCancel(context.Background())

    go func(ctx context.Context) {
        for {
            select {
            case <-ctx.Done():  // if cancel() execute
                forever <- struct{}{}
                return
            default:
                fmt.Println("for loop")
            }

            time.Sleep(500 * time.Millisecond)
        }
    }(ctx)

    go func() {
        time.Sleep(3 * time.Second)
        cancel()
    }()

    <-forever
    fmt.Println("finish")
}
```

## ¿Cómo podemos comprobar el tipo de una variable en tiempo de ejecución?

El mejor sistema es el _type switching_, que evalúa la variable en tiempo de ejecución:

```go
func (e *Easy) SetCode(param interface{}) {
 switch v := param.(type) {
 case int:
  e.code = param
 case string:
  ...
 }
}
```

## Concatenar strings

Se pueden concatenar con `+`:

```go
v := "uno" + ", " + "dos"
```

Con `join` si vienen en un `array`:

```go
strs := []string{"uno", "dos", "tres"}
fmt.Println(strings.Join(strs, ", "))
```

O con `fmt`:

```go
fmt.Printf("%s, %s", "uno", "dos")
```

## ¿Qué son los interfaces y cómo funcionan?

Son un tipo especial en __Go__ que define un conjunto de métodos pero que no provee de su implementación, es decir, describimos la firma de los métodos. Los valores de tipo `interface` pueden sustituirse por cualquier valor que implemente ese conjunto de métodos.

Básicamente son huecos a medida que pueden ser cubiertos por múltiples implementaciones de objetos que encajen en él, que tengan implementados todos los métodos que están requeridos en el interface, pueden tener más métodos, pero no menos.

## ¿Se pueden devolver múltiples valores desde una función?

Sí. Se pueden devolver múltiples valores separados por comas en la declaración de retorno `return`.

## Formatear una cadena sin imprimirla

```go
name := "Jorge"
finalStr := fmt.Sprintf("Hola, %s", name)
```

## Diferencias entre concurrencia y _paralelismo_ en __Go__

### Concurrencia

La concurrencia consiste en manejar el tiempo de cada proceso para ir dándoles salida _"a la vez"_, esto puede suceder en uno o más _cores_ de procesador, pero es __Go__ quién se ocupa de ellos, y el `scheduler` va pausando y retomando cada _gorutina_ concurrente cuando procede.

La concurrencia es una propiedad que permite tener múltiples tareas en progreso pero no necesariamente ejecutándose al mismo tiempo.

La clave de la concurrencia en __Go__ son las gorutinas y los canales.

### _Paralelismo_

El _paralelismo_ consuste en ejecutar __a la vez__ y eso siempre debe suceder en distintos procesadores o distintos cores, utilizando hardware diferente para cada proceso. Habrá dos o más tareas ejecutándose __al mismo tiempo__ al utilizar hardware diferente. _Go_ provee de un mecanismo de __concurrencia__ no de _paralelismo_, sin embargo es capaz de manejar los _cores de CPU_ disponibles y utilizar _paralelismo_ de forma transparente, siempre que eso sea posible dentro de nuestro programa.

## ¿Go soporta excepciones?

No. __Go__ utiliza una aproximación diferente, aprovechando el retorno multivalor es fácil comunicar un error sin sobrecargar el valor de retorno, así que se utiliza el valor de error para señalar un estado anormal.

En una declaración de retorno se puede devolver un valor tipo `error` además de el valor que se espera y si este error es nulo se sabrá que la rutina se ha ejecutado sin errores y el valor devuelto es correcto y usable:

```go
func formatHello(name string) (string, error) {
 if len(name) < 2 {
  return "", errors.New("too short name")
 }
 return fmt.Sprintf("Hello, %s.", name), nil
}
```

Por convención el `error` se devuelve en último lugar.

## ¿Qué es un puntero?

Un puntero o apuntador es una variable que contiene la dirección  del valor de otra variable.

```go
a := 1
b := &a // b es un puntero de a
```

Si hacemos modificaciones sobre el valor de un puntero, en realidad estamos haciendo modificaciones sobre el valor de la variable a cuya dirección apunta. Por tanto si cambiamos `b` estaremos cambiando `a`.

### ¿Es bueno usar punteros en todos los casos?

No siempre, debido al _recolector de basura_. En general se pueden copiar objetos por el _stack_ si están en el mismo ámbito, la misma _gorutina_ en definitiva, pero si pasamos la referencia hacemos que dicho objeto escape de ese ámbito, y por tanto tendrá que ir al _heap_ para que el _recolector de basura_ haga su magia más tarde.

En general, si mantenemos variables de ámbito local, y las pasamos por copia, el compilador puede hacer un buen trabajo determinando el ámbito de vida de dichas variables, asignando y destruyendo el espacio necesario para cada una de ellas.

Sim embargo, si las variables escapan de su ámbito, escapan también de la pila _(stack)_, y el compilador no puede determinar cuando destruirlas, por tanto pasan a ser responsabilidad del _recolector de basura_, que determinará su destrucción en tiempo de ejecución.

### Escape analysis

```bash
go build -gcflags=-m=3 [package]
```

### Optimizaciones específicas de la implementación

Un grafo complejo de objetos y punteros limita el _paralelismo_ y genera más trabajo al _recolector de basura_. El recolector contiene algunas optimizaciones para las estructuras más comunes. Desde el punto de vista de la efectividad debemos tener en cuenta:

- Los valores libres de punteros son segregados de los otros valores.
  - Puede ser ventajoso eliminar los punteros de las estructuras de datos que no los necesiten, esto reduce la presión sobre el caché del recolector del programa. Las estructuras de datos que se basen en índices sobre valores de punteros, aunque no tengan un tipado tan correcto, pueden funcionar mejor.
- El recolector detiene el escaneo al llegar al último puntero en el valor.
  - Podría ser ventajoso agrupar los campos de puntero al principio del valor.
  - En teoría esto lo hará el compilador de forma automática en algún momento, aunque no está implementado a la hora de escribir esto.

### No sobreactuar

No es necesario meterse en este tipo de optimizaciones si el grafo de objetos no es complejo y si el recolector no está empleando mucho tiempo en marcar y escanear estos objetos.

Si no se tiene un problema de desempeño, y no se observa una sobrecarga del _recolector de basura_, __no es necesario complicarse con este tipo de optimizaciones__ y deberíamos centrarnos en la claridad del código escrito y en su efectividad. Mayor claridad y menos líneas de código son un beneficio directo para el proyecto.

## Diferencias entre un canal sin buffer y otro con buffer

### Unbuffered

El emisor queda bloqueado al enviar hasta que el receptor reciba el dato por el canal, mientras el receptor también queda bloqueado hasta que el emisor envíe algo al canal.

### Buffered

El emisor sólo queda bloqueado si no hay espacio libre en el canal, mientras que el receptor se bloquea en la recepción si el canal está vacío.

## Valor por defecto de un boleano

El valor por defecto siempre es `false`.

## ¿Qué es el type assertion o aserción de tipos?

Un _type assertion_ toma el valor de una _interface_ y recupera el valor del tipo explícito especificado. Se utiliza _type conversion_ para convertir tipos diferentes en Go.

## ¿Cuales de los siguientes son tipos derivados en Go?

| Tipo | Respuesta |
| --- | --- |
|Interface types | 🔳 |
|Map types| 🔳 |
|Channel types| 🔳 |
|Todos los anteriores| ✅ |

## ¿Qué es cierto acerca de un bucle for en Go?

|Afirmación|Respuesta|
|---|---|
|Si se ha definido una condición, el bucle se ejecuta mientras se siga cumpliendo dicha condición|🔳|
|Si se ha definido un `range`, el bucle se ejecuta por cada objeto del rango.|🔳|
|Ambas|✅|
|Ninguna|🔳|

## ¿Existe un bucle while en Go?

**NO existe como tal**, aunque se puede simular facilmente con un `for`.

## ¿Cual de las siguientes variables es una cadena en Go?

|Definición|Respuesta|
|----------|---------|
|`x := 10`|🔳|
|`x := "10"`|✅|
|Ninguna de las anteriores|🔳|
|Todas las anteriores|🔳|

## ¿Cual de las siguientes funciones retorna la capacidad de un slice y cuantos elementos puede alojar?

|Función|Respuesta|
|---|---|
|`size()`|🔳|
|`len()`|🔳|
|`cap()`|✅|
|Ninguna|🔳|

## Afirmaciones correctas sobre interfaces

|Afirmación|Respuesta|
|---|---|
|Hay un tipo conocido como `interface` que representa un conjunto de firmas de métodos.|🔳|
|El tipo `struct` implementa estos interfaces al tener las definiciones de los métodos de la firma de esos interfaces.|🔳|
|Ambas|✅|
|Ninguna|🔳|

## ¿Cual es el zero value de un `interface`?

|Valor|Respuesta|
|---|---|
|0|🔳|
|1 |🔳|
|Nil|✅|
|Ninguno|🔳|

## Afirmaciones correctas sobre estructuras

|Afirmación|Respuesta|
|---|---|
|Para acceder a un miembro de la estructura, usamos el operador de acceso `(.)`.|🔳|
|Se usa la palabra reservada `struct` para definir variables de tipo estructura.|🔳|
|Se puede pasar una estructura como un argumento a una función de forma similar a como pasarías cualquier otra variable o puntero.|🔳|
|Todas|✅|

## Qué tipos de datos permiten agrupar o combinar posibles diferentes tipos en uno solo

|Tipo|Respuesta|
|---|---|
|`interface`|🔳|
|`channel`|🔳|
|`struct`|✅|
|Ninguno|🔳|
|Todos|🔳|

## Todas las gorutinas terminan cuando termina la función `main`

✅ Correcto.

De hecho la función `main` también es conocida como `main gorutine`, y cuando esta termina, terminan todas las demás ya que dependen de ella.

## Cómo se declara una variable anónima en Go

|Declaración|Respuesta|
|---|---|
|`@`|🔳|
|`_`|✅|
|`*`|🔳|
|`#`|🔳|

## Valor por defecto de `error`

El valor por defecto de un tipo `error` es __`nil`__.

## Método usado para escribir datos a un fichero

`ioutil.WriteFile()`

## Método para crear un fichero

`os.Create()`
