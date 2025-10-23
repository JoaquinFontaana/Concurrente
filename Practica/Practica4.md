#### CONSIDERACIONES PARA RESOLVER LOS EJERCICIOS DE PASAJE DE MENSAJES ASINCRÓNICO (PMA):
• Los canales son compartidos por todos los procesos.

• Cada canal es una cola de mensajes; por lo tanto, el primer mensaje encolado es el primero en ser atendido.

• Por ser PMA, el send no bloquea al emisor.

• Se puede usar la sentencia empty para saber si hay algún mensaje en el canal, pero no se puede consultar por la cantidad de mensajes encolados.

• Se puede utilizar el if/do no determinístico donde cada opción es una condición boolena donde se puede preguntar por variables locales y/o por empty de canales.
```
    if (cond 1) -> Acciones 1;
    (cond 2) -> Acciones 2;
    ….
    (cond N) -> Acciones N;
    end if
```
 De todas las opciones cuya condición sea Verdadera elige una en forma no determinística y ejecuta las acciones correspondientes. Si ninguna es verdadera, sale del if/do sin ejecutar acción alguna.

• Se debe evitar hacer busy waiting siempre que sea posible (sólo hacerlo si no hay otra opción).

• En todos los ejercicios el tiempo debe representarse con la función delay
___
## 1
Suponga que N clientes llegan a la cola de un banco y que serán atendidos por sus empleados. Analice el problema y defina qué procesos, recursos y canales/comunicaciones serán necesarios/convenientes para resolverlo. Luego, resuelva considerando las siguientes situaciones:

a. Existe un único empleado, el cual atiende por orden de llegada.

b. Ídem a) pero considerando que hay 2 empleados para atender, ¿qué debe modificarse en la solución anterior?

c. Ídem b) pero considerando que, si no hay clientes para atender, los empleados realizan tareas administrativas durante 15 minutos. ¿Se puede resolver sin usar procesos adicionales? ¿Qué consecuencias implicaría?

## A
```c
chan consultas(string,int);
chan respuestas[1..N](string);

Process Cliente[id:1..N]{
    string consulta = ...
    string respuesta;
    consultas.send(consulta,id)
    respuestas[id].receive(respuesta)
}
Process Empleado{
    string consulta;
    int id;
    string respuesta;
    while true{
        consultas.receive(consulta,id);
        //Genera respuesta
        respuestas[id].send(respuesta)
    }
}
```
## B
```c
chan consultas(string,int);
chan respuestas[1..N](string);

Process Cliente[id:1..N]{
    string consulta = ...
    string respuesta;
    consultas.send(consulta,id)
    respuestas[id].receive(respuesta)
}
Process Empleado[1..2]{
    string consulta;
    int id;
    string respuesta;
    while true{
        consultas.receive(consulta,id);
        //Genera respuesta
        respuestas[id].send(respuesta)
    }
}
```
## C
```c
chan consultas(string,int);
chan respuestas[1..N](string);

Process Cliente[id:1..N]{
    string consulta = ...
    string respuesta;
    consultas.send(consulta,id)
    respuestas[id].receive(respuesta)
}
Process Empleado[1..2]{
    string consulta;
    int id;
    string respuesta;
    while true{
        if (!empty(consultas)){
            consultas.receive(consulta,id)
            //Genera respuesta
            respuestas[id].send(respuesta)
        }
        (empty(consultas)){
            //Trabajo administratiivo
            delay(15)
        }
        end if
    }
}
```
## 2
Se desea modelar el funcionamiento de un banco en el cual existen 5 cajas para realizar pagos. Existen P clientes que desean hacer un pago. Para esto, cada una selecciona la caja donde hay menos personas esperando; una vez seleccionada, espera a ser atendido. En cada caja, los clientes son atendidos por orden de llegada por los cajeros. Luego del pago, se les entrega un comprobante. Nota: maximizar la concurrencia.

```C
chan caja[1..5](string,int)
chan espera(int)
chan cajaAsignada[1..P](int)
chan respyesta[1..P](string)
chan avisoAtencion(int)

Process Administrador{
    int cant[1..5]=0
    int total=0
    int id
    int caja
    while total < P{
            if (avistoAtencion.isEmpty() AND !espera.isEmpty()){
                espera.receive(id)
                i = cant.getMinIndice()
                cajaAsignada[id].send(i)
                cant[i]++
                total++
            }
            (!avisotAtencion.isEmpty()){
                avistoAtencion.receive(caja)
                cant[caja]--
            }
            end if
        }
}

Process Caja[id=1..5]{
    int numeroCliente
    string datosPago
    string comprobante
    while true{
        caja[id].receive(datosPago,numeroCliente)
        avistoAtencion.send(id)
        //Procesa la transaccion
        respuesta[numeroCliente].send(comprobante)
    }
}

Process Cliente[id:1..P]{
    int numero_caja
    string consulta=..
    string datosPago=..
    string comprobante
    espera.send(id)
    cajaAsignada[id].receive(numero_caja)
    caja[numero_caja].send(datosPago,id)
    respuesta[id].receive(comprobante)
}
```

