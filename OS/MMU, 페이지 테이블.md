# MMU (Memory Management Unit)
* CPU에 속하고 CPU와 물리 메모리(RAM) 사이에서 가상 주소를 물리 주소로 변환해주는 하드웨어 장치.
* 가상 메모리를 사용하기 위해서 필수적임

### 기능
크게 4가지로 나눌 수 있다.
1. 주소 변환(Address Translation)
2. 메모리 보호(Memory Protection)
3. 캐싱 및 메모리 속성 관리
4. TLB(Translation Lookaside Buffer)

# Page Table
가상 메모리 시스템에서 가상 주소(Virtual)를 물리 주소(Physical)로 변환시킬 때 필요한 정보를 저장하는 자료구조이다.
즉, CPU가 메모리에 접근할 때 MMU가 이 테이블을 참고해서 가상 주소를 물리 주소로 바꾸어 실제 물리 메모리에 접근한다.

### 페이징 (Paging)
<table>
  <tr>
    <td>개념</td><td>설명</td><td></td>
  </tr>
  <tr>
    <td>Page</td>
    <td>프로세스의 가상 메모리를 일정 크기로 나눈 단위</td>
    <td>기본 4KB</td>
  </tr>
  <tr></tr>
</table>

### 주소 변환
### PTE (Page Table Entry)


출처 : <br/>
https://embeddedworld.tistory.com/4<br/>
<hr/><br/><br/>
