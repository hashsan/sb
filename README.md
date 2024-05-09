# sb

sb is site builder

## 想定
```
import "https://hashsan.github.io/sb/sb.js"

sb(name,template.html,replaceTarget='<!--sb-->',saveSpan=2000)

//template.htmlを使って、name.mdとname.htmlをtemplate.htmlと同じ場所に作る。
//replaceTargetを指定しない場合はtemplate.htmlの中にある'<!--sb-->'である。
```

## 開発参考

### 1. url?q=fooといったようなクエリを取得するユーティリティ関数

```
function get(query) {
    var params = new URLSearchParams(window.location.search);
    var value = params.get(query);
    
    if (value === null) {
        throw new Error("クエリ '" + query + "' は存在しません。");
    }
    
    return value;
}
```

### 2. template.htmlをfetchリクエストで取得するユーティリティ関数

```
async function getTemplate(url) {
    try {
        const response = await fetch(url);
        if (!response.ok) {
            throw new Error('Network response was not ok');
        }
        const html = await response.text();
        const parser = new DOMParser();
        const doc = parser.parseFromString(html, 'text/html');
        
        return { html, doc };
    } catch (error) {
        console.error('Fetch Error:', error);
        throw error;
    }
}
```

### 3. viewとeditを切り替えるユーティリティ関数
使用例では、なおかつ、markedで文字列をマークダウンにしてhtmlにしている。
```
import "https://cdnjs.cloudflare.com/ajax/libs/marked/12.0.2/marked.min.js";

function makeFrame() {
    return `
        <div class="frame">
            <div class="view">View Mode</div>
            <div class="edit"
            contenteditable="plaintext-only"
            style="outline:none;min-height:10rem;"
            >Edit Mode</div>
        </div>
    `;
}

function eventFrame(caller) {
    const view = document.querySelector('.view');
    const edit = document.querySelector('.edit');

    const showView = function() {
        if(caller){
          caller(view,edit);
        }
        view.style.display = 'block';
        edit.style.display = 'none';
    }
    const showEdit = function() {
        view.style.display = 'none';
        edit.style.display = 'block';
        edit.focus();
    }

    // 最初は .view を表示
    showView();

    view.addEventListener('click', showEdit);
    edit.addEventListener('blur', showView);
}

/*
var html = makeFrame();
document.body.innerHTML = html;
eventFrame((view,edit)=>{
  const parse = marked.parse
  view.innerHTML = parse(edit.textContent)
  
});
*/
```

### 4.editableをフェッチして、処理をするユーティリティ関数
```
// debounce関数の定義
function debounce(func, delay) {
  let timerId;
  return function(...args) {
    clearTimeout(timerId);
    timerId = setTimeout(() => {
      func.apply(this, args);
    }, delay);
  };
}

// autoEditable関数の定義
function autoEditable(caller, span) {
  // デフォルトで2秒の間隔を設定
  span = span || 2000;

  // contenteditable属性を持つ要素を検索
  const query = '[contenteditable]';
  const el = document.querySelector(query);

  // 要素が見つからない場合はエラーメッセージを出力して終了
  if (!el) return console.error(query + ': query not found.');

  // コールバック関数を定義し、指定された間隔で呼び出す
  const save = () => { if (caller) caller(el); };
  const de_save = debounce(save, span);

  // キーボードイベントを監視し、Ctrl + Sで保存または指定された間隔ごとに保存
  el.addEventListener('keydown', (event) => {
    if (event.ctrlKey && event.key === 's') {
      event.preventDefault(); // デフォルトの保存イベントを防止
      save(); // Ctrl + Sで即時保存
    } else {
      de_save(); // 指定された間隔で保存
    }
  });

  return el; // 処理対象の要素を返す
}

/*
document.body.append(document.createElement('pre'))

autoEditable(el=>{
  
  var pre = document.querySelector('pre')
  console.log(pre,el)
  pre.textContent = el.textContent
})
*/
```



