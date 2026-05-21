# Tema 3 — Agregación y Agrupación

Ejercicios con `GROUP BY`, `HAVING` y funciones de agregado: `COUNT`, `SUM`, `AVG`, `MAX`, `MIN`.

---

## Ejercicio 1

**Cuenta el número de películas que hay en cada categoría. Muestra el nombre de la categoría y el total, ordenado de mayor a menor.**

??? success ":white_check_mark: Solución"
    ```sql
    SELECT c.name AS categoria, COUNT(fc.film_id) AS total_peliculas
    FROM category c
    INNER JOIN film_category fc ON c.category_id = fc.category_id
    GROUP BY c.category_id, c.name
    ORDER BY total_peliculas DESC;
    ```

??? note ":pencil: Explicación"
    `COUNT(fc.film_id)` cuenta cuántas filas de `film_category` hay para cada categoría. Usar `COUNT(*)` también sería correcto aquí.

    `GROUP BY c.category_id, c.name` agrupa el resultado por categoría. Se incluye `c.name` en el GROUP BY porque aparece en el SELECT.

    El alias `total_peliculas` se puede usar en `ORDER BY` porque MySQL evalúa los alias del SELECT antes del ORDER BY.

---

## Ejercicio 2

**Muestra los 5 actores que han participado en más películas, con el número de películas por actor.**

??? success ":white_check_mark: Solución"
    ```sql
    SELECT a.first_name, a.last_name, COUNT(fa.film_id) AS num_peliculas
    FROM actor a
    INNER JOIN film_actor fa ON a.actor_id = fa.actor_id
    GROUP BY a.actor_id, a.first_name, a.last_name
    ORDER BY num_peliculas DESC
    LIMIT 5;
    ```

??? note ":pencil: Explicación"
    `COUNT(fa.film_id)` cuenta las películas de cada actor.

    `ORDER BY num_peliculas DESC` ordena de mayor a menor participación.

    `LIMIT 5` restringe el resultado a los 5 primeros. Combinado con `ORDER BY DESC` es el patrón típico para obtener el "top N".

---

## Ejercicio 3

**Calcula los ingresos totales de cada tienda (`store`). Muestra el `store_id` y el total recaudado, ordenado de mayor a menor.**

??? success ":white_check_mark: Solución"
    ```sql
    SELECT s.store_id, SUM(p.amount) AS ingresos_totales
    FROM store s
    INNER JOIN staff st  ON s.store_id  = st.store_id
    INNER JOIN payment p ON st.staff_id = p.staff_id
    GROUP BY s.store_id
    ORDER BY ingresos_totales DESC;
    ```

??? note ":pencil: Explicación"
    Los pagos (`payment`) se asocian a un `staff_id`, no directamente a una tienda. Por eso es necesario pasar por `staff` para llegar a `store`.

    `SUM(p.amount)` suma todos los importes de los pagos gestionados por los empleados de cada tienda.

---

## Ejercicio 4

**Obtén la duración media de las películas por categoría. Muestra solo las categorías cuya duración media supere los 110 minutos.**

??? success ":white_check_mark: Solución"
    ```sql
    SELECT c.name AS categoria, ROUND(AVG(f.length), 2) AS duracion_media
    FROM category c
    INNER JOIN film_category fc ON c.category_id = fc.category_id
    INNER JOIN film f           ON fc.film_id     = f.film_id
    GROUP BY c.category_id, c.name
    HAVING AVG(f.length) > 110
    ORDER BY duracion_media DESC;
    ```

??? note ":pencil: Explicación"
    `AVG(f.length)` calcula la media de la columna `length` dentro de cada grupo.

    `ROUND(..., 2)` redondea a 2 decimales para una presentación más limpia.

    **`HAVING` vs `WHERE`**: `WHERE` filtra filas antes de agrupar; `HAVING` filtra grupos después de aplicar las funciones de agregado. Aquí necesitamos `HAVING` porque la condición hace referencia a `AVG`, que solo existe tras agrupar.

---

## Ejercicio 5

**Lista los clientes que han realizado más de 30 alquileres. Muestra su nombre y el número de alquileres.**

??? success ":white_check_mark: Solución"
    ```sql
    SELECT c.first_name, c.last_name, COUNT(r.rental_id) AS num_alquileres
    FROM customer c
    INNER JOIN rental r ON c.customer_id = r.customer_id
    GROUP BY c.customer_id, c.first_name, c.last_name
    HAVING COUNT(r.rental_id) > 30
    ORDER BY num_alquileres DESC;
    ```

??? note ":pencil: Explicación"
    La condición `> 30` se aplica sobre una función de agregado, por lo que **obligatoriamente** debe ir en `HAVING`, no en `WHERE`.

    En el `ORDER BY` se puede usar el alias `num_alquileres`; en el `HAVING` hay que repetir la función `COUNT(r.rental_id)`.

---

## Ejercicio 6

**Muestra, para cada clasificación (`rating`), el número de películas, la duración mínima, la duración máxima y la duración media.**

??? success ":white_check_mark: Solución"
    ```sql
    SELECT
        rating,
        COUNT(*)              AS total,
        MIN(length)           AS duracion_min,
        MAX(length)           AS duracion_max,
        ROUND(AVG(length), 1) AS duracion_media
    FROM film
    GROUP BY rating
    ORDER BY rating;
    ```

??? note ":pencil: Explicación"
    Cuando se agrupa directamente sobre una columna de la misma tabla, no se necesita JOIN.

    `MIN`, `MAX` y `AVG` son funciones de agregado que se calculan dentro de cada grupo definido por `GROUP BY rating`.

---

## Ejercicio 7

**Obtén el importe total pagado por cada cliente. Muestra solo los clientes que han gastado más de 150 €, ordenados de mayor a menor gasto.**

??? success ":white_check_mark: Solución"
    ```sql
    SELECT c.first_name, c.last_name,
           SUM(p.amount) AS total_gastado
    FROM customer c
    INNER JOIN payment p ON c.customer_id = p.customer_id
    GROUP BY c.customer_id, c.first_name, c.last_name
    HAVING SUM(p.amount) > 150
    ORDER BY total_gastado DESC;
    ```

??? note ":pencil: Explicación"
    `SUM(p.amount)` suma todos los pagos de cada cliente.

    El filtro `HAVING SUM(p.amount) > 150` descarta los clientes que han gastado 150 € o menos.

    Identificar los clientes de mayor valor económico es un caso de uso típico en análisis de negocio.
