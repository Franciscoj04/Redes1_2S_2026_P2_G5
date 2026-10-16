# Guía Rápida de Configuración para Integrantes 2 y 3

> **IMPORTANTE (Regla de Calificación):** Cada integrante debe hacer `git commit` y `git push` desde su propia cuenta de GitHub con sus avances en el `.pkt` y sus capturas en el `README.md`. Trabajen por turnos sobre el archivo `.pkt` (primero Integrante 2 hace `.pkt` y `push`, luego Integrante 3 hace `pull`, configura ruteo y hace `push`).

---

## PARTE A: Guía para Integrante 2 (Switching, VLANs, Alta Disponibilidad y PCs)

### Paso 1: Asignar IPs a las 28 PCs y Servidores
* Entra a cada PC/Servidor (`Desktop -> IP Configuration`) y copia exactamente la IP, Máscara y Default Gateway que están en la **Sección 5.6 del `README.md`**.

### Paso 2: Configurar Sede Occidente (`SW-OCC-1` y `SW-OCC-2`)
1. En **`SW-OCC-1`**:
   * Configurar VTP Server: `vtp mode server`, `vtp domain bantech16`, `vtp password cisco`.
   * Crear VLANs con nombre exacto:
     * `vlan 15` -> `name Cajas`
     * `vlan 25` -> `name Asesores`
     * `vlan 35` -> `name Gerencia`
     * `vlan 45` -> `name Seguridad`
   * Configurar Trunks: `interface range Gi0/1 - 2` -> `switchport mode trunk`.
   * Asignar puertos de acceso:
     * `Fa0/1 - 2` -> `switchport mode access` -> `switchport access vlan 15`
     * `Fa0/3 - 4` -> `switchport mode access` -> `switchport access vlan 25`
2. En **`SW-OCC-2`**:
   * Configurar VTP Client: `vtp mode client`, `vtp domain bantech16`, `vtp password cisco`.
   * Configurar Trunk hacia `SW-OCC-1`: `interface Gi0/1` -> `switchport mode trunk`.
   * Asignar puertos de acceso (las VLANs ya habrán aparecido solas por VTP):
     * `Fa0/1` -> `switchport mode access` -> `switchport access vlan 35`
     * `Fa0/2 - 3` -> `switchport mode access` -> `switchport access vlan 45`

### Paso 3: Configurar Sede Norte (`SW-NORTE-ROOT`, `SW-NORTE-2`, `SW-NORTE-3`)
1. En **los 3 switches**:
   * Activar Rapid PVST+: `spanning-tree mode rapid-pvst`
   * Crear las 3 VLANs en los 3 switches: `vlan 55` (`name Analisis`), `vlan 65` (`name Auditoria`), `vlan 75` (`name Legal`).
2. En **`SW-NORTE-ROOT`**:
   * Forzar como Root Bridge: `spanning-tree vlan 1,55,65,75 priority 4096`
   * Configurar Trunks: `interface range Gi0/1 - 2, Fa0/24` -> `switchport mode trunk`.
3. En **`SW-NORTE-2`**:
   * Configurar Trunks del anillo: `interface range Gi0/1 - 2` -> `switchport mode trunk`.
   * Puertos de acceso: `interface range Fa0/1 - 3` -> `switchport mode access` -> `switchport access vlan 55`.
4. En **`SW-NORTE-3`**:
   * Configurar Trunks del anillo: `interface range Gi0/1 - 2` -> `switchport mode trunk`.
   * Puertos de acceso:
     * `Fa0/1 - 2` -> `switchport mode access` -> `switchport access vlan 65`
     * `Fa0/3 - 4` -> `switchport mode access` -> `switchport access vlan 75`

### Paso 4: Configurar Sede Oriente (`MS1`, `MS2`, `SW-ORI-1`, `SW-ORI-2`)
1. En **los 4 switches (`MS1`, `MS2`, `SW-ORI-1`, `SW-ORI-2`)**:
   * Crear VLANs: `vlan 85` (`name Boveda`) y `vlan 95` (`name Plataforma`).
2. En **`MS1` y `MS2`** (Switches Multicapa 3650):
   * En puertos `Gi1/0/1`, `Gi1/0/2`, `Gi1/0/3`: ejecutar `switchport trunk encapsulation dot1q` y luego `switchport mode trunk`.
   * Copiar la configuración de interfaces `Vlan85`, `Vlan95` y **HSRP (`standby`)** que ya está escrita en la **Sección 6.3 del `README.md`** (no olvidar `no shutdown` en `interface Vlan85` y `interface Vlan95`).
