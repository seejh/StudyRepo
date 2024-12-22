LIMIT, FETCH, SELECT TOP <br/>
내용 요약<br/>
LIMIT을 사용하여 쿼리에서 반한되는 행 수를 제한, OFFSET으로 행을 건너뜀. <br/>
SQL 표준이 아니므로 사용하는 DB 시스템에 따라 달라짐(LIMIT, FETCH, SELECT TOP) <br/>
여기서는 LIMIT을 기준으로 설명, 다른 절들은 따로 알아봐야한다.(사용법이 크게 다르진 않다.)<br/>


기본 사용법<br/>
<table>
<td>
<pre lang="sql">
SELECT column
FROM table
ORDER BY first_name
LIMIT row_count OFFSET offset;
</pre>
</td>
<td>
LIMIT - 쿼리에서 반환되는 행의 수를 결정<br/>
OFFSET - 행 반환을 시작하기 전에 건너뛰는 행, 생략 가능한 절<br/>
이 절을 사용할 때는 결과를 보장하기 위해 ORDER BY를 사용하는 것이 중요하다.<br/>
이 절(LIMIT, OFFSET)은 SQL 표준이 아님. 예를 들어 SQL SERVER에서는 SELECT TOP으로 사용.
</td>
</table>

<hr/>

예제1) 급여가 높은 상위 5명 직원 조회 <br/>
<table>
<td>
<pre lang="sql">
SELECT employee_id, first_name, salary
FROM employees
ORDER BY salary DESC
LIMIT 5;
</pre>
</td>
<td><img src="https://github.com/user-attachments/assets/1c2adbd7-e6be-423a-bfa8-ec602951e7bf"/></td>
</table>
이 쿼리는 급여를 기준으로 직원을 내림차순으로 정렬한 다음 5개의 행으로 제한해서 가져온다.<br/><br/>

예제2) 받는 급여 수준이 상위에서 두 번째인 직원 조회 <br/>
<table>
<td>
<pre lang="sql">
SELECT employee_id, first_name, salary
FROM employees
ORDER BY salary DESC
LIMIT 1 OFFSET 1;
</pre>
</td>
<td><img src="https://github.com/user-attachments/assets/3334c15d-1676-493d-abca-cb765c1e8dd1"/></td>
</table>
이 쿼리는 직원을 급여(salary)를 기준으로 내림차순 정렬 후 결과 집합에서 두 번째 행을 가져온다.<br/>
모든 직원의 급여가 다르다는 가정 하에 작성한 코드로, 최상위 급여 수준을 가진 인간이 2명이면 원하는 결과 값이 도출되지 않는다.<br/>
또한 2번째 급여 수준을 가진 직원이 두 명 이상 있는 경우 첫 번재 직원만 반환한다.<br/><br/>

예제3) 예제2번의 문제 해결 <br/>
1 상위에서 두 번째 수준의 급여 조회<br/>
2 해당 급여 수준을 가진 인간들을 조회<br/>
이 두 과정을 합쳐준다.(후에 배울 서브쿼리 활용)<br/>

1. 상위에서 두 번째 수준의 급여를 조회
<table>
<td>
<pre lang="sql">
SELECT DISTINCT salary
FROM employees
ORDER BY salary DESC
LIMIT 1, 1;
</pre>
</td>
<td><img src="https://github.com/user-attachments/assets/97db5135-afb4-43af-b74c-024cbca19c51"/></td>
</table>

2. 해당 급여 수준을 가진 인간들 조회
<table>
<td>
<pre lang="sql">
SELECT employee_id, first_name, salary
FROM employees
WHERE salary = 17000;
</pre>
</td>
<td><img src="https://github.com/user-attachments/assets/6f6ba6f8-a809-419c-9f4e-635d1fd4b2a2"/></td>
</table>

이 두 쿼리를 하나로 합침
<table>
<td>
<pre lang="sql">
SELECT employee_id, first_name, salary
FROM employees
WHERE salary = (SELECT DISTINCT salary FROM employees ORDER BY salary DESC LIMIT 1, 1);
</pre>
</td>
<td><img src="https://github.com/user-attachments/assets/6f6ba6f8-a809-419c-9f4e-635d1fd4b2a2"/></td>
</table>

