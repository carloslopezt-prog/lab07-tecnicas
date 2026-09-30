# Tarea: Mi prompt avanzado

Laboratorio 07: Técnicas Avanzadas de Prompting
Autor: Lopez Taype Carlos Ricardo
Herramienta de IA usada: (escribe aquí cuál usaste)

## Tarea elegida

**Explicar un error de compilación de Java a un estudiante que recién está aprendiendo.**

Cuando `javac` muestra un error, el mensaje suele ser difícil de entender para quien empieza. Quiero un prompt que me devuelva siempre la misma explicación: qué significa el error, en qué línea ocurre, cómo corregirlo y si el programa tiene algún otro problema.

Para probar los tres prompts uso siempre el mismo programa y el mismo error:

```text
public class Promedio {
    public static void main(String[] args) {
        int nota1 = 15;
        int nota2 = 18;
        double promedio = (nota1 + nota2) / 2;
        System.out.println("Promedio: " + promedio);
        System.out.println(nombre);
    }
}
```

Mensaje de `javac`:

```text
Promedio.java:7: error: cannot find symbol
        System.out.println(nombre);
                           ^
  symbol:   variable nombre
  location: class Promedio
1 error
```

Además del error de compilación, el programa tiene un problema lógico: `(nota1 + nota2) / 2` es una división entera y da `16.0` en lugar de `16.5`. Lo dejé a propósito para ver qué versión del prompt lo detecta.

## Version 1: prompt basico

**Técnica agregada:** ninguna (prompt básico, zero-shot).

```text
Explicame este error de Java:

public class Promedio {
    public static void main(String[] args) {
        int nota1 = 15;
        int nota2 = 18;
        double promedio = (nota1 + nota2) / 2;
        System.out.println("Promedio: " + promedio);
        System.out.println(nombre);
    }
}

Promedio.java:7: error: cannot find symbol
  symbol:   variable nombre
  location: class Promedio
```

**Por qué:** es el punto de partida. Así puedo ver cómo responde la IA sin ninguna guía y tener con qué comparar.

**Qué observé:** la IA explica el error, pero con un nivel y una longitud que ella decide. Suele usar términos como "símbolo" o "ámbito" sin explicarlos, y el orden de la respuesta cambia cada vez. No mencionó el problema de la división entera.

## Version 2

**Técnicas agregadas:** role prompting y prompt estructurado.

```text
<rol>Actua como profesor de Java para estudiantes de primer ciclo de Diseno y Desarrollo de Software que ya conocen variables y operadores, pero todavia no saben leer los mensajes del compilador.</rol>

<contexto>Mi programa no compila. Te paso el codigo y el mensaje que muestra javac.</contexto>

<codigo>
public class Promedio {
    public static void main(String[] args) {
        int nota1 = 15;
        int nota2 = 18;
        double promedio = (nota1 + nota2) / 2;
        System.out.println("Promedio: " + promedio);
        System.out.println(nombre);
    }
}
</codigo>

<error>
Promedio.java:7: error: cannot find symbol
  symbol:   variable nombre
  location: class Promedio
</error>

<tarea>Explica el error.</tarea>

<formato>Responde en espanol con 3 secciones: Que significa (una frase sencilla), Donde ocurre (numero de linea y parte exacta) y Como corregirlo (codigo corregido).</formato>
```

**Por qué:** el rol específico le indica a quién le explica y con qué vocabulario. Las etiquetas separan mi código y el mensaje de error de las instrucciones, para que la IA no los confunda con lo que debe hacer.

**Qué mejoró:** el lenguaje ahora es sencillo y la respuesta sigue las 3 secciones que pedí. **Qué faltó:** no muestra cómo llegó a la causa, y sigue sin detectar la división entera porque solo le pedí explicar el error.

## Version 3: prompt final

**Técnicas agregadas:** few-shot, chain of thought y autocrítica (sobre la base del rol y la estructura de la versión 2).

