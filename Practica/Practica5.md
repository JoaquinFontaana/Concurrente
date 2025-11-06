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
- Después de escribir una condición por medio de un WHEN siempre se debe escribir un 
accept. 
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
