# Clase 04 — List Comprehensions y Limpieza de Strings

> Procesar datos sucios en una sola línea, sin perder claridad


## ¿Qué vamos a ver hoy?

- List comprehensions: sintaxis y casos de uso
- Comprehensions con condición
- Métodos de limpieza de strings, uno por uno
- Combinar ambos para procesar datos reales


### Qué es una list comprehension

Una list comprehension es una forma de crear una lista nueva en una sola línea, aplicando una transformación y opcionalmente un filtro a cada elemento de otra lista, eso es todo. No es una herramienta distinta a los bucles que ya conoces, es una forma más compacta de escribir el mismo patrón y recorrer una lista para construir otra a partir de ella. Vale la pena aprenderla porque ese patrón aparece todo el tiempo en data analysis. Filtrar filas, transformar valores, limpiar una columna. Python tiene una sintaxis pensada específicamente para escribirlo sin rodeos, y una vez que te acostumbras, es difícil volver al bucle largo.

### El problema de las líneas que no son el problema

Mira este código. Toma una lista de precios y devuelve solo los que superan los $10.000, multiplicados por 1.19.

```python
precios = [8500, 32000, 15900, 7200, 54000, 12300]

resultado = []
for p in precios:
    if p > 10000:
        resultado.append(p * 1.19)
```

Funciona. Pero mira de cerca qué hace cada línea. La idea real —*"multiplicá por 1.19 los precios mayores a 10.000"*— está en una sola línea: `p * 1.19` con la condición `p > 10000`. El resto, `resultado = []` y `resultado.append(...)`, no tiene que ver con el problema. Es solo la mecánica que Python necesita para construir una lista nueva.

Vas a escribir ese mismo patrón (lista vacía, bucle, `.append()`)cada vez que filtres o transformes datos. La comprehension existe para evitar esa repetición.

### La sintaxis

```python
precios = [8500, 32000, 15900, 7200, 54000, 12300]

resultado = [p * 1.19 for p in precios if p > 10000]
```

Una sola línea, mismo resultado que el bucle anterior. La estructura es:

```
[ qué quiero hacer con cada elemento   for elemento in la lista original   if condición opcional ]
```

Se lee de izquierda a derecha, casi como una oración: *"dame `p * 1.19`, para cada `p` en `precios`, si `p > 10000`"*.

Prueba primero sin condición, que es el caso más simple para transformar todos los elementos sin filtrar nada:

```python
nombres = ["ana", "luis", "carla"]
capitalizados = [n.capitalize() for n in nombres]
print(capitalizados)
# ['Ana', 'Luis', 'Carla']
```

`.capitalize()` es un método de string que pone en mayúscula la primera letra y el resto en minúscula, es decir,`"ana"` se convierte en `"Ana"`. Acá no hay `if`, cada nombre de la lista original pasa por `.capitalize()` y el resultado va a la lista nueva. Ahora agrega una condición:

```python
numeros = [4, 15, 8, 23, 7, 42, 16]
pares_grandes = [n for n in numeros if n % 2 == 0 and n > 10]
print(pares_grandes)
# [42, 16]
```

Esta vez la expresión es simplemente `n`. No transformamos el valor, solo decidimos si entra o no a la lista nueva según la condición.

#### Por qué no es solo un atajo

Podrías pensar que esto es puramente estético o el mismo resultado, pero en menos líneas. Hay algo más importante que eso. Cuando leés un bucle con `.append()`, tienes que seguir mentalmente el estado de la lista a través de cada vuelta para entender qué va a contener al final. Cuando lees una comprehension, ves la transformación completa de una sola vez, sin rastrear nada. Esa diferencia se nota cuando el código crece. Un bucle de cinco líneas en el medio de un script de cien líneas es algo que tienes que leer con atención. Una comprehension bien escrita dice lo que hace en el mismo lugar donde aparece.

#### Cuándo NO conviene usar una comprehension

Esto es tan importante como saber escribirlas. Mira este ejemplo:

