# Práctica 3 - Monitores

## Ejercicio 1
Se dispone de un puente por el cual puede pasar un solo auto a la vez. Un auto pide permiso para pasar por el puente, cruza por el mismo y luego sigue su camino

```cpp
Monitor Puente
 cond cola;
 int cant= 0;
 Procedure entrarPuente ()
 while ( cant > 0) wait (cola);
 cant = cant + 1;
 end;
 Procedure salirPuente ()
 cant = cant – 1;
 signal(cola);
 end;
End Monitor;
Process Auto [a:1..M]
 Puente. entrarPuente (a);
 “el auto cruza el puente”
 Puente. salirPuente(a);
End Process;
```

**a.** ¿El código funciona correctamente? Justifique su respuesta.

**b.** ¿Se podría simplificar el programa? ¿Sin monitor? ¿Menos procedimientos? ¿Sin variable condition? En caso afirmativo, rescriba el código.

**c.** ¿La solución original respeta el orden de llegada de los vehículos? Si rescribió el código en el punto b), ¿esa solución respeta el orden de llegada?

### Respuesta

a. Si, es una solución correcta, ya que cuando un auto va a pasar va llamar al procedimiento de Puente entrarPuente() _(no se por qué pasa como argumento "a" supongo que eso si esta mal)_ y va a incrementar la variable cant, mientras esa variable sea mayor a 0, el resto de autos van a esperar dormidos y recién cuando el auto que esta cruzando llame a salirPuente() y disminuya la variable cant, va a despertar a un auto que está esperando para cruzar. Este va a incrementar la variable cant y va a cruzar, y así sucesivamente.

b. Si, se puede simplificar haciendo que el monitor represente el cruce del puente. Esto se debe a que solo se necesita exclusion mutua y esto de por si lo proveen los monitores.

```cpp
Monitor Puente {
    Procedure cruzarPuente (){
        //el auto cruza el puente
    }
}

Process Auto [a:1..M]{
    Puente.cruzarPuente();
}
```

c. No, no lo respeta, ya que cuando un auto se despierta, pasa a competir con el resto de autos para pasar al puente. La b. tampoco lo respeta ya que todos los autos van a competir por el uso del monitor.

## Ejercicio 2

Existen N procesos que deben leer información de una base de datos, la cual es administrada por un motor que admite una cantidad limitada de consultas simultáneas.

a) Analice el problema y defina qué procesos, recursos y monitores serán necesarios/convenientes, además de las posibles sincronizaciones requeridas para resolver el problema.

b) Implemente el acceso a la base por parte de los procesos, sabiendo que el motor de base de datos puede atender a lo sumo 5 consultas de lectura simultáneas.


### Respuesta

```cpp
Monitor BaseDeDatos {
    int nr = 0; 
    cond cv;

    Procedure pasar(){
        while(nr == 5) wait(cv); 
        nr++;
    }

    Procedure salir(){
        nr--;
        signal(cv);
    }
}

Process Proceso [p:1..N]{
    BaseDeDatos.pasar();
    //leer base de datos
    BaseDeDatos.salir();
}
```

## Ejercicio 3

Existen N personas que deben fotocopiar un documento. La fotocopiadora sólo puede ser usada por una persona a la vez. Analice el problema y defina qué procesos, recursos y
monitores serán necesarios/convenientes, además de las posibles sincronizaciones requeridas para resolver el problema. Luego, resuelva considerando las siguientes situaciones:

a) Implemente una solución suponiendo no importa el orden de uso. Existe una función Fotocopiar() que simula el uso de la fotocopiadora.

b) Modifique la solución de (a) para el caso en que se deba respetar el orden de llegada.

c) Modifique la solución de (b) para el caso en que se deba dar prioridad de acuerdo con la edad de cada persona (cuando la fotocopiadora está libre la debe usar la persona de mayor edad entre las que estén esperando para usarla).

d) Modifique la solución de (a) para el caso en que se deba respetar estrictamente el orden dado por el identificador del proceso (la persona X no puede usar la fotocopiadora hasta
que no haya terminado de usarla la persona X-1).

