# Práctica 4 - Pasaje de Mensajes

# PMA

## Ejercicio 1
Suponga que N clientes llegan a la cola de un banco y que serán atendidos por sus empleados. Analice el problema y defina qué procesos, recursos y comunicaciones serán necesarios/convenientes para resolver el problema. Luego, resuelva considerando las siguientes situaciones:

a. Existe un único empleado, el cual atiende por orden de llegada.

b. Ídem a) pero considerando que hay 2 empleados para atender, ¿qué debe
modificarse en la solución anterior?

c. Ídem c) pero considerando que, si no hay clientes para atender, los empleados realizan tareas administrativas durante 15 minutos. ¿Se puede resolver sin usar procesos adicionales? ¿Qué consecuencias implicaría?

### Respuesta

a.
```cpp
chan atencion(int);

Process Cliente [id: 0..N-1] {
    send atencion(id);
}

Process Empleado {
    int idAux;
    while (true) {
        receive atencion(idAux);
        atender(idAux);
    }
}
```

b.
```cpp
chan atencion(int);

Process Cliente [id: 0..N-1] {
    send atencion(id);
}

Process Empleado [id: 0..1] {
    int idAux;
    while (true) {
        receive atencion(idAux);
        atender(idAux);
    }
}
```

c.
```cpp
chan atencion(int);
chan pedido(int);
chan siguiente[2](int);

Process Cliente [id: 0..N-1] {
    send atencion(id);
}

Process Admin {
    int idE;
    int idC;

    while (true) {
        receive pedido(idE);
        if (empty (atencion)) {
            idC = -1;
        } else {
            receive atencion(idC);
            send siguiente[idE](idC);
        }
    }
}

Process Empleado [id: 0..1] {
    int idC;
    while (true) {
        send pedido(id);
        receive siguiente[id](idC);
        if (idC == -1) {
           delay(900)
        } else {
            atender(idC);
        }
    }
}
```

## Ejercicio 2
Se desea modelar el funcionamiento de un banco en el cual existen 5 cajas para realizar pagos. Existen P clientes que desean hacer un pago. Para esto, cada una selecciona la caja donde hay menos personas esperando; una vez seleccionada, espera a ser atendido. En cada caja, los clientes son atendidos por orden de llegada por los cajeros. Luego del pago, se les
entrega un comprobante. __Nota:__ maximizando la concurrencia.

### Respuesta
```cpp
chan pedido[5](text, int);
chan comprobante[P](text);
chan buscarCaja(int);
chan obtenerCaja[P](int);
chan liberarCaja(int);
chan hayPedido(bool);

Process Caja [id: 0..4] {
    int idAux;
    text pago;
    text comprobante;
    while(true) {
        receive pedido[id](pago, idAux);
        generarComprobante(pago, comprobante);
        send comprobante[idAux](comprobante);
    }
}

Process Cliente [id: 0..P-1] {
    int idCaja;
    text pago;
    text comprobante;

    send buscarCaja(id);
    send hayPedido(true);
    receive obtenerCaja[id](idCaja);
    send pedido[idCaja](pago, id);
    receive comprobante[id](comprobante);
    send liberarCaja(idCaja);
    send hayPedido(true);

}


Process Admin {
    int cantEspera[5] = ([5] 0);
    int min;
    int idAux;
    bool pedido;

    while (true) {
        receive hayPedido(pedido);
        if (!empty (buscarCaja) && empty(liberarCaja)) {
            receive buscarCaja(idAux);
            min = cajaMasVacia(cantEspera);
            cantEspera[min]++;
            send obtenerCaja[idAux](min);
        } else {
            if (!empty (liberarCaja)) {
                receive liberarCaja(idAux);
                cantEspera[idAux]--;
            }
        }
    }
}
```

## Ejercicio 3
Se debe modelar el funcionamiento de una casa de comida rápida, en la cual trabajan 2 cocineros y 3 vendedores, y que debe atender a C clientes. El modelado debe considerar que:

- Cada cliente realiza un pedido y luego espera a que se lo entreguen.
- Los pedidos que hacen los clientes son tomados por cualquiera de los vendedores y se lo pasan a los cocineros para que realicen el plato. Cuando no hay pedidos para atender, los vendedores aprovechan para reponer un pack de bebidas de la heladera (tardan entre 1 y 3 minutos para hacer esto).
- Repetidamente cada cocinero toma un pedido pendiente dejado por los vendedores, lo cocina y se lo entrega directamente al cliente correspondiente.

