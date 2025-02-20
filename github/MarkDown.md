<br/>
<hr/>

# 코드블럭
특정 코드 조각을 구분하고 강조하기 위해 사용하는 방법 <br/>
(~~~) 사이에 코드를 집어 넣으면 되고 특정 언어를 명시할 수 있다. <br/>

```c++
#include <iostream>

int main() {
  std::cout << "hi" << std::endl;
}
```

```
~~~c++
#include <iostream>

int main() {
  std::cout << "hi" << std::endl;
}
~~~
```

<br/><br/><br/><br/>
<hr/>

# 테이블

### 생성
태그를 사용하여 테이블 골격을 만들고 속성을 사용해서 꾸며준다. <br/>

<table>
<td>태그</td>
<td>table, th, tr, td</td>
<tr>
<td>속성</td>
<td>border, bordercolor, width, height, align, valign, bgcolor, colspan, rowspan</td>
</tr>
</table>


<table>
  <tr>
    <th>행1 열1 헤더</th>
    <th>행1 열2 헤더</th>
  </tr>
  <tr>
    <td>행2 열1 </td>
  </tr>
  <tr>
    <td>행3 열1 </td>
    <td>행3 열2 </td>
  </tr>
</table>

```html
<table>
<tr>
  <th>행1 열1 헤더</th>
  <th>행1 열2 헤더</th>
</tr>
<tr>
  <td>행2 열1 </td>
</tr>
<tr>
  <td>행3 열1 </td>
  <td>행3 열2 </td>
</tr>

</table>
```

### 크기 조절 (width, height)
<table width="300" height="80">
<td width="250" height="60">width:250, height:60</td>
<td width="150" height="30">width:150, height:30</td>
</table>

```html
<table width="300" height="80">
<td width="250" height="60">width:250, height:60</td>
<td width="150" height="30">width:150, heith:30</td>
</table>
```

### 정렬
1) 가로 정렬 (align - left, right, center)
<table>
<td width="150" align="left">좌측</td>
<td width="150" align="center">중앙</td>
<td width="150" align="right">우측</td>
</table>

~~~html
<td align="left">좌측</td>
<td align="center">중앙</td>
<td align="right">우측</td>
~~~

2) 세로 정렬 (valign - top, bottom)
<table>
<td height="100" valign="top">위</td>
<td valign="middle">중앙</td>
<td valign="bottom">아래</td>
</table>

```html
<td align="top">위</td>
<td align="middle">중앙</td>
<td align="bottom">아래</td>
```

<br/><br/><br/><br/>
<hr/>

# 이미지

```html
<img src="이미지 url" width="" height="" />
```
