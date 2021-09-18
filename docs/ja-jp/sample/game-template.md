description: OpenSiv3D ゲーム開発のサンプル

!!! warning "このページよりも新しいドキュメントがあります"
	このドキュメントは古い OpenSiv3D v0.4.3 向けです。2021 年9 月 18 日に最新の OpenSiv3D v0.6.0 がリリースされました。最新のドキュメントは [Siv3D リファレンス v0.6.0](https://zenn.dev/reputeless/books/siv3d-documentation) です。

# ゲーム開発のテンプレート

## ゲーム開発プロジェクト・テンプレート
本格的なゲーム開発をすぐに始められるプロジェクトテンプレートです。  
リポジトリをダウンロードして使用します。

- https://github.com/Siv3D/GameTemplate

### 機能

- 基本的なファイル分割と SceneManager 対応済み
- OpenSiv3D 用の `.gitignore` が付属
- Visual Studio 用のソリューションファイルと、Xcode 用のプロジェクトファイルを同梱し、どちらからでも開発可能

### 使用手順

- 最新の OpenSiv3D SDK をインストールします
- ゲーム開発プロジェクト・テンプレートをダウンロードします
- macOS の場合、ゲーム開発プロジェクト・テンプレートの `README.md` と同じフォルダに、SDK の `include` と `lib` フォルダをコピーします
- Visual Studio の場合はソリューションファイル、Xcode の場合はプロジェクトファイルを開きます


## シーン遷移（1 ファイル版）
開発初期には開発速度が重要です。  
次のように Main.cpp にすべてのシーンを実装するのもよいでしょう。

<video src="https://github.com/Siv3D/siv3d.docs.images/blob/master/sample/game-template/scene-ja.mp4?raw=true" autoplay loop muted></video>
```C++
# include <Siv3D.hpp>

// シーンの名前
enum class State
{
	Title,
	Game
};

// ゲームデータ
struct GameData
{
	// ハイスコア
	int32 highScore = 0;
};

// シーン管理クラス
using MyApp = SceneManager<State, GameData>;

// タイトルシーン
class Title : public MyApp::Scene
{
private:

	Rect m_startButton = Rect(Arg::center = Scene::Center().movedBy(0, 0), 300, 60);
	Transition m_startTransition = Transition(0.4s, 0.2s);

	Rect m_exitButton = Rect(Arg::center = Scene::Center().movedBy(0, 100), 300, 60);
	Transition m_exitTransition = Transition(0.4s, 0.2s);

public:

	Title(const InitData& init)
		: IScene(init) {}

	void update() override
	{
		m_startTransition.update(m_startButton.mouseOver());
		m_exitTransition.update(m_exitButton.mouseOver());

		if (m_startButton.mouseOver() || m_exitButton.mouseOver())
		{
			Cursor::RequestStyle(CursorStyle::Hand);
		}

		if (m_startButton.leftClicked())
		{
			changeScene(State::Game);
		}

		if (m_exitButton.leftClicked())
		{
			System::Exit();
		}
	}

	void draw() const override
	{
		const String titleText = U"ブロックくずし";
		const Vec2 center(Scene::Center().x, 120);
		FontAsset(U"Title")(titleText).drawAt(center.movedBy(4, 6), ColorF(0.0, 0.5));
		FontAsset(U"Title")(titleText).drawAt(center);

		m_startButton.draw(ColorF(1.0, m_startTransition.value())).drawFrame(2);
		m_exitButton.draw(ColorF(1.0, m_exitTransition.value())).drawFrame(2);

		FontAsset(U"Menu")(U"はじめる").drawAt(m_startButton.center(), ColorF(0.25));
		FontAsset(U"Menu")(U"おわる").drawAt(m_exitButton.center(), ColorF(0.25));

		Rect(0, 500, Scene::Width(), Scene::Height() - 500)
			.draw(Arg::top = ColorF(0.0, 0.0), Arg::bottom = ColorF(0.0, 0.5));

		const int32 highScore = getData().highScore;
		FontAsset(U"Score")(U"High score: {}"_fmt(highScore)).drawAt(Vec2(620, 550));
	}
};

// ゲームシーン
class Game : public MyApp::Scene
{
private:

	// ブロックのサイズ
	static constexpr Size blockSize = Size(40, 20);

	// ボールの速さ
	static constexpr double speed = 480.0;

	// ブロックの配列
	Array<Rect> m_blocks;

	// ボールの速度
	Vec2 m_ballVelocity = Vec2(0, -speed);

	// ボール
	Circle m_ball = Circle(400, 400, 8);

	// パドル
	Rect m_paddle = Rect(Arg::center(Cursor::Pos().x, 500), 60, 10);

	// スコア
	int32 m_score = 0;

public:

	Game(const InitData& init)
		: IScene(init)
	{
		// 横 (Scene::Width() / blockSize.x) 個、縦 5 個のブロックを配列に追加する
		for (auto p : step(Size((Scene::Width() / blockSize.x), 5)))
		{
			m_blocks << Rect(p.x * blockSize.x, 60 + p.y * blockSize.y, blockSize);
		}
	}

	void update() override
	{
		// パドルを操作
		m_paddle = Rect(Arg::center(Cursor::Pos().x, 500), 60, 10);

		// ボールを移動
		m_ball.moveBy(m_ballVelocity * Scene::DeltaTime());

		// ブロックを順にチェック
		for (auto it = m_blocks.begin(); it != m_blocks.end(); ++it)
		{
			// ボールとブロックが交差していたら
			if (it->intersects(m_ball))
			{
				// ボールの向きを反転する
				(it->bottom().intersects(m_ball) || it->top().intersects(m_ball) ? m_ballVelocity.y : m_ballVelocity.x) *= -1;

				// ブロックを配列から削除（イテレータが無効になるので注意）
				m_blocks.erase(it);

				// スコアを加算
				++m_score;

				// これ以上チェックしない  
				break;
			}
		}

		// 天井にぶつかったらはね返る
		if (m_ball.y < 0 && m_ballVelocity.y < 0)
		{
			m_ballVelocity.y *= -1;
		}

		if (m_ball.y > Scene::Height())
		{
			changeScene(State::Title);
			getData().highScore = Max(getData().highScore, m_score);
		}

		// 左右の壁にぶつかったらはね返る
		if ((m_ball.x < 0 && m_ballVelocity.x < 0) || (Scene::Width() < m_ball.x && m_ballVelocity.x > 0))
		{
			m_ballVelocity.x *= -1;
		}

		// パドルにあたったらはね返る
		if (m_ballVelocity.y > 0 && m_paddle.intersects(m_ball))
		{
			// パドルの中心からの距離に応じてはね返る向きを変える
			m_ballVelocity = Vec2((m_ball.x - m_paddle.center().x) * 10, -m_ballVelocity.y).setLength(speed);
		}
	}

	void draw() const override
	{
		FontAsset(U"Score")(m_score).drawAt(Scene::Center().x, 30);

		// すべてのブロックを描画する
		for (const auto& block : m_blocks)
		{
			block.stretched(-1).draw(HSV(block.y - 40));
		}

		// ボールを描く
		m_ball.draw();

		// パドルを描く
		m_paddle.draw();
	}
};

void Main()
{
	// 使用するフォントアセットを登録
	FontAsset::Register(U"Title", 120, U"example/font/AnnyantRoman/AnnyantRoman.ttf");
	FontAsset::Register(U"Menu", 30, Typeface::Regular);
	FontAsset::Register(U"Score", 36, Typeface::Bold);

	// 背景色を設定
	Scene::SetBackground(ColorF(0.2, 0.8, 0.4));

	// シーンと遷移時の色を設定
	MyApp manager;
	manager
		.add<Title>(State::Title)
		.add<Game>(State::Game)
		.setFadeColor(ColorF(1.0));

	while (System::Update())
	{
		if (!manager.update())
		{
			break;
		}
	}
}
```


## タイピングゲームの基本
![](https://github.com/Siv3D/siv3d.docs.images/blob/master/sample/game-template/typing.gif?raw=true)
```C++
# include <Siv3D.hpp>

void Main()
{
	Scene::SetBackground(ColorF(0.8, 0.9, 1.0));

	// 問題文のリスト
	const Array<String> texts =
	{
		U"Practice makes perfect.",
		U"Don't cry over spilt milk.",
		U"Faith will move mountains.",
		U"Nothing ventured, nothing gained.",
		U"Bad news travels fast.",
	};

	String target = texts.choice(), input;

	const Font font(40, Typeface::Bold);

	while (System::Update())
	{
		// テキスト入力（TextInputMode::DenyControl: エンターやタブ、バックスペースは受け付けない）
		TextInput::UpdateText(input, TextInputMode::DenyControl);

		// 誤った入力が含まれていたら削除
		while (!target.starts_with(input))
		{
			input.pop_back();
		}

		// 一致したら次の問題へ
		if (input == target)
		{
			target = texts.choice();
			input.clear();
		}

		font(target).draw(40, 270, ColorF(0.75));

		font(input).draw(40, 270, ColorF(0.1));
	}
}
```

