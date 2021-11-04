template을 이용해 html을 그려주는 라이브러리로 replace해가며 날코딩하던거 보다 가독성이 좋게 작업할 수 있습니다.<br/>

1. js import
- https://cdnjs.com/libraries/handlebars.js 에 접속하여 handlebars.min.js 다운로드 및 프로젝트에 import

2. 사용하기
 2.1. 단일 데이터
> ex) test.html
~~~html
<!-- 생략 -->
<body>
    <div id="draw">
        <script id="test_form" type="text/x-handlebars-template">
            <div>{{data1}}</div>
        </sciprt>
    </div>
</body>
<!-- 생략 -->
~~~

sciprt태그로 감싸며 type을 text/x-handlebars-template로 지정해준 다음 변수명으로 사용할 값을 {{}}로 감싸줍니다. <br/>
> ex) test.js
~~~js
function testHandlebars(){
    var template = Handlebars.compile($('#test_form').html()); //(1)
    var data = {
        'data1': 'testData' //(2)
    };
    $('#draw').html(template(data)); //(3)
}
~~~
(1) script태그의 id를 통해 html접근 <br/>
(2) {{}}를 통해 변수로 지정했던 이름을 key값으로 지정 <br/>
(3) html 내용 교체


 2.2. 반복데이터