3. En **`SW-ORI-1`**:
   * Trunks hacia `MS1/MS2`: `interface range Gi0/1 - 2` -> `switchport mode trunk`.
   * Acceso: `interface range Fa0/1 - 3` -> `switchport mode access` -> `switchport access vlan 85`.
4. En **`SW-ORI-2`**:
   * Trunks hacia `MS1/MS2`: `interface range Gi0/1 - 2` -> `switchport mode trunk`.
   * Acceso: `interface range Fa0/1 - 4` -> `switchport mode access` -> `switchport access vlan 95`.

### Paso 5: Configurar Switches de Data Center (`SW-DC-DIST`, `SW-DC-ACC1`, `SW-DC-ACC2`)
1. En **los 3 switches**:
   * Crear VLANs: `vlan 16` (`name Core_BD`), `vlan 26` (`name Web_Apps`), `vlan 36` (`name NOC`).
2. En **`SW-DC-DIST`**:
   * Trunk hacia `R-Central2`: `interface Gi0/1` -> `switchport mode trunk`.
   * EtherChannel 1 (LACP) hacia `SW-DC-ACC1`:
     * `interface range Fa0/1 - 2` -> `channel-group 1 mode active`
     * `interface Port-channel 1` -> `switchport mode trunk`
   * EtherChannel 2 (LACP) hacia `SW-DC-ACC2`:
     * `interface range Fa0/3 - 4` -> `channel-group 2 mode active`
     * `interface Port-channel 2` -> `switchport mode trunk`
3. En **`SW-DC-ACC1`**:
   * EtherChannel 1 (LACP): `interface range Fa0/1 - 2` -> `channel-group 1 mode active` y luego `interface Port-channel 1` -> `switchport mode trunk`.
   * Puertos de acceso a Servidores:
     * `Fa0/3 - 4` -> `switchport mode access` -> `switchport access vlan 16`
     * `Fa0/5 - 7` -> `switchport mode access` -> `switchport access vlan 26`
4. En **`SW-DC-ACC2`**:
   * EtherChannel 2 (LACP): `interface range Fa0/1 - 2` -> `channel-group 2 mode active` y luego `interface Port-channel 2` -> `switchport mode trunk`.
   * Puertos de acceso a PCs NOC:
     * `Fa0/3 - 4` -> `switchport mode access` -> `switchport access vlan 36`

### Paso 6: Evidencias de Integrante 2
* Guardar el `.pkt`, tomar captura apagando `Gi1/0/2` en `MS1` (demostrando que `MS2` pasa a `Active` en HSRP) y captura de `show spanning-tree` en Sede Norte. Subir al repo con `git commit`.

---

## PARTE B: Guía para Integrante 3 (Backbone Core, Routers, Protocolos y Redistribución)

### Paso 1: Configurar Router-on-a-Stick en Sedes (`R-Occidente`, `R-Norte`, `R-Central2`)
1. **`R-Occidente`:** Copiar el bloque de subinterfaces (`Gi0/0.15`, `.25`, `.35`, `.45`) de la **Sección 6.1 del `README.md`**. Configurar el puerto serial `Se0/0/0` con IP `10.16.0.14 255.255.255.252` y `no shutdown`.
2. **`R-Norte`:** Copiar el bloque de subinterfaces (`Gi0/0.55`, `.65`, `.75`) de la **Sección 6.2 del `README.md`**. Configurar el puerto serial `Se0/0/0` con IP `10.16.0.18 255.255.255.252` y `no shutdown`.
3. **`R-Central2`:**
   * Encender `Gi0/1` (`no shutdown`) y crear subinterfaces:
     * `Gi0/1.16` -> `encapsulation dot1Q 16` -> `ip address 192.168.16.33 255.255.255.240`
     * `Gi0/1.26` -> `encapsulation dot1Q 26` -> `ip address 192.168.16.1 255.255.255.224`
     * `Gi0/1.36` -> `encapsulation dot1Q 36` -> `ip address 192.168.16.49 255.255.255.240`
   * Configurar `Gi0/0` (hacia `R-Central1`) con IP `192.168.16.66 255.255.255.252` (`no shutdown`).
   * Agregar ruta estática por defecto hacia `R-Central1`: `ip route 0.0.0.0 0.0.0.0 192.168.16.65`.

