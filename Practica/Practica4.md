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
___
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
---
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
---
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
---
### 5
Resolver la administración de 3 impresoras de una oficina. Las impresoras son usadas por N administrativos, los cuales están continuamente trabajando y cada tanto envían documentos a imprimir. Cada impresora, cuando está libre, toma un documento y lo imprime, de acuerdo con el orden de llegada.

a) Implemente una solución para el problema descrito.

b) Modifique la solución implementada para que considere la presencia de un director de oficina que también usa las impresas, el cual tiene prioridad sobre los administrativos.

c) Modifique la solución (a) considerando que cada administrativo imprime 10 trabajos y que todos los procesos deben terminar su ejecución.

d) Modifique la solución (b) considerando que tanto el director como cada administrativo imprimen 10 trabajos y que todos los procesos deben terminar su ejecución.

e) Si la solución al ítem d) implica realizar Busy Waiting, modifíquela para evitarlo.
Nota: ni los administrativos ni el director deben esperar a que se imprima el documento.
## A
```C
chan pendientes(string,int)
chan impresiones[1..N](string)
Process Administrativo[id:1..N]{
    string archivo
    string impresion
    while true{
        if (){
            pendientes.send(archivo,id)
        }
        (!empty(impresiones[id])){
            impresiones[id].receive(impresion)
        }
        (){
            //Trabaja
        }
    }
}
Process Impresora[1..3]{
    string archivo
    string impresion
    int id
    while true {
        pendientes.receive(archivo,id)
        //imprime el archivo
        impresiones[id].send(impresion)
    }
}
```
## B
```C
chan pendientes(string,int)
chan impresiones[1..N](string)
chan impresionDirector(string)
chan pendientesDirector(string)

Process Administrativo[id:1..N]{
    string archivo
    string impresion
    while true{
        if (){
            pendientes.send(archivo,id)
        }
        (!empty(impresiones[id])){
            impresiones[id].receive(impresion)
        }
        (){
            //Trabaja
        }
        end if
    }
}

Proces Director{
    string archivo
    string impresion
    while true{
        if (){
            pendientesDirector.send(archivo)
        }
        (!empty(impresionesDirector)){
            impresionesDirector.receive(impresion)
        }
        (){
            //Trabaja
        }
        end if
    }
}

Process Impresora[1..3]{
    string archivo
    string impresion
    int id
    while true {
        if(empty(pendientesDirector) && !empty(pendientes)){
            pendientes.receive(archivo,id)
            //imprime el archivo
            impresiones[id].send(impresion)
        }
        (!empty(pendientesDirector)){
            pendientesDirector.receive(archivo)
            //imprime el archivo
            impresionesDirector.send(impresion)
        }
    }
}
```
### C
```C
chan pendientes(string,int)
chan impresiones[1..N](string)
chan aviso[1..3](int)

Process Administrativo[id:1..N]{
    string archivo
    string impresion
    int cant=0
    termino=false
    while !termino{
        if(cant < 10){
            pendientes.send(archivo,id)
            cant++
        }
        (!empty(impresiones[id]) ){
            impresiones[id].receive(impresion)
            if(cant==10){
                for i=1 to 3{
                    aviso[i].send(1)
                }
            }
        }
        (){
            //Trabaja
        }
    }
}
Process Impresora[id:1..3]{
    string archivo
    string impresion
    int i
    int terminaron=0
    while terminaron < N {
        if(!empty(pendientes)){
            pendientes.receive(archivo,i)
            //imprime el archivo
            impresiones[i].send(impresion)
        }
        (!aviso[id].empty()){
            aviso.receive(i)
            terminaron++
        }
    }
}
```
## D y E (sin busy waiting)
```C
chan pedidos(string,int)
chan pedidosDirector(string,int)
chan impresiones[1..N+1](string)
chan avisoImpresora()
chan terminarImpresora()
chan impresora(string,id)
chan impresoraDirector(string,id)
chan avisoAdmin()

Process Administrador{
    int cant=0
    string pedido
    int id
    while cant < (N+1)*10{
        avisoAdmin.receive()
        if (!empty(pedidos) && empty(pedidosDirector)){
            pedidos.receive(pedido,id)
            impresora.send(pedido,id)
        }
        (!empty(pedidosDirector)){
            pedidosDirector.receive(pedido,id)
            impresoraDirector.send(pedido,id)
        }
        avisoImpresora.send()
        cant++
    }
    for i=1 to 3{
        terminarImpresora.send()
        avisoImpresora.send()
    }
}

Process Administrativo[id:1..N]{
    string impresion
    string pedido
    termino=false
    cant=0
    while(!termino){
        if (!empty(impresiones[id])){
            impresiones[id].receive(impresion)
            if(cant==10){
                termino=true
            }
        }
        (cant < 10){
            pedidos.send(pedido,id)
            avisoAdmin.send()
            cant++
        }
        (){
            //trabaja
        }
        end if
    }
}
Process Director{
    string impresion
    string pedido
    termino=false
    cant=0
    id=N+1
    while(!termino){
        if (!empty(impresiones[id])){
            impresiones[id].receive(impresion)
            if(cant==10){
                termino=true
            }
        }
        (cant < 10){
            pedidosDirector.send(pedido,id)
            avisoAdmin.send()
            cant++
        }
        (){
            //trabaja
        }
        end if
    }
}


Process impresora[1..3]{
    string impresion
    int id
    strig pedido
    termino=false
    while !termino(){
        avisoImpresora.receive()
        if (!empty(terminarImpresora)){
            terminarImpresora.receive()
            termino=true
        }
        (!empty(impresora) && empty(impresoraDirector)){
            impresora.receive(pedido,id)
            //Imprime
            impresiones[id].send(impresion)
        }
        (!empty(impresoraDirector)){
            impresoraDirector.receive(pedido,id)
            //Imprime
            impresiones[id].send(impresion)
        }
        end if
    }
}
```
---
#### CONSIDERACIONES PARA RESOLVER LOS EJERCICIOS DE PASAJE DE MENSAJES SINCRÓNICO (PMS):

