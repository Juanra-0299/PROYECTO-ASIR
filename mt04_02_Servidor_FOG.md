# Guía de Configuración y Clonación Masiva con FOG Project

## Introducción
**FOG Project** es una suite gratuita de código abierto para la gestión de imágenes de disco y clonación masiva de ordenadores a través de la red. Permite a los administradores instalar sistemas operativos, realizar copias de seguridad y configurar decenas de equipos de forma desatendida y remota.

En este escenario, se detalla el uso de FOG exclusivamente para la **clonación de ordenadores con arranque dual (Dual Boot)** que contienen **Windows 10 y Xubuntu**, listo para entornos de administración de sistemas (ASIR).

---

## 1. Requisitos Previos e Infraestructura
Antes de comenzar el proceso de registro y clonación, es fundamental cumplir con las siguientes condiciones de hardware y firmware:

* **Hardware Idéntico:** Se requieren al menos dos ordenadores con componentes idénticos (especialmente discos duros de la misma capacidad, por ejemplo, **1 TB**). Uno actuará como el equipo de origen (*Master*) con el arranque dual ya instalado y configurado, y el otro como destino.
* **Prioridad de Arranque (BIOS):** Para visualizar correctamente el menú de arranque dual (GRUB), la partición de Ubuntu debe tener la máxima prioridad en la sección *Boot* de la BIOS.
* **Seguridad del Firmware:** Las opciones **Secure Boot** y **Fast Boot** deben estar completamente **deshabilitadas** en la BIOS de ambos equipos.

---

## 2. Configuración de Red en la BIOS (Equipo de Origen)
Una vez que los sistemas operativos estén listos y configurados en el equipo *Master*:
1. Reinicie el equipo y acceda a la configuración de la BIOS.
2. Habilite la opción **"Boot from onboard LAN"** (o denominación similar según el fabricante de la placa).
3. Guarde los cambios y salga.
4. Vuelva a ingresar a la BIOS y configure el arranque por red (**PXE Boot / LAN**) como **Boot Option #1**.

---

## 3. Registro del Host en el Servidor FOG
Al reiniciar el equipo por red, se cargará el menú PXE de FOG:

1. Seleccione la opción de **Registro e Inventario** (*Perform Full Host Registration and Inventory*).
2. El sistema solicitará rellenar varios campos en la línea de comandos (no es necesario completarlos todos):
   * **Hostname (Nombre del host):** Introduzca el nombre identificativo. *(Ejemplo: `2ASIR_DualBoot`)*.
   * **Image ID:** Presione `ENTER` para omitirlo por el momento.
   * Continúe presionando `ENTER` para aceptar los valores predeterminados hasta llegar a la solicitud de usuario.
   * **User:** Introduzca el nombre de usuario deseado.
3. Responda con `ENTER` a las 3 preguntas subsiguientes.
4. Espere a que el menú confirme que el host ha sido registrado correctamente.
5. **Apague el equipo** una vez finalizado el registro.

---

## 4. Gestión desde el Panel Web de FOG
Acceda a la interfaz web de administración de FOG introduciendo la dirección IP del servidor en su navegador web (`http://<IP_SERVIDOR_FOG>/fog/management`) e inicie sesión con sus credenciales.

### Paso A: Verificación del Host
* Diríjase a **Hosts** > **List all hosts**.
* Compruebe que el host registrado (`2ASIR_DualBoot`) aparece correctamente en la lista.

### Paso B: Creación de la Imagen de Disco
* Vaya a la sección **Images** y seleccione **Create New Image**.
* Configure los parámetros de la imagen exactamente con la estructura adecuada para sistemas multi-partición:
  * **Image Name:** `2ASIR_DualBoot`
  * **Operative System:** `Linux`
  * **Storage Group:** `default`
  * **Image Type:** `Multiple Partitions - Single Disk (Not Resizable)` *(Nota: Crucial para mantener la estructura de GRUB y Windows intacta)*
  * **Partition:** `Everything`

### Paso C: Asignación de la Imagen al Host
* Regrese a **Hosts** > **List all hosts** y haga clic sobre el nombre de su host (`2ASIR_DualBoot`).
* En el desplegable **Host Image**, seleccione la imagen que acaba de crear (`2ASIR_DualBoot`).
* Haga clic en **Update** para guardar los cambios.

---

## 5. Captura de la Imagen (*Upload*)
1. Dentro del perfil del host en la interfaz web, diríjase a la pestaña superior **Basic Tasks**.
2. Seleccione la opción **Capture** (identificada con un icono amarillo).
3. Confirme la tarea haciendo clic en el botón **Task**.
4. **Encienda el equipo de origen** (se iniciará automáticamente por red a través de PXE).
5. Comenzará el proceso automatizado de captura de datos:
   * El sistema clonará secuencialmente las particiones detectadas. Aunque aparezca el mensaje *"Cloned successfully"* en la primera partición, **no intervenga**, ya que debe procesar las siguientes particiones del arranque dual.
   * Una vez finalizadas todas las particiones, el equipo se reiniciará automáticamente y volverá al menú de host registrado.

---

## 6. Despliegue de la Imagen (*Deploy*) en Equipos de Destino
Para replicar el arranque dual en un nuevo ordenador idéntico:

1. **Arranque el segundo equipo por red (PXE)** y realice el proceso de registro completo como se describió en la Sección 3 (por ejemplo, asígnele el nombre `PC2ASIR_02`).
2. Una vez registrado, apáguelo o déjelo en espera.
3. Acceda al panel web de FOG, busque el nuevo host (`PC2ASIR_02`) y **asígnele la misma imagen** (`2ASIR_DualBoot`) en el campo *Host Image*. Haga clic en **Update**.
4. Diríjase a **Basic Tasks** dentro del perfil de este nuevo host y seleccione **Deploy** (identificado con un icono verde). Confirme la tarea en **Task**.
5. Encienda o reinicie el `PC2ASIR_02`. Al arrancar por red, el servidor FOG iniciará automáticamente el volcado y despliegue de la imagen.
6. Al finalizar el proceso, el nuevo equipo dispondrá del arranque dual completamente funcional (Windows 10 y Xubuntu).

---

## Resolución de Errores Comunes

### Error de Tamaño de Disco Insuficiente (Incompatibilidad de Size)
* **Síntoma:** Error durante el despliegue o la captura que impide completar la clonación masiva.
* **Causa:** Este fallo ocurre habitualmente cuando la imagen de origen fue capturada desde un disco de mayor capacidad (por ejemplo, **1 TB**) e intenta desplegarse en un equipo de destino que cuenta con un disco duro de menor tamaño (por ejemplo, **256 GB**), a pesar de que el resto del hardware de los ordenadores parezca idéntico.
* **Solución:** Asegúrese siempre de que el disco de destino sea de igual o mayor tamaño que el disco de origen del cual se tomó la muestra maestra.
