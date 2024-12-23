MERGE<br/>

내용 요약<br/>
조건을 비교해서 해당 조건에 맞는 데이터가 없으면 데이터를 삽입하고 
조건에 맞는 데이터가 있으면 기존의 데이터를 수정하는 문장이다.<br/>
쓰이는 경우 : 두 개의 테이블의 내용을 비교하여 한쪽에서 다른쪽에 가져오고 싶은 데이터가 있을 때 사용한다.<br/>
A테이블과 B테이블이 있을 때 조건에 해당하는 데이터가 있을 경우 수정, 데이터가 없을 경우 입력<br/>


기본 사용법<br/>
<table>
<td>
<pre lang="sql">
MERGE target_table USING source_table
ON merge_condition
WHEN MATCHED THEN update_statement
WHEN NOT MATCHED THEN insert_statement
WHEN NOT MATCHED BY SOURCE THEN DELETE;
</pre>
</td>
<td>

</td>
</table>

<hr/>

예제1) <br/>
