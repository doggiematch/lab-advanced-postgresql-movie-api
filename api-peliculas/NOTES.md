# Parte 2:

## Paso 1: Pregunta: Por que usamos LEFT JOIN en lugar de INNER JOIN? Que filas se perderian con INNER JOIN?

Con left join listamos todas las peliculas, completas e incompletas. Con inner join solo listamos aquellas peliculas con alguna coincidencia en las tablas relacionadas. Esto haria que no se mostraran las peliculas con director o genero id que no encontraran una fila correspondiente en directores o generos.

# Parte 3:

## Paso 5: Pregunta: Que peliculas tienen usuarios mas entusiastas que la critica? Y al reves?

Las peliculas con usuarios mas entusiastas que la critica son las que tienen una diferencia positiva: Roma, Get Out, The Dark Knight, Dune, Oppenheimer, Inception y Pulp Fiction. La pelicula en la que la critica es mas entusiasta que los usuarios es Barbie.
| Titulo | Nota editorial | Media usuarios | Num. resenas | Diferencia |
| --- | ---: | ---: | ---: | ---: |
| Roma | 7.7 | 10.00 | 1 | 2.30 |
| Get Out | 7.7 | 9.00 | 1 | 1.30 |
| The Dark Knight | 9.0 | 10.00 | 1 | 1.00 |
| Dune | 8.0 | 8.50 | 2 | 0.50 |
| Dune | 8.0 | 8.50 | 2 | 0.50 |
| Oppenheimer | 8.5 | 9.00 | 1 | 0.50 |
| Oppenheimer | 8.5 | 9.00 | 1 | 0.50 |
| Dune | 8.1 | 8.50 | 2 | 0.40 |
| Inception | 8.8 | 9.00 | 3 | 0.20 |
| Inception | 8.8 | 9.00 | 5 | 0.20 |
| Inception | 8.8 | 9.00 | 3 | 0.20 |
| Pulp Fiction | 8.9 | 9.00 | 1 | 0.10 |
| Blade Runner 2049 | 8.0 | 8.00 | 1 | 0.00 |
| Barbie | 6.9 | 6.50 | 2 | -0.40 |

Resultado: 14 filas.

# Parte 4:

## Paso 7: Directores con trayectoria ascendente

No hay ningun director con una trayectoria estrictamente ascendente, entendida como que cada pelicula tenga una nota mayor que la anterior al ordenarlas por año.

# Parte 8:

## 1. Cuando es contraproducente crear un indice?

Principalmente cuando la tabla recibe muchas escrituras (con insert, update o delete), ya que esto hace que PostgreSQL tenga que actualizar tambien los indices cada vez que cambian los datos, lo que hace que las escrituras sean mas lentas.

## 2. Que diferencia hay entre RANK() y DENSE_RANK()?

Mientras que rank deja huecos en la numeracion, dense rank hace lo contrario.

Ejemplo con las peliculas de ciencia ficcion ordenadas por nota:

peliculas_db=# SELECT
peliculas_db-# p.titulo AS pelicula,
peliculas_db-# p.nota,
peliculas_db-# RANK() OVER (
peliculas_db(# ORDER BY p.nota DESC
peliculas_db(# ) AS rank,
peliculas_db-# DENSE_RANK() OVER (
peliculas_db(# ORDER BY p.nota DESC
peliculas_db(# ) AS dense_rank
peliculas_db-# FROM peliculas p
peliculas_db-# JOIN generos g ON g.id = p.genero_id
peliculas_db-# WHERE g.slug = 'ciencia-ficcion'
peliculas_db-# ORDER BY p.nota DESC, p.titulo;
pelicula | nota | rank | dense_rank
-------------------+------+------+------------
Inception | 8.8 | 1 | 1
Inception | 8.8 | 1 | 1
Inception | 8.8 | 1 | 1
Interstellar | 8.6 | 4 | 2
Interstellar | 8.6 | 4 | 2
Dune | 8.1 | 6 | 3
Blade Runner 2049 | 8.0 | 7 | 4
Blade Runner 2049 | 8.0 | 7 | 4
Blade Runner 2049 | 8.0 | 7 | 4
Dune | 8.0 | 7 | 4
Dune | 8.0 | 7 | 4
Arrival | 7.9 | 12 | 5
Arrival | 7.9 | 12 | 5
Gravity | 7.7 | 14 | 6
Tenet | 7.4 | 15 | 7
(15 filas)

Hay 3 filas con la misma nota (8.8), rank asigna el puesto 1 a las tres, pero luego salta directamente al puesto 4, porque las posiciones 2 y 3 quedan ocupadas por ese empate. Contrariamente, dense rank tambien asigna el puesto 1 a las tres filas de Inception, pero la siguiente nota distinta, 8.6, recibe el puesto 2. No deja huecos, no hay saltos.

## 3. Por que el trigger usa AFTER INSERT OR UPDATE OR DELETE en lugar de BEFORE?

El trigger after quiere guardar una pelicula despues de haber sido creada (insert), modificada/actualizada (update) o borrada (delete). El trigger before lo usariamos si quisieramos actuar antes de guardar el cambio para comprobar, por ejemplo, si los datos son validos, modificar algun valor antes de anadirlo/actualizarlo, etc.

En este caso, no queremos cambiar la pelicula ni validar, solo queremos dejar constancia de la operacion que hemos realizado, es decir, registrar el resultado final de la operacion, no algo que todavia podria cambiar o fallar.
