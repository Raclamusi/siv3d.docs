description: OpenSiv3D のチュートリアル

!!! warning "このページよりも新しいドキュメントがあります"
	このドキュメントは古い OpenSiv3D v0.4.3 向けです。2021 年9 月 18 日に最新の OpenSiv3D v0.6.0 がリリースされました。最新のドキュメントは [Siv3D リファレンス v0.6.0](https://zenn.dev/reputeless/books/siv3d-documentation) です。

# 20. リソースの埋め込み
この章では、アプリケーションの実行ファイルに画像や音声などのファイルを埋め込んで、それをプログラムで読み込む方法を学びます。

## 20.1 リソースの埋め込みの基本
Siv3D では、プログラムで使う画像や音声、テキストなどのリソースファイルを .exe や .app に埋め込み、ユーザから見て単一のファイルになるようにまとめることができます。リソースファイルを実行ファイルに埋め込むと、配布が簡単になるだけでなく、ファイルがユーザによって削除されたり、変更されたりすることを防げます。  
埋め込みファイルを変更するとプロジェクトの再ビルドが必要になるため、ファイルを頻繁に更新する開発中はリソースの埋め込みはせず、リリースが近くなってから埋め込みに変更すると効率が良いです。

アプリケーションにファイルを埋め込む手順は以下のとおりです。

### Windows の場合
`App/Resource.rc` に、埋め込みファイルのパスを記述します。`App/Resource.rc` をソリューション エクスプローラー上で右クリックして「コードの表示」で開き、`Resource(example/windmill.png)` のように、埋め込みたいファイルのパスを、埋め込むファイルごとに追記します。デフォルトでは Siv3D の内部処理に必要な `engine/` フォルダの各種ファイルが記述されています。プロジェクトを再ビルドすると .exe にファイルが埋め込まれます。

![](https://github.com/Siv3D/siv3d.docs.images/blob/master/tutorial/20/mac-0.png?raw=true)

### macOS の場合
プロジェクトナビゲータにフォルダをドラッグし「Create folder references」を選択すると、プロジェクトナビゲータ上で青いフォルダアイコンになって表示されます。このフォルダ内のファイルはすべて .app に埋め込まれます。デフォルトでは Siv3D の内部処理に必要な `engine/` フォルダが青いアイコンで表示されています。

![](https://github.com/Siv3D/siv3d.docs.images/blob/master/tutorial/20/mac-1.png?raw=true)

![](https://github.com/Siv3D/siv3d.docs.images/blob/master/tutorial/20/mac-2.png?raw=true)

### Linux の場合
Linux 版ではファイルの埋め込みは実装されていません。代わりに、`resources/` フォルダに必要なリソースファイルを格納し、アプリケーションに同梱します。デフォルトでは Siv3D の内部処理に必要な `engine/` フォルダが `resources/` フォルダに格納されています。

## 20.2 埋め込まれたリソースを読み込む
埋め込まれたファイルをプログラムで読み込むには、これまで `U"example/windmill.png"` と指定していたファイルパスを `Resource()` で囲んで `Resource(U"example/windmill.png")` に変更します（Windows, macOS, Linux 共通）。

ビルドされた実行ファイルを異なるフォルダに移動させてから実行し、埋め込みリソースの画像が表示されれば、埋め込みが成功したことの確認になります。

```C++
# include <Siv3D.hpp>

void Main()
{
	// ファイルから読み込み
	const Texture textureFile(U"example/windmill.png");

	// 埋め込みリソースから読み込み
	const Texture textureResource(Resource(U"example/windmill.png"));

	while (System::Update())
	{
		textureFile.draw(0, 0);

		textureResource.draw(0, 320);
	}
}
```