__Nota:__ maximizar la concurrencia.

### Respuesta
```cpp
chan realizarPedido(text, int);
chan obtenerPlato[C](text);
chan obtenerPedido(int);
chan retornarPedido[3](text,int);
chan pedidoPendiente(text, int);

Process Cliente [id:0..C-1] {
    text pedido;
    text plato;

    send realizarPedido(pedido, id);
    receive obtenerPlato[id](plato);
}

Process Coordinador {
    text pedido;
    int idVendedor;
    int idCliente;

    while (true) {
        receive obtenerPedido(idVendedor);
        if (empty (realizarPedido)) {
            idCliente = -1;
            pedido = "vacio";
        } else {
            receive realizarPedido(pedido, idCliente);
        }
        send retornarPedido[idVendedor](pedido, idCliente);
    }
}

Process Vendedor [id: 0..2] {
    text pedido;
    int idCliente;

    while (true) {
        send obtenerPedido(id);
        receive retornarPedido[id](pedido, idCliente);
        if (pedido != "vacio") {
            send pedidoPendiente(pedido, idCliente);
        } else {
            delay(60..180);
        }
    }
}

Process Cocinero [id: 0..1] {
    text pedido;
    text plato;
    int idCliente;

    while (true) {
        receive pedidoPendiente(pedido, idCliente);
        plato = cocinar(pedido);
        send obtenerPlato[idCliente](plato);
    }
}
``` 

## Ejercicio 4
Simular la atención en un locutorio con 10 cabinas telefónicas, el cual tiene un empleado que se encarga de atender a N clientes. Al llegar, cada cliente espera hasta que el empleado le indique a qué cabina ir, la usa y luego se dirige al empleado para pagarle. El empleado atiende a los clientes en el orden en que hacen los pedidos, pero siempre dando prioridad a los que terminaron de usar la cabina. A cada cliente se le entrega un ticket factura. __Nota:__ maximizar la concurrencia; suponga que hay una función Cobrar() llamada por el empleado que simula que el empleado le cobra al cliente.

### Respuesta
```cpp
chan solicitarCabina(int);
chan obtenerCabina[N](int);
chan pagarEmpleado(int,int);
chan obtenerTicket[N](text);

Process Cliente [id: 0..N-1] {
    int cabina;
    text ticket;

    send solicitarCabina(id);
    receive obtenerCabina[id](cabina);
    usarCabina(cabina);
    send pagarEmpleado(id, cabina);
    receive obtenerTicket[id](ticket);
}

Process Empleado {
    bool cabinasOcup[10] = ([10] false);
    int idAux;
    int cabinaId;
    text ticket;

    while (true) {
        if (!empty (pagarEmpleado)){
            receive pagarEmpleado(idAux, cabinaId);
            cabinasOcup[cabinaId] = false;
            tickect = cobrar(idAux);
            send obtenerTicket[idAux](ticket);
        } else {
            if (!empty (solicitarCabina) && hayCabinaLibre(cabinasOcup)) {
                receive solicitarCabina(idAux);
                cabinaId = obtenerCabinaLibre(cabinasOcup);
                cabinasOcup[cabinaId] = true;
                send obtenerCabina[idAux](cabinaId);
            }
        }
    }
}

```

## Ejercicio 5
Resolver la administración de las impresoras de una oficina. Hay 3 impresoras, N usuarios y 1 director. Los usuarios y el director están continuamente trabajando y cada tanto envían documentos a imprimir. Cada impresora, cuando está libre, toma un documento y lo imprime, de acuerdo con el orden de llegada, pero siempre dando prioridad a los pedidos
del director. __Nota:__ los usuarios y el director no deben esperar a que se imprima el documento.

