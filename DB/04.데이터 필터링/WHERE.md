WHERE<br/>

내용 요약<br/>
WHERE 절을 사용하여 행을 필터링

기본 사용법<br/>
<table>
<td>
<pre lang="sql">
SELECT column
FROM table
WHERE condition;
</pre>
</td>
<td>
WHERE 절은 FROM 절 바로 뒤에 온다. 이 절에는 테이블의 각 행을 평가하는 하나 이상의 논리식이 포함된다. <br/>
논리식으로 행을 평가했을 때 true면 결과 집합에 포함되고 아니라면(FALSE, UNKNOWN) 제외된다.<br/>
SQL에는 TRUE, FALSE, UNKNOWN의 세 가지 논리값이 있다. FALSE, NULL로 평가되면 행이 반환되지 않는다.<br/>
논리식에 사용되는 연산자(=, !=, <, >, <=, >=), 앞에서 부터 같음, 다름, 미만, 보다 큼, 작거나 같음, 크거나 같음<br/>
SELECT 명령문 외에도 UPDATE와 DELETE 명령문에서도 사용하여 업데이트하거나 삭제할 행을 지정할 수 있다.<br/>
</td>
</table>

식에서 사용하는 리터럴 값은 분류(숫자, 문자, 날짜 및 시간)에 따라 사용 형식이 있다. <br/>
숫자 : 형식 없이 그대로 사용 - 예) 100, 30.5 <br/>
문자 : 작은 따옴표 또는 큰 따옴표로 묶어서 사용 - 예) '100', 'John' <br/>
날짜 : DB에 지정된 형식 사용, DB마다 다름 - 예) 'yyyy-mm-dd' <br/>
시간 : DB에 지정된 형식 사용, DB마다 다름 - 예) 'HH:MM:SS' <br/>

<hr/>

예제1) WHERE 절에서 숫자 비교<br/>
<table>
<td>
<pre lang="sql">
SELECT first_name, salary
FROM employees
WHERE salary > 14000
ORDER BY salary DESC;
</pre>
</td>
<td>이 쿼리는 급여가 14000보다 큰 직원을 찾고 급여를 기준으로 결과 집합을 내림차순으로 정렬한다.</td>
</table>

<table>
<td>
<pre lang="sql">
SELECT first_name
FROM employees
WHERE department_id = 5
ORDER BY first_name;
</pre>
</td>
<td>이 쿼리는 5번 부서에 속한 사람을 찾고 first_name을 기준으로 결과 집합을 오름차순 정렬한다.</td>
</table>

예제2) WHERE 절에서 문자 비교<br/>
<table>
<td>
<pre lang="sql">
SELECT first_name, last_name
FROM employees
WHERE last_name = 'Chen';
</pre>
</td>
<td>SQL은 대소문자를 구분하지 않지만 비교의 값에 관해서는 대소문자를 구분한다. 이 쿼리에서는 last_name이 Chen인 사람을 찾는 것이며 chen은 조회되지 않는다.</td>
</table>

예제3) WHERE 절에서 날짜 비교<br/>
<table>
<td>
<pre lang="sql">
SELECT
FROM employees
WHERE hire_date > '1999-01-01'
ORDER BY hire_date DESC;
</pre>
</td>
<td>
이 쿼리는 1999년 1월 1일 이후에 회사에 합류한 직원들을 조회하는 쿼리이다.
</td>
</table>

