# Vue.js

## ＊ディレクティブのまとめ
|ディレクティブ|できること・文法|対象|
|---|---|---|
|v-model|オブジェクトのオプション(dataとか)と結びつける |input,select,textarea|
|v-for|for文: todo(子) in todos(親)|-   |
|v-bind|Model の値を HTML コンポーネントに反映させる|-|
|v-on  |v-on:click="addItem"のように◉◉したら=のイベント(メソッド)を実行。@click="メソッド"とも記述可|今のとこform|
|v-if|if文:v-if="todos.length"|ul|
|v-show|ある条件のときにメッセージを表示 v-show="条件"|li|

参考：https://www.shookuro.com/entry/2018/09/09/100908

## データ バインディング
data と UI を結び付けるという意味で、双方向というのは data を更新すれば UI が更新されてUI を更新すれば data が更新されるという意味になります。

### ＊dataからUIに
```js
(function(){
    'use strict';

    //vm = view model
    var vm = new Vue({
        //el = element
        el: '#app', //CSSのセレクタと同様
        data: {
            name: 'taguchi',
        }

    });
})();
```

```html
   <div id="app">
        <p>Hello {{ name }}!!</p>
        <!-- Hello taguchi!! -->
    </div>

```

### ＊UIからdataに

```html
<p><input type="text" v-model="name"></p>
```
inputタグの中身がリアルタイムで反映される仕組みになる。<br>
v-modelはinput select textarea　に対して使用できるディレクティブ。

## TODOリスト作り

### ＊taskを表示させる

```js
(function(){
    'use strict';

    var vm = new Vue({
        el: '#app',
        data: {
                todos:[
                    'task 1',
                    'task 2',
                    'task 3'
                ]
        }
    });
})();
```


```html
       <ul>
            <li>{{ todos[0] }}</li>
            <li>{{ todos[1] }}</li>
            <li>{{ todos[2] }}</li>
        </ul>

        <!-- ループ処理 -->
        <ul>
            <li v-for="todo in todos">
                {{ todo }}
            </li>
        </ul>
```
v-model だとか v-for とか v- から始まる属性は<strong>ディレクティブ</strong>

### ＊taskを追加する

```html
        <!-- submitされた後の処理を書くためにv-on -->
        <!-- イベントハンドリング -->
        
        <form v-on:submit="addItem">
        <!-- <form @submit="addItem"> -->

            <!-- UIにモデルを結びつける -->
            <input type="text" v-model="newItem">
            <input type="submit" value="送信">

        </form>
```
v-onディレクティブ<br>
v-on:submit・・submitした後に指定されたメソッドを実行<br>
v-on:click　・・clickされた後に....

```js

    var vm = new Vue({
        //el element
        el: '#app',
        data: {
                newItem: '',
                todos:[
                    'task 1',
                    'task 2',
                    'task 3'
                ]
        },
        methods: {
            //addItemは次の処理だよ
            addItem: function(){
                this.todos.push(this.newItem);
                //submitした後にinputboxの中身を空にする
                this.newItem = '';
            }
        }
    });
```

上記だけだとうまくいかないので以下追記する。

```html
        <!-- 規定のページが遷移することを無効化する -->
        <form v-on:submit.prevent="addItem">
        <!-- <form @submit.prevent="addItem"> -->

```

### ＊taskを削除する
main.jsにdeleteItemのメソッドの追加
```js

        methods: {
            addItem: function(){
                this.todos.push(this.newItem);
                this.newItem = '';
            },


            deleteItem: function(index){
                if(confirm('are you sure?')){
                    //index番目から1個削除する
                    this.todos.splice(index,1);
                }
        }
```

```html
    <ul>
      <!-- indexに配列の添字がはいってくる -->
      <li v-for="(todo, index) in todos">
         {{ todo }}
         <span @click="deleteItem(index)" class="command">[x]</span>
      </li>
    </ul>
```

### ＊taskの完了状態を管理する
todosをオブジェクトに編集する
```js
data: {
                newItem: '',
                todos:[{
                    title: 'task 1',
                    isDone: false

                },{
                    title: 'task 2',
                    isDone: false
                },{
                    title: 'task 3',
                    isDone: true
                }
            ]}
```