### Respuesta
```cpp
chan imprimirDocU(text);
chan imprimirDocD(text);
chan hayDoc(bool);


Process Usuario [id: 0..N-1] {
    text doc;

    while (true) {
        doc = generarDoc();
        send imprimirDocU(doc);
        send hayDoc(true);
    }
}

Process Director {
    text doc;

    while (true) {
        doc = generarDoc();
        send imprimirDocD(doc);
        send hayDoc(true);
    }
}

Process Impresora [id: 0..2] {
    text doc;
    bool hay;

    while (true) {
        receive hayDoc(hay);

        if (!empty (imprimirDocD)) {
            receive imprimirDocD(doc);
        } else {
            receive imprimirDocU(doc);
        }
        imprimir(doc);
    }
}
```

# PMS

## Ejercicio 1
Suponga que existe un antivirus distribuido que se compone de R procesos robots Examinadores y 1 proceso Analizador. Los procesos Examinadores están buscando continuamente posibles sitios web infectados; cada vez que encuentran uno avisan la dirección y luego continúan buscando. El proceso Analizador se encarga de hacer todas las pruebas necesarias con cada uno de los sitios encontrados por los robots para determinar si están o no infectados. 

a. Analice el problema y defina qué procesos, recursos y comunicaciones serán necesarios/convenientes para resolver el problema.

b. Implemente una solución con PMS.

### Respuesta

```cpp
Process Examinador [id: 0..R-1] {
    text sitioWeb;

    while (true) {
        sitioWeb = encontrarInfectado();
        Admin!aviso(sitioWeb);
    }
}

Process Analizador {
    text sitioWeb;

    while (true) {
        Admin!pedido();
        Admin?aviso(sitioWeb);
        analizar(sitioWeb);
    }
}

Process Admin {
    cola avisos;
    text sitioWeb;

    do Examinador[*]?Aviso(sitioWeb) -> avisos.push(sitioWeb);
    [] !empty(avisos);Analizador?Pedido() -> 
                    Analizador!Aviso(avisos.pop());
    od
}   
```

## Ejercicio 2
En un laboratorio de genética veterinaria hay 3 empleados. El primero de ellos continuamente prepara las muestras de ADN; cada vez que termina, se la envía al segundo empleado y vuelve a su trabajo. El segundo empleado toma cada muestra de ADN preparada, arma el set de análisis que se deben realizar con ella y espera el resultado para archivarlo. Por último, el tercer empleado se encarga de realizar el análisis y devolverle el
resultado al segundo empleado.

### Respuesta
```cpp

Process Empleado1 {
    text muestra;

    while (true) {
        muestra = realizarMuestra();
        Admin!enviaMuestraPrep(muestra);
    }
}

Process Empleado2 {
    text muestra;
    text set;
    text analisis;

    while (true) {
        Admin!pedido();
        Admin?obtenerMuestraPrep(muestra);
        set = armarSet(muestra);
        Empleado3!enviarSet(set);
        Empleado3?obtenerAnalisis(analisis);
        archivar(analisis);
    }
}

Process Empleado3 {
    text set;
    text analisis;
    while (true) {
        Empleado2?enviarSet(set);
        analisis = analizarSet(set);
        Empleado2!obtenerAnalisis(analisis);
    }
}

Process Admin {
    cola muestras;
    text muestra;

    do Empleado1?enviarMuestraPrep(muestra) -> muestras.push(muestra);
    [] !empty(muestras);Empleado2?Pedido() -> 
                    Empleado2!obtenerMuestraPrep(muestras.pop());
    od
}
```

## Ejercicio 3
En un examen final hay N alumnos y P profesores. Cada alumno resuelve su examen, lo entrega y espera a que alguno de los profesores lo corrija y le indique la nota. Los profesores corrigen los exámenes respetando el orden en que los alumnos van entregando.

a.  Considerando que P=1.

b. Considerando que P>1.

c. Ídem b) pero considerando que los alumnos no comienzan a realizar su examen hasta que todos hayan llegado al aula.

__Nota:__ maximizar la concurrencia y no generar demora innecesaria.

### Respuesta

a.
```cpp

Process Alumno [id: 0..N-1] {
    text examen;
    int nota;

    while (true) {
        examen = resolverExamen();
        Admin!examen(examen, id);
        Profesor?nota(nota);
    }
}

Process Profesor {
    int idAux;
    int nota;
    text examen;

    while (true) { 
        Admin!pedido();
        Admin?examen(examen, idAux);
        nota = corregirExamen(examen);
        Alumno[idAux]!nota(nota);
    }
}

Process Admin {
    text examen;
    cola cola;
    int idAux;

    do Alumno[*]?examen(examen, idAux) -> cola.push(examen, idAux);
    [] !empty(cola);Profesor?pedido() -> 
                    Profesor!examen(cola.pop());
    od
}
```

