## TP4

# Breve descripción de Azure DevOps Pipelines
Azure DevOps Pipelines es un servicio de integración continua (CI) y entrega continua (CD) que forma parte de la plataforma Azure DevOps de Microsoft. Permite automatizar la compilación, pruebas y despliegue de aplicaciones, facilitando la entrega de software de alta calidad de manera rápida y confiable.

## Principales características de Azure DevOps Pipelines:
- Multiplataforma: Soporta aplicaciones para distintas plataformas como Windows, Linux y macOS, permitiendo compilar y desplegar en cualquier entorno.
- Flexibilidad en el lenguaje: Compatible con múltiples lenguajes de programación como .NET, Java, Node.js, Python, PHP, y muchos más.
- Integración con repositorios: Funciona con diversos sistemas de control de versiones, como Git (incluyendo GitHub y Azure Repos), Subversion y TFVC.
- Pipeline como código (YAML): Los pipelines pueden ser definidos mediante archivos YAML, lo que permite versionar y compartir las definiciones de los pipelines junto con el código fuente del proyecto.
- Escalabilidad y personalización: Los pipelines pueden ejecutarse en agentes alojados por Microsoft o en agentes self-hosted (auto-alojados), lo que permite personalizar el entorno de ejecución según las necesidades del proyecto.
- Automatización de todo el ciclo de vida: Desde la compilación del código, pasando por la ejecución de pruebas, hasta el despliegue en entornos de desarrollo, pruebas o producción.
- Azure DevOps Pipelines se integra de manera fluida con otras herramientas de la suite de Azure DevOps, como Boards, Repos, y Test Plans, lo que permite a los equipos tener un flujo de trabajo centralizado y eficiente para gestionar todo el ciclo de vida del desarrollo de software.



# Tipos de Pipelines: Build y Deploy
## 1. Build Pipeline (Pipeline de compilación)
El Build Pipeline es responsable de la compilación del código fuente de la aplicación y la generación de artefactos listos para ser desplegados. Durante este proceso, el código es compilado, se ejecutan pruebas unitarias y se producen artefactos que pueden ser reutilizados en las etapas posteriores de entrega o despliegue. Es el primer paso en un flujo de integración continua (CI).

### Tareas comunes en un Build Pipeline:
- Restaurar dependencias (por ejemplo, usando npm, NuGet, pip, etc.).
- Compilar el código fuente.
- Ejecutar pruebas unitarias o de integración.
- Publicar los artefactos generados para ser usados en otros pipelines o entornos.
## 2. Deploy Pipeline (Pipeline de despliegue)
El Deploy Pipeline toma los artefactos generados en el pipeline de compilación y los despliega en uno o varios entornos, como desarrollo, prueba o producción. Este pipeline permite automatizar la entrega continua (CD), facilitando el despliegue de versiones del software de manera rápida, repetible y segura.

### Tareas comunes en un Deploy Pipeline:
- Descargar o recibir artefactos desde un Build Pipeline.
- Desplegar los artefactos en un servidor de pruebas o producción.
- Ejecutar scripts de migración de bases de datos.
- Verificar el despliegue mediante pruebas funcionales o de aceptación.

### Diferencia clave:
- Build Pipeline: Se enfoca en compilar y probar el código para generar artefactos.
- Deploy Pipeline: Toma esos artefactos y los despliega en uno o más entornos.

# Diferencias entre Editor Clásico y YAML
## Editor Clásico:
- Interfaz gráfica: El editor clásico ofrece una interfaz gráfica de arrastrar y soltar, donde los usuarios pueden añadir tareas al pipeline de manera visual, sin necesidad de escribir código.
- Configuración sencilla: Es ideal para usuarios nuevos o aquellos que prefieren configurar sus pipelines sin manejar archivos de configuración de texto.
- Menor flexibilidad: Aunque permite una fácil configuración, es menos flexible y poderoso que YAML. No es ideal para flujos de trabajo complejos ni para versionar la definición del pipeline junto con el código fuente.

## YAML:
- Pipeline como código: El editor YAML permite definir los pipelines mediante archivos de texto YAML, lo que permite incluir el pipeline como parte del repositorio de código, facilitando la versionabilidad y la colaboración en equipo.
- Flexibilidad y portabilidad: YAML es más flexible, permitiendo personalizar pipelines complejos, definir condiciones, utilizar plantillas y aprovechar funcionalidades avanzadas como matrices de jobs y dependencias entre tareas.
- Mejor para DevOps: Como la definición es código, los cambios en el pipeline pueden gestionarse mediante control de versiones y pull requests, ofreciendo mejor rastreo y control.

