# Práctica 1 - Variables compartidas

## Ejercicio 1

Para el siguiente programa concurrente suponga que todas las variables están inicializadas en 0 antes de empezar. Indique cual/es de las siguientes opciones son verdaderas:

a. En algún caso el valor de x al terminar el programa es 56.

b. En algún caso el valor de x al terminar el programa es 22.

c. En algún caso el valor de x al terminar el programa es 23.

<table>
   <thead>
        <tr>
            <th>
                P1::
            </th>
            <th>
                P2::
            </th>
            <th>
                P3::
            </th>
        </tr>
   </thead>
<tbody>
<tr>
 <td rowspan=4>

```pascal
 if (x = 0) then
    y:= 4*2;
    x:= y + 2;
```

</td>
 <td rowspan=4>

```pascal
    if (x > 0) then
     x:= x + 1;
```

</td>
 <td rowspan=4>

```pascal
    x:= (x*3) + (x*2) + 1;
```

</td>
</tr>
</tbody>
</table>

### Respuesta

1. Verdadero

2. Verdadero

3. Verdadero

## Ejercicio 2

Realice una solución concurrente de grano grueso (utilizando <> y/o <await B; S>) para el
siguiente problema. Dado un número N verifique cuántas veces aparece ese número en un
arreglo de longitud M. Escriba las pre-condiciones que considere necesarias.

### Respuesta

```c

int total = 0; int v[M] = ...; int N = ...;

process recorredor[id: 0..M-1]{
    if (v[id] == N){
        <total := total + 1 ;>
    }
}

```

## Ejercicio 3

Dada la siguiente solución de grano grueso:

a. Indicar si el siguiente código funciona para resolver el problema de
Productor/Consumidor con un buffer de tamaño N. En caso de no funcionar, debe
hacer las modificaciones necesarias.

```c
int cant = 0; int pri_ocupada = 0; int pri_vacia = 0; int buffer[N]

Process Productor:: {
    while (true) { 
        //produce elemento
        <await (cant < N); cant++>
        buffer[pri_vacia] = elemento;
        pri_vacia = (pri_vacia + 1) mod N;
    }
}

Process Consumidor:: { 
    while (true) { 
        <await (cant > 0); cant-- >
        elemento = buffer[pri_ocupada];
        pri_ocupada = (pri_ocupada + 1) mod N;
        //consume elemento
    }
}
```

b. Modificar el código para que funcione para C consumidores y P productores.

### Respuesta

a. Puede suceder que luego del <await (cant < N); cant++> del productor el consumidor intente guardarse el elemento, pero el buffer estaría vacio, por lo que todo lo relaciono con la cola debe estar dentro de <> (es SC).

```c
int cant = 0; int pri_vacia = 0; int pri_ocupada = 0; int buffer[N];

process productor {
    while (true){
        // producir elemento
        <await (cant < N); cant++;
        buffer[pri] = elemento;>
        pri_vacia = (pri_vacia + 1) mod N;
    }
}

process consumidor{ 
    while (true) { 
        <await (cant > 0); cant-- 
        elemento = buffer[pri];>
        pri_ocupada = (pri_ocupada + 1) mod N;
        //consume elemento
    }
}
```

b.

```c
int cant = 0; int pri_vacia = 0; int pri_ocupada = 0; int buffer[N];

process productor[id:0..P-1]{
    while (true){
        // producir elemento
        <await (cant < N); cant++;
        buffer[pri] = elemento;
        pri_vacia = (pri_vacia + 1) mod N;>
    }
}

process consumidor[id:0..C-1]{ 
    while (true) { 
        <await (cant > 0); cant-- 
        elemento = buffer[pri];
        pri_ocupada = (pri_ocupada + 1) mod N;>
        //consume elemento
    }
}
```

## Ejercicio 4

Resolver con SENTENCIAS AWAIT (<> y <await B; S>). Un sistema operativo mantiene 5 instancias de un recurso almacenadas en una cola, cuando un proceso necesita usar una instancia del recurso la saca de la cola, la usa y cuando termina de usarla la vuelve a depositar. 

### Respuesta

```c
colaRecurso c [5]; cant = 0;

process pSo[id: 0..N-1]{
    Recurso recurso;
    while (true){
        <await (cant < 5);
        cant++;
        recurso= c.pop(); >
        // usa recurso
        <c.push(recurso); 
        cant--; >
    }
}
```

### Ejercicio 5
En cada ítem debe realizar una solución concurrente de grano grueso (utilizando <> y/o <await B; S>) para el siguiente problema, teniendo en cuenta las condiciones indicadas en el item. Existen N personas que deben imprimir un trabajo cada una.

a. Implemente una solución suponiendo que existe una única impresora compartida por todas las personas, y las mismas la deben usar de a una persona a la vez, sin importar el orden. Existe una función Imprimir(documento) llamada por la persona que simula el uso de la impresora. Sólo se deben usar los procesos que representan a las Personas.

b. Modifique la solución de (a) para el caso en que se deba respetar el orden de llegada.