b.
```cpp
Process Alumno [id: 0..N-1] {
    text examen;
    int nota;

    while (true) {
        examen = resolverExamen();
        Admin!examen(examen, id);
        Profesor[*]?nota(nota);
    }
}

Process Profesor [id: 0..P-1] {
    int idAux;
    int nota;
    text examen;

    while (true) { 
        Admin!pedido(id);
        Admin?examen(examen, idAux);
        nota = corregirExamen(examen);
        Alumno[idAux]!nota(nota);
    }
}

Process Admin {
    text examen;
    cola cola;
    int idAux;
    int idP;

    do Alumno[*]?examen(examen, idAux) -> cola.push(examen, idAux);
    [] !empty(cola);Profesor[*]?pedido(idP) -> 
                    Profesor[idP]!examen(cola.pop());
    od
}
``` 

c.
```cpp
Process Alumno [id: 0..N-1] {
    text examen;
    int nota;

    while (true) {
        Admin!llegue();
        Admin?comenzar();
        examen = resolverExamen();
        Admin!examen(examen, id);
        Profesor[*]?nota(nota);
    }
}

Process Profesor [id: 0..P-1] {
    int idAux;
    int nota;
    text examen;

    while (true) { 
        Admin!pedido(id);
        Admin?examen(examen, idAux);
        nota = corregirExamen(examen);
        Alumno[idAux]!nota(nota);
    }
}

Process Admin {
    int total = 0;
    int idAux;
    int idP;
    text examen;
    cola cola;

    for i = 1..N --> Alumno[*]?llegue();

    for i = 1..N --> Alumno[i]!comenzar();

    do Alumno[*]?examen(examen, idAux) -> cola.push(examen, idAux);
    [] !empty(cola);Profesor[*]?pedido(idP) -> 
                    Profesor[idP]!examen(cola.pop());
    od
}
```

## Ejercicio 4
En una exposición aeronáutica hay un simulador de vuelo (que debe ser usado con exclusión mutua) y un empleado encargado de administrar su uso. Hay P personas que esperan a que el empleado lo deje acceder al simulador, lo usa por un rato y se retira. El empleado deja usar el simulador a las personas respetando el orden de llegada.

__Nota:__ cada persona usa sólo una vez el simulador. 

### Respuesta
```cpp

Process Persona [id: 0..P-1] {
    Empleado!solicitarPaso(id);
    Empleado?pasar();
    usarSimulador();
    Empleado!salir();
}

Process Empleado {
    cola cola;
    int idAux;
    bool libre = true;

    do Persona[*]?solicitarPaso(idAux) ->
                    if (!libre) {
                        cola.push(idAux);
                    } else {
                        libre = false;
                        Persona[idAux]!pasar();
                    }
    [] Persona[*]?salir() ->
                    if (empty(cola)) {
                        libre = true;
                    } else {
                        idAux = cola.pop();
                        Persona[idAux]!pasar();
                    }
    od
}

```

## Ejercicio 5

En un estadio de fútbol hay una máquina expendedora de gaseosas que debe ser usada por E Espectadores de acuerdo al orden de llegada. Cuando el espectador accede a la máquina en su turno usa la máquina y luego se retira para dejar al siguiente. 

__Nota:__ cada Espectador una sólo una vez la máquina.

### Respuesta
```cpp

Process Espectador [id: 0..E-1] {
    Maquina!solicitarUso(id);
    Maquina?dejarUsar();
    usarMaquina();
    Maquina!liberarUso();
}

Process Maquina {
    cola cola;
    int idAux;
    bool libre = true;

    do Espectador[*]?solicitarUso(idAux) ->
                    if (!libre) {
                        cola.push(idAux);
                    } else {
                        libre = false;
                        Espectador[idAux]!dejarUsar();
                    }
    [] Espectador[*]?liberarUso() ->
                    if (empty(cola)) {
                        libre = true;
                    } else {
                        idAux = cola.pop();
                        Espectador[idAux]!dejarUsar();
                    }
    od
}

``` 
