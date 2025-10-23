# Semáforos
## 1
a) En una estación de trenes, asisten P personas que deben realizar una carga de su tarjeta SUBE en la terminal disponible. La terminal es utilizada en forma exclusiva por cada persona de acuerdo con el orden de llegada. Implemente una solución utilizando únicamente procesos Persona. Nota: la función UsarTerminal() le permite cargar la SUBE en la terminal disponible.

b) Resuelva el mismo problema anterior pero ahora considerando que hay T terminales disponibles. Las personas realizan una única fila y la carga la realizan en la primera terminal que se libera. Recuerde que sólo debe emplear procesos Persona. Nota: la función UsarTerminal(t) le permite cargar la SUBE en la terminal t

### A
```C
libre=true
cola c
sem mutex=1
sem espera[1..P]=0
Process Persona[id:1..P]{
    p(mutex)
    if(libre){
        libre=false
        v(mutex)
    }
    else{
        c.push(id)
        v(mutex)
        p(espera[id])
    }
    usarTerminal()
    p(mutex)
    if(cola.isEmpty()){
        libre=true
        v(mutex)
    }
    else{
        id= c.pop()
        v(mutex)
        v(espera[id])
    }
}
```

### B

```C
sem espera[1..P]=0
terminalAsignada[1..P]
cola terminalesLibres = 1..T
cola c
sem mutex=1 
Process Persona[id=1..P]{
    p(mutex)
    if(terminalesLibres.isEmpty()){
        c.push(id)
        v(mutex)
        p(espera[id])
        t = terminalAsignada[id]
    }
    else{
        t = terminalesLibres.pop()
        v(mutex)
    }
    usarTerminal(t)
    p(mutex)
    if(c.isEmpty()){
        terminalesLibres.push(t)
    }
    else{
        id= c.pop()
        terminalAsignada[id] = t
        v(espera[id])
    }
    v(mutex)
}
```
---
## 2
Implemente una solución para el siguiente problema. Un sistema debe validar un conjunto de 10000 transacciones que se encuentran disponibles en una estructura de datos. Para ello, el sistema dispone de 7 workers, los cuales trabajan colaborativamente validando de a 1 transacción por vez cada uno. 
Cada validación puede tomar un tiempo diferente y para realizarla los workers disponen de la función Validar(t), la cual retorna como resultado un número entero entre 0 al 9. Al finalizar el procesamiento, el último worker en terminar debe informar la cantidad de transacciones por cada resultado de la función de validación. Nota: maximizar la concurrencia. 

```C
sem mutex=1
sem mutex2=1
cola transacciones=1...10000
cantTotal[0..9]=0
terminaron=0

Process Worker[Id:1..7]{
    cant[0..9]=0
    p(mutex)
    while(!transacciones.isEmpty()){
        t = transacciones.pop()
        v(mutex)
        num = validar(t)
        cant[num]++
        p(mutex)
    }
    v(mutex)
    p(mutex2)
    terminaron++
    for i=0 to 9{
        cantTotal[i]=cantTotal[i]+cant[i]
    }
    if(terminaron == 7){
        for i=0 to 9{
            print(cantTotal[i])
        }
    }
    v(mutex2)
}
```
---
## 3
Implemente una solución para el siguiente problema. Se debe simular el uso de una máquina expendedora de gaseosas con capacidad para 100 latas por parte de U usuarios. Además, existe un repositor encargado de reponer las latas de la máquina. Los usuarios usan la máquina según el orden de llegada. Cuando les toca usarla, sacan una lata y luego se retiran. En el caso de que la máquina se quede sin latas, entonces le debe avisar al repositor para que cargue nuevamente la máquina en forma completa. Luego de la recarga, saca una botella y se retira. Nota: maximizar la concurrencia; mientras se reponen las latas se debe permitir que otros usuarios puedan agregarse a la fila.

```C
latas=100
cola c
sem repositor=0;
sem primero=0;
sem mutex=1;
sem mutex2=1
sem espera[1..U]=0
libre=true
cant++
Process Usuarios[id:1..U]{
    p(mutex)
    if(libre){
        libre=false
        v(mutex)
    }
    else{
        cola.push(id)
        v(mutex)
        p(espera[id])
    }
    if(botellas < 1){
        v(repositor)
        p(primero)
    }
    "Toma botella"
    botellas--
    p(mutex)
    if(cola.isEmpty()){
        libre=true
        v(mutex)
    }
    else{
        id = cola.pop();
        v(mutex)
        v(espera[id])
    }
    p(mutex2)
    cant++
    if(cant==U){
        v(repositor)
    }
    v(mutex2)
}

Process Repositor{
    while(cant < U){
        p(repositor)
        if(cant < U){
            for i=1 to 100{
                botellas++
            }
            v(primero)
        }
    }
}
```
---
# Monitores
## 1
Resolver el siguiente problema. En una elección estudiantil, se utiliza una máquina para voto electrónico. Existen N Personas que votan y una Autoridad de Mesa que les da acceso a la máquina de acuerdo con el orden de llegada, aunque ancianos y embarazadas tienen prioridad sobre el resto. La máquina de voto sólo puede ser usada por una persona a la vez. Nota: la función Votar() permite usar la máquina.

```C
Monitor Maquina{
    cond c
    cond cPrioridad
    cond autoridad
    int cantPrioridad=0
    int cantEsperando=0
    bool libre=true
    
    procedure esperar(bool prioridad){
        cantEsperando++
        if(prioridad){
            cantPrioridad++
            signal(autoridad)
            wait(cPrioridad)
        }
        else{
            signal(autoridad)
            wait(c)
        }
    }
    
    procedure salir(){
        signal(autoridad)
        libre=true
    }

    procedure siguiente(){
        while(cantEsperando < 1 || !libre){
            wait(autoridad)
        }
        cantEsperando--
        if(cantPrioridad > 0){
            cantPrioridad--    
            signal(cPrioridad)
        }
        else{
            signal(c)
        }
        libre=false
    }
}

Process Persona[1..N]{
    bool prioridad=...
    Maquina.esperar(prioridad)
    Votar()
    Maquina.salir()
}
Process Autoridad{
    for i=1 to N{
        Maquina.siguiente()
    }
}
```
