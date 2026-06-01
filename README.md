# 🐜 GastoHormi

> Control de **gastos hormiga** — esos pequeños gastos diarios que parecen insignificantes pero juntos se convierten en una deuda silenciosa.

![PHP](https://img.shields.io/badge/PHP-8.2-777BB4?style=flat-square&logo=php&logoColor=white)
![MySQL](https://img.shields.io/badge/MySQL-8.0-4479A1?style=flat-square&logo=mysql&logoColor=white)
![Bootstrap](https://img.shields.io/badge/Bootstrap-5.3-7952B3?style=flat-square&logo=bootstrap&logoColor=white)
![Chart.js](https://img.shields.io/badge/Chart.js-4.4-FF6384?style=flat-square&logo=chartdotjs&logoColor=white)
![XAMPP](https://img.shields.io/badge/XAMPP-Apache%202.4-FB7A24?style=flat-square&logo=xampp&logoColor=white)

---

## ✨ Características

- 📊 **Dashboard** con KPIs del mes, tendencia de 30 días y gráficas interactivas
- 🐜 **Clasificación hormiga** — marca gastos como hormiga o normal y visualiza su impacto anual
- 📋 **Historial** filtrable por fecha, categoría y tipo con paginación
- 📈 **Reportes analíticos**: tendencia mensual, top gastos, heatmap por día de la semana
- 🏷️ **Categorías** personalizables con icono (Font Awesome) y color
- 🔐 **PIN de acceso** — bloqueo opcional con hash bcrypt
- 🎨 **Formato configurable** — símbolo de moneda, código ISO, separador decimal y formato de fecha
- 📤 **Exportar datos** en CSV (compatible con Excel) y JSON
- 🧪 **Generador de datos simulados** por mes para pruebas y demos
- 🧙 **Wizard de primera vez** — asistente de 4 pasos al iniciar la app

---

## 🚀 Instalación

### Requisitos

- [XAMPP 8.x](https://www.apachefriends.org) (Apache 2.4 + MySQL 8 + PHP 8.2+)
- Navegador moderno (Chrome, Firefox, Edge)

### Pasos

```bash
# 1. Clona el repositorio en htdocs
git clone https://github.com/tu-usuario/gastohormiga.git C:/xampp/htdocs/gastohormi

# 2. Inicia Apache y MySQL desde XAMPP Control Panel

# 3. Ejecuta el setup en el navegador
http://localhost/gastohormi/setup.php
```

El setup crea automáticamente la base de datos `gastohormi`, las tablas, 11 categorías predefinidas y ~300 gastos de ejemplo (90 días).

### Primera visita

Al abrir `http://localhost/gastohormi/` se ejecuta el **wizard de bienvenida**:

1. Bienvenida
2. PIN opcional (4 dígitos)
3. Formato de moneda y fechas
4. ¡Listo! → Dashboard

---

## 🗂️ Estructura del proyecto

```
gastohormi/
├── index.php              # Dashboard principal
├── agregar.php            # Registrar nuevo gasto
├── editar.php             # Editar gasto existente
├── historial.php          # Listado con filtros y paginación
├── reportes.php           # Analítica avanzada
├── categorias.php         # Gestión de categorías
├── configuracion.php      # Seguridad · Formato · Datos
├── bienvenida.php         # Wizard de primera vez
├── pin.php                # Pantalla de bloqueo PIN
├── setup.php              # Instalación y seed
│
├── api/
│   ├── gastos.php         # REST: GET POST PUT DELETE
│   ├── categorias.php     # REST: GET POST PUT DELETE
│   ├── reportes.php       # Analytics: dashboard, mensual, heatmap...
│   ├── config.php         # Ajustes: PIN, formato, borrar datos
│   ├── exportar.php       # Descarga CSV / JSON
│   └── simular.php        # Genera datos de ejemplo por mes
│
├── config/
│   └── database.php       # Conexión PDO (getDB())
│
├── includes/
│   ├── header.php         # Sidebar + topbar + auth (PIN / primer_inicio)
│   └── footer.php         # Scripts CDN + inyección de window.ghCfg
│
├── assets/
│   ├── css/style.css      # Estilos: sidebar, KPIs, PIN pad, wizard
│   └── js/app.js          # formatMoney · formatDate · ghTick · chart helpers
│
└── docs/
    ├── informe_tecnico.pdf
    └── manual_despliegue.pdf
```

---

## 🗄️ Base de datos

```sql
-- Tres tablas principales
categorias  (id, nombre, icono, color, es_hormiga, created_at)
gastos      (id, categoria_id, descripcion, monto, fecha, es_hormiga, notas, created_at)
config      (clave PK, valor, updated_at)
```

**Claves en `config`:** `pin_hash` · `moneda_simbolo` · `moneda_codigo` · `separador_decimal` · `formato_fecha` · `primer_inicio`

---

## 🔌 API REST

| Endpoint | Métodos | Descripción |
|----------|---------|-------------|
| `/api/gastos.php` | GET POST PUT DELETE | CRUD de gastos |
| `/api/categorias.php` | GET POST PUT DELETE | CRUD de categorías |
| `/api/reportes.php?tipo=` | GET | `dashboard` `mensual` `heatmap_semana` `hormiga_impact` |
| `/api/config.php` | GET POST | Leer / guardar ajustes y PIN |
| `/api/exportar.php?formato=` | GET | `csv` o `json` |
| `/api/simular.php?mes=&anio=` | GET POST | Consultar / generar datos de ejemplo |

---

## ⚙️ Configuración

Todas las preferencias se guardan en la tabla `config` y se inyectan como `window.ghCfg` antes de cargar `app.js`:

```js
// Generado automáticamente por footer.php
window.ghCfg = {
  moneda_simbolo:    "$",
  moneda_codigo:     "MXN",
  separador_decimal: ".",
  formato_fecha:     "DD/MM/YYYY"
}
```

Las funciones `formatMoney()`, `formatDate()` y `ghTick()` leen este objeto en tiempo de ejecución — cualquier cambio en Configuración se refleja al recargar la página.

---

## 🔐 Seguridad del PIN

El PIN se almacena como hash **bcrypt** (`password_hash` / `PASSWORD_DEFAULT`). El flujo:

```
Cualquier página
      │
      ▼
header.php ──► primer_inicio = '1'? ──► bienvenida.php (wizard)
      │
      ▼
  pin_hash ≠ '' AND !$_SESSION['pin_verificado']? ──► pin.php
      │
      ▼
   App normal
```

> ¿Olvidaste el PIN? Visita `http://localhost/gastohormi/setup.php?reset_pin=1`

---

## 🧪 Generar datos de prueba

```bash
# Generar datos para un mes
POST http://localhost/gastohormi/api/simular.php?mes=6&anio=2026

# Reemplazar datos existentes del mes
POST http://localhost/gastohormi/api/simular.php?mes=6&anio=2026&override=1
```

O desde la interfaz: **Configuración → Datos → Generar datos de ejemplo**

Patrones simulados: café diario (L-V), transporte diario, snacks (días impares), comida rápida (mar/jue), supermercado (sábados), entretenimiento (vie/sab), compras impulsivas (c/8 días), servicios (días 1 y 5).

---

## 📤 Exportar datos

| Formato | URL directa |
|---------|-------------|
| CSV (Excel-compatible, BOM UTF-8) | `GET /api/exportar.php?formato=csv` |
| JSON (con metadatos) | `GET /api/exportar.php?formato=json` |

---

## 🛠️ Stack tecnológico

| Capa | Tecnología |
|------|-----------|
| Backend | PHP 8.2 (procedimental, PDO) |
| Base de datos | MySQL 8 (InnoDB) |
| CSS | Bootstrap 5.3 |
| Iconos | Font Awesome 6.4 |
| Gráficas | Chart.js 4.4 |
| Tipografía | Inter (Google Fonts) |
| Servidor | Apache 2.4 (XAMPP) |
| JS | Vanilla ES2020 (sin frameworks) |

---

## 📄 Documentación

Los documentos técnicos están en la carpeta [`docs/`](docs/):

- [`informe_tecnico.pdf`](docs/informe_tecnico.pdf) — Arquitectura, API, BD, flujos
- [`manual_despliegue.pdf`](docs/manual_despliegue.pdf) — Instalación paso a paso, troubleshooting

---

## 🐛 Solución de problemas

| Problema | Solución |
|----------|---------|
| Error de conexión en setup.php | Verificar que MySQL esté activo en XAMPP |
| Página en blanco / error 500 | Activar `display_errors` en `php.ini`, revisar `apache/logs/error.log` |
| PIN olvidado | `http://localhost/gastohormi/setup.php?reset_pin=1` |
| Dashboard vacío (mes sin datos) | Configuración → Datos → Generar datos de ejemplo |
| CSV con caracteres raros en Excel | Abrir con: Datos → Desde texto/CSV → UTF-8 |

---

## 📝 Licencia

MIT — libre para uso personal, educativo y comercial.

---

<p align="center">
  Hecho con PHP puro · Sin frameworks · Sin dependencias de Node
  <br>
  <sub>Los gastos hormiga parecen pequeños, pero juntos forman grandes deudas 🐜</sub>
</p>