e) Modifique la solución de (b) para el caso en que además haya un Empleado que le indica a cada persona cuando debe usar la fotocopiadora.

f) Modificar la solución (e) para el caso en que sean 10 fotocopiadoras. El empleado le indica a la persona cuál fotocopiadora usar y cuándo hacerlo.

### Respuesta

a. 

```cpp
Process Persona [id:0..N-1]{
    Documento documento; 
    Fotocopia fotocopia;

    Fotocopiadora.Fotocopiar(documento, fotocopia);
}

Monitor Fotocopiadora {

    Procedure fotocopiar(Documento: in text, Fotocopia out text){
        Fotocopia = scan(documento);
    }
}
```

b. 

```cpp

Process Persona [id:0..N-1]{
    Documento documento; 
    Fotocopia fotocopia;

    Fotocopiadora.usar();
    Fotocopia = scan(documento);
    Fotocopiadora.dejar();
}

Monitor Fotocopiadora {
    cond cv; 
    bool libre = true;
    int esperando = 0;

    Procedure usar(){
        if (!libre) {
            esperando++;
            wait(cv);
        }
        else {
            libre = false;
        }
    }

    Procedure dejar(){
        if (esperando > 0) {
            esperando--;
            signal(cv);
        }
        else {
            libre = true;
        }
    }
}
```

c. 

```cpp
Process Persona [id:0..N-1]{
    Documento documento; 
    Fotocopia fotocopia;

    Fotocopiadora.usar(id);
    Fotocopia = scan(documento);
    Fotocopiadora.dejar();
}

Monitor Fotocopiadora {
    bool libre = true;
    cond cv[n];
    int idAux;
    int esperando = 0;
    cola c;

    Procedure usar(idP, edad: in int){
        if (!libre) {
            agregar (c, idP, edad);
            esperando++:
            wait(cv[idP]);
        }
        else {
            libre = false;
        }
    }

    Procedure dejar(){
        if (esperando > 0) {
            sacar(c,idAux);
            esperando--;
            signal(cv[idAux]);
        }
        else {
            libre = true;
        }
    }
}
```

d. 

```cpp
Process Persona [id:0..N-1]{
    Documento documento; 
    Fotocopia fotocopia;

    Fotocopiadora.usar(id);
    Fotocopia = scan(documento);
    Fotocopiadora.dejar();
}

Monitor Fotocopiadora{
    int proximo = 0;
    int cv[n];

    Procedure usar(idP: in int){
        if (idP != proximo) {
            wait(cv[idP]);
        }
    }

    Procedure dejar(){
        proximo++;
        signal(cv[proximo]);
    }
}

```

e. 

```cpp
Process Persona [id:0..N-1]{
    Documento documento; 
    Fotocopia fotocopia;

    Fotocopiadora.usar()
    Fotocopia = scan(documento);
    Fotocopiadora.dejar();
}

Process Empleado{
    int i;
    for i = 0..N-1 {
        Fotocopiadora.asignar()
    }
}

Monitor Fotocopiadora {
    int esperando = 0;
    cond empleado;
    cond persona;
    cond fin;

    Procedure usar(){
       signal(empleado);
       esperando++;
       wait(persona);
    }

    Procedure dejar(){
        signal(fin);
    }

    Procedure asignar(){
        if (esperando == 0){
            wait (empleado);
        }
        esperando--;
        signal(persona);
        wait(fin);
    }
}

```

f. 

```cpp
Process Persona [id:0..N-1]{
    Documento documento; 
    Fotocopia fotocopia;
    Fotocopiadora fotocopiadoraAsig;

    Fotocopiadora.usar(fotocopiadoraAsig);
    Fotocopia = fotocopiadoraAsig.scan(documento);
    Fotocopiadora.dejar(fotocopiadoraAsig);
}

Process Empleado{
    int i;
    for i = 0..N-1 {
        Fotocopiadora.asignar()
    }
}

Monitor Fotocopiadora {
    cola cola;
    cola fotocopiadoras = {1,2,3,4,5,6,7,8,9,10};
    cond empleado;
    cond persona[n] = ([n] = 0);
    cond fotocop;
    int asignada[n] = ([n] = 0);

    Procedure usar(fotocopiadoraAsig: out int, idP: in int){
       signal(empleado);
       cola.push(idP);
       wait(persona[idP]);
       fotocopiadoraAsig = asignada[idP];
    }

    Procedure dejar(fotocopiadoraAsig: in int){
        fotocopiadoras.push(fotocopiadoraAsig);
        signal(fotocop);
    }

    Procesudre asignar(){
        if (cola.empty()){
            wait(empleado);
        }
        idAux = cola.pop();
        if (fotocopiadoras.empty()){
            wait(fotocop);
        }
        asignada[idAux] = fotocopiadoras.pop();
        signal(persona[idAux]);

    }

}

```

