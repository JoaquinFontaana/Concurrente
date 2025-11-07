CONSIDERACIONES PARA RESOLVER LOS EJERCICIOS: 
 
___1. NO SE PUEDE USAR VARIABLES COMPARTIDAS___

___2. Declaración de tareas___
• Especificación de tareas sin ENTRY’s (nadie le puede hacer llamados). 
``` Ada
TASK Nombre; 
TASK TYPE Nombre;   
```
- Especificación de tareas con ENTRY’s (le puede hacer llamados). Los entry’s  funcionan  de  manera  semejante  los  procedimientos:  solo  pueden  recibir  o  enviar información por  medio de los parámetros del entry. NO RETORNAN  VALORES COMO LAS FUNCIONES 
``` Ada
TASK [TYPE] Nombre IS 
  ENTRY e1; 
  ENTRY e2 (p1: IN integer; p2: OUT char; p3: IN OUT float); 
END Nombre;    
```
- Cuerpo de las tareas. 
```Ada 
TASK BODY Nombre IS 
  Codigo que realiza la Tarea; 
END Nombre; 
 ``` 
___3. Sincronización y comunicación entre tareas___
• Entry call para enviar información (o avisar algún evento). 
```Ada
NombreTarea.NombreEntry (parametros);    
```
• Accept  para  atender  un  pedido  de  entry  call  sin  cuerpo  (sólo  para  recibir  el  aviso de un evento para sincronización). Lo usual es que no incluya  parámetros,  aunque  podría  tenerlos.  En  ese  caso,  son  ignorados  por  no  tener cuerpo presente. 
``` Ada
ACCEPT NombreEntry; 
ACCEPT NombreEntry (p1: IN integer; p3: IN OUT float);  
```
• Accept para atender un pedido de entry call con cuerpo. 
```Ada
ACCEPT NombreEntry (p1: IN integer; p3: IN OUT float) do 

Cuerpo del accept donde se puede acceder a los parámetros p1 y p3. 
Fuera del entry estos parámetros no se pueden usar. 

END NombreEntry;  
```
• El accept se puede hacer en el cuerpo de la tarea que ha declarado el entry en  su  especificación.  Los  entry  call  se  pueden  hacer  en  cualquier  tarea  o  en  el  programa principal.   
• Tanto  el  entry  call  como  el  accept  son  bloqueantes,  ambas  tareas  continúan  trabajando cuando el cuerpo del accept ha terminado su ejecución. 
 
___4. Select para ENTRY CALL.___
• Select ...OR DELAY: espera a lo sumo x tiempo a que la tarea correspondiente haga el  accept del entry call realizado. Si pasó el tiempo entonces realiza el código  opcional.
```Ada
SELECT  
 NombreTarea.NombreEntry(Parametros); 
 Sentencias; 
OR DELAY x 
 Código opcional; 
END SELECT;   
```
• Select ...ELSE: si la tarea correspondiente no puede realizar el accept inmediatamente  (en  el  momento  que  el  procesador  está  ejecutando  esa  línea  de  código)  entonces  se  ejecuta el código opcional. 
```Ada
SELECT  
 NombreTarea.NombreEntry(Parametros); 
 Sentencias; 
ELSE 
 Código opcional; 
END SELECT; 
```  
- En los select para entry call sólo puede ponerse un entry call y una única opción (OR  DELAY o ELSE);   

___5. Select para ACCEPT.___
- En  los  select  para  los  accept  puede  haber  más  de  una  alternativa  de  accept,  pero  no  puede  haber  alternativas  de  entry  call  (no  se  puede mezclar  accept  con  entries).  Cada  alternativa de ACCEPT puede ser o no condicional (uso de cláusula WHEN). 
```Ada 
SELECT  
  ACCEPT e1 (parámetros); 
  Sentencias1; 
  END e1; 
  OR    
  ACCEPT e2 (parámetros) IS cuerpo; 
  END e2; 
  OR  
  WHEN (condición) => ACCEPT e3 (parámetros) IS cuerpo; 
    Sentencias3 
  END e3; 
            
END SELECT;

```
Funcionamiento:  Se  evalúa  la  condición  booleana  del  WHEN  de  cada  alternativa  (si  no lo tiene se considera TRUE). Si todas son FALSAS se sale del select. En otro caso,  de las alternativas cuya condición es verdadera se elige en forma no determinística una  que  pueda  ejecutarse  inmediatamente  (es  decir  que  tiene  un  entry  call  pendiente).  Si  ninguna de ellas se puede ejecutar inmediatamente el select se bloquea hasta que haya  un entry call para alguna alternativa cuya condición sea TRUE.  
- Se puede poner una opción OR DELAY o ELSE (no las dos a la vez).  
- Dentro de la condición booleana de una alternativa (en el WHEN) se puede preguntar  por la cantidad de entry call pendientes de cualquier entry de la tarea. NombreEntry’count  
- Después de escribir una condición por medio de un WHEN siempre se debe escribir un accept. 
---
### 1
Se requiere modelar un puente de un único sentido que soporta hasta 5 unidades de peso.  El peso de los vehículos depende del tipo: cada auto pesa 1 unidad, cada camioneta pesa 2  unidades y  cada  camión  3  unidades.  Suponga  que  hay  una  cantidad  innumerable  de vehículos  (A  autos,  B  camionetas  y  C  camiones).  Analice  el  problema  y  defina  qué  tareas, recursos y sincronizaciones serán necesarios/convenientes para resolverlo. 