```python
# No hagas esto. Es una comprehension, pero es ilegible.
resultado = [x*2 if x % 2 == 0 else x*3 for x in datos if x is not None and x > 0]
```

Es código válido. Funciona. Pero nadie lo entiende de un vistazo, ni siquiera quien lo escribió la semana pasada. La regla práctica es, si para escribir la comprehension necesitas más de una condición de filtro o más de una transformación condicional, mejor vuelve al bucle normal con `for` e `if`. Un bucle largo pero legible es preferible a una línea corta pero confusa.

### El problema de los datos que no se pueden convertir

Ahora el segundo tema de la clase. Mirá esta lista de precios tal como realmente llega desde un sistema externo:

```python
precios_crudos = ["$45.000", "$1,250.50", " $8500 ", "N/A", "$32.000"]
```

Intenta convertir el primero a número:

```python
float(precios_crudos[0])
```

Esto lanza `ValueError: could not convert string to float: '$45.000'`. El símbolo de moneda no es un carácter numérico — Python no sabe qué hacer con él. Necesitás quitarlo antes de convertir.

#### strip(): el problema de los espacios sobrantes

Mirá el tercer elemento de la lista: `" $8500 "`. Tiene espacios antes y después. Esos espacios suelen aparecer cuando los datos vienen de un campo de formulario o de un archivo exportado mal. Probá convertir igual:

```python
float(" $8500 ")
```

Falla por la misma razón: ni los espacios ni el `$` son parte de un número válido. Para quitar espacios del principio y del final de un string, usás `.strip()`:

```python
" $8500 ".strip()
# "$8500"
```

`.strip()` no toca los caracteres del medio — solo limpia los bordes. Sigue habiendo un `$` ahí, así que todavía no podés convertir. Necesitás otra herramienta para eso.

#### replace(): el problema de los símbolos que no son números

`$45.000` tiene un signo de pesos que `float()` no entiende. Para eliminarlo, usás `.replace()`, que busca un substring y lo reemplaza por otro — en este caso, por nada:

```python
"$45.000".replace("$", "")
# "45.000"
```

Mejor. Pero todavía queda un punto en el medio, y ese punto en `"45.000"` es un separador de miles, no un punto decimal. Si intentás `float("45.000")` ahora, Python lo va a interpretar como el número `45.0` — perdiendo tres ceros. Necesitás quitar también ese punto:

```python
"45.000".replace(".", "")
# "45000"
```

Ahora sí: `float("45000")` da `45000.0`, correcto.

#### Encadenar métodos: resolver todo en una expresión

Cada método de string que usaste devuelve un string nuevo. Eso significa que podés aplicar el siguiente método directamente sobre el resultado del anterior, sin guardar resultados intermedios:

```python
sucio = "  $45.000  "
limpio = sucio.strip().replace("$", "").replace(".", "")
print(limpio)
# "45000"
```

Python ejecuta de izquierda a derecha: primero `.strip()` quita los espacios de los bordes, después el primer `.replace()` quita el `$`, después el segundo `.replace()` quita el punto. Cada paso trabaja sobre el resultado del paso anterior.

#### Un caso que no se resuelve solo con replace

Mirá el segundo precio de la lista original: `"$1,250.50"`. Tiene una coma como separador de miles y un punto como separador decimal — formato distinto al anterior. Si aplicás la misma limpieza:

```python
"$1,250.50".replace("$", "").replace(",", "").replace(".", "")
# "125050"
```

Eso está mal. El resultado debería ser `1250.50`, pero al borrar el punto junto con la coma, perdiste la separación decimal. El problema es que el mismo carácter — el punto — significa cosas distintas según el formato de origen: separador de miles en un caso, separador decimal en otro. Python no puede adivinar cuál es cuál sin que vos le digas la regla.

Esto no es un error de sintaxis que se arregla con más métodos. Es una decisión que tenés que tomar mirando tus datos: ¿qué formato usa esta fuente en particular? En un dataset real, conviene revisar una muestra de los datos antes de asumir cualquier convención.

### Validar antes de convertir