## Ejercicio 4
Existen N vehículos que deben pasar por un puente de acuerdo con el orden de llegada.Considere que el puente no soporta más de 50000kg y que cada vehículo cuenta con su propio peso (ningún vehículo supera el peso soportado por el puente).

### Respuesta

```cpp
Process Vehiculo [id: 0..N-1]{
    int peso;

    Puente.pasar(peso);
    //cruzar puente
    Puente.dejar(peso);
}

Monitor Puente{
    bool libre = true;
    int esperando = 0;
    int acum = 0;
    cond auto;
    cond pesoAdecuado;

    Procedure pasar (peso: in int){
        if (!libre){
            esperando++;
            wait(auto);
        }
        else {
            libre = false;
        }
        while (acum + peso > 50000){
            wait(pesoAdecuado);
        }
        acump += peso;
        if (esperando > 0){
            esperando--;
            signal(auto);
        }
        else {
            libre = true;
        }
    }

    Procedure dejar (peso: in int){
        acum -= peso;
        signal(pesoAdecuado);
    }
}

```

## Ejercicio 5
En un corralón de materiales se deben atender a N clientes de acuerdo con el orden de llegada. Cuando un cliente es llamado para ser atendido, entrega una lista con los productos que
comprará, y espera a que alguno de los empleados le entregue el comprobante de la compra realizada.

a) Resuelva considerando que el corralón tiene un único empleado.

b) Resuelva considerando que el corralón tiene E empleados (E > 1)

### Respuesta

a. 

```cpp
Process Cliente[i:0..N-1]{
    text lista;
    text comprobante;

    Corralon.llegada(lista,comprobante);
}

Process Empleado{
    text lista;
    text comprobante;
    int j;

    for j = 0..N-1{
        Corralon.obtenerLista(lista);
        comprobante = comprobar(lista);
        Corralon.dejarComprobante(comprobante);
    }
}

Monitor Corralon{
    text listaC;
    text compE;
    cond empleado;
    cond cliente;
    cond datos;
    cond atencion;
    cont recibiComp;
    int esperando;

    Procedure llegada(lista: in text, compr: out text){
        signal(empleado);
        esperando++;
        wait(cliente);
        listaC = lista;
        signal(datos);
        wait(atencion);
        compr = compE;
        esperando--;
        signal(recibiComp);
    }

    Procedure obtenerLista(lista: out text){
        if (esperando == 0){
            wait(empleado);
        }
        signal(cliente);
        wait(datos);
        lista = listaC;
    }

    Procedure dejarComprobante(compr: in text){
        compE = compr;
        signal(atencion);
        wait(recibiComp);
    }
}

```

b. 

