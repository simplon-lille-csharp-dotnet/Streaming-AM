
# Streaming-AM

## Installation

Installer PostGreSQL : https://www.postgresql.org/download/  
Créer une db nommée "streaming-am"  
Dans cette base de données, ouvrir le fichier (alt+o) "initialisation-db.sql"
## Requètes

les titres et dates de sortie des films du plus récent au plus ancien :

```bash
SELECT title, release_year  
FROM public.movie  
ORDER BY release_year DESC 
```

les noms, prénoms et âges des acteurs/actrices de plus de 30 ans dans l'ordre alphabétique (prénom d'abord, puis nom)
```bash
SELECT firstname, lastname, EXTRACT(YEAR FROM AGE(current_date, birthdate)) AS age
	FROM public.actor
	WHERE EXTRACT(YEAR FROM AGE(current_date, birthdate)) < 30
	ORDER BY firstname ASC,
	         lastname ASC;
```

la liste des acteurs/actrices principaux pour un film donné
```bash
SELECT 
    actor.firstname,
    actor.lastname,
	movie.title
FROM 
    public.actor
JOIN 
    public.actor_movie ON actor.id_actor = actor_movie.id_actor
JOIN 
    public.movie ON actor_movie.id_movie = movie.id_movie
WHERE 
	movie.title = 'The Brave Little Toaster Goes to Mars';
```

la liste des films pour un acteur/actrice donné
```bash
SELECT 
    actor.firstname,
    actor.lastname,
	movie.title
FROM 
    public.actor
JOIN 
    public.actor_movie ON actor.id_actor = actor_movie.id_actor
JOIN 
    public.movie ON actor_movie.id_movie = movie.id_movie
WHERE 
	actor.lastname = 'Pinar';
```

ajouter un film
```bash
INSERT INTO public.movie(title, duration, release_year, id_director)
VALUES ('La Classe américaine : Le Grand Détournement', 70.00, '1993-12-31', 69);
```

ajouter un acteur/actrice
```bash
INSERT INTO public.actor(firstname, lastname, birthdate)
VALUES ('Dustin', 'Hoffman', '1937-08-08');
```

modifier un film
```bash
UPDATE public.movie
	SET title = 'Rage of the Elephant King: Babar''s Brutal Odyssey'
	WHERE title = 'Babar The Movie';
```

supprimer un acteur/actrice
```bash
DELETE FROM public.actor
	WHERE id_actor = 222;
```

afficher les 3 derniers acteurs/actrices ajouté(e)s
```bash
SELECT firstname, lastname, created
	FROM public.actor
	ORDER BY created DESC
	LIMIT 3;
```

## Requètes bonus

La moyenne du nombre de films des acteurs/trices de plus de 50
```bash
SELECT AVG(nb_films) AS moyenne_nb_films
	FROM
	(
	SELECT actor.id_actor, COUNT(actor_movie.id_movie) AS nb_films
	FROM public.actor
	JOIN public.actor_movie ON actor.id_actor = actor_movie.id_actor
	WHERE EXTRACT(YEAR FROM AGE(current_date, actor.birthdate)) >= 50
	GROUP BY actor.id_actor
	);
```

Le réalisateur ayant le plus de film mis en favoris et combien de ses films ont été mis en favoris.
```bash
SELECT director.firstname, director.lastname, COUNT(movie.title) AS nb_film
	FROM public.director
	JOIN public.movie ON director.id_director = movie.id_director
	GROUP BY director.firstname, director.lastname
	ORDER BY COUNT(*) DESC
	LIMIT 1;
```

Les films qui ont plus d'acteurs que la moyenne des acteurs par film.
```bash
SELECT movie.title, COUNT(actor_movie.id_actor) AS nb_actor
FROM public.movie
JOIN public.actor_movie ON movie.id_movie = actor_movie.id_movie
GROUP BY movie.title
HAVING COUNT(actor_movie.id_actor) > (
    SELECT AVG(nb_actors) AS moyenne_nb_actors
    FROM (
        SELECT movie.id_movie, COUNT(actor_movie.id_actor) AS nb_actors
        FROM public.movie
        JOIN public.actor_movie ON actor_movie.id_movie = movie.id_movie
        GROUP BY movie.id_movie
    ) 
)
ORDER BY nb_actor DESC
```
Écrivez un script de transaction qui ajoute un nouveau film, puis l'ajoute aux films favoris d'un utilisateur spécifique, en s'assurant que les deux opérations réussissent ou échouent ensemble. (Astuce : Utilisez BEGIN TRANSACTION, COMMIT, et ROLLBACK)
```bash
DO $$
	DECLARE
		movieInsertedCount int;
BEGIN
	INSERT INTO public.movie (title, duration, release_year, id_director)
	VALUES ('Rambo XIIII', 119, '2019-07-23', 65);

	INSERT INTO public.favorite_movie (id_user,id_movie) 
	VALUES (1,LASTVAL());

	select COUNT(*) INTO movieInsertedCount from public.favorite_movie where id_movie =lastVal();

	IF movieInsertedCount = 0 THEN
		ROLLBACK;
	ELSE
		COMMIT;
	END IF;
END$$;
```



## 🔗 Links

[![linkedin](https://img.shields.io/badge/linkedin-0A66C2?style=for-the-badge&logo=linkedin&logoColor=white)](https://www.linkedin.com/in/alexandre-merlin-82a395a8/)


