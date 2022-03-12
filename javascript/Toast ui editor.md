오픈소스 Markdown WYSIWYG Editor인 Toast ui 에디터를 사용해보면서 괜찮은것 같아서 다음에도 사용할 것을 대비해 기록해보고자 합니다. <br><br>

> ex) tui_test.html
~~~html
<!DOCTYPE html>
<html>
    <!-- (1) -->
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/codemirror/5.48.4/codemirror.min.css"/>
    <link rel="stylesheet" href="https://uicdn.toast.com/editor/latest/toastui-editor.min.css" />

    <!-- (2) -->
    <link rel="stylesheet" href="https://uicdn.toast.com/tui-color-picker/latest/tui-color-picker.min.css"/>
    <link rel="stylesheet" href="https://uicdn.toast.com/editor-plugin-color-syntax/latest/toastui-editor-plugin-color-syntax.min.css"/>
    
    <!-- (3) -->
    <div id="editor"></div>

    <!-- (2) -->
    <script src="https://uicdn.toast.com/tui-color-picker/latest/tui-color-picker.min.js"></script>

    <!-- (1) -->
    <script src="https://uicdn.toast.com/editor/latest/toastui-editor-all.min.js"></script>
    <script src="https://uicdn.toast.com/editor-plugin-color-syntax/latest/toastui-editor-plugin-color-syntax.min.js"></script>

    <script>
        const { Editor } = toastui;
        const { colorSyntax } = Editor.plugin;

        const editor = new toastui.Editor({
            el: document.querySelector('#editor'),
            previewStyle: 'vertical',
            height: '500px',
            initialEditType: 'wysiwyg',
            plugins: [colorSyntax], //(4)
            hooks: {
                addImageBlobHook: (blob, callback) => { //(5)
                    const imageUrl = setEditorImage(blob); //(6)
                    callback(imageUrl, '이미지 설명');
                }
            }
        });
      
        function setEditorImage(image){
            //이미지 업로드 처리
        }
    </script>
</div>

</html>
~~~

(1): Toast ui editor를 사용하기 위해 기본적으로 필요한 js, css <br>
(2): 에디터에서 글자 색을 변경하기 위한 플러그인. 기본 에디터의 크기를 경량화 하기 위해 여러 기능들을 플러그인으로 따로 분리시킨 것으로 보입니다. <br>
(3): 에디터로 사용될 엘리먼트 <br>
(4): 사용할 플러그인 종류 <br>
(5): 이미지 등록 이벤트로 toast ui 에서는 이미지를 base64로 변환하여 저장하기에 해당 메소드를 이용해 파일 서버에 따로 저장가능 <br>
(6): setEditorImage는 이미지 업로드 하기 위한 커스텀 메소드로 image url만 리턴받아 callback 함수를 호출 <br>

<br>

> Editor 기본 생성 옵션

- el: 에디터가 생성될 엘리먼트
- previewStyle: 마크다운 모드에서 편집중인 콘텐츠 프리뷰 옵션 [tab/vertical]
- height: 에디터 편집 영역 높아
- initialEditType: 초기 모드 설정 [markdown/wysiwyg]
- initialValue: 에디터 내용 초기값

<br><br>

> 생성화면
<img src="/img/20220313_1.png" witdh="350px">

(1): color 플러그인이 추가 <br>
(2): markdown, wysiwyg 모드가 나뉨

