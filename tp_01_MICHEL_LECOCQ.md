# Revision

## Exercice nouvelle colonne

1. Ajoutez la colonne salary, salaire annuel, dans la table pilots, définissez son type. Vous donnerez la requête SQL pour modifier la table. Puis faites une requête pour ajouter les salaires respectifs suivants :

```text
+--------+--------+
| name   | salary |
+--------+--------+
| Alan   |  2000  |
| Tom    |  1500  |
| Yi     |  1500  |
| Sophie |  2000  |
| Albert |  2000  |
| Yan    |  1500  |
| Benoit |  2000  |
| Jhon   |  3000  |
| Pierre |  3000  |
+--------+--------+
```

COMMANDE:

 			AJOUT DE COLONNE
```sql
ALTER TABLE `pilots`
ADD salary DECIMAL(8,2) UNSIGNED ;
```
 			AJOUT DES SALAIRES
```sql
UPDATE `pilots`
SET `salary`= (CASE 
	WHEN `name` IN ("Alan","Sophie","Albert","Benoit") THEN "2000"
    WHEN `name` IN ("Tom","Yi","Yan") THEN "1500"
    WHEN `name` IN ("Jhon","Pierre") THEN "3000"
END);
```

## Exercice 1

1. Quel est le salaire moyen.


COMMANDE : 
```sql 
SELECT ROUND(AVG(salary),2) AS `average_salary` FROM pilots;
```
RÉSULTAT : 
```text 
| # average_salary |
|------------------|
| '2055.56'      |

```

2. Calculez le salaire moyen par compagnie.

COMMANDE:
```sql
SELECT compagny, ROUND(AVG(salary),2) AS `average_salary` FROM pilots 
GROUP BY compagny;
```
RESULTAT:

```text
| # compagny | average_salary |
|------------|----------------|
| NULL       | NULL           |
| 'AUS'      | '2000.00'    |
| 'CHI'      | '3000.00'    |
| 'FRE1'     | '2250.00'    |
| 'SIN'      | '1500.00'    |
```


3. Quels sont les pilots qui sont au-dessus du salaire moyen.
COMMANDE:
```sql
SELECT `name` FROM pilots 
WHERE salary > (SELECT AVG(salary) AS `average_salary` FROM pilots);
```
RESULTAT:
```text
| # name   |
|----------|
| 'Jhon'   |
| 'Pierre' |
```
4. Calculez l'étendu des salaires.
COMMANDE:
```sql
SELECT 	MIN(salary) AS `min_salary`,
		MAX(salary) AS `max_salary`,
        MAX(salary) - MIN(salary) AS salary_range
FROM pilots;
```
RESULTAT:
```text
| # min_salary | max_salary | salary_range |
|--------------|------------|--------------|
| '1500'       | '3000'     | '1500'       |

```
5. Quels sont les noms des compagnies qui payent leurs pilotes au-dessus de la moyenne ?

VARIANTE 1 Au moins un pilote de la compagnie a un salaire au dessus de la moyenne 
    
COMMANDE:
```sql
SELECT `name` FROM compagnies 
WHERE comp IN (
	SELECT DISTINCT compagny FROM pilots
    WHERE salary > (SELECT AVG(salary) FROM pilots)
);
```
RESULTAT:
```text
|#   name  |
|-------------|
|'CHINA Air'  |
|'Air  France'|
```

VARIANTE 2 Tous les pilotes de la compagnie ont un salaire au dessus de la moyenne

COMMANDE:
```sql
SELECT `name` FROM compagnies 
WHERE comp IN (
	SELECT compagny
    FROM pilots 
    GROUP BY compagny 
    HAVING MIN(salary) > (SELECT AVG(salary) FROM pilots)
);
``` 
RESULTAT:
```text
|#   name  |
|-------------|
|'CHINA Air'  |

```

VARIANTE 3 La moyenne des salaires de la compagnie est supérieur a la moyenne générale
COMMANDE:
```sql
SELECT `name` FROM compagnies 
WHERE comp IN (
	SELECT compagny FROM pilots
    GROUP BY compagny
    HAVING AVG(salary) > (SELECT AVG(salary) FROM pilots)
);
```


RESULTAT:
```text
|#   name  |
|-------------|
|'CHINA Air'  |
|'Air  France'|
```
6. Quels sont les pilotes qui par compagnie dépasse(nt) le salaire moyen ?
(les pilotes dont le salaire est supérieur a la moyenne de leur compagnie)
COMMANDE:
```sql
SELECT `name`,salary,compagny,avg_comagny_salary FROM pilots
JOIN (SELECT compagny AS c_compagny , ROUND(AVG(salary),2) AS avg_comagny_salary FROM pilots GROUP BY compagny) AS avg_compagny
ON compagny=c_compagny
HAVING salary>avg_comagny_salary;
```
RESULTAT (j’ai ajouté la compagnie et la moyenne de salaire de la compagnie):
```text
| # name | salary    | compagny | avg_comagny_salary |
|--------|-----------|----------|--------------------|
| 'Jhon' | '3000.00' | 'FRE1'   | '2250.00'          |

```
## Exercice 2

1. Faites une requête qui diminue de 40% le salaire des pilotes de la compagnie AUS.
COMMANDE:
```sql
UPDATE pilots
SET salary= 0.6 * salary
WHERE compagny="AUS";

SELECT `name`,salary FROM pilots
WHERE compagny="AUS";
```
RESULTAT:
```text
| # name   | salary    |
|----------|-----------|
| 'Alan'   | '1200.00' |
| 'Sophie' | '1200.00' |
| 'Albert' | '1200.00' |
| 'Benoit' | '1200.00' |

```

2. Vérifiez que les salaires des pilotes australiens sont tous inférieurs aux autres salaires des pilotes des autres compagnies.
COMMANDE:
```sql
SELECT `name`,compagny FROM pilots 
WHERE compagny = "AUS"
AND salary < ALL(SELECT salary FROM pilots 
	WHERE compagny <> "AUS"
);
```

RESULTAT:
```text
| # name   | compagny |
|----------|----------|
| 'Alan'   | 'AUS'    |
| 'Sophie' | 'AUS'    |
| 'Albert' | 'AUS'    |
| 'Benoit' | 'AUS'    |

```
## Exercices de recherche

Pour chaque question ci-dessous créez la requête :

1. On aimerait savoir quels sont les types d'avions en commun que la compagnie AUS et FRE1 exploitent.

Indications : l'intersection de deux ensembles en MySQL s'implémente comme suit :

```sql
SELECT DISTINCT value FROM `table1`
WHERE value IN (
  SELECT value 
  FROM `table2`
);
```

COMMANDE:
```sql
SELECT DISTINCT plane FROM pilots 
WHERE compagny="AUS" 
AND plane IN (
	SELECT plane FROM pilots
    WHERE compagny="FRE1"
);
```
RESULTAT
```text
| # plane |
|---------|
| 'A320'  |

```


2. Quels sont les types d'avion que ces deux compagnies AUS et FRE1 exploitent (c'est l'UNION ici) ?

Indications : Pensez à utiliser l'opérateur UNION.

Sans utiliser l’union:

COMMANDE:
```sql
SELECT DISTINCT plane FROM pilots WHERE compagny IN ("AUS","FRE1");
```

en utilisant l’union
COMMANDE:
```sql
SELECT DISTINCT plane FROM pilots WHERE compagny ="AUS"
UNION
SELECT DISTINCT plane FROM pilots WHERE compagny = "FRE1";
```

RESULTAT
```text
| # plane |
|---------|
| 'A320'  |
| 'A380'  |
```