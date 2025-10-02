# Practica 2
---
## Ejercicio 1
Se dispone de un puente por el cual puede pasar un solo auto a la vez. Un auto pide permiso
para pasar por el puente, cruza por el mismo y luego sigue su camino.
```C
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
a. ¿El código funciona correctamente? Justifique su respuesta.
b. ¿Se podría simplificar el programa? ¿Sin monitor? ¿Menos procedimientos? ¿Sin variable condition? En caso afirmativo, rescriba el código.
c. ¿La solución original respeta el orden de llegada de los vehículos? Si rescribió el código en el punto  ¿esa solución respeta el orden de llegada?


### A
Esta bien en terminos de que no va a haber dos al mismo tiempo, pero no respeta el orden.

--- 
## Ejercicio 2
Existen N procesos que deben leer información de una base de datos, la cual es administrada
por un motor que admite una cantidad limitada de consultas simultáneas.

a) Analice el problema y defina qué procesos, recursos y monitores/sincronizaciones
serán necesarios/convenientes para resolverlo.

b) Implemente el acceso a la base por parte de los procesos, sabiendo que el motor de
base de datos puede atender a lo sumo 5 consultas de lectura simultáneas.

### A & B
``` C
Process Persona [1..N]{
    BD.acceder()
    "Realiza la operacion de lectura"
    BD.terminar_sesion()
}

Monitor BD{
    cond cola;
    cant = 0;

    procedure acceder(){
        while(cant == 5) wait(cola);
        cant++
    }

    procedure terminar_sesion(){
        signal(cola);
        cant--;
    }
}
```

---
## Ejercicio 3
Existen N personas que deben fotocopiar un documento. La fotocopiadora sólo puede ser usada por una persona a la vez. Analice el problema y defina qué procesos, recursos y monitores serán necesarios/convenientes, además de las posibles sincronizaciones requeridas para resolver el problema. Luego, resuelva considerando las siguientes situaciones:

a) Implemente una solución suponiendo no importa el orden de uso. Existe una función
Fotocopiar() que simula el uso de la fotocopiadora.

b) Modifique la solución de (a) para el caso en que se deba respetar el orden de llegada.

c) Modifique la solución de (b) para el caso en que se deba dar prioridad de acuerdo con la edad de cada persona (cuando la fotocopiadora está libre la debe usar la persona de mayor edad entre las que estén esperando para usarla).

d) Modifique la solución de (a) para el caso en que se deba respetar estrictamente el orden dado por el identificador del proceso (la persona X no puede usar la fotocopiadora hasta que no haya terminado de usarla la persona X-1).

e) Modifique la solución de (b) para el caso en que además haya un Empleado que le indica
a cada persona cuando debe usar la fotocopiadora.

f) Modificar la solución (e) para el caso en que sean 10 fotocopiadoras. El empleado le indica a la persona cuál fotocopiadora usar y cuándo hacerlo.

``` C
Monitor Fotocopiadora{
    cond cola;
    cant=0
}

Process Persona[1..N]{

}
```

### A
``` C
Monitor Fotocopiadora{
    cond cola;
    cant=0

    procedure usar(){
        while(cant>0) wait(cola);
        cant++
    }

    procedure terminar(){
        cant--
        signal(cola)
    }
}

Process Persona[1..N]{
    fotocopiadora.usar()
    Fotocopiar()
    fotocopiadora.terminar()
}
```

### B
```C
    Monitor Fotocopiadora{
    cond cola;
    cant=0
    libre=true

    procedure usar(){
        if(!libre){
            cant++
            wait(cola)
        }
        libre=false
    }

    procedure terminar(){
        if(cant>0){
            signal(cola)
            cant--
        }
        else libre=true
    }
}

Process Persona[1..N]{
    fotocopiadora.usar()
    Fotocopiar()
    fotocopiadora.terminar()
}
```
### C
``` C
Monitor Fotocopiadora{
    cond espera[1..N];
    colaOrdenada fila;
    libre=true
    cant=0

    procedure usar(edad,id){
        if(!libre){
            cant++
            fila.push(edad,id)
            wait(espera[id])
        }
        libre=false
    }

    procedure terminar(){
        if(cant>0){
            edad,id = fila.pop();
            signal(espera[id])
            cant--
        }
        else libre=true
    }
}

