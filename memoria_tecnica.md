# Memoria ténica

## Portada (título, integrantes, tutor/a)
## Análisis y mejora de la infraestrucura informática del IES Padre Poveda
Integrantes: 
 - Ángel Luis Aranda
 - Juan Ramón Jiménez
 - Jesús Cortés

Curso: 2º ASIR
## Índice
## Resumen ejecutivo

## [1. Introducción y objetivos](#1-introducción-y-objetivos)
## [2. Análisis de requisitos](#2-análisis-de-requisitos)
## [3. Diseño técnico (diagramas, topologías)](#3-diseño-técnico-diagramas-topologías)
## [4. Implementación (procedimientos, comandos, scripts)](#4-implementación-procedimientos-comandos-scripts)
## [5. Pruebas y resultados (evidencias)](#5-pruebas-y-resultados-evidencias)
## [6. Seguridad y gestión de riesgos](#6-seguridad-y-gestión-de-riesgos)
## [7. Conclusiones y mejoras propuestas](#7-conclusiones-y-mejoras-propuestas)
## [8. Anexos (código, scripts, listados de configuración, FUENTES)](#8-anexos-código-scripts-listados-de-configuración-fuentes)


## Resumen ejecutivo


## 1. Introducción y objetivos


## 2. Análisis de requisitos


## 3. Diseño técnico (diagramas, topologías)


## 4. Implementación (procedimientos, comandos, scripts)


## 5. Pruebas y resultados (evidencias)


## 6. Seguridad y gestión de riesgos


## 7. Conclusiones y mejoras propuestas


## 8. Anexos (código, scripts, listados de configuración, FUENTES)































# Proyecto de infraestructura de red

## Descripción general
En el departamento contamos con un router con **IP pública dinámica**, por lo que utilizaremos **DDNS** para permitir el acceso desde el exterior. El switch es **configurable** y se usará para conectar:

- Equipos del profesorado.
- La red de alumnado.
- La **DMZ** con los servicios del centro.

## Cambios propuestos
Se sustituirá el proxy actual por uno nuevo y se creará una **DMZ**. El nuevo proxy dispondrá de **tres interfaces de red**:

1. **Interfaz de entrada**: conexión desde el departamento (salida a Internet).
2. **Interfaz DMZ**: red donde residirán los servidores.
3. **Interfaz de alumnado**: salida a la red del alumnado.

## Servicios en la DMZ

- **DNS**: filtrado de acceso a sitios web y mejora de rendimiento mediante caché.
- **Web**: alojamiento del sitio web del centro.
- **FTP**: repositorio para el profesorado con acceso desde todas las aulas.
- **Mail**: opcional; actualmente el correo está gestionado por Google mediante acuerdos con la Junta de Andalucía.

## Servicios fuera de la DMZ (switch del alumnado)
En el switch del alumnado, encargado de distribuir Internet a todas las aulas, se desplegarán los siguientes servidores:

- **FOG**: restauración y despliegue de equipos sin necesidad de pendrive.
- **RADIUS**: autenticación de profesorado para acceso al servidor FTP y un posible servidor **LDAP**.