### 3
Se debe modelar el funcionamiento de una casa de comida rápida, en la cual trabajan 2 cocineros y 3 vendedores, y que debe atender a C clientes. El modelado debe considerar que:
- Cada cliente realiza un pedido y luego espera a que se lo entreguen.
- Los pedidos que hacen los clientes son tomados por cualquiera de los vendedores y se lo pasan a los cocineros para que realicen el plato. Cuando no hay pedidos para atender, los vendedores aprovechan para reponer un pack de bebidas de la heladera (tardan entre 1 y 3 minutos para hacer esto).
- Repetidamente cada cocinero toma un pedido pendiente dejado por los vendedores, lo cocina y se lo entrega directamente al cliente correspondiente.
Nota: maximizar la concurrencia
```C 
chan pedidos(string,int)
chan pedientesCocina(string,int)
chan entrega[1..C](string)

process Cliente[id:1..C]{
    string pedido=...
    string plato
    pedidos.send(pedido,id)
    entrega[id].receive(plato)
}
Process Vendedor[1..3]{
    string pedido
    int id
    while true{
        if (!empty(pedidos)){
            pedidos.receive(pedido,id)
            pendientesCocina.send(pedido,id)
        }
        (empty(pedidos)){
            if (){
                //Repone pack de bebidas
                delay(3)
            }
            (){
                //Repone pack de bebidas
                delay(1)
            }
            end if
        }
        end if
    }
}

Process Cocinero[1..2]{
    string pedido
    int id
    string plato
    while true{
        pendientesCocina.receive(pedido,id)
        //Cocina el pedido
        entrega[id].send(plato)
    }
}
```
### 4
Simular la atención en un locutorio con 10 cabinas telefónicas, el cual tiene un empleado que se encarga de atender a N clientes. Al llegar, cada cliente espera hasta que el empleado le indique a qué cabina ir, la usa y luego se dirige al empleado para pagarle. El empleado atiende a los clientes en el orden en que hacen los pedidos. A cada cliente se le entrega un ticket factura por la operación.
a) Implemente una solución para el problema descrito.
b) Modifique la solución implementada para que el empleado dé prioridad a los que terminaron de usar la cabina sobre los que están esperando para usarla.
Nota: maximizar la concurrencia; suponga que hay una función Cobrar() llamada por el empleado que simula que el empleado le cobra al cliente.

### A

```C
chan espera(int)
chan cabinaAsignada[1..C](int)
chan colaPago(string,int)
chan avisoSalida(int)
chan facturas[1..C](string)

Process Empleado{
    cola cabinasLibres(1..10)
    int id
    string datosPago
    string factura
    int cabina
    while true{
        if (!empty(espera) && !cabinaLibres.isEmpty()){
            espera.receive(id)
            cabina = cabinaAsignada.pop()
            cabinaAsignada[id].send(cabina)
        }
        (!empty(colaPago) || cabinaLibres.isEmpty()){
            colaPago.receive(datosPago,id,cabina)
            cabinasLibres.push(cabina)
            factura = Cobrar()
            facturas[id].send(factura)
        }
        end if
    }
}

Process Cliente[id:1..C]{
    int cabina
    string datosPago
    string factura
    espera.send(id)
    cabinaAsignada.receive(cabina)
    //Usa la cabina
    colaPago.send(datosPago,id,cabina)
    facturas[id].receive(factura)
}

```
### B
```C
chan espera(int)
chan cabinaAsignada[1..C](int)
chan colaPago(string,int,int)
chan facturas[1..C](string)

Process Empleado{
    cola cabinasLibres(1..10)
    int id
    string datosPago
    string factura
    int cabina
    while true{
        if (!empty(espera) && !cabinaLibres.isEmpty() && empty(colaPago)){
            espera.receive(id)
            cabina = cabinaAsignada.pop()
            cabinaAsignada[id].send(cabina)
        }
        (!empty(colaPago) || cabinaLibres.isEmpty()){
            colaPago.receive(datosPago,id,cabina)
            cabinasLibres.push(cabina)
            factura = Cobrar()
            facturas[id].send(factura)
        }
        end if
    }
}

Process Cliente[id:1..C]{
    int cabina
    string datosPago
    string factura
    espera.send(id)
    cabinaAsignada.receive(cabina)
    //Usa la cabina
    colaPago.send(datosPago,id,cabina)
    facturas[id].receive(factura)
}

```