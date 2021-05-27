## MongoDB 컬렉션 Join 조회
NoSQL 에는 Join을 지원하지 않기에 유사하게 표현할 수 있는 방법을 생각해 봤습니다. <br/>
참조 방식과 임베디드 방식 두 방식으로 해보았고, <br/>
아래 설명드릴 예제는 개인적인 생각으로 보다 더 나은 방법이 있을 수 있습니다.
<br/><br/><br/>

### Team, Member (참조) / Team_member (임베디드) 컬렉션 생성
> ex) team collection
~~~json
{
  "id": 1,
  "teamName": "A_TEAM",
  "event": "Soccer",
  "members": [
      1
  ]
},
{
  "id": 2,
  "teamName": "B_TEAM",
  "event": "Baseball",
  "members": [
      2, 3
  ]
}
~~~
<br/>

> ex) member collection
~~~json
{
  "_id": 1,
  "memberName": "Anonymous1"
},
{
  "_id": 2,
  "memberName": "Anonymous2"
},
{
  "_id": 3,
  "memberName": "Anonymous3"
}
~~~
<br/>

> ex) team_member collection
~~~json
{
  "id": 1,
  "teamName": "A_TEAM",
  "event": "Soccer",
  "members": [
      {
        "_id": 1,
        "memberName": "Anonymous1"
      }
  ]
},
{
  "id": 2,
  "teamName": "B_TEAM",
  "event": "Baseball",
  "members": [
      {
        "_id": 2,
        "memberName": "Anonymous2"
      },
      {
        "_id": 3,
        "memberName": "Anonymous3"
      }
  ]
}
~~~

참조방식은 team에서 member의 인덱스만 가지고 있고, <br/>
임베디드는 Document별로 member에 대한 내용까지 포함되어있습니다.
<br/><br/>

### 애플리케이션 테스트 코드
Service, Repository는 간략하게 작성하였으므로 테스트 코드만 추가하겠습니다. <br/>
아래 소스 보기를 통해 상세 내용을 확인할 수 있습니다.
<br/><br/>

> ex) TeamTests.java
~~~java
@SpringBootTest
public class TeamTests {
    Logger logger = LoggerFactory.getLogger(TeamTests.class);

    @Autowired
    TeamService teamService;

    @Autowired
    MemberService memberService;

    @Autowired
    TeamMemberService teamMemberService;

    //참조
    @Test
    public void testSelectAllTeamReference(){
        List<TeamDoc> teamList = teamService.selectAllTeam();

        for(TeamDoc teamDoc : teamList){
            logger.info(teamDoc.getTeamName() + " " + teamDoc.getEvent());
            for(Long memberId : teamDoc.getMembers()){
                Optional<MemberDoc> member = memberService.selectMemberById(memberId);
                logger.info(member.get().getMemberName());
            }
        }
    }
    
    //임베디드
    @Test
    public void testSelectAllTeamEmbed(){
        List<TeamMemberDoc> teamList = teamMemberService.selectAllTeamMember();

        for(TeamMemberDoc teamMember : teamList){
            logger.info(teamMember.getTeamName() + " " + teamMember.getEvent());

            for(MemberDoc member : teamMember.getMembers()){
                logger.info(member.getMemberName());
            }
        }
    }
}
~~~
참조는 조회된 team의 members 아이디 개수만큼 조회를 해야하고, <br/>
임베디드는 한번의 트랜잭션으로 조회할 수 있습니다. <br/>
하지만 Document에는 크기 제한이 있고 Document의 크기를 늘리는 것이 insert 하는 것보다 느리기에<br/>
데이터가 쌓이다 보면 문제가 될 것으로 보입니다. <br/>
따라서 RDBMS의 외래키처럼 member에서 team의 인덱스를 갖는 구조가 좋아보입니다.
<br/><br/>

> ex) team collection
~~~json
{
  "team_id" : 1
},
{
  "team_id" : 2
}
~~~
<br/>

> ex) member collection
~~~json
{
  "member_id" : 1
  , "team_id" : 1
},
{
  "member_id" : 2
  , "team_id" : 2
},
{
  "member_id" : 3
  , "team_id" : 2
},
~~~

<br/><br/>

예제 소스: https://bit.ly/3hMIa34
