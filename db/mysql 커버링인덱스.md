<s>커버링 인덱스란 쿼리의 조건을 충족시키는데 필요한 모든 데이터들을 인덱스에서만 추출할 수 있는 인덱스를 의미합니다.
B-Tree 스캔만으로 원하는 데이터를 가져오기에, 불필요한 데이터의 데이터블록을 읽지 않아서 성능이 좋습니다.
<br/>

가장 많이 사용된 예시로 limit를 이용한 페이징 쿼리를 사용하겠습니다. (데이터 100만건) <br/>

테스트용 테이블
~~~sql
CREATE TABLE IF NOT EXISTS `board` (
  `board_no` bigint NOT NULL AUTO_INCREMENT,
  `title` varchar(50) COLLATE utf8mb4_general_ci NOT NULL,
  `content` varchar(50) COLLATE utf8mb4_general_ci NOT NULL,
  `cal1` varchar(50) COLLATE utf8mb4_general_ci NOT NULL,
  `cal2` varchar(50) COLLATE utf8mb4_general_ci NOT NULL,
  `cal3` varchar(50) COLLATE utf8mb4_general_ci NOT NULL,
  `cal4` varchar(50) COLLATE utf8mb4_general_ci NOT NULL,
  `cal5` varchar(50) COLLATE utf8mb4_general_ci NOT NULL,
  `cal6` varchar(50) COLLATE utf8mb4_general_ci NOT NULL,
  `cal7` varchar(50) COLLATE utf8mb4_general_ci NOT NULL,
  `cal8` varchar(50) COLLATE utf8mb4_general_ci NOT NULL,
  `reg_date` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP,
  PRIMARY KEY (`board_no`)
) ENGINE=InnoDB AUTO_INCREMENT=344094 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_general_ci;
~~~

실행계획 및 쿼리 실행 시간을 통해 차이점을 확인할 수 있고, 데이터양이 더 많아지면 차이는 더 날 것입니다.

> 일반 쿼리

> 커버링 인덱스 쿼리
</s>

===2021.09.09===<br/>
사내 DBA분을 통해 확인해본 결과
~~~sql
select *
from board
order by board_no desc
limit 1000000, 10


select b.*
from board b
join (select board_no
    from board
    order by board desc
    limit 1000000, 10
) as sb
on b.board_no = sb.board_no
~~~

위 두 쿼리는 차이가 없다고 했습니다. <br/>
직접 200만개 데이터를 가지고 테스트해본 결과도 크게 차이는 없는것 같았습니다...<br/>
DBA분도 직접 테스트해보고 제가 참고한 블로그 글들도 확인해보셨지만 잘못된 내용이라고 하네여.. <br/>
참고로 테스트는 AWS Aurora mysql 5.7 버전에서 하였습니다. <br/>
버전마다 다른건지... 뭐가 맞는건지..
