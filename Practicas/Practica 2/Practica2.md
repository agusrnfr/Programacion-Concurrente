# Práctica 2 - Semáforos

## Ejercicio 1
Existen N personas que deben ser chequeadas por un detector de metales antes de poder ingresar al avión.

a. Analice el problema y defina qué procesos, recursos y semáforos serán
necesarios/convenientes, además de las posibles sincronizaciones requeridas para resolver el problema.

b. Implemente una solución que modele el acceso de las personas a un detector (es decir, si el detector está libre la persona lo puede utilizar; en caso contrario, debe esperar).

c. Modifique su solución para el caso que haya tres detectores. 

### Respuesta

a y b.

```c
sem mutex = 1;

process persona[i:0..N-1]{
    P(mutex)
    //Usar detector
    V(mutex)
}
```

c.

```c
sem mutex = 3;

process persona[i:0..N-1]{
    P(mutex)
    //Usar detector
    V(mutex)
}
```

## Ejercicio 2

Un sistema de control cuenta con 4 procesos que realizan chequeos en forma
colaborativa. Para ello, reciben el historial de fallos del día anterior (por simplicidad, de tamaño N). De cada fallo, se conoce su número de identificación (ID) y su nivel de gravedad (0=bajo, 1=intermedio, 2=alto, 3=crítico). Resuelva considerando las siguientes situaciones:

**a.** Se debe imprimir en pantalla los ID de todos los errores críticos (no importa el orden).

**b.** Se debe calcular la cantidad de fallos por nivel de gravedad, debiendo quedar los resultados en un vector global.

**c.** Ídem b. pero cada proceso debe ocuparse de contar los fallos de un nivel de
gravedad determinado

### Respuesta

a.

```c
sem mutex = 1; int cant = 0; colaFallos c[N];

process controlador[i:0..3]{
    Fallo fallo;
    int nivel;
    P(mutex)
    while (cant < N){
        fallo = c.pop();
        cant++;
        V(mutex);
        nivel = fallo.getNivel();
        if (nivel == 3){
            print(fallo.getID());
        }
        P(mutex);
    }
    V(mutex)
}
```

b.

```c
sem mutex = 1; int cant = 0; colaFallos c[N]; sem semNivel[4] = ([4] 1);
array contFallos[4] = ([4] 0);

process controlador[id:0..3]{
    Fallo fallo;
    int nivel;
    P(mutex)
    while (cant < N){
        fallo = c.pop();
        cant++;
        V(mutex);
        nivel = fallo.getNivel();
        if (nivel == 3){
            print(fallo.getID());
        }
        P(semNivel[nivel]);
        contFallos[nivel]++;
        V(semNivel[nivel]);
        P(mutex);
    }
    V(mutex);
}
```

c.

```c
sem mutex = 1; int cant = 0; colaFallos c[N]; array contFallos[4] = ([4] 0);

process controlador[id:0..3]{
    Fallo fallo;
    int nivel;
    P(mutex)
    while (cant < N){
        fallo = c.pop();
        cant++;
        V(mutex);
        nivel = fallo.getNivel();
        if (nivel == id){
            if (nivel == 3){
                print(fallo.getID());
            }
            cantFallos[id]++;
        }
        else{
            P(mutex); cant--;
            c.push(fallo);
            V(mutex);
        }
        P(mutex);
    }
    V(mutex);
}
```

## Ejercicio 3

Un sistema operativo mantiene 5 instancias de un recurso almacenadas en una cola. Además, existen P procesos que necesitan usar una instancia del recurso. Para eso, deben
sacar la instancia de la cola antes de usarla. Una vez usada, la instancia debe ser encolada nuevamente.

### Respuesta

```c
sem mutex = 5; sem acceso = 1; colaRecurso c[5];

process pSo[id:0..P-1]{
    Recurso recurso;
    while (true){
        P(mutex);
        P(acceso); // SIN ESTO PODRÍAN HABER 2 INTENTANDO SACAR EL MISMO ELEMENTO DE LA COLA
        recurso = c.pop();
        V(acceso);
        // usar recurso
        P(acceso);
        c.push(recurso);
        V(acceso);
        V(mutex);
    }
}
```

## Ejercicio 4
Suponga que existe una BD que puede ser accedida por 6 usuarios como máximo al mismo tiempo. Además, los usuarios se clasifican como usuarios de prioridad alta y usuarios de prioridad baja. Por último, la BD tiene la siguiente restricción:
* no puede haber más de 4 usuarios con prioridad alta al mismo tiempo usando la BD.
* no puede haber más de 5 usuarios con prioridad baja al mismo tiempo usando la BD.

