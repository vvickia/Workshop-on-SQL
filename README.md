# Workshop-on-SQL
В рамках курса по базам данных были выполнены некоторые SQL-запросы на локальном студенческом сервере. Примеры:  

1. Сколько всего планет в каталоге?  
```SQL
SELECT COUNT(*) FROM planetdata;
```  

2. У скольки звезд есть хотя бы одна планета?  
```SQL
SELECT COUNT(DISTINCT pl_hostname) FROM planetdata;
```  

3. Найти все планеты, у которых не известен эксцентриситет орбиты. Сколько их в каталоге?  
```SQL
SELECT COUNT pl_hostname, pl_orbeccen FROM planetdata WHERE pl_orbeccen IS null;
```  

4. Сколько планет было открыто в 2019 году?  
```SQL
SELECT COUNT(*) FROM planetdata WHERE pl_discyear = 2019;
```  

5. Чему равна максимальная масса среди всех планет? Найти планету с такой массой.  
```SQL
SELECT MAX(pl_masse) FROM planetdata;
SELECT pl_name, pl_masse FROM planetdata WHERE pl_masse = 239000;
```  

6. Чему равна минимальная большая полуось орбиты среди всех планет? Найти планету с такой большой полуосью орбиты.  
```SQL
SELECT MIN(pl_orbsmax) FROM planetdata;
SELECT pl_name, pl_orbsmax FROM planetdata WHERE pl_orbsmax = 0.0044;
```  

7. Сколько планет было открыто разными методами? Ответ представить в виде таблицы: Метод | Число планет. Отсортировать в порядке убывания числа планет.  
```SQL
SELECT pl_discmethod, COUNT(*) FROM planetdata GROUP BY pl_discmethod ORDER BY 2;
```  

8. Сколько планет расположено в Южном полушарии небесной сферы?  

```SQL  
SELECT decl FROM planetdata WHERE decl < 0;
SELECT COUNT(*) AS "planets..."FROM planetdata WHERE decl < 0;
```  

9. Сколько звезд с планетами расположено между $\delta = -1$ и $\delta =~38$?  
```SQL  
(SELECT decl FROM planetdata WHERE decl > −1) INTERSECT (SELECT decl FROM planetdata WHERE decl < 38);
SELECT COUNT(*) AS "stars..."FROM planetdata WHERE decl > −1 AND decl < 38;
```  

10. Какие планеты лучше всего наблюдать 21 марта? Видны ли они из Санкт-Петербурга?  
```SQL  
(SELECT pl_name FROM planetdata WHERE RA > 175) INTERSECT (SELECT pl_name FROM planetdata WHERE RA < 185) INTERSECT (SELECT pl_name FROM planetdata WHERE decl > −30);
```  
	
11. Сколько планет открыто у звезд спектрального класса F и G?  
```SQL  
SELECT COUNT(*) FROM planetdata WHERE pl_name IN ((SELECT DISTINCT pl_name FROM planetdata WHERE st_spectype IN ("F")) UNION (SELECT DISTINCT pl_name FROM planetdata WHERE st_spectype IN ("G")));
SELECT COUNT(*) AS "stars..."FROM planetdata WHERE st_spectype LIKE "\F\"OR st_spectype LIKE "\G\";
```  
	
12. Выбрать планеты, которые находятся на расстоянии от 0.7 до 1.5 а.е. от звезд класса G2V. Похожи ли эти планеты на Землю?  
```SQL  
SELECT COUNT(*) AS "planetki..." FROM planetdata WHERE st_spectype = "G2 V" AND pl_orbsmax > 0.7 AND pl_orbsmax < 1.5;
```  
	
13. Каков возраст звезд, у которых проводилось измерение лучевых скоростей? У всех ли он известен?  
```SQL  
SELECT st_age, starname FROM planetdata INNER JOIN solstars ON solstars.starname = pl_hostname;
```  
	
14. Вывести названия и спектральные классы звезд из таблицы *planetdata* и *solstars* для всех тех звезд, у которых хотя бы в одном измерении лучевых скоростей экспозиция составляла 600 с. Для контроля вывести время экспозиции. Совпадают ли спектральные классы в таблице?  
```SQL  
SELECT DISTINCT pl_hostname, sptype FROM planetdata, solstars, velocities WHERE (velocities.exptime = 600 AND velocities.starname = pl_hostname AND solstars.starname = pl_hostname);
```  

15. Вывести звездные величины в полосе B и расстояния для звезд, у которых лучевая скорость от 19 до 24 км/с.  
```SQL  
SELECT DISTINCT bmag, ras FROM planetdata, solstars, velocities WHERE (velocities.radvel > 19 AND velocities.radvel < 24);
```  
	
16. У каких звезд измерялись лучевые скорости после полуночи 15 января 2015 года по UT?  
```SQL  
SELECT DISTINCT starname FROM velocities WHERE epochbgd > 2457037.5;
```  

17. Какая минимальная экспозиция потребовалась для измерения лучевой скорости у звезды HD 216437? Какое при этом получилось соотношение «сигнал–шум»?  
```SQL  
SELECT MIN(exptime) FROM velocities WHERE (starname = "HD 216437") UNION SELECT snr FROM velocities WHERE (exptime = 300);
SELECT snr FROM velocities WHERE exptime = (SELECT MIN(exptime) FROM velocities WHERE starname = "HD 216437") AND starname = "HD 216437";
```  
	
18. Каково расстояние до самой яркой в полосе $B$ звезды, у которой проводились измерения лучевых скоростей? Что это за звезда?  
```SQL  
SELECT starname, MIN(bmag) FROM solstars;
SELECT pl_hostname, st_dist FROM planetdata WHERE pl_hostname = "HD 114729";
SELECT pl_hostname, st_dist FROM planetdata WHERE pl_hostname = (SELECT starname FROM solstars WHERE bmag = (SELECT MIN(bmag) FROM solstars));
```  
	
19. Вывести для всех звезд средние значения измеренных лучевых скоростей.  
```SQL  
SELECT starname, AVG(radvel) FROM velocities GROUP BY starname;
```  
	
20. Узнать из таблицы *velocities*, сколько измерений лучевых скоростей было выполнено для каждой звезды, и убедиться, что в столбце *nummeans* таблицы *solstars* записано именно число измерений для каждой звезды. Вывести результат сопоставления в порядке убывания количества измерений.  
```SQL  
(SELECT n_means FROM solstars ORDER BY 1) EXCEPT (SELECT COUNT(starname) FROM velocities GROUP BY starname ORDER BY 1);
```  
	
21. Каков средний радиус звезд, которые содержатся в таблице *solstars*?  
```SQL  
SELECT AVG(st_rad) FROM planetdata, solstars WHERE pl_hostname = solstars.starname;
```	