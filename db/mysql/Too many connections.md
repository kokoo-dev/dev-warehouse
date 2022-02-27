로컬에서 서버를 실행하다 발생한 Too many connections 에러가 발생했고 이에 대한 이유 및 해결방법을 설명합니다. <br>
일단 해당 에러는 max_connections로 설정되어 있는 값보다 더 많은 connection 연결을 시도해서 발생하는 에러입니다. <br>
제가 사용한 DB의 경우 Aurora Mysql의 max_connections가 200으로 설정되어있었고
어플리케이션에서 설정한 커넥션풀 수랑 개발자 인원 수, 구동중인 개발 및 테스트 서버 대수를 생각해 봤을때 전혀 부족한 값은 아니었습니다. <br>
프로세스 리스트를 확인하여 kill하거나 max_connextions를 늘릴 수도 있지만 이는 임시방편이라 생각했고 다시 같은 상황이 반복될 것 같았고 DB스펙에 비해 과하게 설정하는 것은 메모리가 부족하게될 수 있습니다.
(Aurora Mysql 경우 인스턴스의 메모리 값을 기준으로 계산식이 세워져 있었습니다.) <br>

~~~sql
show full processlist;
select count(*) from information_schema.processlist;
~~~
두 명령어로 확인해봤을때 사용중인 스레드는 200개 이상, 대부분의 Command상태가 Sleep으로 되어있었습니다. <br>
Sleep의 경우 현재 클라이언트가 새로운 명령을 보내길 대기하고 있는 상태입니다. <br>
이렇게 쌓인 Sleep상태의 connection들이 대기시간만큼 명령을 기다리고 있었고 계속 누적되어가고 있었던 것입니다. <br>
그래서 interactive_timeout, wait_timeout 설정을 변경해주어야 합니다. <br>
interactive_timeout는 활동중인 커넥션이 닫히기까지 서버가 대기하는 시간, wait_timeout은 활동하지 않는 커넥션을 끊을때까지 서버가 대기하는 시간이며 <br>
둘의 기본 값은 28800으로 8시간입니다. 권장값은 3600, 즉 1시간입니다. <br>

> my.cnf
~~~cnf
[mysqld]
interactive_timeout = 3600
wait_timeout = 3600
~~~

my.cnf를 수정하여 해당 값을 변경해주거나 RDS의 경우 파라미터 그룹을 수정해주면 적용유형이 dynamic이기에 서버 재기동 없이 변경값을 적용할 수 있습니다. <br>
수정 후 connection들은 1시간 안에 해제가 되겠지만 이전에 쌓여있던 것들은 이전 설정의 영향을 받아 계속 남아있게 되고 당장 필요한 연결을 못할 수 있습니다. <br>
이 경우 kill명령어를 입력해 직접 종료해줍니다. <br>

~~~sql
kill {processlist_id}
~~~
기본적으로는 processlist를 조회해서 나온 아이디값을 지정해서 하나씩 kill해주면 되나,
RDS를 운영할 경우 You are not owner of thread 에러를 만나게 됩니다. <br>
이는 접속한 계정이 아닌 다른 계정으로 kill하려 할 때 나오며 이 때를 대비해 RDS에서는 프로시저를 만들어 놨습니다. <br>

~~~sql
CALL mysql.rds_kill({processlist_id});
CALL mysql.rds_kill_query({processlist_id});
~~~
두 명령어 중 골라서 사용하면 됩니다.