### Paso 2: Configurar `R-Central1` (Frontera Data Center)
* Interfaz `Gi0/1` (hacia `R-Central2`): IP `192.168.16.65 255.255.255.252` (`no shutdown`).
* Interfaz `Gi0/0` (hacia `SW-CORE-1`): IP `10.16.0.26 255.255.255.252` (`no shutdown`).
* Copiar las 3 rutas estáticas hacia las VLANs del Data Center y la redistribución en `router ospf 1` que están en la **Sección 6.4 del `README.md`**.

### Paso 3: Configurar el Backbone Core (`SW-CORE-1`, `SW-CORE-2`, `R-CORE-3`) y `MS1`
1. En **`SW-CORE-1`**:
   * Activar ruteo: `ip routing`
   * EtherChannel Capa 3 con `SW-CORE-2`:
     * `interface range Gi1/0/1 - 2` -> `no switchport` -> `channel-group 1 mode active`
     * `interface Port-channel 1` -> `no switchport` -> `ip address 10.16.0.1 255.255.255.252`
   * Enlace Fibra (`Gi1/1/1` hacia `R-CORE-3`): `no switchport` -> `ip address 10.16.0.5 255.255.255.252` -> `no shutdown`.
   * Enlace Cobre (`Gi1/0/4` hacia `R-Central1`): `no switchport` -> `ip address 10.16.0.25 255.255.255.252` -> `no shutdown`.
   * Configurar `router ospf 1`: anunciar redes `10.16.0.0/30`, `10.16.0.4/30` y `10.16.0.24/30` en `area 0`.
2. En **`SW-CORE-2`**:
   * Activar ruteo (`ip routing`) y configurar `Port-channel 1` con IP `10.16.0.2 255.255.255.252` (ver **Sección 6.5 del `README.md`**).
   * Enlace Cobre (`Gi1/0/3` hacia `R-CORE-3`): `no switchport` -> `ip address 10.16.0.9 255.255.255.252` -> `no shutdown`.
   * Enlace Cobre (`Gi1/0/4` hacia `MS1`): `no switchport` -> `ip address 10.16.0.21 255.255.255.252` -> `no shutdown`.
   * Copiar la redistribución mutua **OSPF <-> RIPv2** de la **Sección 6.5 del `README.md`**.
3. En **`MS1` (Conexión de Oriente al Backbone)**:
   * En `interface Gi1/0/24` (que va hacia `SW-CORE-2`): `no switchport` -> `ip address 10.16.0.22 255.255.255.252` -> `no shutdown`.
   * Verificar que `router rip` (`version 2`, `no auto-summary`) tenga `network 10.0.0.0` y `network 172.16.0.0` (también agregar `router rip` en `MS2` con `network 172.16.0.0`).
4. En **`R-CORE-3`**:
   * Enlace Fibra (`Gi0/1/0` hacia `SW-CORE-1`): IP `10.16.0.6 255.255.255.252` (`no shutdown`).
   * Enlace Cobre (`Gi0/0` hacia `SW-CORE-2`): IP `10.16.0.10 255.255.255.252` (`no shutdown`).
   * Enlace Serial (`Se0/0/0` hacia `R-Occidente`): IP `10.16.0.13 255.255.255.252`, `clock rate 128000`, `no shutdown`.
   * Enlace Serial (`Se0/0/1` hacia `R-Norte`): IP `10.16.0.17 255.255.255.252`, `clock rate 128000`, `no shutdown`.
   * Copiar el bloque de redistribución mutua **OSPF <-> EIGRP 16** de la **Sección 6.5 del `README.md`**.

### Paso 4: Evidencias de Integrante 3
* Verificar `ping` entre PCs de distintas sedes (ej. de `PC-OCC-1` en Occidente a `SRV-BD-1` en Data Center y a `PC-ORI-1` en Oriente).
* Tomar capturas de `show ip route` en `R-CORE-3`, `SW-CORE-2` y `R-Central1`, más las capturas de `ping`, agregarlas a la **Sección 7 del `README.md`** y hacer `git commit` / `git push` desde su cuenta de GitHub.