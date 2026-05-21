# Tema 2 — JOINs

Ejercicios de combinación de tablas con `INNER JOIN`, `LEFT JOIN` y múltiples tablas.

---

## Ejercicio 1

**Muestra el nombre completo de cada actor junto con el título de cada película en la que ha participado. Ordena por apellido del actor y luego por título de película.**

??? success ":white_check_mark: Solución"
    ```sql
    SELECT a.first_name, a.last_name, f.title
    FROM actor a
    INNER JOIN film_actor fa ON a.actor_id = fa.actor_id
    INNER JOIN film f        ON fa.film_id  = f.film_id
    ORDER BY a.last_name ASC, f.title ASC;
    ```

??? note ":pencil: Explicación"
    La tabla `film_actor` es una **tabla puente** que resuelve la relación muchos a muchos entre actores y películas.

    Se necesitan **dos JOINs**: primero conectar `actor` con `film_actor` (por `actor_id`), y luego `film_actor` con `film` (por `film_id`).

    Los alias `a`, `fa` y `f` hacen el código más corto y evitan ambigüedades al referirse a columnas como `actor_id` que existen en varias tablas.

    `ORDER BY a.last_name ASC, f.title ASC` ordena primero por apellido y, dentro del mismo apellido, por título de película.

---

## Ejercicio 2

**Lista el título de cada película junto con el nombre de su categoría.**

??? success ":white_check_mark: Solución"
    ```sql
    SELECT f.title, c.name AS categoria
    FROM film f
    INNER JOIN film_category fc ON f.film_id      = fc.film_id
    INNER JOIN category c       ON fc.category_id = c.category_id
    ORDER BY c.name ASC, f.title ASC;
    ```

??? note ":pencil: Explicación"
    Al igual que con actores, la relación película-categoría pasa por la tabla intermedia `film_category`.

    `AS categoria` renombra la columna `name` de `category` para que el resultado sea más descriptivo.

    Ordenar primero por categoría y luego por título produce un resultado agrupado visualmente por género.

---

## Ejercicio 3

**Muestra todos los clientes (`customer`) junto con la dirección de su tienda (`store`). Incluye clientes aunque no tengan tienda asignada.**

??? success ":white_check_mark: Solución"
    ```sql
    SELECT c.first_name, c.last_name, s.store_id,
           a.address, ci.city, co.country
    FROM customer c
    LEFT JOIN store s    ON c.store_id    = s.store_id
    LEFT JOIN address a  ON s.address_id  = a.address_id
    LEFT JOIN city ci    ON a.city_id     = ci.city_id
    LEFT JOIN country co ON ci.country_id = co.country_id
    ORDER BY c.last_name;
    ```

??? note ":pencil: Explicación"
    `LEFT JOIN` conserva todas las filas de la tabla izquierda (`customer`) aunque no haya coincidencia en las tablas de la derecha. En ese caso las columnas de la derecha aparecen como `NULL`.

    La dirección de la tienda requiere encadenar cuatro tablas: `store → address → city → country`.

    Este tipo de consultas es muy habitual cuando se trabaja con datos geográficos normalizados en varias tablas.

---

## Ejercicio 4

**Lista los empleados (`staff`) y el nombre de la tienda donde trabajan, incluyendo su dirección completa.**

??? success ":white_check_mark: Solución"
    ```sql
    SELECT st.first_name, st.last_name,
           s.store_id,
           a.address, ci.city, co.country
    FROM staff st
    INNER JOIN store s    ON st.store_id   = s.store_id
    INNER JOIN address a  ON s.address_id  = a.address_id
    INNER JOIN city ci    ON a.city_id     = ci.city_id
    INNER JOIN country co ON ci.country_id = co.country_id;
    ```

??? note ":pencil: Explicación"
    Se usa `INNER JOIN` porque en este caso todos los empleados tienen una tienda asignada. No hay necesidad de conservar filas sin coincidencia.

    Nótese que el alias de `staff` es `st` para evitar confusión con `store` (`s`).

    El patrón `store → address → city → country` es recurrente en Sakila; conviene memorizarlo.

---

## Ejercicio 5

**Obtén los títulos de las películas que están en el inventario de la tienda con `store_id = 1`, sin repeticiones.**

??? success ":white_check_mark: Solución"
    ```sql
    SELECT DISTINCT f.title
    FROM film f
    INNER JOIN inventory i ON f.film_id = i.film_id
    WHERE i.store_id = 1
    ORDER BY f.title;
    ```

??? note ":pencil: Explicación"
    `inventory` contiene una fila por cada copia física de una película en una tienda. Una película puede tener varias copias, por eso se usa `DISTINCT` para evitar repetir el título.

    El filtro `WHERE i.store_id = 1` se aplica después del JOIN, sobre el conjunto combinado de filas.

---

## Ejercicio 6

**Lista los clientes que han realizado algún alquiler, mostrando su nombre completo, el título de la película alquilada y la fecha de alquiler. Ordena por fecha descendente.**

??? success ":white_check_mark: Solución"
    ```sql
    SELECT c.first_name, c.last_name,
           f.title,
           r.rental_date
    FROM customer c
    INNER JOIN rental r    ON c.customer_id  = r.customer_id
    INNER JOIN inventory i ON r.inventory_id = i.inventory_id
    INNER JOIN film f      ON i.film_id      = f.film_id
    ORDER BY r.rental_date DESC;
    ```

??? note ":pencil: Explicación"
    El alquiler (`rental`) no referencia directamente la película; referencia el `inventory_id` (la copia física). Por eso es necesario pasar por `inventory` para llegar a `film`.

    La cadena completa es: `customer → rental → inventory → film`.

    Ordenar por `rental_date DESC` muestra primero los alquileres más recientes.

---

## Ejercicio 7

**Muestra los clientes que NO han realizado ningún alquiler.**

??? success ":white_check_mark: Solución"
    ```sql
    SELECT c.first_name, c.last_name, c.email
    FROM customer c
    LEFT JOIN rental r ON c.customer_id = r.customer_id
    WHERE r.rental_id IS NULL;
    ```

??? note ":pencil: Explicación"
    El patrón `LEFT JOIN ... WHERE columna_derecha IS NULL` es la forma clásica de encontrar registros sin correspondencia.

    El `LEFT JOIN` mantiene todos los clientes; cuando un cliente no tiene alquileres, los campos de `rental` son `NULL`. Filtrando por `r.rental_id IS NULL` quedamos solo con esos clientes.

    Alternativa equivalente usando `NOT EXISTS` o `NOT IN` con subconsulta (ver Tema 4).