___a. Realice la solución suponiendo que todos los vehículos tienen la misma prioridad.___

___b. Modifique la solución para que tengan mayor prioridad los camiones que el resto de los vehículos.___
### A
```Ada 
Respetando orden de llegada

Task TYPE Puente IS
  ENTRY esperar();
  ENTRY ingresarAuto()
  ENTRY ingresarCamioneta()
  ENTRY ingresarCamion()
  ENTRY salir(peso: IN int);
End Puente;

Task BODY Puente IS
int capacidad = 5
bool esperando = false
Loop
  Select 
    WHEN(!esperando) => ACCEPT esperar() IS
      esperando = true;
    End esperar;
    OR
    ACCEPT salir(peso) IS 
      capacidad += peso;
    END salir;
    OR
    WHEN(capacidad >= 1) ACCEPT ingresarAuto() IS
      capacidad -= 1;
      esperando = false;
    End ingresarAuto;
    OR
    WHEN (capacidad >= 2) ACCEPT ingresarCamioneta() IS 
      capacidad -= 2
      esperando = false;
    END ingresarCamioneta;
    OR
    WHEN(capacidad >= 3) ACCEPT ingresarCamion() IS 
      capacidad -= 3
      esperando = false;
    END ingresarCamion; 
  END SELECT;
End Loop;
END Puente;

TASK TYPE Auto;
END Auto;

autos: array(1..A) of Auto;

TASK BODY Auto IS
  int peso = 1
  Puente.esperar()
  Puente.ingresarAuto()
  //Pasa por el puente
  Puente.salir(peso)
END Auto;

Task TYPE Camioneta;
camionetas:(1..B) of Camioneta;
END Camioneta;

TASK BODY Camioneta IS
  int peso= 2
  Puente.esperar();
  Puente.ingresarCamioneta()
  //Pasa por el puente
  Puente.salir(peso)
END Camioneta;

Task Type Camion;
END Camion;

camiones: array(1..C) of Camion;

TASK BODY Camion IS
  int peso= 3
  Puente.esperar()
  Puente.pasarCamion()
  //Pasa por el puente
  Puente.salir(peso)
END CAMION
```

### B

```Ada
Task TYPE Puente IS
  ENTRY ingresarAuto()
  ENTRY ingresarCamioneta()
  ENTRY ingresarCamion()
  ENTRY salir(peso: IN int);
End Puente;

Task BODY Puente IS
int capacidad = 5
Loop
  Select 
    ACCEPT salir(peso) IS 
      capacidad += peso;
    END salir;
    OR
    WHEN(pasarCamion'count < 1 && capacidad >= 1) ACCEPT ingresarAuto() IS
      capacidad -= 1;
      esperando = false;
    End ingresarAuto;
    OR
    WHEN (pasarCamion'count < 1 && capacidad >= 2) ACCEPT ingresarCamioneta() IS 
      capacidad -= 2
      esperando = false;
    END ingresarCamioneta;
    OR
    WHEN(capacidad >= 3) ACCEPT ingresarCamion() IS 
      capacidad -= 3
      esperando = false;
    END ingresarCamion; 
  END SELECT;
End Loop;
END Puente;

TASK TYPE Auto;
END Auto;

autos: array(1..A) of Auto;

TASK BODY Auto IS
  int peso = 1
  Puente.ingresarAuto()
  //Pasa por el puente
  Puente.salir(peso)
END Auto;

Task TYPE Camioneta;
camionetas:(1..B) of Camioneta;
END Camioneta;

TASK BODY Camioneta IS
  int peso= 2
  Puente.ingresarCamioneta()
  //Pasa por el puente
  Puente.salir(peso)
END Camioneta;

Task Type Camion;
END Camion;

camiones: array(1..C) of Camion;

TASK BODY Camion IS
  int peso= 3
  Puente.pasarCamion()
  //Pasa por el puente
  Puente.salir(peso)
END CAMION
```
---
### 2
Se quiere modelar el funcionamiento de un banco, al cual llegan clientes que deben realizar un pago y retirar un comprobante. Existe un único empleado en el banco, el cual atiende de acuerdo con el orden de llegada.  