`"N/A"`, el cuarto elemento de la lista original, no tiene ningún símbolo que limpiar. El problema ahí no es de formato, es que simplemente no es un número. `float("N/A")` va a fallar sin importar qué tan bien lo limpies antes.

Para manejar esto sin que el programa se detenga, necesitas lo que ya viste en la clase anterior: `try/except`.

```python
def limpiar_precio(valor):
    try:
        limpio = valor.strip().replace("$", "").replace(",", "")
        return float(limpio)
    except ValueError:
        return None
```

Esta función intenta limpiar y convertir. Si falla porque el valor no es convertible de ninguna manera, como `"N/A"` devuelve `None` en lugar de interrumpir el programa.

### Todo junto: comprehension más limpieza de strings

Ahora tiens las dos piezas, una función que limpia un precio, y la comprehension que aplica esa función a toda la lista.

```python
def limpiar_precio(valor):
    try:
        limpio = valor.strip().replace("$", "").replace(",", "")
        return float(limpio)
    except ValueError:
        return None

precios_crudos = ["$45.000", "$1,250.50", " $8500 ", "N/A", "$32.000"]

precios_limpios = [limpiar_precio(p) for p in precios_crudos]
print(precios_limpios)
# [45000.0, 1250.5, 8500.0, None, 32000.0]

precios_validos = [p for p in precios_limpios if p is not None]
print(precios_validos)
# [45000.0, 1250.5, 8500.0, 32000.0]
```

Fijate cómo se reparten las responsabilidades. La función concentra la limpieza y el manejo de errores, esa lógica necesita su propio bloque con `try/except`, así que no podría ir dentro de una comprehension sin volverse ilegible. Las comprehensions hacen el trabajo de aplicar esa función a toda la lista y de filtrar los resultados válidos. Cada herramienta se usa donde es la más clara, no donde sea posible forzarla.

### Un caso más: nombres de clientes

Las comprehensions con limpieza de strings no son solo para números. Este ejemplo normaliza una lista de nombres que llegó con mayúsculas inconsistentes, espacios sobrantes y un campo vacío:

```python
nombres_crudos = ["  ana torres", "LUIS GÓMEZ", "Clara Díaz  ", "", "  pedro ruiz  "]

nombres_limpios = [n.strip().title() for n in nombres_crudos if n.strip() != ""]

print(nombres_limpios)
# ['Ana Torres', 'Luis Gómez', 'Clara Díaz', 'Pedro Ruiz']
```

`.title()` es un método nuevo, convierte la primera letra de cada palabra a mayúscula y el resto a minúscula. La condición `if n.strip() != ""` descarta el string vacío, comparándolo después de quitarle los espacios, así un valor como `"   "` (solo espacios) también queda afuera, no solo `""`.


## Resumen

| Concepto | Para qué sirve |
|----------|----------------|
| `[expr for x in iterable]` | Transformar cada elemento de una lista |
| `[expr for x in iterable if cond]` | Transformar y filtrar en una sola línea |
| `.strip()` | Eliminar espacios al principio y al final de un string |
| `.replace(a, b)` | Reemplazar todas las apariciones de un substring |
| `.title()` | Poner en mayúscula la primera letra de cada palabra |
| Encadenar métodos | Aplicar varias limpiezas en una sola expresión |
| Comprehension + función | Separar la limpieza compleja (en una función) de la aplicación a la lista (en la comprehension) |


## Recursos adicionales

- [Python Docs — List Comprehensions](https://docs.python.org/3/tutorial/datastructures.html#list-comprehensions)
- [Python Docs — String Methods](https://docs.python.org/3/library/stdtypes.html#string-methods)
- [Real Python — List Comprehensions](https://realpython.com/list-comprehension-python/)
- [Real Python — Python String Methods](https://realpython.com/python-strings/)


## Práctica

→ [Ver ejercicios](./practica/notebook.ipynb)

---

*← [Clase 03 — Funciones y Manejo de Errores](../clase-03/README.md) · [Módulo 1](../README.md) · Clase 05 — Introducción a NumPy →*