Process Persona[ID:1..N]{
    edad=Random(0..100)
    fotocopiadora.usar(edad,ID)
    Fotocopiar()
    fotocopiadora.terminar()
}
```

### D
``` C
Monitor Fotocopiadora{
    cond espera[1..N];
    proximo=0

    procedure usar(id){
        if(proximo != id){
            wait(espera[id])
        }
    }

    procedure terminar(){
        if(proximo < N){
            proximo++
            signal(espera[proximo])
        }
    }
}

Process Persona[id:0..N]{
    fotocopiadora.usar(id)
    Fotocopiar()
    fotocopiadora.terminar()
}
```
### E
``` C
Monitor Fotocopiadora{
    cond cola;
    cond empleado;
    cant=0
    libre=true

    procedure usar(){
        cant++
        signal(empleado)
        wait(cola)
    }

    procedure avisar(){
        while(cant==0 || !libre){
            wait(empleado)
        }
        cant--
        libre=false
        signal(cola)
    }

    procedure terminar(){
        libre=true
        signal(empleado)
    }
}

Process Persona[1..N]{
    fotocopiadora.usar()
    Fotocopiar()
    fotocopiadora.terminar()
}

Process Empleado{
    for i=0 to N{
        fotocopiadora.avisar()
    }
}
```
---
## Ejercicio 4
Existen N vehículos que deben pasar por un puente de acuerdo con el orden de llegada Considere que el puente no soporta más de 50000kg y que cada vehículo cuenta con su propio peso (ningún vehículo supera el peso soportado por el puente).

```C
Monitor Puente{
    capacidad=50000
    cond cola;
    cant=0

    procedure ingresar(peso){
        if(cant > 0 || capacidad < peso){
            cant++
            wait(cola)
            cant--
        }
        capacidad=capacidad-peso
    }

    procedure salir(peso){
        capacidad = capacidad + peso
        if(cant>0){
            signal(cola)
        }
    }
}

Process Vehiculo[1..N]{
    peso=Random()
    puente.ingresar(peso)
    "Pasa por el puente"
    puente.salir(peso)
}
```
---
## Ejercicio 5
En un corralón de materiales se deben atender a N clientes de acuerdo con el orden de llegada Cuando un cliente es llamado para ser atendido, entrega una lista con los productos que comprará, y espera a que alguno de los empleados le entregue el comprobante de la compra realizada.

a) Resuelva considerando que el corralón tiene un único empleado.

b) Resuelva considerando que el corralón tiene E empleados (E > 1). Los empleados no deben terminar su ejecución.

c) Modifique la solución (b) considerando que los empleados deben terminar su ejecución cuando se hayan atendido todos los clientes.

### A
```C 
Monitor Corralon{
    cond cola;
    cond empleado;
    cant=0
    comprobantes[1..N]
    colaOrdenada listaPendientes

    procedure entregarLista(listaProductos,id){
        insertar(listaPendientes,listaProductos,id)
        cant++
        signal(empleado)
        wait(cola)    
    }

    procedure entregarComprobante(){
        if(cant == 0){
            wait(empleado)
        }
        listaProductos,id = listaPendientes.pop()
        comprobantes[id] = generarComprobante(listaProductos)
        cant--
        signal(cola)
    }

    procedure tomarComprobante(id){
        return comprobantes[id]
    }
}

Process Cliente[id:1..N]{
    listaProductos=...
    Corralon.entregarLista(listaProductos,id)
    comprobante = Corralon.tomarComprobante(id)
}

Process Empleado{
    for i=1 to N{
        Corralon.entregarComprobante()
    }
}
```
### B
```C
Monitor Corralon{
    cond espera[1..N];
    cond empleado;
    cant=0
    comprobantes[1..N]
    colaOrdenada listaPendientes

    procedure entregarLista(listaProductos,id){
        insertar(listaPendientes,listaProductos,id)
        cant++
        signal(empleado)
        wait(espera[id])    
    }

    procedure entregarComprobante(){
        while(cant == 0){
            wait(empleado)
        }
        listaProductos,id = listaPendientes.pop()
        comprobantes[id] = generarComprobante(listaProductos)
        cant--
        signal(espera[id])
    }

    procedure tomarComprobante(id){
        return comprobantes[id]
    }
}

Process Empleado[1..E]{
    while(true){
        Corralon.entregarComprobante()
    }
}

