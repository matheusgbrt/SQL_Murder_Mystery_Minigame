Enquanto scrollava o LinkedIn, vi uma postagem de um joguinho que podia ser resolvido usando SQL. Fiquei intrigado e decidi resolvê-lo.
Para quem quiser, aqui está o link para poder jogar: [link](https://mystery.knightlab.com/)


Vamos começar assim:

Se você quiser ver apenas os resultados finais com as queries completas, aqui vai: Se não, passe desse bloco e siga o roteiro.

```sql
--Gets the murderer name
select 
	morty.name
from
  (select 
	  gfnc_filtered.person_id,
	  gfnc_filtered.name
  from (
	  select 
		  *
	  from 
		  get_fit_now_check_in gfnc
	  inner join 
		  get_fit_now_member gfnm
		  on gfnc.membership_id = gfnm.id
	  where check_in_date = 20180109
  ) as gfnc_filtered
  cross join (
	  select 
		  gfnc2.check_in_time,
		  gfnc2.check_out_time
	  from get_fit_now_check_in gfnc2
	  inner join get_fit_now_member gfnm
		  on gfnc2.membership_id = gfnm.id
	  inner join person p
		  on gfnm.person_id = p.id
	  where 
		  p.name like '%Annabel%' 
		  and p.address_street_name = 'Franklin Ave'
  ) as annabel_times
  where 
	  gfnc_filtered.check_in_time <=annabel_times.check_out_time 
	  and gfnc_filtered.check_out_time >= annabel_times.check_in_time) 
as annabel
inner join
	(select 
	p.*
from 
	get_fit_now_member gfnm
inner join 
	person p
	on gfnm.person_id = p .id
inner join drivers_license  dl
	on p.license_id = dl.id
where
	dl.plate_number like '%H42W%'
	and substr(gfnm.id,0,4) = '48Z'
	and gfnm.membership_status = 'gold')
as morty
on annabel.person_id = morty.id


--Gets the name of the woman that hired the murderer
select 
	p.name
from 
	drivers_license dl
inner join 
	person p on dl.id = p.license_id
inner join 
	income i on p.ssn = i.ssn
inner join
	(select 
		person_id,
		count(*) 
	 from 
		 facebook_event_checkin
	where 
		date between 20171200 and 20180100 
		and event_name = 'SQL Symphony Concert'
	group by 
		person_id
	having 
		count(*) = 3) fbe 
	on p.id = fbe.person_id
where 
	hair_color = 'red' 
	and gender = 'female' 
	and car_make = 'Tesla' 
	and car_model = 'Model S' 
	and height between 65 and 67
```


Podemos começar desenvolvendo os blocos dessas queries.

Vamos primeiro para o assassino.
A primeira dica é que precisamos pegar o relatório da cena do crime. Isso não nos traz dados estruturados, apenas as próximas dicas

```sql
select 
	* 
from 
	crime_scene_report 
where 
	date = 20180115 
	and type = 'murder' 
	and city = 'SQL City'
```

| crime_scene_report |        |                                                                                                                                                                                           |          |
| ------------------ | ------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------- |
| date               | type   | description                                                                                                                                                                               | city     |
| 20180115           | murder | Security footage shows that there were 2 witnesses. The first witness lives at the last house on "Northwestern Dr". The second witness, named Annabel, lives somewhere on "Franklin Ave". | SQL City |

Usando essas dicas, podemos buscar os dados pessoais das testemunhas. Vamos montar ambos ao mesmo tempo.

```sql
SELECT 
    * 
FROM 
    person
WHERE 
    (name LIKE '%Annabel%' AND address_street_name = 'Franklin Ave')
    OR
    (address_street_name = 'Northwestern Dr' AND address_number = (
        SELECT MAX(address_number) 
        FROM person 
        WHERE address_street_name = 'Northwestern Dr'
    ));
```

| person |                |            |                |                     |           |
| ------ | -------------- | ---------- | -------------- | ------------------- | --------- |
| id     | name           | license_id | address_number | address_street_name | ssn       |
| 14887  | Morty Schapiro | 118009     | 4919           | Northwestern Dr     | 111564949 |
| 16371  | Annabel Miller | 490173     | 103            | Franklin Ave        | 318771143 |

As coisas começam a ficar estranhas com subqueries. Não são tão performáticas, porém não temos uma quantia grande de dados a serem buscados, então isso não afetará tanto a performance.
Vamos adicionar as entrevistas das testemunhas trazendo apenas as informações que nos interessam.

```sql
SELECT 
    p.id,p.name,i.transcript
FROM 
    person p
inner join
	interview i
	on p.id = i.person_id
WHERE 
    (name LIKE '%Annabel%' AND address_street_name = 'Franklin Ave')
    OR
    (address_street_name = 'Northwestern Dr' AND address_number = (
        SELECT MAX(address_number) 
        FROM person 
        WHERE address_street_name = 'Northwestern Dr'
    ));

```

| person.id | name           | transcript                                                                                                                                                                                                                      |
| --------- | -------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 14887     | Morty Schapiro | I heard a gunshot and then saw a man run out. He had a "Get Fit Now Gym" bag. The membership number on the bag started with "48Z". Only gold members have those bags. The man got into a car with a plate that included "H42W". |
| 16371     | Annabel Miller | I saw the murder happen, and I recognized the killer from my gym when I was working out last week on January the 9th.                                                                                                           |

Agora temos algumas pistas. Novamente não estão estruturadas, precisamos montar uma nova query.
Vamos utilizar a porção do Morty primeiro.


```sql
select 
	p.*
from 
	get_fit_now_member gfnm
inner join 
	person p
	on gfnm.person_id = p .id
inner join drivers_license  dl
	on p.license_id = dl.id
where
	dl.plate_number like '%H42W%'
	and substr(gfnm.id,0,4) = '48Z'
	and gfnm.membership_status = 'gold'
```

| person |                |            |                |                       |           |
| ------ | -------------- | ---------- | -------------- | --------------------- | --------- |
| id     | name           | license_id | address_number | address_street_name   | ssn       |
| 67318  | Jeremy Bowers  | 423327     | 530            | Washington Pl, Apt 3A | 871539279 |

Encontramos apenas uma pessoa que se enquadra na descrição do Morty.
Vamos utilizar a descrição da Annabel agora. Ela é mais vaga quanto a situação, mas ainda podemos utilizar.
Vamos capturar quem estava presente na academia no momento que a Annabel disse que estava lá.

```sql
select 
	gfnc_filtered.person_id,
	gfnc_filtered.name
from (
	select 
		*
	from 
		get_fit_now_check_in gfnc
	inner join 
		get_fit_now_member gfnm
		on gfnc.membership_id = gfnm.id
	where check_in_date = 20180109
) as gfnc_filtered
cross join (
	select 
		gfnc2.check_in_time,
		gfnc2.check_out_time
	from get_fit_now_check_in gfnc2
	inner join get_fit_now_member gfnm
		on gfnc2.membership_id = gfnm.id
	inner join person p
		on gfnm.person_id = p.id
	where 
		p.name like '%Annabel%' 
		and p.address_street_name = 'Franklin Ave'
) as annabel_times
where 
	gfnc_filtered.check_in_time <=annabel_times.check_out_time 
	and gfnc_filtered.check_out_time >= annabel_times.check_in_time
```

| person_id | name           |
| --------- | -------------- |
| 67318     | Jeremy Bowers  |
| 28819     | Joe Germuska   |
| 16371     | Annabel Miller |

Que curioso, Jeremy estava lá.
Vamos juntar ambas as queries para termos a cruza de dados.

```sql
select 
	morty.name
from
  (select 
	  gfnc_filtered.person_id,
	  gfnc_filtered.name
  from (
	  select 
		  *
	  from 
		  get_fit_now_check_in gfnc
	  inner join 
		  get_fit_now_member gfnm
		  on gfnc.membership_id = gfnm.id
	  where check_in_date = 20180109
  ) as gfnc_filtered
  cross join (
	  select 
		  gfnc2.check_in_time,
		  gfnc2.check_out_time
	  from get_fit_now_check_in gfnc2
	  inner join get_fit_now_member gfnm
		  on gfnc2.membership_id = gfnm.id
	  inner join person p
		  on gfnm.person_id = p.id
	  where 
		  p.name like '%Annabel%' 
		  and p.address_street_name = 'Franklin Ave'
  ) as annabel_times
  where 
	  gfnc_filtered.check_in_time <=annabel_times.check_out_time 
	  and gfnc_filtered.check_out_time >= annabel_times.check_in_time) 
as annabel
inner join
	(select 
	p.*
from 
	get_fit_now_member gfnm
inner join 
	person p
	on gfnm.person_id = p .id
inner join drivers_license  dl
	on p.license_id = dl.id
where
	dl.plate_number like '%H42W%'
	and substr(gfnm.id,0,4) = '48Z'
	and gfnm.membership_status = 'gold')
as morty
on annabel.person_id = morty.id
inner join
	interview
	on morty.id = interview.person_id
```

Era evidente que o único que se encaixava com os critérios era Jeremy Bowers. Vamos adicionar a entrevista dele no resultado.

| name          | person.id | transcript                                                                                                                                                                                                                                       |
| ------------- | --------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| Jeremy Bowers | 67318     | I was hired by a woman with a lot of money. I don't know her name but I know she's around 5'5" (65") or 5'7" (67"). She has red hair and she drives a Tesla Model S. I know that she attended the SQL Symphony Concert 3 times in December 2017. |