• Los canales son punto a punto y no deben declararse.

• No se puede usar la sentencia empty para saber si hay algún mensaje en un canal.

• Tanto el envío como la recepción de mensajes es bloqueante.

• Sintaxis de las sentencias de envío y recepción:
 ```
 Envío: nombreProcesoReceptor!port (datos a enviar)
 Recepción: nombreProcesoEmisor?port (datos a recibir)
 ```
El port (o etiqueta) puede no ir. Se utiliza para diferenciar los tipos de mensajes que se podrían comunicarse entre dos procesos.

• En la sentencia de comunicación de recepción se puede usar el comodín * si el origen es un proceso dentro de un arreglo de procesos. Ejemplo: Clientes[*]?port(datos).

• Sintaxis de la Comunicación guardada:
```
 Guarda: (condición booleana); sentencia de recepción → sentencia a realizar
 ```
 Si no se especifica la condición booleana se considera verdadera (la condición booleana sólo puede hacer referencia a variables locales al proceso).
 Cada guarda tiene tres posibles estados:

 • Elegible: la condición booleana es verdadera y la sentencia de comunicación se puede resolver inmediatamente.

 • No elegible: la condición booleana es falsa.

 • Bloqueada: la condición booleana es verdadera y la sentencia de comunicación no se puede resolver inmediatamente.

 Sólo se puede usar dentro de un if o un do guardado:
 El if funciona de la siguiente manera: de todas las guardas elegibles se selecciona una en forma no determinística, se realiza la sentencia de comunicación correspondiente, y luego las acciones asociadas a esa guarda. Si todas las guardas tienen el estado de no elegibles, se sale sin hacer nada. Si no hay ninguna guarda elegible, pero algunas están en estado bloqueado, se queda esperando en el if hasta que alguna se vuelva elegible.

El do funciona de la siguiente manera: sigue iterando de la misma manera que el if hasta que todas las guardas hasta que todas las guardas sean no elegibles.
___
## 1
Suponga que existe un antivirus distribuido que se compone de R procesos robots Examinadores y 1 proceso Analizador. Los procesos Examinadores están buscando continuamente posibles sitios web infectados; cada vez que encuentran uno avisan la dirección y luego continúan buscando. El proceso Analizador se encarga de hacer todas las pruebas necesarias con cada uno de los sitios encontrados por los robots para determinar si están o no infectados.

a) Analice el problema y defina qué procesos, recursos y comunicaciones serán necesarios/convenientes para resolverlo.

b) Implemente una solución con PMS sin tener en cuenta el orden de los pedidos.

