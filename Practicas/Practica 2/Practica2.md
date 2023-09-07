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

a. Se debe imprimir en pantalla los ID de todos los errores críticos (no importa el orden).

b. Se debe calcular la cantidad de fallos por nivel de gravedad, debiendo quedar los resultados en un vector global.

c. Ídem b. pero cada proceso debe ocuparse de contar los fallos de un nivel de
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