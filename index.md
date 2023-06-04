## 確実なコンフリクト解消方法
現在、私が所属する現場では、BitbucketとSourceTreeを使用してプログラムのソース管理を行っている。Bitbucketのリポジトリにプログラムを保存し、Gitツールの一つであるSourceTreeを用いて、ブランチの作成や修正のコミット等を行っている。
プログラムの修正のおおよその手順は下記の通りとなる。
1. SourceTreeを用いてBitbucketのリポジトリのブランチを作成し、それをチェックアウトする
2. チェックアウトしたソースを修正する
3. 修正が完了したら、コミットしたのち、Bitbucketのブランチへプッシュする
4. ブランチを本線にマージしてもらうためのプルリクエスト（プルリクという）を発行する

このプルリクエストを発行した時点で、他人の修正とコンフリクトを起こしていた場合は、「マージできない」旨のメッセージを受け取ることととなる。
この場合、コンフリクトを解消する確実な方法として、自分の変更を退避（スタッシュという）したのち、リポジトリの最新版から新たにブランチを切り直し、そのブランチに退避した変更を当ててからコミットし、再度プルリクを出すという方法がある。ただこの方法は少し時間がかかり、修正が集中している箇所を直しているときには、プルリクを出し直したときには、再び別の修正によりコンフリクトが発生するということが度々発生した。
そこでもう少し効率的なコンフリクト解消方法がないか調査してみた。

## 一般的なコンフリクト解消方法
プルリク申請時に判明したコンフリクトの一般的な対処方法は、コンフリクトを発生させた修正を自身のブランチにマージし、コンフリクト箇所をローカル環境で修正したのち、再度プルリクを出し直す方法であることがわかった。この方法では、再度プルリクエストを作成する必要がなくなる。
その方法には下記の通りとなる。
1. 自身のローカルブランチをチェックアウトする
2. 自身の変更に、最新のブランチをマージする
3. コンフリクトが発生した箇所をエディタで修正する
4. コンフリクトが解消した修正をコミットしたのち、リポジトリへプッシュする
5. プルリクのコンフリクトの警告が消えていることを確認する

## 具体例
上記コンフリクトの解消方法を例を使って説明しよう。今、Bitbucketのリポジトリにdevelopというメインのブランチがあり、そのブランチは下記のファイル「index.html」を持つとする。このブランチから、サブブランチ「branchA」と「branchB」を作成し、それぞれ下記のように修正する。

* develop - index.html *
```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
    <h1>Header1</h1>
    <p>Text</p>
    <h1>Header2</h1>
    <p>Text</p>
</body>
</html>
```
* <branchA - index.html> *
```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>TitleA</title> <!-- ここを修正-->
</head>
<body>
    <h1>Header1</h1>
    <p>TextA</p> <!-- ここも修正 -->
    <h1>Header2</h1>
    <p>Text</p>
</body>
</html>
```
* <branchB - index.html> *
```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>TitleB</title> <!-- branchAとコンフリクトを起こす修正-->
</head>
<body>
    <h1>Header1</h1>
    <p>Text</p>
    <h1>Header2</h1>
    <p>TextB</p> <!-- branchAとはコンフリクトを起こさない修正 -->
</body>
</html>
```
branchAでdevelopへのプルリクエストを作成し、それが受理されたあと、branchBがdevelopへのプルリクエストを作成すると、コンフリクトが発生し、下記のようなエラーメッセージが表示され、ローカルのSourceTreeでは下記のように表示される

* <エラーメッセージ> *
![エラーメッセージ](https://github.com/umidori/conflict/blob/main/img/conflict.png)

* <ローカルのSourceTreeでの表示> *
![エラーメッセージ](https://github.com/umidori/conflict/blob/main/img/SourceTree(comment).png)

コンフリクトを解消するには、SourceTreeで下記の手順を実施する
1. メニューの端末をクリックし、端末を起動する
![ローカルのSourceTreeでの表示](:/7ae65c018a7e4eb2be898a42820d146d)
2. 起動した端末で下記のコマンドを実行する（2のコマンドは環境により不要な場合がある）
	```text
	$ git checkout branchB
	$ git config pull.rebase false
	$ git pull origin develop
	```
	**<コマンド実施後のSourceTreeの表示>**
	![branchB-pull origin.png](https://github.com/umidori/conflict/blob/main/img/branchB-pull%20origin.png)
3. コンフリクトが反映されたindex.htmlが生成されるので、それをエディタ等で修正し、コンフリクトを解消する]
	* <コンフリクトが反映されたindex.html> *
	```html
	<!DOCTYPE html>
	<html>
	<head>
		<meta charset="UTF-8">
	<<<<<<< HEAD
		<title>TitleB</title>
	=======
		<title>TitleA</title>
	>>>>>>> 4b16511746fde7f9a371945cd8dc4033c3d60093
	</head>
	<body>
		<h1>Header1</h1>
		<p>TextA</p>
		<h1>Header2</h1>
		<p>TextB</p>
	</body>
	</html>
	```
	* <修正後のindex.html（例）> *
	```html
	<!DOCTYPE html>
	<html>
	<head>
		<meta charset="UTF-8">
		<title>TitleAB</title> <!-- コンフリクト解消後のタイトル（例）-->
	</head>
	<body>
		<h1>Header1</h1>
		<p>TextA</p>
		<h1>Header2</h1>
		<p>TextB</p>
	</body>
	</html>
	```
4. コンフリクトが解消したindex.htmlをコミットし、リモートにプッシュする
* <プッシュ後のSourceTreeの表示> *
![branchB-conflict solved.png](:/cb06ce95e96a42a7bdc498f92c607108)

これで、branchBのプルリクエストのエラーは解消され、受理されるのを待つ状態になる。