```text
<rol>Actua como profesor de Java para estudiantes de primer ciclo de Diseno y Desarrollo de Software que ya conocen variables y operadores, pero todavia no saben leer los mensajes del compilador.</rol>

<contexto>Mi programa no compila. Te paso el codigo y el mensaje que muestra javac.</contexto>

<ejemplo>
Codigo: int edad = 20
Error: Main.java:3: error: ';' expected

Respuesta:
1. Razonamiento: el mensaje dice que falta algo; la flecha apunta al final de la linea 3; en Java cada instruccion termina con punto y coma.
2. Que significa: Java esperaba un punto y coma al final de la instruccion y no lo encontro.
3. Donde ocurre: linea 3, al final de "int edad = 20".
4. Como corregirlo: int edad = 20;
5. Revision: revise el resto del codigo y no encontre otros problemas.
</ejemplo>

<codigo>
public class Promedio {
    public static void main(String[] args) {
        int nota1 = 15;
        int nota2 = 18;
        double promedio = (nota1 + nota2) / 2;
        System.out.println("Promedio: " + promedio);
        System.out.println(nombre);
    }
}
</codigo>

<error>
Promedio.java:7: error: cannot find symbol
  symbol:   variable nombre
  location: class Promedio
</error>

<tarea>Piensa paso a paso: lee el mensaje, ubica la linea y determina la causa. Luego responde con las 5 secciones del ejemplo. En la seccion 5, revisa tu propia explicacion y todo el codigo: si hay otro error o un resultado incorrecto aunque compile, indicalo y corrigelo.</tarea>

<formato>Responde en espanol, con las mismas 5 secciones numeradas del ejemplo y el codigo corregido completo al final.</formato>
```

**Por qué:** el ejemplo (few-shot) fija el formato de las 5 secciones. Pedir el razonamiento paso a paso (chain of thought) me permite comprobar cómo llegó a la causa. La autocrítica lo obliga a revisar el código completo, no solo la línea del error.

**Qué mejoró:** la respuesta tiene siempre las 5 secciones en el mismo orden, muestra el razonamiento y, en la revisión final, señala que `(nota1 + nota2) / 2` es una división entera que da `16.0` en lugar de `16.5`, y propone `/ 2.0`.

## Tecnicas usadas en el prompt final

| Técnica | Parte del prompt final | Para qué sirve aquí |
|---|---|---|
| Role prompting | `<rol>` "profesor de Java para estudiantes de primer ciclo..." | Fija el nivel y el vocabulario de la explicación |
| Prompt estructurado | Etiquetas `<rol>`, `<contexto>`, `<ejemplo>`, `<codigo>`, `<error>`, `<tarea>`, `<formato>` | Separa mi código y el error de las instrucciones |
| Few-shot | `<ejemplo>` con el caso del punto y coma | Hace que la respuesta siempre tenga las 5 secciones |
| Chain of thought | `<tarea>`: "Piensa paso a paso: lee el mensaje, ubica la linea..." y sección 1 "Razonamiento" | Permite verificar cómo llegó a la causa |
| Autocrítica | `<tarea>`: "revisa tu propia explicacion y todo el codigo..." y sección 5 "Revision" | Detecta problemas que el mensaje de `javac` no muestra |

## Evaluacion del resultado

| Criterio | Cumple (Sí / No) |
|---|---|
| ¿Explica el error en lenguaje sencillo, sin términos sin explicar? | Sí |
| ¿Indica la línea correcta (línea 7) y la parte exacta? | Sí |
| ¿El código corregido compila? | Sí |
| ¿Usa las 5 secciones numeradas del formato pedido? | Sí |
| ¿Muestra el razonamiento paso a paso? | Sí |
| ¿Detecta la división entera (`16.0` en vez de `16.5`)? | Sí |
| ¿Hay alguna afirmación incorrecta o repetida en la respuesta? | No |

## Por que elegi estas tecnicas

Elegí role prompting porque el mismo error se explica distinto a un principiante que a un desarrollador, y necesitaba que el vocabulario fuera el de un estudiante de primer ciclo. Usé prompt estructurado porque mi entrada mezcla tres cosas (instrucciones, código y mensaje del compilador) y las etiquetas evitan que la IA las confunda. Agregué few-shot porque quería el mismo formato en cada consulta, sin corregirlo a mano. Elegí chain of thought porque así puedo revisar cómo llegó a la causa y no solo creerle. Añadí autocrítica porque el compilador solo reporta el error de sintaxis y no la división entera, que es un error lógico. No usé descomposición porque mi tarea es pequeña y se resuelve en un solo pedido; dividirla en varios mensajes habría sido más lento sin mejorar el resultado.
