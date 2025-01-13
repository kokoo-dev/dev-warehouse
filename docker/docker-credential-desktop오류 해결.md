docker compose up 명령어 실행 시 <br>
**error getting credentials - err: exec: "docker-credential-desktop": executable file not found in $PATH, out: ``** 오류 해결

~~~sh
vi ~/.docker/config.json
~~~

~~~json
// config.json
// credsStore -> credStore로 변경
{
  "auths": {},
  "credsStore": "desktop", 
  "currentContext": "desktop-linux"
}
~~~