Indique si la solución presentada es la más adecuada. Justifique la respuesta. 

```pascal
Var
sem: semaphoro := 6;
alta: semaphoro := 4;
baja: semaphoro := 5
```
<table style="width:100%">
   <thead>
        <tr>
            <th>
                Process Usuario-Alta [I:1..L]:: 
            </th>
            <th>
                Process Usuario-Baja [I:1..K]:: 
            </th>
        </tr>
   </thead>
<tbody>
<tr>
 <td rowspan=4>

```cpp
 { P (sem);
 P (alta);
 //usa la BD
 V(sem);
 V(alta);
 }
```

</td>
 <td rowspan=4>

```cpp
{ P (sem);
 P (baja);
//usa la BD
 V(sem);
 V(baja);
 }
```
</td>
</td>
</tr>
</tbody>
</table>

### Respuesta

Están mal declaradas las variables, deberían estar:

```java
sem sem = 6;
sem alta = 4;
sem baja = 5;
```
Y ademas se debería hacer primero `p(alta)`  o `p(baja)` y luego `p(sem)`, ya que si no podría ocurrir que usuarios estén bloqueando la entrada a la BD sin poder acceder porque su prioridad alcanzo el limite y estarían evitando que otros de distinta prioridad puedan acceder (cuando deberían poder hacerlo porque su prioridad no alcanzo el máximo)
_Ej.:_ 6 de alta bloquean la BD pero solo 4 pudieron acceder a ella y los otros 2 no pueden hacerlo pero aun asi tienen la BD bloqueada evitando que entren 2 de prioridad baja (que deberían poder). No se maximiza la concurrencia y hay demora innecesaria. 

## Ejercicio 5
En una empresa de logística de paquetes existe una sala de contenedores donde se preparan las entregas. Cada contenedor puede almacenar un paquete y la sala cuenta con capacidad para N contenedores. Resuelva considerando las siguientes situaciones:

**a)** La empresa cuenta con 2 empleados: un empleado Preparador que se ocupa de preparar los paquetes y dejarlos en los contenedores; un empelado Entregador que se ocupa de tomar los paquetes de los contenedores y realizar la entregas.
Tanto el Preparador como el Entregador trabajan de a un paquete por vez.

**b)** Modifique la solución a) para el caso en que haya P empleados Preparadores.

**c)** Modifique la solución a) para el caso en que haya E empleados Entregadores.

**d)** Modifique la solución a) para el caso en que haya P empleados Preparadores y E empleadores Entregadores.

### Respuesta

a)

```cpp
Paquete c[N]; 
sem vacio = 1; 
sem lleno = 0;

Process preparador {
    while (true){
        //preparar paquete;
        P(vacio);
        c.push(paquete);
        V(lleno);
    }
}

Process entregador {
    while (true){
        P(lleno);
        paquete = c.pop();
        V(vacio);
        //entregar paquete;
    }
}
```

b)

```cpp
Paquete c[N];
sem vacio = 1;
sem lleno = 0;
sem mutexP = 1;

Process preparador[i:0..P-1] {
    while (true){
        //preparar paquete;
        P(vacio);
        P(mutexP);
        c.push(paquete);
        V(mutexP);
        V(lleno);
    }
}

Process entregador {
    while (true){
        P(lleno);
        paquete = c.pop();
        V(vacio);
        //entregar paquete;
    }
}
```

c)

```cpp
Paquete c[N];
sem vacio = 1;
sem lleno = 0;
sem mutexE = 1;

Process preparador {
    while (true){
        //preparar paquete;
        P(vacio);
        c.push(paquete);
        V(lleno);
    }
}

Process entregador[i:0..E-1] {
    while (true){
        P(lleno);
        P(mutexE);
        paquete = c.pop();
        V(mutexE);
        V(vacio);
        //entregar paquete;
    }
}
```

d)

```cpp
Paquete c[N];
sem vacio = 1;
sem lleno = 0;
sem mutexP = 1;
sem mutexE = 1;

Process preparador[i:0..P-1] {
    while (true){
        //preparar paquete;
        P(vacio);
        P(mutexP);
        c.push(paquete);
        V(mutexP);
        V(lleno);
    }
}

Process entregador[i:0..E-1] {
    while (true){
        P(lleno);
        P(mutexE);
        paquete = c.pop();
        V(mutexE);
        V(vacio);
        //entregar paquete;
    }
}
```