メソッドも編集する
```js
            addItem: function(){
                //インスタンス生成
                var item = {
                    title: this.newItem,
                    isDone: false
                };

                //引数をitemに
                this.todos.push(item);
                this.newItem = '';
            },
```

これだけだとオブジェクト全部が表示されてしまうため、todo.titleにする
```html

      <li v-for="(todo, index) in todos">
         {{ todo.title }}
         <span @click="deleteItem(index)" class="command">[x]</span>
      </li>

```

オブジェクトに管理状態を持たせたが、管理状態をわかるようにチェックボックスをつけてあげる。

チェックボックスの追加
v-modelディレクティブを使いisDone(オブジェクトのプロパティ)と結びつける。
```html
      <li v-for="(todo, index) in todos">
          <!-- isDoneがtrueならチェック、falseならばチェックなし -->
          <input type="checkbox" v-model="todo.isDone">
         {{ todo.title }}
         <span @click="deleteItem(index)" class="command">[x]</span>
      </li>
```

Doneというクラスを与え、終わったタスクのスタイル(打ち消し線)を変える<br>
以下どちらかを記述する
```html
<!-- todo の isDone が true の場合は、 done クラスとなる -->
<span v-bind:class="{done: todo.isDone}">{{ todo.title }}</span>
<!-- v-onディレクティブ -->
<span  :class="{done: todo.isDone}">{{ todo.title }}</span>
```

### ＊Todoがなかった場合にメッセージを表示させる
v-ifとv-viewの2つのディレクティブを使う方法がある<br>
1.v-view<br>


```html
    <ul>
        <li v-for="(todo, index) in todos">
            <input type="checkbox" v-model="todo.isDone">
            <span :class="{done: todo.isDone}">{{ todo.title }}</span>
            <span @click="deleteItem(index)" class="command">[x]</span>
        </li>
        <!-- todosの配列がなかった場合は以下のメッセージを表示 -->
        <li v-view="!todos.length">Nothing to do,yay!</li>
    </ul>
```

2.v-if<br>
ulごとに処理をわけてかく
```html
    <ul v-if="todos.length">
        <li v-for="(todo, index) in todos">
            <input type="checkbox" v-model="todo.isDone">
            <span :class="{done: todo.isDone}">{{ todo.title }}</span>
            <span @click="deleteItem(index)" class="command">[x]</span>
        </li>
    </ul>
              
    <ul v-else>
        <li>Nothing to do, yay!</li>
    </ul>
```

### ＊taskの残数を確認する
メモ：なんかしらんけどv-ifだと動かないっぽい<br>
spanタグを使い、残っているタスクと総タスクの表示<br>
```html
    <h1>My Todos
      <span class="info">{{ remaining }}/{{ todos.length }}</span>
    </h1>
        
        <ul>
            <li v-for="(todo, index) in todos">
             <input type="checkbox" v-model="todo.isDone">
             <span :class="{done: todo.isDone}">{{ todo.title }}</span>
             <span @click="deleteItem(index)" class="command">[x]</span>
            </li>
             <li v-show="!todos.length">Nothing to do, yay!</li>
        </ul>

```

Vueインスタンスの中にコンポーネントオプションのcomputedを追加する<br>
公式:https://012-jp.vuejs.org/api/options.html<br>
filter関数について<br>
https://www.sejuku.net/blog/21887<br>
メモ：コールバック関数・・・他の関数に引数として渡す関数のこと。<br>

```js
      computed: {
        remaining: function() {
            //filter..isDoneがfalseである項目の数を調べる
          var items = this.todos.filter(function(todo) {
            return !todo.isDone;
          });
          return items.length;
        }
      }
```

### ＊完了したtaskを一括削除する

```js

      methods: {
        addItem: function() {
          var item = {
            title: this.newItem,
            isDone: false
          };
          this.todos.push(item);
          this.newItem = '';
        },
        deleteItem: function(index) {
          if (confirm('are you sure?')) {
            this.todos.splice(index, 1);
          }
        },
        purge: function() {
            if (!confirm('delete finished??')) {
                //処理を行わないようにreturn
              return;
            }

            // this.todos = this.todos.filter(function(todo) {
            //     return !todo.isDone;
            //   });

            //filterの処理が重複以下記述
            //終わったTodoの数を配列で返す
            this.todos = this.remaining;
          }

      },
      computed: {
        remaining: function() {
        //   var items = this.todos.filter(function(todo) {
        //     return !todo.isDone;
        //   });
        //   return items.length;

        return this.todos.filter(function(todo){
            return !todo.isDone;
        });
        }
      }
```

