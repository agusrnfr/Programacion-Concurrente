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