## Ejercicio 6
Existen N personas que deben imprimir un trabajo cada una. Resolver cada ítem usando semáforos:

**a)** Implemente una solución suponiendo que existe una única impresora compartida por todas las personas, y las mismas la deben usar de a una persona a la vez, sin importar el orden. Existe una función `Imprimir(documento)` llamada por la persona que simula el uso de la impresora. Sólo se deben usar los procesos que representan a las Personas.

**b)** Modifique la solución de a) para el caso en que se deba respetar el orden de llegada.

**c)** Modifique la solución de (a) para el caso en que se deba respetar estrictamente el orden dado por el identificador del proceso (la persona X no puede usar la impresora hasta que no haya terminado de usarla la persona X-1).

**d)** Modifique la solución de (b) para el caso en que además hay un proceso Coordinador que le indica a cada persona que es su turno de usar la impresora.

**e)** Modificar la solución (d) para el caso en que sean 5 impresoras. El coordinador le indica a la persona cuando puede usar una impresora, y cual debe usar. 

### Respuesta
_Nota: Este ejercicio fue consultado y esta bien resuelto._

a)
    
```cpp
sem mutex = 1;

Process persona[i:0..N-1]{
    Documento documento;
    P(mutex);
    Imprimir(documento);
    V(mutex);
}
```

b)

```cpp
colaLlegada c;
sem espera[P] = ([P] 0);
sem mutex = 1;
bool libre = true;

Process persona [i:0..P-1]{
    Documento documento;
    int aux;
    P(mutex);
    if (libre){
        libre = false;
        V(mutex);
    }
    else {
        c.push(i);
        V(mutex);
        P(espera[i]);
    }
    Imprimir(documento);
    P(mutex);
    if (c.isEmpty()){
        libre = true;
    }
    else {
        aux = c.pop();
        V(espera[aux]);
    }
    V(mutex);
}
```

c)

```cpp
int proximo = 0;
sem espera[P] = ([P] 0);

Process persona [i:0..P-1]{
    Documento documento;
    if (proximo != i){ //se podria sacar este if pero se deberia inicializar el semaforo del proceso con i=0 en 1
        P(espera[i]);
    }
    Imprimir(documento);
    proximo++;
    V(espera[proximo]);
}
```

d)

```cpp
sem espera[P] = ([P] 0);
sem mutex = 1;
sem listo = 0;
sem llena = 0;
colaLlegada c;

Process persona [i:0..P-1]{
    Documento documento;
    P(mutex);
    c.push(i);
    V(mutex);
    V(llena);
    P(espera[i]);
    Imprimir(documento);
    V(listo);
}

Process coordinador{
    int aux;
    for i = 0..P-1{
        P(llena);
        P(mutex);
        aux = c.pop();
        V(mutex);
        V(espera[aux]);
        P(listo);
    }
}
```

e)

```cpp
sem espera[P] = ([P] 0);
sem mutex = 1;
sem llena = 0;
colaLlegada c;
int impresoras[5] = {0,1,2,3,4};
sem cantImpres = 5;
int impersona[P] = ([P] -1);
sem mutexImpresoras = 1;

Process persona[i:0..P-1]{
    Documento documento;
    P(mutex);
    c.push(i);
    V(mutex);
    V(llena);
    P(espera[i]);
    Imprimir(documento, impersona[i]);
    P(mutexImpresoras);
    impresoras.push(impersona[i]);
    V(mutexImpresoras);
    V(impresoras);
}

Process coordinador{
    int aux;
    int impaux;
    for i = 0..P-1{
        P(llena);
        P(mutex);
        aux = c.pop();
        V(mutex);
        P(impresoras);
        P(mutexImpresoras);
        impaux = impresoras.pop();
        V(mutexImpresoras);
        impersona[aux] = impaux;
        V(espera[aux]);
    }
}
```

## Ejercicio 7
Suponga que se tiene un curso con 50 alumnos. Cada alumno debe realizar una tarea y existen 10 enunciados posibles. Una vez que todos los alumnos eligieron su tarea, comienzan a realizarla. Cada vez que un alumno termina su tarea, le avisa al profesor y se queda esperando el puntaje del grupo, el cual está dado por todos aquellos que comparten el mismo enunciado. Cuando un grupo terminó, el profesor les otorga un puntaje que representa el orden en que se terminó esa tarea de las 10 posibles.

_**Nota:**_ Para elegir la tarea suponga que existe una función `elegir` que le asigna una tarea a un alumno (esta función asignará 10 tareas diferentes entre 50 alumnos, es decir, que 5 alumnos tendrán la tarea 1, otros 5 la tarea 2 y así sucesivamente para las 10 tareas).