c) Modifique el inciso (b) para que el Analizador resuelva los pedidos en el orden en que se hicieron.

### A y B
```C
Process Examinador[1..R]{
    string sitio
    bool infectado
    while(true){
        //Selecciona sitio a examinar
        //Examina el sitio
        if(infectado){
            analizador!(sitio)
        }
    }
}

Process Analizador{
    string sitio
    bool infectado
    cola sitiosInfectados;
    while true{
        Examinador[*]?(sitio)
        //Analiza el sitio
        if(infectado){
            sitiosInfectados.push(sitio)
        }
    }
}
```
### C
```C
Process Examinador[1..R]{
    string sitio
    bool infectado
    while(true){
        //Selecciona sitio a examinar
        //Examina el sitio
        if(infectado){
            admin!(sitio)
        }
    }
}

Process Analizador{
    string sitio
    bool infectado
    cola sitiosInfectados;
    while true{
        Admin!pedido()
        Admin?sitio(sitio)
        //Analiza el sitio
        if(infectado){
            sitiosInfectados.push(sitio)
        }
    }
}

Process Admin{
    cola pedidos;
    string sitio
    contador = 0
    do 
        □ (contador < R); Examinador[*]?(sitio) ->
            contador++ 
            cola.push(sitio)

        □ (!pedidos.isEmpty()); Analizador?pedido() ->
            sitio = cola.pop()
            Analizador!sitio(sitio)
    od
}
```
---
### 2
En un laboratorio de genética veterinaria hay 3 empleados. El primero de ellos continuamente prepara las muestras de ADN; cada vez que termina, se la envía al segundo empleado y vuelve a su trabajo. El segundo empleado toma cada muestra de ADN preparada, arma el set de análisis que se deben realizar con ella y espera el resultado para archivarlo. Por último, el tercer empleado se encarga de realizar el análisis y devolverle el resultado al segundo empleado. 

```C
Process Preparador{
    cola muestras;
    muestra
    do
        □ (!muestras.isEmpty()); Archivador!muestras(muestras.pop())

        □ true; -> //Prepara muestra
                    muestras.push(muestra)
    od
}

Process Archivador{
    muestras
    setAnalisis 
    string resultado
    do 
        □ (!muestra); Preparador?muestras(muestra) -> //Prepara el set de analisis
                                                     Analizador!analisis(muestra,setAnalisis)
                                                     muestra = null                                                   
        □ Analizador?resultados(resultado) --> //Archiva el resultado
    od
}

Process Analizador{
    setAnalisis
    muestra
    resultado
    do 
        □ Archivador?analisis(muestra,setAnalisis) -> //Realiza analisis
                                                        Archivador!resultados(resultado) 
    od
}
```
---
### 3
En un examen final hay N alumnos y P profesores. Cada alumno resuelve su examen, lo
entrega y espera a que alguno de los profesores lo corrija y le indique la nota. Los
profesores corrigen los exámenes respetando el orden en que los alumnos van entregando.
a) Considerando que P=1.
b) Considerando que P>1.
c) Ídem b) pero considerando que los alumnos no comienzan a realizar su examen hasta
que todos hayan llegado al aula.
Nota: maximizar la concurrencia; no generar demora innecesaria; todos los procesos deben
terminar su ejecución
___
### 4. 
En una exposición aeronáutica hay un simulador de vuelo (que debe ser usado con
exclusión mutua) y un empleado encargado de administrar su uso. Hay P personas que
esperan a que el empleado lo deje acceder al simulador, lo usa por un rato y se retira.
a) Implemente una solución donde el empleado sólo se ocupa de garantizar la exclusión
mutua (sin importar el orden).
b) Modifique la solución anterior para que el empleado los deje acceder según el orden de
su identificador (hasta que la persona i no lo haya usado, la persona i+1 debe esperar).
c) Modifique la solución a) para que el empleado considere el orden de llegada para dar
acceso al simulador.
Nota: cada persona usa sólo una vez el simulador.
___
### 5.
 En un estadio de fútbol hay una máquina expendedora de gaseosas que debe ser usada por
E Espectadores de acuerdo con el orden de llegada. Cuando el espectador accede a la
máquina en su turno usa la máquina y luego se retira para dejar al siguiente. Nota: cada
Espectador una sólo una vez la máquina.