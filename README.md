# 🛡️ Packet Tracer: Configuración de ACL Estándar para IPv4 con Nombre

<div align="center">

**Laboratorio CISCO - Access Control Lists Estándar con Nombre**

[![Cisco Packet Tracer](https://img.shields.io/badge/Cisco-Packet_Tracer-1BA0D7?style=for-the-badge&logo=cisco&logoColor=white)](https://www.netacad.com)
[![ACL Protocol](https://img.shields.io/badge/Protocol-ACL_Estándar-00A86B?style=for-the-badge)](https://www.cisco.com/)
[![CCNA](https://img.shields.io/badge/Certification-CCNA-blue?style=for-the-badge)](https://www.cisco.com/c/en/us/training-events/training-certifications/certifications/associate/ccna.html)
[![License](https://img.shields.io/badge/License-MIT-yellow?style=for-the-badge)](LICENSE)

[🎯 Objetivos](#-objetivos) • 
[📊 Tabla de Direcciones](#️-tabla-de-asignación-de-direcciones) • 
[📋 Escenario](#️-aspectos-básicosescenario) • 
[⚙️ Configuración](#️-configuración-paso-a-paso) • 
[🔍 Verificación](#️-verificación) • 
[👨‍💻 Autor](#️-autor)

</div>

---

## 📋 Descripción del Proyecto
Este laboratorio de Cisco Packet Tracer implementa una **ACL Estándar con Nombre** para controlar el acceso a un servidor de archivos crítico. La política de seguridad requiere que solo la estación de trabajo del administrador web (PC1) y el servidor web tengan acceso al servidor de archivos, mientras que se debe denegar todo el resto del tráfico. Este escenario demuestra cómo las ACL estándar proporcionan un control básico pero efectivo basado únicamente en direcciones IP de origen.

### 🎯 Objetivos
**Parte 1:** Configurar y aplicar una ACL estándar con nombre  
**Parte 2:** Verificar la implementación y operación de la ACL  

### 📋 Aspectos Básicos/Escenario
El administrador de red ejecutivo ha solicitado crear una ACL estándar con nombre para prevenir el acceso no autorizado a un servidor de archivos que contiene la base de datos para aplicaciones web. Solo la estación de trabajo del administrador web (PC1) y el servidor web necesitan acceder al servidor de archivos. Todo el resto del tráfico debe ser denegado.

---

## 📊 Tabla de Asignación de Direcciones

| Dispositivo | Interfaz | Dirección IP | Máscara de Subred | Gateway |
|-------------|----------|--------------|-------------------|---------|
| **R1** | F0/0 | 192.168.10.1 | 255.255.255.0 | N/A |
| **R1** | F0/1 | 192.168.20.1 | 255.255.255.0 | N/A |
| **R1** | E0/0/0 | 192.168.100.1 | 255.255.255.0 | N/A |
| **R1** | E0/1/0 | 192.168.200.1 | 255.255.255.0 | N/A |
| **File Server** | NIC | 192.168.200.100 | 255.255.255.0 | 192.168.200.1 |
| **Web Server** | NIC | 192.168.100.100 | 255.255.255.0 | 192.168.100.1 |
| **PC0** | NIC | 192.168.20.3 | 255.255.255.0 | 192.168.20.1 |
| **PC1** | NIC | 192.168.20.4 | 255.255.255.0 | 192.168.20.1 |
| **PC2** | NIC | 192.168.10.3 | 255.255.255.0 | 192.168.10.1 |

### 🌐 Topología de Red
```
          [Web Server]        [File Server]
         192.168.100.100     192.168.200.100
                |                   |
          E0/0/0:192.168.100.1     E0/1/0:192.168.200.1
                       [R1]
                 /               \
        F0/0:192.168.10.1       F0/1:192.168.20.1
              |                         |
      [PC2:192.168.10.3]    [PC0:192.168.20.3, PC1:192.168.20.4]
```

---

## ⚙️ Configuración Paso a Paso

### Parte 1: Configurar y Aplicar una ACL Estándar con Nombre

#### Paso 1: Verificar Conectividad Inicial
Antes de configurar la ACL, verificar que todas las estaciones de trabajo pueden hacer ping tanto al servidor web como al servidor de archivos.

```cisco
! Desde cada PC, verificar conectividad:
PC0> ping 192.168.100.100  ! Ping al Web Server
PC0> ping 192.168.200.100  ! Ping al File Server

PC1> ping 192.168.100.100
PC1> ping 192.168.200.100

PC2> ping 192.168.100.100
PC2> ping 192.168.200.100
```

**Resultado esperado:** Todos los pings deben ser exitosos antes de aplicar la ACL.

#### Paso 2: Configurar ACL Estándar con Nombre
```cisco
! Crear ACL estándar con nombre (nombre exacto: File_Server_Restrictions)
R1(config)# ip access-list standard File_Server_Restrictions

! Permitir solo a PC1 (192.168.20.4)
R1(config-std-nacl)# permit host 192.168.20.4

! Permitir solo al Web Server (192.168.100.100)
R1(config-std-nacl)# permit host 192.168.100.100

! Denegar explícitamente todo el resto (aunque es implícito, es buena práctica)
R1(config-std-nacl)# deny any

! Salir del modo ACL
R1(config-std-nacl)# exit
```

**Nota importante:** 
- El nombre de la ACL **distingue entre mayúsculas y minúsculas**: `File_Server_Restrictions`
- Las instrucciones deben estar en el orden especificado
- La regla `deny any` es redundante (denegación implícita), pero se incluye para claridad

#### Verificar Configuración ACL
```cisco
! Verificar contenido de la ACL con números de secuencia
R1# show access-lists

! Salida esperada:
Standard IP access list File_Server_Restrictions
    10 permit host 192.168.20.4
    20 permit host 192.168.100.100
    30 deny any
```

#### Paso 3: Aplicar la ACL con Nombre
```cisco
! Acceder a la interfaz F0/1 (conectada a la red del File Server)
R1(config)# interface FastEthernet 0/1

! Aplicar la ACL en dirección OUT
R1(config-if)# ip access-group File_Server_Restrictions out

! Guardar configuración
R1(config-if)# end
R1# copy running-config startup-config
```

**Justificación de la aplicación:**
- **Interfaz:** F0/1 es la interfaz conectada a la red del servidor de archivos (192.168.200.0/24)
- **Dirección:** OUT filtra el tráfico que **sale** hacia el servidor de archivos
- **Ubicación:** Eficiente - cerca del destino que se está protegiendo

---

## 🔍 Verificación de la Implementación

### Paso 1: Verificar Configuración y Aplicación
```cisco
! Verificar ACL configurada
R1# show access-lists
! Debe mostrar las 3 reglas con números de secuencia

! Verificar aplicación en interfaz
R1# show ip interface FastEthernet 0/1
! Buscar: "Outgoing access list is File_Server_Restrictions"

! Alternativa: ver en running-config
R1# show running-config interface FastEthernet 0/1
```

### Paso 2: Verificar Operación de la ACL
Realizar pruebas de conectividad desde cada dispositivo:

#### Desde PC0 (192.168.20.3)
```cisco
PC0> ping 192.168.100.100  ! Al Web Server - ✅ DEBE FUNCIONAR
PC0> ping 192.168.200.100  ! Al File Server - ❌ DEBE FALLAR (no está en ACL)
```

#### Desde PC1 (192.168.20.4)
```cisco
PC1> ping 192.168.100.100  ! Al Web Server - ✅ DEBE FUNCIONAR
PC1> ping 192.168.200.100  ! Al File Server - ✅ DEBE FUNCIONAR (está en ACL)
```

#### Desde PC2 (192.168.10.3)
```cisco
PC2> ping 192.168.100.100  ! Al Web Server - ✅ DEBE FUNCIONAR
PC2> ping 192.168.200.100  ! Al File Server - ❌ DEBE FALLAR (no está en ACL)
```

#### Desde Web Server (192.168.100.100)
```cisco
Web_Server> ping 192.168.200.100  ! Al File Server - ✅ DEBE FUNCIONAR (está en ACL)
```

### Paso 3: Verificar Contadores ACL
```cisco
! Verificar cuántos paquetes han coincidido con cada regla
R1# show access-lists

! Salida esperada después de pruebas:
Standard IP access list File_Server_Restrictions
    10 permit host 192.168.20.4 (5 matches)     ! PC1 accediendo
    20 permit host 192.168.100.100 (3 matches)  ! Web Server accediendo
    30 deny any (8 matches)                     ! Otros dispositivos bloqueados
```

### Resumen de Resultados Esperados
| Dispositivo | Destino | Resultado | Regla ACL que coincide |
|-------------|---------|-----------|------------------------|
| **PC0** | Web Server | ✅ Permitido | No aplica (ACL solo en F0/1) |
| **PC0** | File Server | ❌ Denegado | deny any (30) |
| **PC1** | Web Server | ✅ Permitido | No aplica (ACL solo en F0/1) |
| **PC1** | File Server | ✅ Permitido | permit host 192.168.20.4 (10) |
| **PC2** | Web Server | ✅ Permitido | No aplica (ACL solo en F0/1) |
| **PC2** | File Server | ❌ Denegado | deny any (30) |
| **Web Server** | File Server | ✅ Permitido | permit host 192.168.100.100 (20) |

---

## 💡 Conceptos Fundamentales Aprendidos

### 🎯 ACL Estándar con Nombre
- **Propósito:** Filtrado basado únicamente en dirección IP de origen
- **Alcance:** Simple pero efectivo para políticas básicas
- **Nombre:** Descriptivo, distingue mayúsculas/minúsculas
- **Ubicación:** Aplicada cerca del destino protegido

### 🔧 Comandos Clave
```cisco
! Crear ACL estándar con nombre
ip access-list standard [NOMBRE]

! Agregar reglas
permit host [IP]    ! Permitir host específico
deny any            ! Denegar todo lo demás (explícito)

! Aplicar a interfaz
interface [interfaz]
 ip access-group [NOMBRE] [in|out]

! Verificación
show access-lists
show ip interface [interfaz]
```

### 📊 Diferencias entre ACL Estándar y Extendida
| Característica | ACL Estándar | ACL Extendida |
|----------------|--------------|---------------|
| **Criterios** | Solo dirección origen | Origen, destino, protocolo, puerto |
| **Complejidad** | Simple | Compleja |
| **Rendimiento** | Más rápido | Más lento |
| **Ubicación** | Cerca del destino | Cerca del origen |
| **Uso típico** | Control básico de acceso | Control granular de servicios |

### 🌐 Política Implementada
```
POLÍTICA: Solo PC1 y Web Server → File Server
IMPLEMENTACIÓN:
1. permit host 192.168.20.4      (PC1)
2. permit host 192.168.100.100   (Web Server)
3. deny any                      (todos los demás)
```

---

## 🚀 Solución de Problemas Comunes

### ❌ La ACL no está filtrando tráfico
```cisco
! Verificar aplicación correcta
R1# show ip interface f0/1
! Debe mostrar: "Outgoing access list is File_Server_Restrictions"

! Verificar dirección (debe ser OUT)
R1# show running-config interface f0/1

! Verificar que la ACL existe
R1# show access-lists
```

### ❌ Dispositivo permitido no puede acceder
```cisco
! Verificar IP del dispositivo
PC1> ipconfig
! Debe ser 192.168.20.4

! Verificar regla en ACL
R1# show access-lists | include 192.168.20.4

! Verificar conectividad básica (sin ACL)
R1(config)# interface f0/1
R1(config-if)# no ip access-group File_Server_Restrictions out
! Probar ping sin ACL
! Restaurar ACL después de prueba
```

### ❌ Contadores no incrementan
```cisco
! Limpiar contadores y probar nuevamente
R1# clear access-list counters

! Realizar prueba específica
PC0> ping 192.168.200.100

! Verificar contadores
R1# show access-lists
! La regla "deny any" debe mostrar matches
```

### 📋 Checklist de Verificación
- [ ] ACL creada con nombre correcto (case-sensitive)
- [ ] Reglas en orden correcto (permits primero, deny al final)
- [ ] ACL aplicada en interfaz F0/1
- [ ] Dirección correcta (OUT)
- [ ] Contadores incrementando en pruebas
- [ ] Configuración guardada (copy run start)

---

## 📚 Recursos Adicionales

### Documentación Oficial Cisco
- [Cisco Standard ACL Configuration](https://www.cisco.com/c/en/us/td/docs/ios-xml/ios/sec_data_acl/configuration/15-mt/sec-acl-15-mt-book.html)
- [Named ACL Configuration Guide](https://www.cisco.com/c/en/us/support/docs/security/ios-firewall/23602-confaccesslists.html)
- [ACL Placement Best Practices](https://www.cisco.com/c/en/us/support/docs/ip/access-lists/13608-21.html)

### Libros Recomendados
- "CCNA 200-301 Official Cert Guide" - ACL Chapter
- "Cisco IOS Access Lists" - O. Held
- "Network Security Fundamentals" - Cisco Press

### Laboratorios Relacionados
- **ACL Estándar Numerada:** Configuración básica con números
- **ACL Extendida:** Filtrado por protocolo y puerto
- **ACL Reflexivas:** Control de sesiones bidireccional
- **ACL Basadas en Tiempo:** Filtrado por horario

### 🎓 Preguntas de Práctica CCNA
1. ¿Por qué las ACL estándar se aplican cerca del destino?
2. ¿Cuál es la diferencia entre `permit host 192.168.1.1` y `permit 192.168.1.1 0.0.0.0`?
3. ¿Qué sucede si no hay una regla `deny any` al final de una ACL?
4. ¿Cómo afecta la dirección (IN/OUT) al filtrado de tráfico?

---

## 👨‍💻 Autor

<div align="center">

**Darwin Manuel Ovalles Cesar**

<p align="center">
<a href="https://www.linkedin.com/in/darwin-manuel-ovalles-cesar-dev" target="_blank">
<img align="center" src="https://raw.githubusercontent.com/rahuldkjain/github-profile-readme-generator/master/src/images/icons/Social/linked-in-alt.svg" alt="LinkedIn - Darwin Ovalles" height="40" width="50" />
</a>
<a href="https://github.com/dovalless" target="_blank">
<img align="center" src="https://raw.githubusercontent.com/rahuldkjain/github-profile-readme-generator/master/src/images/icons/Social/github.svg" alt="GitHub - Darwin Ovalles" height="40" width="50" />
</a>
</p>

💼 **LinkedIn**: [darwin-manuel-ovalles-cesar-dev](https://www.linkedin.com/in/darwin-manuel-ovalles-cesar-dev/)  
🌐 **GitHub**: [@dovalless](https://github.com/dovalless)  
🎓 **Certificaciones**: CCNA, Network+, A+  

*"Las ACL estándar son la primera línea de defensa en la seguridad de red. Aunque simples en concepto, cuando se implementan correctamente proporcionan una protección robusta para recursos críticos como servidores de archivos y bases de datos."*

**#Cisco #PacketTracer #ACL #StandardACL #NetworkSecurity #CCNA #AccessControl**

</div>

---

## 📄 Licencia

Este proyecto está bajo la Licencia MIT. Consulta el archivo [LICENSE](LICENSE) para más detalles.

```
MIT License
Copyright (c) 2024 Darwin Manuel Ovalles Cesar
```

---

## 🙏 Agradecimientos

- **Cisco Networking Academy** - Por Packet Tracer y recursos educativos
- **Administradores de Red** - Por implementar políticas de seguridad efectivas
- **Comunidad de Networking** - Por compartir mejores prácticas

<div align="center">

### ⭐ Si este laboratorio te ayudó a entender ACL estándar, compártelo ⭐

### 🔄 **Reflexión Final:**
*"Configurar una ACL estándar es como hacer una lista de invitados para una fiesta privada: solo las direcciones IP en la lista pueden entrar. Es simple, directo y extremadamente efectivo cuando sabes exactamente quién necesita acceso a qué recursos."*

**Desarrollado con 💙 para la comunidad de seguridad de redes**

---
*Laboratorio completado: Packet Tracer - Configuración de ACL estándar para IPv4 con nombre*  
*Habilidades demostradas: ACL Estándar, ACL con Nombre, Políticas de Acceso, Verificación de Contadores*

</div>
```