___a) Implemente una solución donde los clientes llegan y se retiran sólo después de haber sido atendidos.___

___b) Implemente una solución donde los clientes se retiran si esperan más de 10 minutos para realizar el pago.___

___c) Implemente una solución donde los clientes se retiran si no son atendidos  inmediatamente.___

___d)  Implemente  una  solución  donde  los  clientes  esperan  a  lo  sumo  10  minutos para  ser atendidos. Si pasado ese lapso no fueron atendidos, entonces solicitan atención una vez más y se retiran si no son atendidos inmediatamente.___
### A
``` Ada
TASK TYPE Empleado IS
  ENTRY pagar(datosPago: IN String, comprobante: OUT String);
END Empleado;

TASK BODY Empleado IS
  Loop
    SELECT accept pagar(datosPago,comprobante) IS
      //procesa el pago
      comprobante = generarComprobante();
    END SELECT;
  END LOOP
END Empleado 

Task TYPE Cliente IS
END CLIENTE;

clientes: array(1..C) of Cliente;

Task BODY Cliente IS\
  string datosPago
  string comprobante
  Empleado.pagar(datosPago,comprobante)
```

### B
``` Ada
TASK TYPE Empleado IS
  ENTRY pagar(datosPago: IN String, comprobante: OUT String);
END Empleado;

TASK BODY Empleado IS
  Loop
    SELECT accept pagar(datosPago,comprobante) IS
      //procesa el pago
      comprobante = generarComprobante();
    END SELECT;
  END LOOP
END Empleado 

Task TYPE Cliente IS
END CLIENTE;

clientes: array(1..C) of Cliente;

Task BODY Cliente IS
  string datosPago
  string comprobante
  SELECT Empleado.pagar(datosPago,comprobante)
  OR DELAY 10
  END SELECT
END Cliente;  
```

### C
``` Ada
TASK TYPE Empleado IS
  ENTRY pagar(datosPago: IN String, comprobante: OUT String);
END Empleado;

TASK BODY Empleado IS
  Loop
    SELECT accept pagar(datosPago,comprobante) IS
      //procesa el pago
      comprobante = generarComprobante();
    END SELECT;
  END LOOP
END Empleado 

Task TYPE Cliente IS
END CLIENTE;

clientes: array(1..C) of Cliente;

Task BODY Cliente IS
  string datosPago
  string comprobante
  SELECT Empleado.pagar(datosPago,comprobante)
  END SELECT
END Cliente;  
```

### D
``` Ada
TASK TYPE Empleado IS
  ENTRY pagar(datosPago: IN String, comprobante: OUT String);
END Empleado;

TASK BODY Empleado IS
  Loop
    SELECT accept pagar(datosPago,comprobante) IS
      //procesa el pago
      comprobante = generarComprobante();
    ELSE
    END SELECT;
  END LOOP
END Empleado 

Task TYPE Cliente IS
END CLIENTE;

clientes: array(1..C) of Cliente;

Task BODY Cliente IS
  string datosPago
  string comprobante
  SELECT Empleado.pagar(datosPago,comprobante)
  OR DELAY 10
    SELECT Empleado.pagar(datosPago,comprobante)
    ELSE
    END SELECT
  END SELECT
END Cliente;  
```
---
### 3
Se  dispone  de  un  sistema  compuesto  por  1  central  y  2  procesos  periféricos,  que  se  comunican continuamente. Se requiere modelar su funcionamiento considerando las siguientes condiciones: 
- La  central  siempre  comienza  su  ejecución  tomando  una  señal  del  proceso  1;  luego  toma  aleatoriamente  señales  de  cualquiera  de  los  dos  indefinidamente.  Al  recibir  una  señal de proceso 2, recibe señales del mismo proceso durante 3 minutos. 
- Los  procesos  periféricos  envían  señales  continuamente  a  la  central.  La  señal  del  proceso  1  será  considerada  vieja  (se  deshecha)  si  en  2  minutos  no  fue  recibida.  Si  la señal del proceso 2 no puede ser recibida inmediatamente, entonces espera 1 minuto y  vuelve a mandarla (no se deshecha).
  
