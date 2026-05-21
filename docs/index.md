# Ejercicios SQL — Base de Datos Sakila

Bienvenido a esta colección de ejercicios SQL resueltos sobre la base de datos **Sakila**, una base de datos de ejemplo oficial de MySQL que simula el sistema de gestión de un videoclub.

## ¿Qué es la base de datos Sakila?

Sakila contiene tablas que representan películas, actores, clientes, alquileres, tiendas y empleados de un videoclub. Es ideal para practicar SQL porque incluye relaciones variadas, datos realistas y una estructura bien normalizada.

Las tablas principales son:

| Tabla | Descripción |
|-------|-------------|
| `film` | Catálogo de películas |
| `actor` | Lista de actores |
| `film_actor` | Relación muchos a muchos entre películas y actores |
| `category` | Categorías de películas |
| `film_category` | Relación entre películas y categorías |
| `customer` | Clientes del videoclub |
| `rental` | Registros de alquileres |
| `inventory` | Inventario de copias físicas |
| `payment` | Pagos realizados |
| `store` | Tiendas |
| `staff` | Empleados |
| `address` / `city` / `country` | Datos geográficos |

## Estructura de los ejercicios

Cada ejercicio sigue el mismo formato:

1. **Enunciado** — descripción del problema a resolver
2. **Solución** (plegada) — la consulta SQL correcta
3. **Explicación** (plegada) — análisis detallado de la consulta

Haz clic sobre las secciones plegadas para desplegarlas.

## Instalación de la base de datos

Puedes descargar el archivo SQL de la base de datos desde el repositorio e importarla en tu servidor MySQL:

```bash
mysql -u root -p < Sakila_BD/sakila-schema.sql
mysql -u root -p < Sakila_BD/sakila-data.sql
```

## Temas

| Tema | Contenido |
|------|-----------|
| [Tema 1 — Consultas Básicas](tema1-consultas-basicas/ejercicios.md) | SELECT, WHERE, ORDER BY, LIMIT |
| [Tema 2 — JOINs](tema2-joins/ejercicios.md) | INNER JOIN, LEFT JOIN, múltiples tablas |
| [Tema 3 — Agregación](tema3-agregacion/ejercicios.md) | GROUP BY, HAVING, funciones de agregado |
| [Tema 4 — Subconsultas](tema4-subconsultas/ejercicios.md) | Subconsultas correlacionadas y no correlacionadas, EXISTS |
| [Tema 5 — Vistas](tema5-vistas/ejercicios.md) | CREATE VIEW, vistas actualizables, DROP VIEW |
