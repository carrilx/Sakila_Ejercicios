# Tema 5 — Vistas

Ejercicios de creación, uso y eliminación de vistas (`VIEW`) en MySQL.

---

## Ejercicio 1

**Crea una vista llamada `peliculas_con_categoria` que muestre el título de cada película junto con su categoría.**

??? success ":white_check_mark: Solución"
    ```sql
    CREATE VIEW peliculas_con_categoria AS
    SELECT f.film_id, f.title, c.name AS categoria
    FROM film f
    INNER JOIN film_category fc ON f.film_id      = fc.film_id
    INNER JOIN category c       ON fc.category_id = c.category_id;

    -- Uso de la vista
    SELECT * FROM peliculas_con_categoria WHERE categoria = 'Action';
    ```

??? note ":pencil: Explicación"
    Una vista es una **consulta almacenada con nombre**. No guarda datos físicamente; cada vez que se consulta, ejecuta la query subyacente.

    `CREATE VIEW nombre AS SELECT ...` es la sintaxis básica.

    Una vez creada, se usa como si fuera una tabla: `SELECT * FROM peliculas_con_categoria`.

---

## Ejercicio 2

**Crea una vista llamada `clientes_activos` que muestre el nombre, apellido y email de los clientes activos (`active = 1`).**

??? success ":white_check_mark: Solución"
    ```sql
    CREATE VIEW clientes_activos AS
    SELECT customer_id, first_name, last_name, email
    FROM customer
    WHERE active = 1;

    -- Comprobación
    SELECT COUNT(*) FROM clientes_activos;
    ```

??? note ":pencil: Explicación"
    Esta vista se basa en una **sola tabla** y no incluye funciones de agregado ni GROUP BY, por lo que es una **vista actualizable** en MySQL.

    Esto significa que se pueden realizar operaciones `INSERT`, `UPDATE` y `DELETE` sobre ella, y los cambios se aplicarán sobre la tabla `customer`.

---

## Ejercicio 3

**Inserta un cliente nuevo a través de la vista `clientes_activos`. Comprueba que aparece en la tabla `customer`.**

??? success ":white_check_mark: Solución"
    ```sql
    -- Insertar a través de la vista
    INSERT INTO clientes_activos (customer_id, first_name, last_name, email)
    VALUES (600, 'Prueba', 'Usuario', 'prueba@sakila.com');

    -- Verificar en la tabla base
    SELECT * FROM customer WHERE customer_id = 600;
    ```

??? note ":pencil: Explicación"
    Al hacer `INSERT` sobre la vista, MySQL escribe directamente en la tabla base `customer`.

    La vista es solo una "ventana" a esa tabla; no tiene almacenamiento propio.

    **Nota**: el `INSERT` puede fallar si no se proporcionan valores para columnas obligatorias (`NOT NULL`) de la tabla base que no están incluidas en la vista.

---

## Ejercicio 4

**Crea una vista llamada `resumen_alquileres` que muestre, para cada cliente, su nombre completo y el número total de alquileres que ha realizado.**

??? success ":white_check_mark: Solución"
    ```sql
    CREATE VIEW resumen_alquileres AS
    SELECT c.customer_id,
           CONCAT(c.first_name, ' ', c.last_name) AS nombre_completo,
           COUNT(r.rental_id) AS total_alquileres
    FROM customer c
    LEFT JOIN rental r ON c.customer_id = r.customer_id
    GROUP BY c.customer_id, c.first_name, c.last_name;

    -- Uso: clientes con más de 30 alquileres
    SELECT * FROM resumen_alquileres
    WHERE total_alquileres > 30
    ORDER BY total_alquileres DESC;
    ```

??? note ":pencil: Explicación"
    Esta vista usa `GROUP BY` y `COUNT`, por lo que **no es actualizable**: no se pueden hacer INSERT, UPDATE ni DELETE sobre ella.

    Sin embargo, es muy útil para consultar: oculta la complejidad del JOIN y el GROUP BY.

    El `LEFT JOIN` garantiza que los clientes sin alquileres también aparezcan (con `total_alquileres = 0`).

---

## Ejercicio 5

**Intenta insertar una fila en la vista `resumen_alquileres`. ¿Qué ocurre?**

??? success ":white_check_mark: Solución"
    ```sql
    INSERT INTO resumen_alquileres (customer_id, nombre_completo, total_alquileres)
    VALUES (999, 'Nuevo Cliente', 0);
    -- ERROR 1471: The target table resumen_alquileres of the INSERT is not insertable-into
    ```

??? note ":pencil: Explicación"
    MySQL lanza el error `1471` porque la vista no es actualizable al contener `GROUP BY` y funciones de agregado.

    Las condiciones para que una vista sea actualizable en MySQL incluyen:

    - Basada en una sola tabla sin JOIN.
    - Sin `DISTINCT`, `GROUP BY`, `HAVING`, `UNION`.
    - Sin funciones de agregado (`COUNT`, `SUM`, `AVG`, etc.).

    Para modificar datos en casos como este sería necesario hacerlo directamente sobre las tablas base.

---

## Ejercicio 6

**Crea una vista llamada `top_peliculas` que muestre las 10 películas más alquiladas, con su título y número de alquileres.**

??? success ":white_check_mark: Solución"
    ```sql
    CREATE VIEW top_peliculas AS
    SELECT f.film_id, f.title,
           COUNT(r.rental_id) AS num_alquileres
    FROM film f
    INNER JOIN inventory i ON f.film_id      = i.film_id
    INNER JOIN rental r    ON i.inventory_id = r.inventory_id
    GROUP BY f.film_id, f.title
    ORDER BY num_alquileres DESC
    LIMIT 10;

    -- Consultar la vista
    SELECT * FROM top_peliculas;
    ```

??? note ":pencil: Explicación"
    Las vistas pueden incluir `ORDER BY` y `LIMIT`, aunque si se añade un `ORDER BY` externo al consultar la vista, puede sobreescribir el orden interno.

    Esta vista encapsula una consulta de tres tablas y una agregación, simplificando enormemente el acceso a esta información.

---

## Ejercicio 7

**Elimina las vistas `peliculas_con_categoria`, `clientes_activos`, `resumen_alquileres` y `top_peliculas`.**

??? success ":white_check_mark: Solución"
    ```sql
    DROP VIEW IF EXISTS peliculas_con_categoria;
    DROP VIEW IF EXISTS clientes_activos;
    DROP VIEW IF EXISTS resumen_alquileres;
    DROP VIEW IF EXISTS top_peliculas;
    ```

??? note ":pencil: Explicación"
    `DROP VIEW` elimina la **definición** de la vista. No elimina los datos de las tablas base.

    `IF EXISTS` evita un error si la vista ya no existe. Es una buena práctica incluirlo para hacer los scripts más robustos.

    Diferencia clave:

    - `DROP TABLE`: elimina estructura **y** datos físicos.
    - `DROP VIEW`: elimina solo la definición de la consulta; los datos siguen intactos en las tablas base.
