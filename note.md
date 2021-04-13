# 1.解决iframe高度问题

- js

```js
//父页面
window.addEventListener('message', e => {
  if (e.data.iframeHeight) {
    this.iframeHeight = e.data.iframeHeight;
  }
});
//1.html 
;((origin) => {
  window.onload = () => {
    setInterval(() => {
      top.postMessage({
        iframeHeight: Math.max(document.body.scrollHeight, document.documentElement.scrollHeight)
      }, origin);
    }, 200)
```

- vue

  ```js
  // 父页面 
  https://github.com/davidjbradshaw/iframe-resizer/blob/master/docs/use_with/vue.md
  
  //子页面 注入这个js
  https://cdn.bootcdn.net/ajax/libs/iframe-resizer/4.2.9/iframeResizer.contentWindow.min.js
  ```

  

  # 2. 文字渐变
  
  ```js
  background-image: -webkit-gradient(
      linear,
      0 0,
      0 bottom,
      from(rgba(255, 255, 255, 1)),
      to(rgba(0, 138, 255, 0.6))
  );
  background-clip: text;
  -webkit-text-fill-color: transparent;
  ```
  
  # 3. 复制
  
  ```html
   <div class="copy_cont" @click="copy"><a-icon type="copy" /></div>
  <textarea
      id="copy_data"
      :style="{ width: '100%', padding: '10px', resize: 'none' }"
      disabled
  >
      <script scr="https://cdn.bootcdn.net/ajax/libs/iframe-			       resizer/4.2.9/iframeResizer.contentWindow.min.js">
      </script>
  </textarea>
  
  ```
  
  ```js
  // 复制到粘贴板
  copy() {
      const content = document.getElementById('copy_data').value;
      var input = document.createElement('input');
      document.body.appendChild(input);
      input.value = content;
      input.select();
      document.execCommand('copy');
      if (document.execCommand('copy')) {
          this.$message.success('复制成功!');
      }
  },
  ```
  
  