Process Cliente[1..N]{
    listaProductos=...
    Corralon.entregarLista(listaProductos,id)
    comprobante = Corralon.tomarComprobante(id)
}
```
### C
```C
Monitor Corralon{
    cond espera[1..N];
    cond empleado;
    cant=0
    comprobantes[1..N]
    colaOrdenada listaPendientes
    contador=0 

    procedure entregarLista(listaProductos,id){
        insertar(listaPendientes,listaProductos,id)
        cant++
        signal(empleado)
        wait(espera[id])    
    }

    procedure entregarComprobante(){
        while(cant == 0 && contador < N){
            wait(empleado)
        }
        if (contador < N){
            listaProductos,id = listaPendientes.pop()
            comprobantes[id] = generarComprobante(listaProductos)
            cant--
            contador++
            signal(espera[id])
        }
        else {
            signalAll(empleado)
        }
    }

    procedure tomarComprobante(comprobante:out,id){
        comprobante = comprobantes[id]
    }

    procedure quedanClientes(ok:out){
        ok = contador < N
    }
}

Process Empleado[1..E]{
    bool ok
    corralon.quedanClientes(ok)
    while(ok){
        Corralon.entregarComprobante()
        corralon.quedanClientes(ok)
    }
}

Process Cliente[1..N]{
    listaProductos=...
    comprobante=...
    Corralon.entregarLista(listaProductos,id)
    Corralon.tomarComprobante(comprobante,id)
}
```

### C con DOS MONITORES (Mejora de eficiencia)
```C
// Monitor para gestionar las listas de productos
Monitor GestorListas{
    cond empleado;
    colaOrdenada listaPendientes
    cant=0
    
    procedure entregarLista(listaProductos,id){
        insertar(listaPendientes,listaProductos,id)
        cant++
        signal(empleado)
    }
    
    procedure tomarLista(listaProductos:out,id:out){
        while(cant == 0){
            wait(empleado)
        }
        listaProductos,id = listaPendientes.pop()
        cant--
    }
    
    procedure hayTrabajos(resultado:out){
        resultado = (cant > 0)
    }
}

// Monitor para gestionar los comprobantes
Monitor GestorComprobantes{
    cond espera[1..N];
    comprobantes[1..N]
    contador=0
    
    procedure esperarComprobante(id){
        wait(espera[id])
    }
    
    procedure entregarComprobante(comprobante,id){
        comprobantes[id] = comprobante
        contador++
        signal(espera[id])
    }
    
    procedure tomarComprobante(comprobante:out,id){
        comprobante = comprobantes[id]
    }
    
    procedure quedanClientes(ok:out){
        ok = contador < N
    }
}

Process Cliente[1..N]{
    listaProductos=...
    comprobante=...
    
    GestorListas.entregarLista(listaProductos,id)
    GestorComprobantes.esperarComprobante(id)
    GestorComprobantes.tomarComprobante(comprobante,id)
}

Process Empleado[1..E]{
    bool ok, hayTrabajo
    listaProductos, comprobante, id
    
    GestorComprobantes.quedanClientes(ok)
    while(ok){
        GestorListas.tomarLista(listaProductos,id)
        comprobante = generarComprobante(listaProductos)
        GestorComprobantes.entregarComprobante(comprobante,id)
        GestorComprobantes.quedanClientes(ok)
    }
}
```

**Ventajas de usar dos monitores:**

1. **Menor contención**: Los empleados solo compiten por acceso al `GestorListas` cuando toman trabajos, no durante todo el procesamiento
2. **Mayor paralelismo**: Mientras un empleado procesa un comprobante, otros pueden tomar nuevas listas sin bloqueo
3. **Separación de responsabilidades**: 
   - `GestorListas`: Maneja la cola de trabajos pendientes
   - `GestorComprobantes`: Maneja la entrega de resultados
4. **Mejor rendimiento**: Los empleados pasan menos tiempo bloqueados esperando acceso al monitor

---
## 6 
Existe una comisión de 50 alumnos que deben realizar tareas de a pares, las cuales son corregidas por un JTP. Cuando los alumnos llegan, forman una fila. Una vez que están todos en fila, el JTP les asigna un número de grupo a cada uno. Para ello, suponga que existe una función AsignarNroGrupo() que retorna un número “aleatorio” del 1 al 25.
Cuando un alumno ha recibido su número de grupo, comienza a realizar su tarea. Al terminarla, el alumno le avisa al JTP y espera por su nota. Cuando los dos alumnos del grupo completaron la tarea, el JTP les asigna un puntaje (el primer grupo en terminar tendrá como nota 25, el segundo 24, y así sucesivamente hasta el último que tendrá nota 1). Nota: el JTP no guarda el número de grupo que le asigna a cada alumno.

```C
Monitor Fila{
    cant=0
    cond cola;
    cond jtp;
    nroGrupo[1..50]

    procedure esperar(){
        cant++
        if(cant < 50){
            wait(cola)
        }
        else{
            signal(jtp)
            wait(cola)
        }
    }

    procedure asignarGrupo(){
        if(cant < 50){
            wait(jtp)
        }
        for i=1 to 50{
            nroGrupo[i] = AsignarNroGrupo()
        }
        signalAll(cola)
    }

    procedure leerNroGrupo(grupo:out,id){
        grupo = nroGrupo[id]
    }
}