Olha! Jeremy foi contratado para esse trabalho. Encontramos o assassino, mas quem é o mandante?
Utilizando a descrição de Jeremy, vamos pegar as CNHs para ver quem se encaixa no perfil.

```sql
select 
	* 
from 
	drivers_license 
where 
	hair_color = 'red' 
	and gender = 'female' 
	and car_make = 'Tesla'
	and car_model = 'Model S' 
	and height between 65 and 67
```

| drivers_license |     |        |           |            |        |              |          |           |
| --------------- | --- | ------ | --------- | ---------- | ------ | ------------ | -------- | --------- |
| id              | age | height | eye_color | hair_color | gender | plate_number | car_make | car_model |
| 202298          | 68  | 66     | green     | red        | female | 500123       | Tesla    | Model S   |
| 291182          | 65  | 66     | blue      | red        | female | 08CM64       | Tesla    | Model S   |
| 918773          | 48  | 65     | black     | red        | female | 917UU3       | Tesla    | Model S   |

Encontramos três mulheres, vamos adicionar os dados pessoais delas no resultado.
```sql
select 
	p.* 
from 
	drivers_license dl
inner join 
	person p 
	on dl.id = p.license_id
where 
	hair_color = 'red' 
	and gender = 'female' 
	and car_make = 'Tesla' 
	and car_model = 'Model S' 
	and height between 65 and 67
```

