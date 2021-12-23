# 2021-12-23 (mark down js)

typora 가 electron 으로 구현된 editor 라고 해서 electron 에서 mark down 을 보여주기 위해 사용 가능한 것을 찾다가 marked.js 를 찾았다.

https://gist.github.com/Back-Tracking/1cac3f8a577c70a6f6645cbcc5dc82d5

```html
<body>
  <div id="editor">
    <textarea id="input"># hoge</textarea>
    <div id="result"></div>
  </div>
  <script>
    document.getElementById('input').onkeyup = function(e) {
      document.getElementById('result').innerHTML =
        marked(document.getElementById('input').value);
    };
  </script>
</body>
```



+

marked js 를 cdn 으로 사용할 때 나오는 에러

refused to load the script because violates the following content security policy directive

https://stackoverflow.com/questions/34950009/chrome-extension-refused-to-load-the-script-because-it-violates-the-following-c