``` Ada
TASK TYPE Central IS
  ENTRY comunicacionUno(señal: IN string);
  ENTRY comunicacionDos(señal: IN string);
END Central

TASK BODY Central IS
  string señal
  ACCEPT comunacionUno(señal)
  LOOP
    SELECT 
      ACCEPT comunacionUno(señal) IS
      END comunicacionUno;
      OR
      ACCEPT comunicacionDos(señal) IS
        //No se como hacer la iteracion por 3 minutos
      END comunicacionDos;
    END SELECT
  END LOOP
END Central;

TASK TYPE Proceso1 IS
END Proceso1;

TASK BODY Proceso1 IS
  string señal
  LOOP 
    //genera señal
    SELECT Central.comunicacionUno(señal)
    OR DELAY 2
    END SELECT;
  END LOOP;
END Proceso1;

TASK TYPE Proceso2 IS
END Proceso2;

TASK BODY Proceso2 IS
  string señal;
  LOOP
    //genera señal
    SELECT Central.comunicacionDos(señal)
    OR DELAY 1
      central.comunicacionDos(señal)
    END SELECT
  END LOOP
END Proceso2;
```
---
### 4
En  una  clínica  existe  un  médico  de  guardia  que  recibe  continuamente  peticiones  de  atención de las E  enfermeras que trabajan en su piso y de las  P  personas que llegan a la  clínica ser atendidos.  
Cuando una persona necesita que la atiendan espera a lo sumo 5 minutos a que el médico lo  haga, si pasado ese tiempo no lo hace, espera 10 minutos y vuelve a requerir la atención del médico. Si no es atendida tres veces, se enoja y se retira de la clínica. 
Cuando una enfermera requiere la atención del médico, si este no lo atiende inmediatamente le  hace  una  nota  y  se  la  deja  en  el  consultorio  para  que  esta  resuelva  su  pedido  en  el momento  que  pueda  (el  pedido  puede  ser  que  el  médico  le  firme  algún  papel). Cuando  la petición  ha  sido  recibida  por  el  médico  o  la  nota  ha  sido  dejada  en  el  escritorio,  continúa trabajando y haciendo más peticiones.
El médico atiende los pedidos dándole prioridad a los enfermos que llegan para ser atendidos. Cuando atiende un pedido, recibe la solicitud y la procesa durante un cierto tiempo. Cuando está libre aprovecha a procesar las notas dejadas por las enfermeras. 
 
