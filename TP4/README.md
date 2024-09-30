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



![image](https://github.com/user-attachments/assets/f3657c0f-386b-40f4-84ca-3bb45d2ffeb1)

![image](https://github.com/user-attachments/assets/69bc66b1-746f-41c0-a155-408c5b9cf457)

![image](https://github.com/user-attachments/assets/ef4dfcea-47ef-40c6-87d3-21f9c8843429)


## 4.3 Explicación: ¿Por qué es necesario contar con una tarea de Publish en un pipeline que corre en un agente de Microsoft en la nube?
Cuando utilizas un agente Microsoft-hosted en Azure DevOps, el entorno en el que se ejecuta el pipeline es temporal. Una vez que el pipeline finaliza, todos los archivos y datos que se generaron durante la ejecución del pipeline son eliminados. Esto significa que si no publicas los artefactos o resultados generados durante la ejecución, no tendrás acceso a ellos después de que el pipeline termine.

La tarea de Publicar artefactos (Publish Pipeline Artifacts) se encarga de almacenar los resultados (artefactos) de la compilación en una ubicación permanente dentro de Azure DevOps, como el almacenamiento de artefactos de pipeline. De esta manera, puedes descargarlos, reutilizarlos en otros pipelines o desplegarlos en otros entornos, incluso después de que el agente de Microsoft haya terminado su trabajo y haya liberado sus recursos.

<img width="1511" alt="image" src="https://github.com/user-attachments/assets/f686b40c-f714-4eb4-a894-263f1b9ae910">


