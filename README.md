# Portfolio DevSecOps: Scripts Automatizados, Infraestructura y Seguridad CI/CD

[![DevSecOps - Semgrep SAST Scan](https://github.com/karyparrauni/trabajo-15/actions/workflows/semgrep.yml/badge.svg)](https://github.com/karyparrauni/trabajo-15/actions)

## 📌 Visión General del Portfolio (TP01 a TP15)
Este repositorio centraliza y consolida el recorrido práctico completo de la materia, abarcando desde la administración base en sistemas Linux y la automatización mediante scripting, hasta la orquestación en la nube y la integración de la cadena completa de **DevSecOps** en pipelines de CI/CD.

| Bloque Temático | Trabajos Prácticos | Tecnologías y Conceptos Clave |
|---|---|---|
| **Fundamentos de Operaciones** | **TP01 – TP04** | Scripting automatizado en Bash, principio de menor privilegio (`devops-deploy`), flujo de trabajo colaborativo Gitflow y diagnóstico de conectividad de red en formato YAML. |
| **Contenerización y Redes** | **TP05 – TP06** | Empaquetado en Docker (Flask API), optimización por capas, usuarios no-root y orquestación multicapa con Docker Compose (Nginx, Flask, PostgreSQL). |
| **CI/CD y Observabilidad** | **TP07 – TP08** | Pipelines automatizados en GitHub Actions, pruebas locales con `act`, instrumentación de métricas con Prometheus y tableros en Grafana. |
| **Orquestación en Kubernetes** | **TP09 – TP10** | Administración declarativa en K8s (Pods, Secrets, PVC), enrutamiento con Ingress Controllers y parametrización mediante Charts personalizados en Helm. |
| **Infraestructura como Código** | **TP11 – TP12** | Aprovisionamiento modular de infraestructura con Terraform y consolidación del portfolio unificado en un ecosistema integrable. |
| **Cadena DevSecOps Integral** | **TP13 – TP15** | **DAST** dinámico con OWASP ZAP, **Modelado de Amenazas** con Threagile, y **SAST** multilenguaje con Semgrep y freno de mano (*Andon Cord*). |

---

## 🛡️ TP15: Análisis Estático de Seguridad (SAST) con Semgrep

El **TP15** complementa las etapas previas de seguridad (OWASP ZAP y Threagile) mediante la incorporación de **Static Application Security Testing (SAST)** automatizado en GitHub Actions. A diferencia de analizadores tradicionales, **Semgrep** funciona como un motor multilenguaje capaz de auditar todas las capas tecnológicas del proyecto en un único flujo de trabajo.

### 📜 Capas Auditadas y Reglas Aplicadas

| Capa Tecnológica | Archivos Auditados | Conjunto de Reglas (`--config`) | Riesgos y Vulnerabilidades Detectadas |
|---|---|---|---|
| **Backend Python** | `app/backend/app.py`, rutas API | `OWASP Top 10`, `p/python` | Inyección SQL, funciones peligrosas (`eval`), desinfección de entradas y configuración CORS. |
| **Contenedores** | `Dockerfile` (frontend/backend) | `p/dockerfile` | Ejecución como usuario root, imágenes base vulnerables y manejo inseguro de capas. |
| **Infraestructura (IaC)** | `guia-11/*.tf`, módulos | `p/terraform` | Reglas de Ingress permisivas (`0.0.0.0/0`), credenciales expuestas y almacenamiento inseguro. |
| **Orquestación (K8s/Helm)** | `devops-tp12/chart`, manifiestos | `p/kubernetes`, `p/owasp-top-ten` | Ausencia de límites de recursos (`limits/requests`), Secrets en texto plano y permisos excesivos. |

---

## 🚨 Estrategia de Seguridad: Guardián Estricto (*Andon Cord*)

El pipeline implementa una evaluación en dos niveles para equilibrar la visibilidad de reportes con la protección del despliegue:

1. **Modo Informativo (Reportes e Históricos):** Los pasos de generación de reportes finalizan con `|| true`. Esto garantiza que los artefactos (`semgrep-results.json`, `semgrep.sarif`) y la tabla en `$GITHUB_STEP_SUMMARY` se publiquen en cada ejecución, sin importar los hallazgos.
2. **Andon Cord (Freno de Mano DevSecOps):** El paso final ejecuta Semgrep con la instrucción `--severity=ERROR --error`. Ante cualquier vulnerabilidad crítica (por ejemplo, ejecución remota por tubería como `curl | bash`), el paso fuerza la finalización con `exit code 1`, interrumpiendo el flujo de integración antes del despliegue.

---

## ⚙️ Estructura del Workflow (`.github/workflows/semgrep.yml`)
1. **Renderizado de Helm:** Genera el YAML estático (`helm template`) hacia `.semgrep-tmp/` para analizar plantillas de Kubernetes antes de aplicar los manifiestos.
2. **Escaneo Multilenguaje:** Analiza el código fuente, la infraestructura y los contenedores.
3. **Resumen de Auditoría:** Construye un cuadro en Markdown con el estado de cada capa en `$GITHUB_STEP_SUMMARY`.
4. **Carga de Artefactos:** Publica la carpeta comprimida `semgrep-report` (`semgrep-results.json`).
5. **Code Scanning (SARIF):** Carga los resultados en la pestaña **Security** de GitHub.
6. **Guardia Andon Cord:** Aplica la verificación estricta de severidad para detener el pipeline si existen bloqueantes.

---

## 💻 Ejecución y Auditoría en Entorno Local

Para verificar el código localmente antes de enviar los cambios al repositorio remoto:

```bash
# 1. Activar entorno virtual e instalar Semgrep
source .venv/bin/activate
python3 -m pip install semgrep

# 2. Escaneo general informativo (excluyendo carpetas secundarias)
semgrep scan --config=auto --exclude=.venv --exclude='**/.terraform/**' .

# 3. Escaneo de validación estricta (Andon Cord local)
semgrep scan --config=p/owasp-top-ten --severity=ERROR .
📁 Entregables del TP15
Workflow: .github/workflows/semgrep.yml
Reporte de Auditoría: Artefacto semgrep-report (semgrep-results.json) adjunto al run exitoso.
Evidencias: Captura del fallo controlado por Andon Cord (curl-pipe-shell), commit de mitigación y ejecuciones posteriores completamente en verde (✅).

---
