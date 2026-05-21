# Tema 1 — Consultas Básicas

Ejercicios de `SELECT`, `WHERE`, `ORDER BY` y `LIMIT` sobre la base de datos Sakila.

---

## Ejercicio 1

**Obtén el título, el año de lanzamiento y la duración de todas las películas, ordenadas por duración de mayor a menor.**

??? success ":white_check_mark: Solución"
    ```sql
    SELECT title, release_year, length
    FROM film
    ORDER BY length DESC;
    ```

??? note ":pencil: Explicación"
    `SELECT title, release_year, length` selecciona únicamente las tres columnas pedidas, evitando recuperar datos innecesarios.

    `FROM film` indica la tabla fuente.

    `ORDER BY length DESC` ordena el resultado de mayor a menor duración. Sin `DESC`, el orden por defecto sería ascendente (`ASC`).

---

## Ejercicio 2

**Lista los 10 actores cuyo apellido empieza por la letra 'S', mostrando nombre y apellido juntos en una sola columna llamada `nombre_completo`.**

??? success ":white_check_mark: Solución"
    ```sql
    SELECT CONCAT(first_name, ' ', last_name) AS nombre_completo
    FROM actor
    WHERE last_name LIKE 'S%'
    LIMIT 10;
    ```

??? note ":pencil: Explicación"
    `CONCAT(first_name, ' ', last_name)` une las dos columnas de texto con un espacio entre ellas.

    `AS nombre_completo` asigna un alias a la expresión calculada para que aparezca con ese nombre en el resultado.

    `WHERE last_name LIKE 'S%'` filtra los actores cuyo apellido comienza por 'S'. El `%` es el comodín que representa cualquier secuencia de caracteres.

    `LIMIT 10` restringe el resultado a los primeros 10 registros.

---

## Ejercicio 3

**Muestra las películas con una duración de alquiler (`rental_duration`) superior a 5 días y una tarifa de alquiler (`rental_rate`) inferior a 3,00 €. Ordénalas por tarifa ascendente.**

??? success ":white_check_mark: Solución"
    ```sql
    SELECT title, rental_duration, rental_rate
    FROM film
    WHERE rental_duration > 5
      AND rental_rate < 3.00
    ORDER BY rental_rate ASC;
    ```

??? note ":pencil: Explicación"
    Se combinan dos condiciones con `AND`: ambas deben cumplirse simultáneamente para que una fila se incluya en el resultado.

    `rental_duration > 5` filtra películas con periodo de alquiler de más de 5 días.

    `rental_rate < 3.00` filtra por precio. MySQL usa el punto como separador decimal.

    `ORDER BY rental_rate ASC` ordena de precio más bajo a más alto.

---

## Ejercicio 4

**Obtén los títulos de las películas cuya descripción contiene la palabra 'Documentary'. Muestra también la descripción.**

??? success ":white_check_mark: Solución"
    ```sql
    SELECT title, description
    FROM film
    WHERE description LIKE '%Documentary%';
    ```

??? note ":pencil: Explicación"
    `LIKE '%Documentary%'` busca la subcadena 'Documentary' en cualquier posición de la descripción. El `%` al principio indica que puede haber texto antes, y el `%` al final indica que puede haber texto después.

    Este tipo de búsqueda **no es sensible a mayúsculas** en MySQL con la configuración por defecto (`utf8mb4_general_ci`).

    Para búsquedas en grandes volúmenes de texto es más eficiente usar `FULLTEXT` índices con `MATCH ... AGAINST`.

---

## Ejercicio 5

**Lista los diferentes valores de la columna `rating` (clasificación por edades) que existen en la tabla `film`, sin repeticiones.**

??? success ":white_check_mark: Solución"
    ```sql
    SELECT DISTINCT rating
    FROM film
    ORDER BY rating;
    ```

??? note ":pencil: Explicación"
    `SELECT DISTINCT` elimina duplicados del resultado: solo aparece cada valor de `rating` una vez.

    En Sakila los valores posibles son: `G`, `PG`, `PG-13`, `R`, `NC-17`. Son los estándares de clasificación de la MPAA (sistema de clasificación cinematográfico de EE. UU.).

    Esta consulta es útil para inspeccionar los valores únicos de una columna antes de filtrar por ella.

---

## Ejercicio 6

**Muestra el título y el coste de reposición (`replacement_cost`) de las películas cuyo coste esté entre 10,00 y 20,00 (ambos inclusive). Ordénalas por coste descendente.**

??? success ":white_check_mark: Solución"
    ```sql
    SELECT title, replacement_cost
    FROM film
    WHERE replacement_cost BETWEEN 10.00 AND 20.00
    ORDER BY replacement_cost DESC;
    ```

??? note ":pencil: Explicación"
    `BETWEEN 10.00 AND 20.00` es equivalente a `replacement_cost >= 10.00 AND replacement_cost <= 20.00`. Ambos extremos son **inclusivos**.

    Es una forma más legible de expresar rangos numéricos o de fechas.

    `BETWEEN` también funciona con cadenas (orden lexicográfico) y fechas.

---

## Ejercicio 7

**Obtén los títulos de las películas que tienen clasificación 'PG' o 'PG-13', ordenados alfabéticamente.**

??? success ":white_check_mark: Solución"
    ```sql
    SELECT title, rating
    FROM film
    WHERE rating IN ('PG', 'PG-13')
    ORDER BY title ASC;
    ```

??? note ":pencil: Explicación"
    `IN ('PG', 'PG-13')` es equivalente a `rating = 'PG' OR rating = 'PG-13'`, pero es más conciso y legible.

    El operador `IN` acepta una lista de valores separados por comas entre paréntesis.

    Para listas largas, `IN` es claramente preferible a encadenar múltiples `OR`.
