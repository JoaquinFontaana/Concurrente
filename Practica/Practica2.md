# Practica 2

---
## 10
A una cerealera van T camiones a descargarse trigo y M camiones a descargar maíz. Sólo hay lugar para que 7 camiones a la vez descarguen, pero no pueden ser más de 5 del mismo tipo de cereal.

a) Implemente una solución que use un proceso extra que actúe como coordinador entre los camiones. El coordinador debe atender a los camiones según el orden de llegada. Además, debe retirarse cuando todos los camiones han descargado.

b) Implemente una solución que no use procesos adicionales (sólo camiones). No importa el orden de llegada para descargar. Nota: maximice la concurrencia.

### A 
```C
cola llegada;
sem mutex=1;
sem esperaTrigo[1..T]=0
sem esperaMaiz[1..M]=0
sem cordinador=0:
sem lugaresTrigo=5
sem lugaresMaiz=5
sem lugares=7

Process Trigo[ID:1..T]{
    p(mutex)
    llegada.push(id,"trigo")
    v(mutex)
    v(cordinador)
    p(esperaTrigo[id])
    "Descarga camion"
    v(lugares)
    v(lugaresTrigo)
}

Process Maiz[ID:1..M]{
    p(mutex)
    llegada.push(id,"maiz")
    v(mutex)
    v(cordinador)
    p(esperaMaiz[id])
    "Descarga camion"
    v(lugares)
    v(lugaresMaiz)

}

Process Cordinador{
    cant=M+T
    while(cant > 0){
        p(cordinador)
        id,tipo = llegada.pop()
        p(lugares)
        if(tipo == "maiz"){
            p(lugaresMaiz)
            v(esperaMaiz[id])
        }
        else{
            p(lugaresTrigo)
            v(esperaTrigo[id])
        }
        cant--
    }
}
```
### B
```C
sem lugares=7
sem lugaresTrigo=5
sem lugaresMaiz=5

Process Trigo[1..T]{
    p(lugaresTrigo)
    p(lugares)
    "Descarga camion"
    v(lugaresTrigo)
    v(lugares)
}

Process Maiz[1..M]{
    p(lugaresMaiz)
    p(lugares)
    "Descarga camion"
    v(lugaresMaiz)
    v(lugares)
}
```