remaining.lengthで配列の数を返すように記述
```html
<h1>My Todos
    <span class="info">{{ remaining.length }}/{{ todos.length }}</span>
</h1>
```

### ＊LocalStrageを使いデータを永続化する
LocalStrage…HTML5から導入されたAPI、Web Storageの一種<br>
クッキーと似ている。<br>
参考：https://www.granfairs.com/blog/staff/local-storage-01

Vueインスタンスにwatchを追加<br>
watch..オブジェクトの追加や削除を検知する
deep watcher..オブジェクトの値の変更までも検知する
以下コードだと新しいtaskを追加した時にはsaveされるが、チェックボックスにチェックをいれたときにはsaveされない<br>
単に todos をこのように watch しただけだと todos の配列自体に変更があったときにはこちらの処理は実行してくれるのですが、
配列の中身の要素(今回はtitleやisDone)の変更までは監視してくれないという仕組みになっているためです。

```js
    watch: {
          //todosに変更があったときは次の処理をしなさいよ
          todos: function(){
              localStorage.setItem('todos',JSON.stringify(this.todos));
              alert('Data saved!');
          }
```

なので、以下のように(deep watcher)書き換える
```js
 watch: {
        todos:{
            handler: function(){
              localStorage.setItem('todos',JSON.stringify(this.todos));
              alert('Data saved!');
          },
          deep: true
        }
```
データの保存はできているが、読み出しができない状態。<br>
※データがLocalstorageにsaveされているかどうかの確認は<br>
検証モード->application->Localstorage->file-><br>
|key||value|
|---|---|---|
|todos||0:{title:"task",isDone:true}|
|todos||1:{title:"task2",isDone:false}|
↑のような感じでsaveされている<br>

読み出しをできるようにするために以下の通り記載

```js
mounted: function(){
    this.todos = JSON.parse(localStorage.getItem('todos'));
 }
```

## コンポーネント
```html
<div id="app">
      <!-- Component -->
      <like-component message="LIKE"></like-component>
      <like-component message="test"></like-component>

      <!-- message属性を与えていない場合はdefaultのlikeになる -->
      <like-component></like-component>
  </div>
```

```js
　(function() {
    'use strict';

    //コンポーネントごとにオブジェクトを作ってあげる
    var likeComponent = Vue.extend({
        
        //プロパティ
        props: {
            //like-componentに与えているカスタム属性
            message: {
                type:String,
                default: "like",
            }
        },
        data: function() {
            return {
                //初期値
                count: 0
            }
        },
        template: '<button @click="countUp">{{ message }} {{ count }}</button>',
        methods: {
            countUp: function() {
                this.count++;
                
            }
        }
    });

　   var app = new Vue({
        el: '#app',
        components: {
            'like-component': likeComponent //オブジェクト
        },
    });
  
  })();

```

### ＊$emitを使ってイベントを発火 (トータルのいいね数を表示させるには)
コンポーネントから親要素にデータを渡す<br>
Component からイベントを発火して、親要素の方でそれを検出するという手法が一般的なので見ていきましょう。
$emit...Vue インスタンスに定義されたメソッドで、ひとつ以上の引数を持ちます <br>
参考:https://www.hypertextcandy.com/vuejs-components-introduction-emit-events

```js
//like-componentのオブジェクト内 追加
        methods: {
            countUp: function() {
                this.count++;

                //$emit カスタムイベント
                this.$emit('increment');
            }
        }

//appのオブジェクト内 追加
        data: {
            total:0
        },
        methods: {
            incrementTotal: function() {
                this.total++;
            }
        }
```

```html
<div id="app">
      <p>Total likes:{{ total }}</p>
      <!-- Component -->
      <!-- v-onディレクティブを与える -->
      <like-component message="LIKE" @increment="incrementTotal"></like-component>
      <like-component message="test" @increment="incrementTotal"></like-component>
      <like-component @increment="incrementTotal"></like-component>
  </div>
```