### Respuesta
_Nota: Este ejercicio fue consultado y esta bien resuelto._

```cpp
sem mutex = 1;
sem barreraAlumnos = 0;
sem despiertoMaestro = 0;
sem seDioPuntaje[10] = ([10] 0);
int puntajeTarea[10] = ([10] 0);
cola finalizadas;

Process alumno[i:0..49]{
    int tarea; int puntaje; int j;
    P(mutex);
    tarea = elegir();
    contador++;
    if (contador == 50){
        for j = 1..50 -> V(barreraAlumnos);
    }
    V(mutex);
    P(barreraAlumnos);
    //realizar tarea
    P(mutex);
    finalizadas.push(tarea);
    V(mutex);
    V(despiertoMaestro);
    P(seDioPuntaje[tarea]);
    puntaje = puntajeTarea[tarea];
}

Process profesor{
    int puntaje=10; int contadorTarea[10] = ([10] 0); int j; int tarea; int i;
    for j = 1..50{
        P(despiertoMaestro);
        P(mutex);
        tarea = finalizadas.pop();
        V(mutex);
        contadorTarea[tarea]++;
        if (contadorTarea[tarea] == 5){
            puntajeTarea[tarea] = puntaje;
            for i = 1..5 -> V(seDioPuntaje[tarea]);
            puntaje--;
        }
    }
}
```

## Ejercicio 8
Una fábrica de piezas metálicas debe producir T piezas por día. Para eso, cuenta con E empleados que se ocupan de producir las piezas de a una por vez (se asume T>E). La
fábrica empieza a producir una vez que todos los empleados llegaron. Mientras haya piezas por fabricar, los empleados tomarán una y la realizarán. Cada empleado puede tardar distinto tiempo en fabricar una pieza. Al finalizar el día, se le da un premio al empleado que más piezas fabricó.

### Respuesta
_Nota: Este ejercicio fue consultado y esta bien resuelto._

## Ejercicio 9
Resolver el funcionamiento en una fábrica de ventanas con 7 empleados (4 carpinteros, 1 vidriero y 2 armadores) que trabajan de la siguiente manera:
* Los carpinteros continuamente hacen marcos (cada marco es armando por un único carpintero) y los deja en un depósito con capacidad de almacenar 30 marcos.

* El vidriero continuamente hace vidrios y los deja en otro depósito con capacidad para 50 vidrios.

* Los armadores continuamente toman un marco y un vidrio (en ese orden) de los depósitos correspondientes y arman la ventana (cada ventana es armada por un único armador).

### Respuesta

## Ejercicio 10
A una cerealera van T camiones a descargarse trigo y M camiones a descargar maíz. Sólo hay lugar para que 7 camiones a la vez descarguen, pero no pueden ser más de 5 del mismo
tipo de cereal.

_**Nota:**_ no usar un proceso extra que actué como coordinador, resolverlo entre los camiones.

### Respuesta

## Ejercicio 11
En un vacunatorio hay **un empleado** de salud para vacunar a **50 personas**. El empleado de salud atiende a las personas de acuerdo con el orden de llegada y de a 5 personas a la
vez. Es decir, que cuando está libre debe esperar a que haya al menos 5 personas esperando, luego vacuna a las 5 primeras personas, y al terminar las deja ir para esperar por otras 5. Cuando ha atendido a las 50 personas el empleado de salud se retira. 

_**Nota:**_ todos los procesos deben terminar su ejecución; asegurarse de no realizar _Busy Waiting_; suponga que el empleado tienen una función `VacunarPersona()` que simula que el empleado está vacunando a UNA persona. 

### Respuesta
_Nota: Este ejercicio fue consultado y esta bien resuelto._

## Ejercicio 12
Simular la atención en una Terminal de Micros que posee 3 puestos para hisopar a **150 pasajeros**. En cada puesto hay **una Enfermera** que atiende a los pasajeros de acuerdo con el orden de llegada al mismo. Cuando llega un pasajero se dirige al puesto que tenga menos gente esperando. Espera a que la enfermera correspondiente lo llame para hisoparlo, y luego se retira.

_**Nota:**_ sólo deben usar procesos Pasajero y Enfermera.
Además, suponer que existe una función `Hisopar()` que simula la atención del pasajero por parte de la enfermera correspondiente.

### Respuesta

_Nota: Este ejercicio fue consultado y esta bien resuelto._