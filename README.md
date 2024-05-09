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



