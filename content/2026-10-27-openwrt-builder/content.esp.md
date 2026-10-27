---
classification: article
status: published
topics:
  - openwrt
  - embedded-linux
  - firmware
  - build-automation
  - open-source
related_projects:
  - ../../projects/openwrt-builder.md
  - ../../projects/openwrt-wireless-mesh.md
publications:
  - channel: medium
    url: https://medium.com/@demonccc/de-una-necesidad-personal-a-una-herramienta-para-la-comunidad-openwrt-builder-f87d34ae5418
---

# De una necesidad personal a una herramienta para la comunidad: OpenWrt Builder

*OpenWrt Builder permite generar firmware personalizado mediante distintas estrategias de compilación, optimizando tiempos y recursos. En este artículo explico cómo funciona la herramienta y cómo utilizarla para habilitar las dos radios Wi-Fi en el TP-Link Archer A9 v6.*

## Cómo empezó todo

Llevo tiempo utilizando OpenWrt, tanto en mi red doméstica como en proyectos freelance para distintos clientes. En casa administro varios routers de distintas arquitecturas y los actualizo periódicamente. En el ámbito profesional, lo he aprovechado para implementar soluciones de conectividad y seguridad —como VPNs, proxies y filtrado de tráfico— adaptando la red sin las limitaciones del firmware del fabricante.

Sin embargo, con cada nueva versión me encontraba repitiendo el mismo flujo manual: preparar el entorno de compilación, seleccionar paquetes, aplicar parches al código fuente y generar la imagen final para cada equipo.

El caso determinante fue el TP-Link Archer A9 v6: su radio Wi-Fi de 2.4 GHz requería parches específicos en el controlador ath9k, obligándome a repetir la integración manual en cada actualización.

De ahí surgió OpenWrt Builder: una herramienta para automatizar estas compilaciones personalizadas, separando la configuración de cada equipo de los distintos métodos de construcción.

## 1. OpenWrt Builder: separar la configuración del proceso de compilación

El objetivo de OpenWrt Builder no es reemplazar las herramientas oficiales, sino permitir declarar qué se quiere construir y automatizar el proceso con Python dentro de Docker, tanto en ejecuciones locales como en GitHub Actions.

Las compilaciones se definen mediante profiles: configuraciones independientes que indican la versión de OpenWrt, los paquetes, los repositorios adicionales y los parámetros de cada equipo. El nombre sigue la convención fabricante-modelo-versión; el método de compilación se define dentro del profile, no en su nombre.

Ejemplo de estructura:

```text
profiles/
└── <vendor>-<model>-<openwrt_version>/
    ├── README.md
    ├── settings
    ├── packages
    ├── feeds
    ├── git-packages
    └── source-build-targets  (solo release-patched)
```

Siguiendo este esquema, el perfil del TP-Link Archer A9 v6 para la versión 25.12.6 se organiza bajo la carpeta tplink-archer_a9_v6-25.12.6/.

Así, el Builder se ocupa del entorno Docker, las dependencias y los artefactos finales, mientras cada profile define qué necesita el dispositivo. Para agregar un equipo o una versión, alcanza con incorporar su configuración.

Distintas necesidades, distintas estrategias

No todas las personalizaciones requieren el mismo esfuerzo de cómputo. Incorporar LuCI o un par de paquetes adicionales no justifica recompilar el kernel completo; por el contrario, modificar un controlador de hardware sí exige intervenir el código fuente.

Para resolver esto, OpenWrt Builder ofrece cuatro modos de compilación adaptados a distintas necesidades:

| Modo | Qué hace | Ventaja principal | Limitación |
| --- | --- | --- | --- |
| imagebuilder | Arma una imagen con paquetes binarios oficiales, sin compilar código. | Muy rápido; permite generar firmware localmente o en GitHub Actions con configuración declarativa en Git. | No incorpora cambios de código, kernel o controladores. |
| release-patched | Parte de una release exacta y recompila los componentes afectados por parches; ensambla con ImageBuilder. | Reutiliza los binarios oficiales y acota la compilación a lo necesario. | Exige compatibilidad estricta con la versión base y los módulos. |
| selective-source | Compila desde fuentes el firmware, los paquetes seleccionados y sus dependencias. | Permite seguir ramas y personalizar fuentes sin compilar todo el catálogo. | Puede requerir compilar kernel y herramientas; tarda más. |
| full-source | Habilita una compilación amplia del árbol de OpenWrt y su universo de paquetes. | Máxima amplitud para desarrollo y pruebas de paquetes. | Mayor tiempo, consumo de recursos y posibilidad de fallos ajenos al firmware. |

