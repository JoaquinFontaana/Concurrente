## 1
Resolver con SEMÁFOROS el siguiente problema. La Clave Única de Identificación Tributaria (CUIT) es una clave que se utiliza en el sistema tributario de la República Argentina para poder identificar correctamente a las personas físicas o jurídicas. Consta de un total de once (11) cifras numéricas, siendo la última un dígito verificador (del 0 al 9). Una empresa cuenta con una lista de CUITs que debe procesar, debiendo informar la cantidad de CUITs por dígito verificador. Para ello, dispone de un software que emplea 5 workers, los cuales trabajan colaborativamente procesando de a una CUIT por vez cada uno. Al finalizar el procesamiento, el último worker en terminar debe informar los resultados del procesamiento. Notas: la función obtenerDV(CUIT) retorna el dígito verificador para la CUIT recibida como entrada. La lista de CUITs se almacena como una cola global y la solución debe maximizar la concurrencia

```C
cantTotal[0..9]=0
terminaron=0
cola cuits
sem sem_cuits=1
sem sem_total=1

Process Worker[1..5]{
   cant[0..9]=0
   p(sem_cuits)
   while(!cuits.isEmpty()){
        cuit = cuits.pop()
        v(sem_cuits)
        digito = obtenerDV(cuit)
        cant[digito]++
        p(sem_cuits)
   }
   v(sem_cuits)
   p(sem_total)
   terminaron++
   for i=0 to 9{
        cantTotal[i]+=cant[i]
   }
   if(terminaron == 5){
        for i=0 to 9{
            print(cantTotal[i])
        }
   }
   v(sem_total)
}
```
## 2
Resolver con MONITORES la siguiente situación. En un negocio hay UN empleado que diseña tarjetas digitales. El empleado debe atender los pedidos de C clientes, de acuerdo con el orden en que se hacen los pedidos. El cliente envía las indicaciones, y el empleado en base a eso diseña la tarjeta y se la envía al cliente. Notas: maximizar la concurrencia; existe una función HacerTarjeta(indicaciones) que simula el armado de la tarjeta por parte del empleado; todos los procesos deben terminar su ejecución.
```C
Monitor Pedidos{
    int contador=0
    cond empleado;
    cola pedidos;
    indicaciones[1..C]
    cond espera[1..C]
    tarjetas[1..C]

    procedure pedirTarjeta(indicaciones,id){
        indicaciones[id]=indicaciones
        pedidos.push(id)
        signal(empleado)
        wait(espera[id])
    }

    procedure tomarPedido(id:out,indicaciones:out){
        if(pedidos.isEmpty()){
            wait(empleado)
        }
        contador++
        id = pedidos.pop()
        indicaciones = indicaciones[id]
    }

    procedure enviarTarjeta(id,tarjeta){
        signal(espera[id])
        tarjetas[id] = tarjeta
    }

    procedure quedanClientes(ok:out){
        ok = contador < C
    }

    procedure recibirTarjeta(id,tarjeta:out){
        tarjeta = tarjetas[id]
    }
}
Process Empleado{
    bool ok
    indicaciones
    tarjeta
    int id
    Pedidos.quedanClientes(ok)
    while(ok){
        Pedidos.tomarPedido(id,indicaciones)
        tarjeta = HacerTarjeta(indicaciones)
        Pedidos.enviarTarjeta(id,tarjeta)
        Pedidos.quedanClientes(ok)
    }
}

Process Cliente[id:1..C]{
    tarjeta
    indicaciones
    Pedidos.pedirTarjeta(indicaciones,id)
    Pedidos.recibirTarjeta(id,tarjeta)
}

```