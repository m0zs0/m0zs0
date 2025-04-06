```console
1. Feladat
SELECT `letszam` 
  FROM `megye` 
  WHERE `nev`= 'Vas';

SELECT `letszam` AS "A Vas megyeiek létszáma"
  FROM `megye` 
  WHERE `nev`= 'Vas';

SELECT `letszam` AS "A Vas megyeiek létszáma"
  FROM `megye` 
  WHERE `nev`= 'Vas';

3. Feladat: Összekapcsolás WHERE-el
SELECT SUM(aero.letszam) AS "Somogyi részvétel"
  FROM megye AS m, aerob AS aero
  WHERE m.kod = aero.mkod 
    AND nev = 'Somogy';

SELECT SUM(aerob.letszam) AS "Somogyi részvétel"
  FROM megye INNER JOIN aerob ON 
    megye.kod = aerob.mkod
  WHERE nev = 'Somogy';

4. Feladat
SELECT aerob.letszam AS "Zalai egészségesek"
  FROM megye, allapot, aerob
  WHERE megye.kod = aerob.mkod AND aerob.allkod = allapot.kod
  	AND aerob.nem = 1
    AND allapot.nev = "egészséges"
    AND megye.nev = "Zala";

SELECT aerob.letszam AS "Zalai egészségesek"
  FROM (megye INNER JOIN aerob ON megye.kod = aerob.mkod) INNER JOIN allapot ON aerob.allkod = allapot.kod
  WHERE aerob.nem = 1
    AND allapot.nev = "egészséges"
    AND megye.nev = "Zala";

5. Feladat: Allekérdezés
SELECT COUNT(letszam) AS "Hevesnél kevesebb"
  FROM megye 
  	WHERE letszam < (SELECT letszam
                      FROM megye
                      WHERE nev = "Heves" );

6. Feladat
SELECT SUM(aerob.letszam) / megye.letszam * 100  AS "Tanulók aránya Pest megyében"
  FROM megye, aerob
  WHERE megye.kod = aerob.mkod
  	AND megye.nev = "Pest";

SELECT SUM(aerob.letszam) / megye.letszam * 100  AS "Tanulók aránya Pest megyében"
  FROM megye INNER JOIN aerob ON megye.kod = aerob.mkod 
  WHERE megye.nev = "Pest";

7. Feladat
SELECT megye.nev, aerob.letszam AS "Egészséges lányok" 
  FROM megye, aerob, allapot
  WHERE megye.kod = aerob.mkod AND aerob.allkod = allapot.kod
   AND aerob.nem = 0
   AND allapot.nev = "egészséges"
  ORDER BY aerob.letszam DESC;

8. Feladat
SELECT megye.nev, SUM(aerob.letszam) / megye.letszam * 100 AS "Arány"
  FROM megye, aerob
  WHERE megye.kod = aerob.mkod
  GROUP BY megye.kod
  ORDER BY Arány DESC
  LIMIT 1;

9. Feladat
SELECT megye.nev, SUM(aerob.letszam) / megye.letszam AS "Arány"
  FROM megye, aerob, allapot
  WHERE megye.kod = aerob.mkod AND aerob.allkod = allapot.kod
  	AND (allapot.nev = "fejlesztést igényel" OR allapot.nev = "fokozott fejlesztést igényel")
    -- AND NOT allapot.nev = "egészséges" 
  GROUP BY megye.kod
  HAVING Arány > 0.25
  ORDER BY 2;

```