Elegir la estrategia adecuada optimiza el tiempo de ejecución, el consumo de recursos y la facilidad de mantenimiento.

### ImageBuilder: personalizar firmware sin compilar

Es el método más ágil. Utiliza el ImageBuilder oficial provisto por OpenWrt para cada arquitectura, ensamblando imágenes a partir de paquetes binarios precompilados sin procesar código fuente.

Toda la selección de software queda versionada en Git dentro del perfil, lo que elimina la necesidad de acudir a herramientas web externas o configurar entornos manualmente.

**Ventajas**
- Eficiencia de recursos: al omitir la compilación de herramientas, kernel y bibliotecas, reduce drásticamente el tiempo de ejecución.
- Autonomía: permite generar imágenes consistentes tanto en entornos locales como en flujos de CI/CD.
- Mantenimiento: para actualizar a otra versión de OpenWrt normalmente alcanza con cambiar la referencia del profile y revisar la compatibilidad de los paquetes.

**Desventajas**
- Limitado a los paquetes y controladores oficiales provistos como binarios.
- Imposibilidad de aplicar parches al kernel o a los drivers existentes.

Uso recomendado: dispositivos soportados oficialmente donde solo se requiere personalizar la selección de paquetes o archivos de configuración iniciales.

### Release-patched: modificar lo necesario sin reconstruir todo

Diseñado para escenarios donde el firmware estable funciona correctamente pero requiere parches específicos en controladores o módulos puntuales.

Esta estrategia combina el SDK y el ImageBuilder oficiales con un árbol de fuentes parcheado. El SDK puede compilar también dependencias necesarias, pero solo los paquetes personalizados declarados se incorporan a la imagen final; el resto se obtiene de la release oficial exacta.

**Ventajas**
- Compilación acotada: reduce el trabajo al compilar los componentes modificados y sus dependencias necesarias.
- Estabilidad: mantiene la paridad estricta con la versión oficial base.

**Desventajas**
- Los parches deben ser compatibles con la versión exacta del SDK seleccionado.

Uso recomendado: cuando se requiere aplicar correcciones específicas o soporte de hardware sobre una versión estable sin recompilar todo el sistema.

### Selective-source: compilar desde fuentes sin construir todo el catálogo

Indicado cuando se trabaja sobre ramas en desarrollo o cuando los parches modifican aspectos estructurales que exceden el alcance del SDK. Este modo compila el firmware y la selección de paquetes declarada desde el código fuente, generando las herramientas y el kernel necesarios pero omitiendo el resto del catálogo de OpenWrt.

**Ventajas**
- Flexibilidad para incorporar cambios profundos en el sistema base o seguir ramas de desarrollo activas.
- Control preciso sobre las fuentes y las opciones de compilación.

**Desventajas**
- Mayor tiempo de ejecución y consumo de recursos informáticos.
- Requiere mayor atención al resolver posibles conflictos entre dependencias.

Uso recomendado: desarrollos activos, personalizaciones avanzadas o pruebas sobre versiones en desarrollo (snapshots).

### Full-source: construir desde el código fuente completo

Ofrece acceso completo a la cadena de compilación de OpenWrt, permitiendo procesar todo el árbol de fuentes y el catálogo entero de paquetes disponibles.

**Ventajas**
- Mayor control sobre el proceso de compilación y los paquetes disponibles.
- Ideal para validaciones masivas de software, auditorías o construcción de repositorios propios.

**Desventajas**
- Demanda un tiempo sustancial de CPU y almacenamiento elevado.
- Propenso a fallos en paquetes opcionales ajenos al objetivo principal del dispositivo.

Uso recomendado: tareas de mantenimiento de repositorios, investigación o desarrollo integral sobre OpenWrt.

## 2. Archer A9 v6: cómo habilitar las dos radios Wi-Fi

El TP-Link Archer A9 v6 utiliza la plataforma ath79 y cuenta con dos radios: Qualcomm QCN5502 (2.4 GHz, controlada por ath9k) y Qualcomm QCA9984 (5 GHz, gestionada por ath10k). En las compilaciones oficiales de la rama 25.12, la radio de 2.4 GHz permanece desactivada en el Device Tree Source (DTS), el archivo que describe el hardware al kernel, porque el controlador oficial todavía no incorpora soporte para QCN5502. Para habilitarla no basta con cambiar su estado a okay: también es necesario que ath9k reconozca el chip y utilice correctamente los datos de calibración ART.

### Adaptar el soporte existente

