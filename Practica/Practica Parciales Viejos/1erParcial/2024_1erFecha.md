## 1
```C
sem cajeros=0;
sem timer=0;
sem espera[1..N];
datosPago[1..N];
tupla respuesta[1..N]
cola c;
int disponibles=E
sem sem_disponibles=1
sem sem_cola=1

Process Timer{
    //espera hora de venta
    wait()
    //Se hace la hora de venta
    for i=1 to C{
        v(cajeros)
    }
}

Process Comprador[id:1..N]{
    datosPago=...
    p(sem_cola)
    cola.push(id)
    datosPago[id]=datosPago
    cantEsperando++
    v(sem_cola)
    v(cajeros)
    p(espera[id])
    resultado,comprobante=respuesta[id]
}

Process Cajero[1..C]{
    p(timer)
    while(true){
        p(cajeros)
        p(sem_cola)
        id = cola.pop()
        v(sem_cola)
        datos = datosPago[id]
        resultado = false
        comprobante = null
        p(sem_disponibles)
            if(disponibles > 0 ){
                compra = cobrar(datos)
                if(compra){
                    disponibles--
                    v(sem_disponibles)
                    resultado = true
                    comprobante = generarComprobante(compra)
                }
            }
        v(sem_disponibles)
        respuesta[id]= resultado,comprobante
    }
}
```
## 2
## A
```C
Process Persona[1..N]{
    Mirador.ingresar()
    //usa el mirador
    Mirador.salir()
}
Monitor Mirador{
    cond fila;
    bool libre = true;
    int cantEsperando=0;

    procedure ingresar(){
        if(libre){
            libre=false
        }
        else{
            cantEsperando++
            wait(fila)
        }
    }

    procedure salir(){
        if(cantEsperando>0){
            signal(fila)
            cantEsperando--
        }
        else{
            libre=true
        }
    }
}
```
## b
```C
Monitor Mirador{
    cond prioridad
    cond fila
    int cantEsperando=0
    bool libre = true
    int cantPrioridad=0
    procedure ingresar(edad){
        if(libre =true){
            libre=false
        }
        else{
            cantEsperando++
            if(edad>60){
                cantPrioridad++
                wait(prioridad)
            }
            else{
                wait(fila)
            }
        }
    }

    procedure salir(){
        if(cantEsperando>0){
            cantEsperando--
            if(cantPrioridad > 0){
                cantPrioridad--
                signal(prioridad)
            }
            else{
                signal(fila)
            }
        }
        else{
            libre=true
        }
    }
}
```