# Agentes Microsoft-hosted y Self-hosted
## Agentes Microsoft-hosted:
Descripción: Son agentes administrados por Microsoft, que están alojados en la nube. No requieren configuración por parte del usuario, y vienen preconfigurados con las herramientas más comunes para la compilación y despliegue.
### Ventajas:
- No requieren mantenimiento.
- Vienen preconfigurados con muchas herramientas populares.
- Escalabilidad automática para manejar cargas de trabajo adicionales.
### Desventajas:
- Limitaciones en cuanto al tiempo de ejecución (tiempo de ejecución limitado por trabajo).
- No se pueden personalizar los entornos más allá de las configuraciones disponibles.

Cuándo usar: Ideal para proyectos pequeños o medianos, donde no se requiere control total del entorno o cuando se necesita un entorno listo para usar.

## Agentes Self-hosted:
Descripción: Son agentes que configuras y gestionas tú mismo, ya sea en tus propios servidores o en máquinas virtuales. Ofrecen total control sobre el entorno de compilación, lo que permite personalizarlo según las necesidades del proyecto.
### Ventajas:
- Control total sobre el entorno, lo que permite incluir herramientas, librerías o configuraciones específicas.
- No hay limitaciones en el tiempo de ejecución, lo que lo hace adecuado para tareas largas o complejas.
### Desventajas:
- Requieren mantenimiento y monitoreo.
- Mayor responsabilidad de gestión, como actualizaciones, seguridad y recursos del servidor.

Cuándo usar: Adecuado cuando necesitas un entorno de compilación personalizado o si tienes cargas de trabajo largas o específicas que no se adaptan a los agentes Microsoft-hosted.

2 - Pasos del TP
2.1 Verificar acceso a Pipelines concedido