A partir del trabajo de Wenli Looi, hdfla y spike0en, adapté los parches a OpenWrt 25.12.5 y 25.12.6, ajustando la calibración ART y la compatibilidad del controlador ath9k. La construcción del firmware se realiza con release-patched.

El resultado de estos ajustes está publicado en mi fork (github.com/demonccc/openwrt), destacando la rama openwrt-25.12.6-archerA9v6. A nivel de DTS, el bloque para la interfaz inalámbrica queda definido de la siguiente manera:

```dts
&wmac {
    compatible = "qca,qcn5502-wifi";
    status = "okay";
    nvmem-cells = <&cal_art_1000>, <&macaddr_info_8 0>;
    nvmem-cell-names = "calibration", "mac-address";
};
```

### Compilar el firmware con OpenWrt Builder

Para este equipo, el perfil de configuración utiliza la estrategia release-patched. De este modo, el SDK oficial compila los módulos Wi-Fi modificados mientras el Builder ensambla la imagen con el DTS adaptado sobre la base estable:

```ini
METHOD=source
BUILD_MODE=release-patched
REPOSITORY=https://github.com/demonccc/openwrt.git
REF=openwrt-25.12.6-archerA9v6
BASE_REF=v25.12.6
TARGET=ath79
SUBTARGET=generic
DEVICE=tplink_archer-a9-v6
SDK=auto
```

REF identifica la rama de mi fork que contiene los parches para el Archer, mientras que BASE_REF=v25.12.6 fija la release oficial utilizada como base para el SDK, ImageBuilder y los componentes que no se modifican. En packages se selecciona el software por nombre y OpenWrt resuelve sus feeds y dependencias. El archivo feeds es obligatorio, aunque puede quedar vacío cuando no hay repositorios adicionales. FEED_NAMES es un ajuste opcional para limitar las actualizaciones de feeds en los modos que lo admiten. En release-patched, source-build-targets delimita qué componentes se recompilan localmente; las dependencias que el SDK necesite construir no se incorporan automáticamente como paquetes personalizados al firmware.

Para reproducir el build en Linux con Docker:

```bash
git clone https://github.com/demonccc/openwrt-builder.git
cd openwrt-builder
docker run --rm \
  --user "$(id -u):$(id -g)" \
  -e HOME=/tmp \
  -v "$PWD:/workspace" \
  demonccc/openwrt-builder:latest \
  python3 scripts/build.py build \
--profile tplink-archer_a9_v6-25.12.6 \
--output artifact \
  --cache-dir .cache/openwrt-builder
```

El proceso se puede ejecutar con Docker, tanto localmente como mediante GitHub Actions. Al finalizar, genera una imagen para instalar en el Archer A9 v6 y probar ambas radios Wi-Fi, además de un BUILD_INFO que permite distinguir los componentes oficiales de los compilados localmente. La release publicada para OpenWrt 25.12.6 está enlazada en las referencias.

## 3. Dos proyectos que se complementan

La combinación de OpenWrt Builder y el caso práctico del Archer A9 v6 refleja cómo la automatización y la modularidad facilitan el soporte de hardware complejo en sistemas embebidos. Desacoplar las definiciones del entorno de compilación permite mantener imágenes personalizadas sin incurrir en tareas repetitivas.

El objetivo principal de compartir estos proyectos es ofrecer a la comunidad una metodología documentada y reproducible para generar imágenes personalizadas, adaptar perfiles a otros routers y continuar mejorando el soporte de hardware en OpenWrt.

## Repositorios y referencias

**Proyectos publicados**
- OpenWrt Builder — https://github.com/demonccc/openwrt-builder
- Profile Archer A9 v6 — https://github.com/demonccc/openwrt-builder/tree/main/profiles/tplink-archer_a9_v6-25.12.6
- Fork de OpenWrt con soporte QCN5502 — https://github.com/demonccc/openwrt (ramas específicas para 25.12.5 y 25.12.6)
- Firmware Archer A9 v6 — OpenWrt 25.12.6: https://github.com/demonccc/openwrt-builder/releases/tag/tplink-archer_a9_v6-25.12.6-2026.10.26-1603-23

**Trabajos de referencia y créditos**
- OpenWrt — https://github.com/openwrt/openwrt
- Wenli Looi — autor de la serie original de parches QCN550x
- hdfla/openwrt-qcn550x — https://github.com/hdfla/openwrt-qcn550x
- hdfla/openwrt-actions-qcn550x — https://github.com/hdfla/openwrt-actions-qcn550x
- spike0en/openwrt — https://github.com/spike0en/openwrt
- OpenWrt SDK e ImageBuilder — https://github.com/openwrt/docker