```cpp
Process Cliente[id:0..N-1]{
    int idE;
    text comprobante;
    text lista;
    
    Corralon.llegada(idE);
    Escritorio[idE].atencion(lista,comprobante);
}

Process Empleado[id:0..E-1]{
    int j;
    for j = 0..N-1{
        Corralon.proximo(id);
        Escritorio[id].obtenerLista(lista);
        comprobante = comprobar(lista);
        Escritorio[id].dejarComprobante(comprobante);
    }
}

Monitor Corralon{
    int cantLibres = 0;
    int esperando = 0; 
    cola eLibres;
    cond esperaC;

    Procedure llegada (idE: out int){
        if (cantLibres == 0){
            esperando++;
            wait(esperaC);
        }
        else {
            cantLibres--;
        }
        idE = eLibres.pop();
    }

    Procedure proximo(idE: in int){
        eLibres.push(idE);
        if (esperando > 0){
            esperando--;
            signal(esperaC);
        }
        else {
            cantLibres++;
        }
    }
}

Monitor Escritorio[id:0..E-1]{
    text listaC;
    text compE;
    bool hayDatos = false;
    cond datos;
    cond atencionE;

    Procedure atencion(list: in text; comp: out text){
        listaC = list;
        hayDatos = true;
        signal(datos);
        wait(atencionE);
        comp = compE;
        signal(datos);
    }

    Procedure obtenerLista(list: out text){
        if (!hayDatos){
            wait(datos);
        }
        list = listaC;
    }

    Procedure dejarComprobante(comp: in text){
        compE = comp;
        signal(atencionE);
        wait(datos);
        hayDatos = false;
    }

}

```

## Ejercicio 6

Existe una comisión de 50 alumnos que deben realizar tareas de a pares, las cuales son corregidas por un JTP. Cuando los alumnos llegan, forman una fila. Una vez que están todos en fila, el JTP les asigna un número de grupo a cada uno. Para ello, suponga que existe una función AsignarNroGrupo() que retorna un número “aleatorio” del 1 al 25. Cuando un alumno ha recibido su número de grupo, comienza a realizar su tarea. Al terminarla, el alumno le avisa al JTP y espera por su nota. Cuando los dos alumnos del grupo completaron la tarea, el JTP
les asigna un puntaje (el primer grupo en terminar tendrá como nota 25, el segundo 24, y así sucesivamente hasta el último que tendrá nota 1). __Nota__: el JTP no guarda el número de grupo que le asigna a cada alumno. 

### Respuesta

```cpp
Process Alumno[id:0..49]{
    int nro;
    int nota;

    Tarea.esperarFila(id);
    Tarea.recibirNro(id,nro);
    //realizar tarea
    Tarea.obtenerNota(nro,nota);
}

Process JTP {
    int i;
    int nroAux;
    int puntaje = 25;
    int tareasContador[25] = ([25] = 0);

    Tarea.esperarAlumnos();
    for i = 1..50 {
        nroAux = asignarNroGrupo();
        Tarea.asignarNroGrupo(nroAux);
    }

    for i = 1..50 {
        Tarea.recibirTarea(nroAux);
        tareasContador[nroAux]++;
        if (tareasContador[nroAux] == 2){
            Tarea.corregirTarea(nroAux,puntaje);
            puntaje--;
        }
    }
}

Monitor Tarea {
    int cant = 0;
    int idAux;
    cola fila;
    cola finalizadas;
    int nroAsig[50] = ([50] = -1);
    int notaTarea[25] = ([25] = -1);
    cond profesor;
    cond esperando[50];
    cond notaLista[25];

    Procedure esperarFila (idA: in int) {
        cant++;
        fila.push(idA);
        if (cant == 50){
            signal(profesor);
        }
    }

    Procedure esperarAlumnos(){
        if (cant < 50){
            wait(profesor);
        }
    }

    Procedure recibirNro (idA: in int; nro: out int) {
        wait(esperaNro[idA]);
        nro = nroAsig[idA];
    }

    Procedure asignarNro(nro: in int) {
        idAux = fila.pop();
        nroAsig[idAux] = nro;
        signal(esperaNro[idAux]);
    }

    Procedure obtenerNota (nro: in int; nota: out int) {
        finalizadas.push(nro);
        signal(profesor);
        wait(notaLista[nro]);
        nota = notaTarea[nro];
    }

    Procedure recibirTarea(nro: out int){
        if (finalizadas.empty()){
            wait(profesor);
        }
        nro = finalizadas.pop();
    }

    Procedure corregirTarea (nro: in int, puntaje: in int){
        notaTarea[nro] = puntaje;
        signal_all(notaLista[nro]);
    }
}

```