```Ada
TASK TYPE Medico IS
  ENTRY atencion();
  ENTRY peticion(pedido: IN String);
  ENTRY leerNota(pedido: IN String);
END Medico

TASK BODY MEDICO IS
  LOOP
    SELECT 
      ACCEPT atencion() DO
        //Atiende al paciente
      END atencion
      OR
      WHEN(atencion'count=0) => ACCEPT peticion(pedido) DO
        //Resuelve el pedido
      END peticion
      OR 
      ELSE 
        SELECT Nota.leerNota(pedido)
          //Lee el pedido y lo resuelve
        OR ELSE
        END SELECT
    END SELECT
  END LOOP
END MEDICO

TASK TYPE Enfermera IS
END Enfermera
enfermeras: array(1..E) of Enfermera;

TASK BODY Enfermera IS 
  string pedido;
  LOOP
    //trabajando
    SELECT Medico.peticion(pedido)
    OR ELSE 
      Nota.dejarNota(pedido)
    END SELECT
  END LOOP
END Enfermera

TASK TYPE Persona IS
END Persona
personas: array(1..P) of Persona;

TASK BODY Persona IS
  bool atendido = false
  int contador = 3
  while(contador > 0 AND !atendido)
    SELECT Medico.atencion(); 
      atendido=true
    OR DELAY 5
      contador--
    END SELECT;
    DELAY 10;
END Persona

TASK TYPE Nota IS
  ENTRY dejarNota(pedido:IN String);
  ENTRY leerNota(pedido:OUT String)
END NOTAS;

TASK BODY Nota IS
  pedidos: cola(string)
  LOOP 
    SELECT 
      WHEN(leerNota'count=0 OR pedidos.isEmpty()) => ACCEPT dejarNota(pedido) IS
        pedidos.push(pedido)
      END dejarNota
      OR
      WHEN(!pedidos.isEmpty()) => ACCEPT leerNota(pedido) IS
        pedido = cola.pop()
      END leerNota
    END Select
  END LOOP
END Nota
```
---
### 5
En  un  sistema  para  acreditar  carreras  universitarias,  hay  UN  Servidor  que  atiende  pedidos de  U  Usuarios  de  a  uno  a  la  vez  y  de  acuerdo  con  el  orden  en  que  se  hacen  los  pedidos. Cada  usuario  trabaja  en  el  documento  a  presentar,  y  luego  lo  envía  al  servidor;  espera  la respuesta de este que le indica si está todo bien o hay algún error. Mientras haya algún error,  vuelve a trabajar con el documento y a enviarlo al servidor. Cuando el servidor le responde que está todo bien, el usuario se retira. Cuando un usuario envía un pedido espera a lo sumo minutos a que sea recibido por el servidor, pasado ese tiempo espera un minuto y vuelve a  intentarlo (usando el mismo documento).
```Ada
TASK TYPE Servidor IS
  ENTRY comprobarDocumento(documento: IN string,resultado: OUT bool)
END Servidor

TASK BODY Servidor IS
  bool res
  LOOP
    ACCEPT comprobarDocumento(documento,resultado) DO
      //comprueba el documento
      resultado = res;
    END comprobarDocumento
  END LOOP
END Servidor

TASK TYPE Usuario IS
END Usuario
usuarios: array(1..U) of Usuario

TASK BODY Usuario IS
  resultado = false
  string documento
  bool recibido
  while(!resultado)
    //trabaja en  el documento
    recibido = false
    while(!recibido)
      SELECT Servidor.comprobarDocumento(documento,resultado)
        recibido=true
      OR DELAY 3
        DELAY 1
      END SELECT
    END LOOP
  END LOOP
END Usuario
```
### 6
 En una playa hay 5 equipos de 4 personas cada uno (en total son 20 personas donde cada una  conoce  previamente  a  que  equipo  pertenece).  Cuando  las  personas  van  llegando  esperan  con  los  de  su  equipo  hasta  que  el  mismo  esté  completo  (hayan  llegado  los  4 integrantes), a partir de ese momento el equipo comienza a jugar. El juego consiste en que cada integrante del grupo junta 15 monedas de a una en una playa (las monedas pueden ser de  1,  2  o  5  pesos)  y  se  suman  los  montos  de  las  60  monedas  conseguidas  en  el  grupo.  Al finalizar  cada  persona  debe  conocer  el  grupo  que  más  dinero  junto.  Nota:  maximizar  la concurrencia.  Suponga  que  para  simular  la  búsqueda  de  una  moneda  por  parte  de  una persona existe una función Moneda() que retorna el valor de la moneda encontrada. 
 ```Ada 
TASK TYPE Persona IS
  ENTRY comenzar();
  ENTRY equipoGanador(equipoMax: IN integer);
END PERSONA

TYPE Equipo IS array(1..4) of Persona;
equipos: array(1..5) of Equipo;

TASK BODY Persona IS
  int equipo
  int cantMonedas = 0 
  int moneda
  ADMIN.llegoPersona(equipo)
  ACCEPT comenzar();
  while(cantMonedas < 15)
    //busca moneda
    moneda = Moneda()
    ADMIN.sumarMoneda(equipo,moneda)
    cantMonedas+=1
  END LOOP
  ACCEPT equipoGanador(equipoMax)
END Persona

TASK TYPE ADMIN IS
  ENTRY llegoPersona(equipo: IN Integer);
  ENTRY sumarMoneda(equipo: IN Integer,moneda: IN Integer)
END ADMIN

TASK BODY ADMIN IS
  total: array(1..5) of Integer;
  contadorPersonas: array(1..5) of Integer;
  totalMonedas: array(1..5) of Integer;
  integer cantMonedas = 0;
  While(cantMonedas < 5*60) 
    SELECT
      ACCEPT llegoPersona(equipo) DO
        contadorPersonas(equipo)++
        IF (contadorPersonas(equipo) = 4) 
          FOR i IN 1..4 LOOP
            equipos(equipo)(i).comenzar()
          END LOOP
        END IF
      END llegoPersona
      OR
      ACCEPT sumarMoneda(equipo,moneda) DO
        cantMonedas++
        totalMonedas(equipo) += moneda
      END sumarMoneda
    END SELECT
  END LOOP
  integer equipoMax= totalMonedas.max() //funcion que devuelve el indice con el valor maximo del array
  FOR i IN 1..5 LOOP
    FOR j IN 1..4 LOOP
      equipos(i)(j).equipoGanador(equipoMax)
    END LOOP
  END LOOP
END ADMIN
 ```