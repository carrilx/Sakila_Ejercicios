# Tema 4 — Subconsultas

Ejercicios con subconsultas escalares, correlacionadas, `IN`, `NOT IN`, `EXISTS` y `NOT EXISTS`.

---

## Ejercicio 1

**Obtén los títulos de las películas cuya duración supera la duración media de todas las películas.**

??? success ":white_check_mark: Solución"
    ```sql
    SELECT title, length
    FROM film
    WHERE length > (
        SELECT AVG(length)
        FROM film
    )
    ORDER BY length DESC;
    ```

??? note ":pencil: Explicación"
    La subconsulta `(SELECT AVG(length) FROM film)` es **escalar y no correlacionada**: devuelve un único valor y se ejecuta una sola vez antes que la consulta principal.

    Su resultado (la media de duración) se usa como umbral de comparación en el `WHERE` de la consulta externa.

---

## Ejercicio 2

**Lista los actores que han participado en más películas que la media de participaciones de todos los actores.**

??? success ":white_check_mark: Solución"
    ```sql
    SELECT a.first_name, a.last_name, COUNT(fa.film_id) AS peliculas
    FROM actor a
    INNER JOIN film_actor fa ON a.actor_id = fa.actor_id
    GROUP BY a.actor_id, a.first_name, a.last_name
    HAVING COUNT(fa.film_id) > (
        SELECT AVG(conteo)
        FROM (
            SELECT COUNT(film_id) AS conteo
            FROM film_actor
            GROUP BY actor_id
        ) AS subq
    )
    ORDER BY peliculas DESC;
    ```

??? note ":pencil: Explicación"
    La subconsulta del `HAVING` está anidada en dos niveles:

    - La más interna cuenta películas por actor.
    - La intermedia envuelve ese conteo en una subconsulta derivada para poder aplicar `AVG` sobre él.

    Las **subconsultas derivadas** (en el `FROM`) deben tener un alias obligatoriamente en MySQL: aquí se usa `subq`.

    No se puede hacer `AVG(COUNT(...))` directamente; hay que materializar los conteos primero como subconsulta.

---

## Ejercicio 3

**Encuentra los títulos de las películas que nunca han sido alquiladas.**

??? success ":white_check_mark: Solución"
    ```sql
    SELECT f.title
    FROM film f
    WHERE f.film_id NOT IN (
        SELECT DISTINCT i.film_id
        FROM inventory i
        INNER JOIN rental r ON i.inventory_id = r.inventory_id
    )
    ORDER BY f.title;
    ```

??? note ":pencil: Explicación"
    La subconsulta obtiene los `film_id` de todas las películas que tienen al menos un alquiler registrado.

    `NOT IN` devuelve las películas cuyo `film_id` no aparece en esa lista.

    **Precaución con `NULL`**: si la subconsulta puede devolver `NULL`, `NOT IN` puede producir resultados inesperados. En este caso no hay riesgo porque `film_id` en `inventory` no es nulo.

---

## Ejercicio 4

**Obtén los clientes cuya cuota total pagada supera la cuota media de todos los clientes.**

??? success ":white_check_mark: Solución"
    ```sql
    SELECT c.first_name, c.last_name,
           SUM(p.amount) AS total_pagado
    FROM customer c
    INNER JOIN payment p ON c.customer_id = p.customer_id
    GROUP BY c.customer_id, c.first_name, c.last_name
    HAVING SUM(p.amount) > (
        SELECT AVG(total)
        FROM (
            SELECT SUM(amount) AS total
            FROM payment
            GROUP BY customer_id
        ) AS promedios
    )
    ORDER BY total_pagado DESC;
    ```

??? note ":pencil: Explicación"
    Igual que en el ejercicio 2, se necesita una subconsulta derivada para calcular el promedio de sumas.

    La subconsulta más interna calcula el total pagado por cada cliente; la intermedia calcula la media de esos totales.

    Ese valor promedio se usa en `HAVING` para filtrar solo los clientes que superan la media.

---

## Ejercicio 5

**Lista las categorías en las que existe al menos una película con clasificación 'R'.**

??? success ":white_check_mark: Solución"
    ```sql
    SELECT DISTINCT c.name AS categoria
    FROM category c
    WHERE EXISTS (
        SELECT 1
        FROM film_category fc
        INNER JOIN film f ON fc.film_id = f.film_id
        WHERE fc.category_id = c.category_id
          AND f.rating = 'R'
    )
    ORDER BY c.name;
    ```

??? note ":pencil: Explicación"
    `EXISTS` devuelve verdadero si la subconsulta devuelve al menos una fila. No importa qué se seleccione dentro, por eso se usa `SELECT 1` por convención.

    La subconsulta es **correlacionada**: hace referencia a `c.category_id` de la consulta externa. Se ejecuta una vez por cada fila de `category`.

    `EXISTS` suele ser más eficiente que `IN` cuando la subconsulta puede devolver muchas filas, porque se detiene en cuanto encuentra la primera coincidencia.

---

## Ejercicio 6

**Encuentra los clientes que NO han realizado ningún pago en el año 2005.**

??? success ":white_check_mark: Solución"
    ```sql
    SELECT c.first_name, c.last_name, c.email
    FROM customer c
    WHERE NOT EXISTS (
        SELECT 1
        FROM payment p
        WHERE p.customer_id = c.customer_id
          AND YEAR(p.payment_date) = 2005
    )
    ORDER BY c.last_name;
    ```

??? note ":pencil: Explicación"
    `NOT EXISTS` devuelve verdadero cuando la subconsulta no encuentra ninguna fila. Es la negación de `EXISTS`.

    La condición `YEAR(p.payment_date) = 2005` filtra los pagos del año 2005.

    Este patrón es preferible a `NOT IN` cuando la subconsulta puede contener `NULL`, ya que `NOT EXISTS` maneja los nulos correctamente.

---

## Ejercicio 7

**Lista los títulos de las películas cuyo coste de reposición es mayor que el coste de reposición de cualquier película de la categoría 'Horror'.**

??? success ":white_check_mark: Solución"
    ```sql
    SELECT title, replacement_cost
    FROM film
    WHERE replacement_cost > ALL (
        SELECT f.replacement_cost
        FROM film f
        INNER JOIN film_category fc ON f.film_id      = fc.film_id
        INNER JOIN category c       ON fc.category_id = c.category_id
        WHERE c.name = 'Horror'
    )
    ORDER BY replacement_cost DESC;
    ```

??? note ":pencil: Explicación"
    `> ALL (subconsulta)` exige que el valor sea mayor que **todos** los valores devueltos por la subconsulta. Es equivalente a `> MAX(...)` en este contexto.

    La contraparte es `> ANY`, que exige que el valor sea mayor que **al menos uno** de los valores.

    La subconsulta obtiene los costes de reposición de todas las películas de la categoría 'Horror'.