c. Modifique la solución de (a) para el caso en que se deba respetar el orden dado por el identificador del proceso (cuando está libre la impresora, de los procesos que han solicitado su uso la debe usar el que tenga menor identificador).

d. Modifique la solución de (b) para el caso en que además hay un proceso Coordinador que le indica a cada persona que es su turno de usar la impresora.

### Respuesta

_Nota: Este ejercicio fue consultado y esta bien resuelto._

a.

```c
process persona[id: 0..N-1] {
    Documento documento;
    <imprimir(documento); >
}
```

b. 

```c
int numero = 0; int proximo = 0; array[0..N-1] = ([n] = 0);

process persona[i:0..N-1]{
    Documento documento;
    <turno[i] = numero; numero + 1; >
    <await turno[i] == proximo; >
    imprimir(documento);
    proximo = proximo + 1;
}
```

c. 

```c
colaProcesos c; int siguiente = -1;

process persona[i:0..N-1]{
    Documento documento;
    <if (siguiente == -1){
        siguiente = i;
    } 
    else {
        agregar(c, i); 
    }>
    <await (siguiente == i)>
    imprimir(documento);
    <if (c.isEmpty()){
        siguiente = -1;
    }
    else {
        siguiente = sacar(c);
    }>
}
```

d. 

```c
int numero = 0; int actual = -1; turno[0:n-1] = ([n] = 0); bool listo = false;

process persona[i:0..n-1]{
    <turno[i] = numero; numero + 1; >
    <await actual == turno[i]; >
    imprimir(documento);
    listo = true;
}

process coordinador{
    for j = 0..n-1 {
        actual = j;
        <await listo;>
        listo = false;
    }
}
```

__Nota:__ Solo el coordinador modifica actual y solo el puede estar accediendo a esa variable en ese momento ya que las personas van a estar paradas en el await, por lo tanto no hace falta que este entre <>, ya que lo único que podría pasar es que la persona haga una verificación cuando no se termino de actualizar el valor de j pero no representa problema ya que tarde o temprano la va a verificar. Lo mismo sucede cuando listo se pone en true, no hace falta que este entre <>.

## Ejercicio 6
Dada la siguiente solución para el Problema de la Sección Crítica entre dos procesos (suponiendo que tanto SC como SNC son segmentos de código finitos, es decir que terminan en algún momento), indicar si cumple con las 4 condiciones requeridas:

```c
int turno = 1;
Process SC1:: { 
    while (true) { 
        while (turno == 2) 
            skip;
        SC;
        turno = 2;
        SNC;
    }
}

Process SC2:: { 
    while (true) { 
        while (turno == 1) 
            skip;
        SC;
        turno = 1;
        SNC;
    }
}

```

### Respuesta
Si, se cumplen las siguientes 4 condiciones:

__1. Exclusion Mutua__: Garantiza la exclusión mutua. Solo un proceso (SC1 o SC2) puede estar en la sección crítica a la vez debido al uso de la variable turno. Cuando un proceso entra en la sección crítica, establece el turno para el otro proceso una vez que sale de esta.

__2. Ausencia de Dead Lock__:  Garantiza la ausencia de deadlock. No hay ningún escenario en el que ambos procesos queden bloqueados y no puedan continuar. Ambos procesos intentan entrar en la sección crítica, pero si un proceso está en la sección crítica, el otro proceso simplemente espera a que sea su turno. No hay bloqueo mutuo.

__3.Ausencia de Demora Innecesaria__: Garantiza la ausencia de demora innecesaria. Cuando un proceso quiere entrar en la sección crítica, solo debe esperar a que sea su turno. No hay una espera indefinida o innecesaria, ya que tan pronto como sea su turno, el proceso podrá ingresar a la sección crítica.

__4.Eventual Entrada__: Garantiza la eventual entrada de todos los procesos. Esto se debe a que el turno se alterna entre los dos procesos, lo que significa que ambos procesos eventualmente tendrán la oportunidad de ingresar a la sección crítica.

## Ejercicio 7

Desarrolle una solución de grano fino usando sólo variables compartidas (no se puede usar las sentencias await ni funciones especiales como TS o FA). En base a lo visto en la clase 3 de teoría, resuelva el problema de acceso a sección crítica usando un proceso coordinador. En este caso, cuando un proceso SC[i] quiere entrar a su sección crítica le avisa al coordinador, y espera a que éste le dé permiso. Al terminar de ejecutar su sección crítica, el proceso SC[i] le avisa al coordinador. __Nota:__ puede basarse en la solución para implementar barreras con “Flags y coordinador” vista en la teoría 3.

### Respuesta

_Nota: Este ejercicio fue consultado y esta bien resuelto._

Asumo que el coordinador da permiso siguiendo el orden de id de los procesos.

```c
int actual = -1;

process sc[i: 0..n-1]{
    while (true){
        while(actual <> i) skip;
        //SC
        listo = true;
        while(listo) skip;
    }
}

process coordinador{
    while(true){
        for j = 0..n-1 {
            actual = j;
            while(!listo) skip;
            listo = false;
        }
    }
}
```