### Ejercicio 7
En un entrenamiento de fútbol hay 20 jugadores que forman 4 equipos (cada jugador conoce el equipo al cual pertenece llamando a la función DarEquipo()). Cuando un equipo está listo
(han llegado los 5 jugadores que lo componen), debe enfrentarse a otro equipo que también esté listo (los dos primeros equipos en juntarse juegan en la cancha 1, y los otros dos equipos juegan en la cancha 2). Una vez que el equipo conoce la cancha en la que juega, sus jugadores
se dirigen a ella. Cuando los 10 jugadores del partido llegaron a la cancha comienza el partido, juegan durante 50 minutos, y al terminar todos los jugadores del partido se retiran (no es necesario que se esperen para salir).

### Respuesta

```cpp

Process Jugador[id:0..19]{
    int nroEquipo;
    int canchaId;

    Jugar.elegirEquipo(nroEquipo);
    equipo[nroEquipo].listo(canchaId);
    cancha[canchaId].llegada();
}

Process Partido[id: 0..1] {
    Cancha[id].iniciar();
    //Delay 50
    Cancha[id].terminar();
}

Monitor Jugar {

    Procedure elegirEquipo(nro: out int){
        nro = DarEquipo();
    }
}

Monitor Equipo[id: 1..4] {
    int equipoCant = 0;
    cond espera;
    int cancha;

    Procedure Listo (canchaId: out int) {
        equipoCant++;
        if (equipoCant < 5){
            wait(espera);
        } 
        else {
            Administrador.calcularCancha(cancha);
            signal_all(espera);
        }
        canchaId = cancha;
    }
}

Monitor Administrador {
    int totalFormados = 0;
    
    Procedure calcularCancha(canchaId: out int) {
        totalFormados++;
        if (totalFormados <= 2){
            canchaId = 1;
        }
        else {
            canchaId = 2;
        }
    }
}

Monitor Cancha[id: 0..1] {
    int cant = 0;
    cond espera;
    cond inicio;

    Procedure llegada() {
        cant++;
        if (cant == 10){
            signal(inicio)
        }
        wait(espera);
    }

    Procedure iniciar(){
        if (cant < 10){
            wait(inicio);
        }
    }

    Procedure terminar(){
        signal_all(espera);
    }

}   

```

### Ejercicio 8
Se debe simular una maratón con C corredores donde en la llegada hay UNA máquina expendedoras de agua con capacidad para 20 botellas. Además, existe un repositor encargado
de reponer las botellas de la máquina. Cuando los C corredores han llegado al inicio comienza la carrera. Cuando un corredor termina la carrera se dirigen a la máquina expendedora, espera
su turno (respetando el orden de llegada), saca una botella y se retira. Si encuentra la máquina sin botellas, le avisa al repositor para que cargue nuevamente la máquina con 20 botellas; espera a que se haga la recarga; saca una botella y se retira. __Nota__: mientras se reponen las botellas se debe permitir que otros corredores se encolen. 

### Respuesta

```cpp

Process Corredor[id: 0..C-1]{
    
    Carrera.listo();
    Carrera.iniciar();
    //Core
    Carrera.accesoMaq();
    Maquina.usar();
    Carrera.dejarMaq();
}

Process repositor{
    while (true){
        Maquina.reponerAgua();
    }
}

Monitor Carrera {
    int cant = 0;
    int espera = 0;
    cond iniciar;
    cond cola;
    bool libre = true;

    Procedure listo(){
        cant++;
        if (cant == C){
            signal_all(iniciar);
        }
    }

    Procedure iniciar(){
        if (cant < C){
            wait(iniciar);
        }
    }

    Procedure accesoMaquina(){
        if (!libre){
            espera++;
            wait(cola);
        }
        else {
            libre = false;
        }
    }

    Procedure dejarMaq(){
        if (espera > 0){
            espera--;
            signal(cola);
        }
        else {
            libre = true;
        }
    }
}

Monitor Maquina {
    int botellas = 20;
    cond reponer;
    cond esperarBot;

    Procedure usar() {
        if (botellas == 0){
            signal(reponer);
            wait(esperarBot);
        }
        botellas--;
    }

    Procedure reponerAgua(){
        if (botellas > 0){
            wait(reponer);
        }
        botellas = 20;
        signal(esperarBot);
    }
}