description: OpenSiv3D のチュートリアル

!!! warning "このページよりも新しいドキュメントがあります"
	このドキュメントは古い OpenSiv3D v0.4.3 向けです。2021 年9 月 18 日に最新の OpenSiv3D v0.6.0 がリリースされました。最新のドキュメントは [Siv3D リファレンス v0.6.0](https://zenn.dev/reputeless/books/siv3d-documentation) です。

# 18. テキストファイル

この章では、テキストファイルの内容を読み込んだり、テキストファイルに文字を書き込んだりする方法を学びます。

## 18.1 テキストファイルから 1 行ずつ読み込む
テキストファイルからテキストを読み込むには `TextReader` の機能を使います。`TextReader` のコンストラクタ引数に、読み込みたいテキストファイルのパスを渡します。このファイルパスは、実行ファイルがあるフォルダ（開発中は App フォルダ）を基準とする相対パスか、絶対パスを使用します。リリース用のアプリを作るときには、のちの章で説明する「リソース」パスの使用を推奨します。ファイルが存在しない場合など、オープンに失敗したかどうかは `if (!reader)` で調べられます。

オープン直後は、読み込み位置はファイルの先頭にセットされています。`TextReader::readLine()` に `String` 型の変数を参照で渡すと、読み込み位置を始点とし、次に見つかった改行またはファイル終端までの 1 行分を読み込んだ内容をその変数に格納し、読み込み位置をそこまで進めて `true` を返します。読み込み位置がすでにファイルの終端にあって、これ以上読み込めないときには `false` を返します。次のプログラムのように `while` ループと組み合わせると便利です。

```C++
# include <Siv3D.hpp>

void Main()
{
    // ファイルをオープンする
	TextReader reader(U"example/test.txt");

	// オープンに失敗
	if (!reader)
	{
		throw Error(U"Failed to open `test.txt`");
	}

	// 行の内容を読み込む変数
	String line;

	// 行数の表示用のカウント
	size_t i = 0;

	// 終端に達するまで 1 行ずつ読み込む
	while (reader.readLine(line))
	{
		Print << U"{}: {}"_fmt(i++, line);
	}

	while (System::Update())
	{

	}

    // reader のデストラクタで自動的にファイルがクローズされる
}
```

## 18.2 ファイルをオープン・クローズするタイミングを制御する
`TextReader` のコンストラクタでファイルをオープンせずに、`TextReader::open()` でファイルをオープンすることもできます。オープンに成功した場合は `true`, 失敗した場合は `false` を返します。

また通常は `TextReader` のデストラクタが実行されるタイミングでファイルがクローズされますが、例えば内容を読み込んだあとにテキストファイルを削除したり、別の内容を書き込みたい場合には、ファイルがオープンされたままだと操作ができないため、すぐにクローズしたいというケースがあるでしょう。そうしたときには `TextReader::close()` でファイルを明示的にクローズします。

```C++
# include <Siv3D.hpp>

void Main()
{
	TextReader reader;

	// オープンに失敗
	if (!reader.open(U"example/test.txt"))
	{
		throw Error(U"Failed to open `test.txt`");
	}

	// 行の内容を読み込む変数
	String line;

	// 行数の表示用のカウント
	size_t i = 0;

	// 1 行ずつ読み込む
	while (reader.readLine(line))
	{
		Print << U"{}: {}"_fmt(i++, line);
	}

	// ファイルを明示的にクローズ
	reader.close();

	while (System::Update())
	{

	}
}
```

## 18.3 ファイルにテキストを書き込む
ファイルにテキストを書き込むには `TextWriter` の機能を使います。`TextWriter` のコンストラクタ引数に、書き込み先のファイルのパスを渡します。このファイルパスは、実行ファイルがあるフォルダ（開発中は App フォルダ）を基準とする相対パスか、絶対パスを使用します。ファイルが使用中だったり、ファイル名が不正なものだったりしてオープンに失敗したかどうかは `if (!writer)` で調べられます。

同名のファイルがすでに存在する場合はそれを破棄してからオープンし、存在しない場合は新しい空のファイルを作成してオープンします。

書き込み方法は `Print` と似ていて、オープン済みの `TextWriter` の変数に向かって `<<` で書き込みたい 1 行分の文字列や値を送ります。書き込みの最後には改行が自動で挿入されます。これを避けたい場合はメンバ関数 `.write()` を使って書き込みたい内容を送ります。`.writeln()` は改行が挿入されます。

```C++
# include <Siv3D.hpp>

void Main()
{
	// 書き込み用のファイルをオープンする
	// （同名のファイルがすでに存在する場合はそれを破棄してからオープン）
	TextWriter writer(U"tutorial.txt");

	// オープンに失敗
	if (!writer)
	{
		throw Error(U"Failed to open `tutorial.txt`");
	}

	// 文章を 1 行書き込む
	writer << U"Hello, Siv3D!";

	// 値や文字を　1 行書き込む
	writer << 123 << U',' << 456 << Point(10, 20);

	// 1 文字ずつ書き込む（改行無し）
	writer.write(U'A');
	writer.write(U'B');

	// 1 文字書き込んで改行も書き込む
	writer.writeln(U'C');

	// 値を書き込む（改行無し）
	writer.write(777);
	writer.write(U',');
	writer.write(888);

	while (System::Update())
	{

	}

    // writer のデストラクタで自動的にファイルがクローズされる
}
```

出力結果 (tutorial.txt)
```
Hello, Siv3D!
123,456(10, 20)
ABC
777,888
```

### オープンとクローズのタイミングの制御
`TextWriter` は `TextReader` と同じように、`.open()` を使ってファイルをオープン, `.close()` を使ってファイルをクローズできます。

```C++
# include <Siv3D.hpp>

void Main()
{
	TextWriter writer;

	// オープンに失敗
	if (!writer.open(U"tutorial.txt"))
	{
		throw Error(U"Failed to open `tutorial.txt`");
	}

	// 文章を 1 行書き込む
	writer << U"Hello, Siv3D!";

	// 値や文字を　1 行書き込む
	writer << 123 << U',' << 456 << Point(10, 20);

	// 1 文字ずつ書き込む（改行無し）
	writer.write(U'A');
	writer.write(U'B');

	// 1 文字書き込んで改行も書き込む
	writer.writeln(U'C');

	// 値を書き込む（改行無し）
	writer.write(777);
	writer.write(U',');
	writer.write(888);

	// ファイルを明示的にクローズ
	writer.close();

	while (System::Update())
	{

	}
}
```


## 18.4 既存のテキストファイルにテキストを追加で書きこむ
テキストを既存のテキストファイルの末尾から追加で書き込みたい場合は、`TextWriter` でのオープン時にファイルオープンモードとして `OpenMode::Append` (追加モード) を指定します。同名のテキストファイルが存在しなかった時は、通常どおり新しいファイルを作成します。

```C++
# include <Siv3D.hpp>

void Main()
{
    // 追加モードでテキストファイルをオープン
	TextWriter writer(U"tutorial.txt", OpenMode::Append);

	// オープンに失敗
	if (!writer)
	{
		throw Error(U"Failed to open `tutorial.txt`");
	}

	// 文章を追加する
	writer.write(U"\n------");

	while (System::Update())
	{

	}
}
```

18.3 で作った tutorial.txt に追加した結果
```
Hello, Siv3D!
123,456(10, 20)
ABC
777,888
------
```


## 18.5 テキストファイルのプログラムあれこれ

### スコープによる制御

ファイルのオープン・クローズのタイミングの制御は、`{ } ` でスコープを作り、スコープの終了時にデストラクタで自動的にクローズされるようにするのも良い方法です。

```C++
# include <Siv3D.hpp>

void Main()
{
	{
		TextWriter writer(U"tutorial.txt");

		// 1 行書き込む
		writer.write(U"Hello, Siv3D!");

		// ここで自動的にクローズ
	}

	String text;

	{
        // TextWriter でオープンしたままだと、ここでオープンに失敗する
		TextReader reader(U"tutorial.txt");

		// 1 行読み込む
		reader.readLine(text);

		// ここで自動的にクローズ
	}

	Print << text;

	while (System::Update())
	{

	}
}
```