| persons |                  |            |                |                     |           |
| ------- | ---------------- | ---------- | -------------- | ------------------- | --------- |
| id      | name             | license_id | address_number | address_street_name | ssn       |
| 78881   | Red Korb         | 918773     | 107            | Camerata Dr         | 961388910 |
| 90700   | Regina George    | 291182     | 332            | Maple Ave           | 337169072 |
| 99716   | Miranda Priestly | 202298     | 1883           | Golden Ave          | 987756388 |

Agora vamos adicionar a renda anual delas.
```sql
select 
	p.*,
	i.annual_income 
from 
	drivers_license dl
inner join 
	person p 
	on dl.id = p.license_id
left join 
	income i 
	on p.ssn = i.ssn
where 
	hair_color = 'red' 
	and gender = 'female' 
	and car_make = 'Tesla' 
	and car_model = 'Model S' 
	and height between 65 and 67
```

| persons | + income         |            |                |                     |           |               |
| ------- | ---------------- | ---------- | -------------- | ------------------- | --------- | ------------- |
| id      | name             | license_id | address_number | address_street_name | ssn       | annual_income |
| 78881   | Red Korb         | 918773     | 107            | Camerata Dr         | 961388910 | 278000        |
| 90700   | Regina George    | 291182     | 332            | Maple Ave           | 337169072 | null          |
| 99716   | Miranda Priestly | 202298     | 1883           | Golden Ave          | 987756388 | 310000        |

Como podemos ver, Regina não possui renda anual, portanto ela não se encaixa na descrição.
Resta apenas uma pista para descobrirmos quem foi a mandante. Vamos buscar nos eventos do Facebook quem foi no SQL Symphony Concert três vezes no mês de dezembro de 2017.

```sql
select 
	person_id,
	count(*) 
from 
	facebook_event_checkin
where 
	date between 20171200 and 20180100 
	and event_name = 'SQL Symphony Concert'
group by 
	person_id
having 
	count(*) = 3

```

| person_id | count(*) |
| --------- | -------- |
| 24556     | 3        |
| 99716     | 3        |

Encontramos duas pessoas, vamos adicionar esse resultado à query anterior.

```sql
select 
	p.*,
	i.annual_income 
from 
	drivers_license dl
inner join 
	person p on dl.id = p.license_id
inner join 
	income i on p.ssn = i.ssn
inner join
	(select 
		person_id,
		count(*) 
	 from 
		 facebook_event_checkin
	where 
		date between 20171200 and 20180100 
		and event_name = 'SQL Symphony Concert'
	group by 
		person_id
	having 
		count(*) = 3) fbe 
	on p.id = fbe.person_id
where 
	hair_color = 'red' 
	and gender = 'female' 
	and car_make = 'Tesla' 
	and car_model = 'Model S' 
	and height between 65 and 67
```

| final query result |                  |            |            |        |          |           |        |               |       |
| ------------------ | ---------------- | ---------- | ---------- | ------ | -------- | --------- | ------ | ------------- | ----- |
| person.id          | name             | license_id | hair_color | gender | car_make | car_model | height | annual_income | count |
| 99716              | Miranda Priestly | 202298     | red        | female | Tesla    | Model S   | 66     | 310000        | 3     |

A única pessoa que se enquadra na descrição é Miranda Priestly. Chegamos ao resultado final composto em apenas uma query!