![image](https://github.com/user-attachments/assets/f3657c0f-386b-40f4-84ca-3bb45d2ffeb1)

![image](https://github.com/user-attachments/assets/69bc66b1-746f-41c0-a155-408c5b9cf457)

![image](https://github.com/user-attachments/assets/ef4dfcea-47ef-40c6-87d3-21f9c8843429)


<img width="1511" alt="image" src="https://github.com/user-attachments/assets/f686b40c-f714-4eb4-a894-263f1b9ae910">




## 4.3 Explicación: ¿Por qué es necesario contar con una tarea de Publish en un pipeline que corre en un agente de Microsoft en la nube?
Cuando utilizas un agente Microsoft-hosted en Azure DevOps, el entorno en el que se ejecuta el pipeline es temporal. Una vez que el pipeline finaliza, todos los archivos y datos que se generaron durante la ejecución del pipeline son eliminados. Esto significa que si no publicas los artefactos o resultados generados durante la ejecución, no tendrás acceso a ellos después de que el pipeline termine.

La tarea de Publicar artefactos (Publish Pipeline Artifacts) se encarga de almacenar los resultados (artefactos) de la compilación en una ubicación permanente dentro de Azure DevOps, como el almacenamiento de artefactos de pipeline. De esta manera, puedes descargarlos, reutilizarlos en otros pipelines o desplegarlos en otros entornos, incluso después de que el agente de Microsoft haya terminado su trabajo y haya liberado sus recursos.

Punto 4: Descargar el resultado del pipeline y correr localmente el software compilado.
<img width="869" alt="image" src="https://github.com/user-attachments/assets/8d770496-6f11-436b-ae5b-0671a1bdbfe5">

<img width="643" alt="image" src="https://github.com/user-attachments/assets/99ec20ac-e196-4529-a454-e300809989d8">


<img width="1001" alt="image" src="https://github.com/user-attachments/assets/31bc6c01-aaf6-4cfa-8b61-b7f909818a06">

Punto 5: Habilitar el editor clásico de pipelines. Explicar las diferencias claves entre este tipo de editor y el editor YAML.
<img width="629" alt="image" src="https://github.com/user-attachments/assets/82fd2410-f3b1-4336-ae61-806073b645b5">


El editor clásico de Azure DevOps es una interfaz gráfica que facilita la creación de pipelines a través de formularios y opciones desplegables, lo que lo hace ideal para principiantes o usuarios que prefieren trabajar sin escribir código. Aunque es fácil de usar, tiene limitaciones en cuanto a flexibilidad y personalización avanzada. Además, la configuración de pipelines en el editor clásico no se integra automáticamente con el control de versiones, lo que puede dificultar la colaboración y el seguimiento de cambios en proyectos más grandes.
Por otro lado, el editor YAML permite definir pipelines como código, lo que ofrece una mayor flexibilidad y control sobre la configuración del pipeline. Al estar almacenado directamente en el repositorio, el archivo YAML se versiona junto con el código fuente, facilitando la colaboración y la trazabilidad de los cambios. Aunque tiene una curva de aprendizaje más pronunciada, especialmente para usuarios no familiarizados con YAML, el editor YAML es más adecuado para proyectos complejos que requieren una configuración más detallada y reutilizable. En resumen, el editor clásico es más accesible, mientras que YAML proporciona un control más robusto y es ideal para entornos de desarrollo más avanzados.


Punto 6: Crear un nuevo pipeline con el editor clásico. Descargar el resultado del pipeline y correr localmente el software compilado.
<img width="654" alt="image" src="https://github.com/user-attachments/assets/76c80e6a-c4f9-412b-bff2-58f3f429d70f">

<img width="647" alt="image" src="https://github.com/user-attachments/assets/98d10996-ba2e-4fcb-ae80-33fc26539b81">

<img width="639" alt="image" src="https://github.com/user-attachments/assets/83734b72-5cf8-4b85-be48-516902992c27">

<img width="996" alt="image" src="https://github.com/user-attachments/assets/9729afd1-e3c9-4ea7-a596-96dece093555">

Punto 7: Configurar CI en ambos pipelines (YAML y Classic Editor). Mostrar resultados de la ejecución automática de ambos pipelines al hacer un commit en la rama main.
<img width="622" alt="image" src="https://github.com/user-attachments/assets/441ca94f-ef20-43fb-bfa2-24f93459ab6a">

<img width="685" alt="image" src="https://github.com/user-attachments/assets/f18a3e22-4f8d-47ef-9731-0c768b6ddd43">

<img width="675" alt="image" src="https://github.com/user-attachments/assets/dc32542e-09fc-48ed-94a7-87053ea600b6">


Punto 8: Explicar la diferencia entre un agente MS y un agente Self-Hosted. Qué ventajas y desventajas hay entre ambos? Cuándo es conveniente y/o necesario usar un Self-Hosted Agent?
Un agente MS es un agente en la nube gestionado por Azure DevOps. Es fácil de usar, sin necesidad de gestionar infraestructura, pero tiene menos control sobre el entorno y puede haber tiempos de espera en momentos de alta demanda.
Un agente Self-Hosted es un agente que configuras en tu propia infraestructura, dándote control total sobre el entorno y sin limitaciones de tiempo de ejecución. Sin embargo, requiere mantenimiento y gestión continua.
Usa un agente Self-Hosted cuando necesitas un entorno personalizado, integrar software específico o evitar las limitaciones de los agentes en la nube.

Punto 9: Crear un Pool de Agentes y un Agente Self-Hosted

<img width="706" alt="image" src="https://github.com/user-attachments/assets/f8bb4045-72ee-48b6-9685-b39561cbd613">

<img width="681" alt="image" src="https://github.com/user-attachments/assets/0632a1f7-af96-40a8-81a5-dd50e8165b44">

<img width="672" alt="image" src="https://github.com/user-attachments/assets/8e31e1a9-2401-42fb-84a9-c952db9fcd08">

Punto 10: Instalar y correr un agente en nuestra máquina local.

<img width="648" alt="image" src="https://github.com/user-attachments/assets/a18e2e0e-e984-47f2-bbfb-08fa5dc599d8">

Punto 11: Crear un pipeline que use el agente Self-Hosted alojado en nuestra máquina local.

<img width="651" alt="image" src="https://github.com/user-attachments/assets/dbaf1c42-e8ae-45dd-8287-362e44258733">

<img width="631" alt="image" src="https://github.com/user-attachments/assets/5ce1ba70-b325-465e-9254-8fb4ca5376b0">

Punto 12: Buscar el resultado del pipeline y correr localmente el software compilado.
<img width="632" alt="image" src="https://github.com/user-attachments/assets/dfd51ebd-47c4-40f3-8f78-71838ac22881">

<img width="1005" alt="image" src="https://github.com/user-attachments/assets/dc9efbb7-a2c1-4b59-93c6-3939bc7e7c79">

Punto 13: Crear un nuevo proyecto en ADO clonado desde un repo que contenga una aplicación en Angular
<img width="640" alt="image" src="https://github.com/user-attachments/assets/72e74dc6-41bc-40f8-adc5-cc0a9d9b9f61">

Punto 14: Configurar un pipeline de build para un proyecto de tipo Angular como el clonado.
<img width="1003" alt="image" src="https://github.com/user-attachments/assets/dca61916-0156-4900-a407-77de6f9c8bed">

Punto 15: Habilitar CI para el pipeline.


<img width="376" alt="image" src="https://github.com/user-attachments/assets/bdfd78c2-16ef-4bfd-ba97-2571bd77e930">