Monitor ProcesarTareas{
    cond espera[1..25]
    cola tareaTerminada;
    cond jtp
    notas[1..25]
    terminaron[1..25]=0
    nota=25

    procedure esperarAlumnos(){
        while(nota > 0){
            if(tareaTerminada.isEmpty()){
                wait(jtp)
            }
            nroGrupo = tareaTerminada.pop()
            terminaron[nroGrupo]++
            if(terminaron[nroGrupo] == 2){
                notas[nroGrupo]= nota
                signalAll(espera[nroGrupo])
                nota--
            }
        }
    }

    procedure terminarTarea(nroGrupo){
        tareaTerminada.push(nroGrupo)
        signal(jtp)
        wait(espera[nroGrupo])
    }

    procedure leerNota(nota:out,nroGrupo){
        nota = notas[nroGrupo]
    }
}

Process Alumno[id:1..50]{
    nroGrupo=...
    nota=...
    Fila.esperar()
    Fila.LeerNroGrupo(nroGrupo,id)
    "Hace su tarea"
    ProcesarTareas.terminarTarea(nroGrupo)
    ProcesarTareas.leerNota(nota,nroGrupo)
}

Process JTP{
    Fila.asignarGrupo()
    ProcesarTareas.esperarAlumnos()
}
```
---
## 7
Se debe simular una maratón con C corredores donde en la llegada hay UNA máquina expendedoras de agua con capacidad para 20 botellas. Además, existe un repositor encargado de reponer las botellas de la máquina. Cuando los C corredores han llegado al inicio comienza la carrera. Cuando un corredor termina la carrera se dirigen a la máquina expendedora, espera su turno (respetando el orden de llegada), saca una botella y se retira. Si encuentra la máquina sin botellas, le avisa al repositor para que cargue nuevamente la máquina con 20 botellas; espera a que se haga la recarga; saca una botella y se retira. Nota: mientras se reponen las botellas se debe permitir que otros corredores se encolen. 

```C
Monitor Maquina{
    botellas=20
    cond repositor;
    cond consumidor;
    contador=0

    procedure tomarBotella(botella:out,sucess:out){
        if(botellas>0){
            botella='Saca botella'
            botellas--
            contador++
            sucess=true
            if(contador == C){
                signal(repositor)
            }
        }
        else{
            signal(repositor)
            wait(consumidor)
        }

    }

    procedure reponerBotellas(){
        while(contador < C){
            if(botellas > 0){
                wait(repositor)
            }
            if(contador < C){
                for i=1 to 20{
                    botellas++
                }
                signal(consumidor)
            }
        }
    }
}
Monitor Fila{
    cond cola;
    maquinaOcupada=false
    cant=0

    procedure esperar(){
        if(maquinaOcupada){
            cant++
            wait(cola)
        }
        maquinaOcupada = true
    }

    procedure terminar(){
        if(cant>0){
            cant--
            signal(cola)
        }
        else{
            maquinaOcupada = false
        }
    }

}
Monitor Carrera{
    cond espera:
    cant=0
    procedure esperarComienzo(){
        cant++
        if(cant < C){
            wait(espera)
        }
        else{
            signalAll(espera)
        }
    }
}
Process Corredor[1..C]{
    botella=...
    sucess=false
    Carrera.espera()
    "Corre la carrera"
    Fila.esperar()
    while(!sucess){
        Maquina.tomarBotella(botella)
    }
    Fila.terminar()
}

Process Repositor{
    Maquina.reponerBotellas()
}

```