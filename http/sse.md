Server-Sent Events(SSE)는 서버가 클라이언트로 단방향 실시간 업데이트를 푸시하기 위한 HTTP 기반 통신 기술

### 필요 조건
- HTTP/1.1 이상
- Response Content Type은 **text/event-stream** 이어야 함

### 예제

<br>

Server
~~~kt
import org.springframework.http.MediaType
import org.springframework.web.bind.annotation.GetMapping
import org.springframework.web.bind.annotation.RequestMapping
import org.springframework.web.bind.annotation.RestController
import reactor.core.publisher.Flux
import java.time.Duration
import java.time.LocalTime


@RestController
@RequestMapping(value = ["/sse"])
class SseController {
    @GetMapping(value = [""], produces = [MediaType.TEXT_EVENT_STREAM_VALUE])
    fun getFlux(): Flux<String> {
        return Flux.interval(Duration.ofSeconds(3))
            .map { sequence: Long ->
                "Event: $sequence, ${LocalTime.now()}"
            }
            .doOnCancel {
                TODO("disconnected")
            }
    }
}
~~~

<br>

Client
~~~js
const eventSource = new EventSource("/sse")

eventSource.onmessage = function(event) {
    console.log(`Server Message: ${event.data}`)
};

eventSource.onerror = function() {
    console.error("An error occurred");
};
~~~