description: OpenSiv3D のチュートリアル

!!! warning "このページよりも新しいドキュメントがあります"
	このドキュメントは古い OpenSiv3D v0.4.3 向けです。2021 年9 月 18 日に最新の OpenSiv3D v0.6.0 がリリースされました。最新のドキュメントは [Siv3D リファレンス v0.6.0](https://zenn.dev/reputeless/books/siv3d-documentation) です。

# 3. 動きを作る

この章では、「動き」の表現に役立つ Siv3D の機能を学びます。

## 3.1 経過時間を使ったアニメーション

### Scene::Time()
`Scene::Time()` はプログラムが起動されてからの経過時間（秒）を `double` 型の値で返します。この値を使って簡単なアニメーションを作成できます。

### Scene::Center()
画面の中心座標を返す関数です。画面のサイズが 800x600 のときには `Point(400, 300)` を返します。

<video src="https://github.com/Siv3D/siv3d.docs.images/blob/master/tutorial/3/1-0.mp4?raw=true" autoplay loop muted></video>

```C++
# include <Siv3D.hpp>

void Main()
{
	Scene::SetBackground(Palette::White);

	while (System::Update())
	{
		const double t = Scene::Time();

		// 円の半径が、時間の経過に伴って大きくなる
		Circle(Scene::Center(), t * 50).draw(ColorF(0.25));
	}
}
```

### Scene::DeltaTime()
`Scene::DeltaTime()` は、直前のフレームからの経過時間 (秒) を `double` 型の値で返します。`Scene::Time()` を使わずに、この値を加算していくことでアニメーションを作成することもできます。
```C++
# include <Siv3D.hpp>

void Main()
{
	Scene::SetBackground(Palette::White);

	double r = 0.0;

	while (System::Update())
	{
		// 経過時間分だけ大きくする
		r += Scene::DeltaTime() * 50;

		Circle(Scene::Center(), r).draw(ColorF(0.25));
	}
}
```

### step
Siv3D に用意されている、ループを短く書ける機能です。`for (auto i : step(N))` は `for (int i = 0; i < N; ++i)`と同じ働きです。

<video src="https://github.com/Siv3D/siv3d.docs.images/blob/master/tutorial/3/1-1.mp4?raw=true" autoplay loop muted></video>

```C++
# include <Siv3D.hpp>

void Main()
{
	Scene::SetBackground(Palette::White);

	while (System::Update())
	{
		const double t = Scene::Time();

		for (auto i : step(12))
		{
			// 長方形が、時間の経過に伴って横に大きくなる
			RectF(-50 * i, i * 50, t * 100, 50).draw(ColorF(0.25));
		}
	}
}
```

### OffsetCircular
円周に沿った動きをつくるのに最適な円座標クラスです。オフセット `offset` と円座標系の動径座標 `r`, 角度座標 θ (`theta`) の 3 つの要素で位置を表現します。シーン上の座標 `offset` を中心とする半径 `r` の円を考え、その円周上で 12 時の方向を 0° として時計回りに `theta` の位置が `OffsetCircular(offset, r, theta)` です。

![](https://github.com/Siv3D/siv3d.docs.images/blob/master/tutorial/3/1-2.png?raw=true)

![](https://github.com/Siv3D/siv3d.docs.images/blob/master/tutorial/3/1-3.gif?raw=true)

```C++
# include <Siv3D.hpp>

void Main()
{
	Scene::SetBackground(Palette::White);

	// 中心位置のオフセット
	const Vec2 center = Scene::Center();
	
	// 円座標系における動径座標
	constexpr double r = 200.0;

	while (System::Update())
	{
		const double t = Scene::Time();

		for (auto i : step(12))
		{
			// 円座標系における角度座標
			const double theta = i * 30_deg + t * 30_deg;

			const Vec2 pos = OffsetCircular(center, r, theta);

			Circle(pos, 20).draw(ColorF(0.25));
		}
	}
}
```


## 3.2 ストップウォッチ
経過時間の計測やリセットを便利に行える `Stopwatch` クラスがあります。

### Stopwatch の経過時間とリスタート

`Stopwatch` のコンストラクタ引数に `true` を渡すと、作成と同時に計測を開始します。`Stopwatch::sF()` はその時点での経過時間（秒）を `double` 型で返します。`Stopwatch::restart()` すると、経過時間をリセットして再び 0 から計測を開始（リスタート）します。

### MouseL.down()
マウスの左ボタンがクリック（タッチディスプレイの場合は画面がタッチ）されたかを、`if (MouseL.down())` で調べられます。次のサンプルでは、画面上をマウスでクリックするたびに `Stopwatch` をリスタートします。

<video src="https://github.com/Siv3D/siv3d.docs.images/blob/master/tutorial/3/2-0.mp4?raw=true" autoplay loop muted></video>

```C++
# include <Siv3D.hpp>

void Main()
{
	Scene::SetBackground(Palette::White);

	const Vec2 center = Scene::Center();

	// ストップウォッチ（作成と同時に計測開始）
	Stopwatch stopwatch(true);

	while (System::Update())
	{
		// もし左クリックされたら
		if (MouseL.down())
		{
			// ストップウォッチをリセットして再び 0 から計測
			stopwatch.restart();
		}

		// ストップウォッチの経過時間（秒）を double 型で取得 
		const double t = stopwatch.sF();

		// シーンの中心の円の半径が、時間の経過に伴って大きくなる
		Circle(center, t * 50).draw(ColorF(0.25));
	}
}
```

### Stopwatch の一時停止と再開

ストップウォッチが計測中かどうかは `if (Stopwatch::isRunning())` で調べられます。ストップウォッチの計測を一時停止するには `Stopwatch::pause()`, 一時停止を解除して計測を再開するには `Stopwatch::resume()` します。

<video src="https://github.com/Siv3D/siv3d.docs.images/blob/master/tutorial/3/2-1.mp4?raw=true" autoplay loop muted></video>

```C++
# include <Siv3D.hpp>

void Main()
{
	Scene::SetBackground(Palette::White);

	const Vec2 center = Scene::Center();

	// ストップウォッチ（作成と同時に計測開始）
	Stopwatch stopwatch(true);

	while (System::Update())
	{
		if (MouseL.down())
		{
			// ストップウォッチが計測中なら
			if (stopwatch.isRunning())
			{
				// ストップウォッチを一時停止
				stopwatch.pause();
			}
			else // ストップウォッチが一時停止中なら
			{
				// ストップウォッチを再開
				stopwatch.resume();
			}
		}

		// ストップウォッチの経過時間（秒）を double 型で取得 
		const double t = stopwatch.sF();

		Circle(center, 120).drawArc(t * 140_deg, 240_deg, 60, 0, ColorF(0.4));

		Circle(center, 180).drawArc(t * 90_deg, 160_deg, 60, 0, ColorF(0.6));

		Circle(center, 240).drawArc(t * 50_deg, 120_deg, 60, 0, ColorF(0.8));
	}
}
```

## 3.3 周期的なアニメーション
Siv3D で周期的に移動・点滅・拡大縮小するようなアニメーションを作るときには、`Periodic::` 名前空間に用意されている関数群を使うと便利です。

### Periodic::Square0_1()
指定した周期で 0.0 か 1.0 を交互に返す関数です。周期は `2s` (2 秒) や `0.5s` (0.5 秒) のように時間リテラルを使って記述します。周期の前半では 1.0 を、残りの半分では 0.0 を返します。

<video src="https://github.com/Siv3D/siv3d.docs.images/blob/master/tutorial/3/3-0.mp4?raw=true" autoplay loop muted></video>

```C++
# include <Siv3D.hpp>

void Main()
{
	Scene::SetBackground(ColorF(0.25));

	while (System::Update())
	{
		// 2 秒周期（1 秒点灯、1 秒消灯）で明滅を繰り返す
		if (Periodic::Square0_1(2s))
		{
			Circle(Scene::Center(), 200).draw();
		}
	}
}
```

### Periodic::Triangle0_1()
0.0 から一定の速度で値が大きくなって 1.0 に、そして一定の速度で小さくなって 0.0 に、という変化を指定した周期で繰り返す関数です。

![](https://github.com/Siv3D/siv3d.docs.images/blob/master/tutorial/3/3-1.gif?raw=true)

```C++
# include <Siv3D.hpp>

void Main()
{
	Scene::SetBackground(ColorF(0.25));

	while (System::Update())
	{
		// 2 秒周期で、一定速度での左右移動を繰り返す
		const double x = 50 + 700 * Periodic::Triangle0_1(2s);
		
		Circle(x, 300, 50).draw();
	}
}
```

### Periodic::Sine0_1()
指定した周期で、0.0～1.0 の範囲で正弦波（サインカーブ）を描く数値の変化を返す関数です。

![](https://github.com/Siv3D/siv3d.docs.images/blob/master/tutorial/3/3-2.gif?raw=true)

```C++
# include <Siv3D.hpp>

void Main()
{
	Scene::SetBackground(ColorF(0.25));

	while (System::Update())
	{
		// 2 秒周期で、サインカーブの速度での左右移動を繰り返す
		const double x = 50 + 700 * Periodic::Sine0_1(2s);
		
		Circle(x, 300, 50).draw();
	}
}
```

### Periodic::Sawtooth0_1()
指定した周期で、0.0 → 1.0 への変化を繰り返す関数です。

![](https://github.com/Siv3D/siv3d.docs.images/blob/master/tutorial/3/3-3.gif?raw=true)

```C++
# include <Siv3D.hpp>

void Main()
{
	Scene::SetBackground(ColorF(0.25));

	while (System::Update())
	{
		// 2 秒周期で、左 → 右への移動を繰り返す 
		const double x = 50 + 700 * Periodic::Sawtooth0_1(2s);
		
		Circle(x, 300, 50).draw();
	}
}
```

### Periodic::Jump0_1()
指定した周期で、地面からジャンプしたときの速度のような数値変化を繰り返す関数です。

![](https://github.com/Siv3D/siv3d.docs.images/blob/master/tutorial/3/3-4.gif?raw=true)

```C++
# include <Siv3D.hpp>

void Main()
{
	Scene::SetBackground(ColorF(0.25));

	while (System::Update())
	{
		// 2 秒周期で、ジャンプのような移動を繰り返す 
		const double h = 500 * Periodic::Jump0_1(2s);
		
		Circle(400, 550 - h, 50).draw();
	}
}
```

## 3.4 トランジション

### Transition
値が少しずつ大きくなって最大値に到達する。そこから徐々に小さくなって最小値に戻る、といった挙動をプログラムするときには `Transition` を使うと便利です。`Transition` のコンストラクタには、最小値から最大値に増加する所要時間と、最大値から最小値に減少する所要時間を設定します。あとは毎フレーム、`Transition::update()` に、増加の場合は `true` を、減少の場合は `false` を渡せば、設定された速度で値が変化します。`Transitgion::value()` で現在の値を取得できます。

### MouseL.pressed()
マウスの左ボタンが押されている（タッチディスプレイの場合は画面がタッチされている）かを、`if (MouseL.pressed())` で調べられます。次のサンプルでは、左ボタンが押されていると扇形が大きくなり、離されていると小さくなります。

![](https://github.com/Siv3D/siv3d.docs.images/blob/master/tutorial/3/4-0.gif?raw=true)

```C++
# include <Siv3D.hpp>

void Main()
{
	Scene::SetBackground(ColorF(0.25));

	// 2.0 秒かけて 0.0 から 1.0 になる速度で増加し
	// 0.5 秒かけて 1.0 から 0.0 になる速度で減少するトランジション
	Transition transition(2.0s, 0.5s);

	while (System::Update())
	{
		if (MouseL.pressed())
		{
			// マウスの左ボタンが押されていたら増加
			transition.update(true);
		}
		else
		{
			// 押されていなかったら減少
			transition.update(false);
		}

		const double t = transition.value();

		Circle(Scene::Center(), 200).drawPie(0_deg, 360_deg * t);
	}
}
```

`MouseL.pressed()` は `bool` 型の値を返すので、このプログラムは次のように短く書けます。

```C++ hl_lines="14"
# include <Siv3D.hpp>

void Main()
{
	Scene::SetBackground(ColorF(0.25));

	// 2.0 秒かけて 0.0 から 1.0 になる速度で増加し、
	// 0.5 秒かけて 1.0 から 0.0 になる速度で減少するトランジション
	Transition transition(2.0s, 0.5s);

	while (System::Update())
	{
		// マウスの左ボタンが押されていたら増加、押されていなかったら減少
		transition.update(MouseL.pressed());

		const double t = transition.value();

		Circle(Scene::Center(), 200).drawPie(0_deg, 360_deg * t);
	}
}
```


## 3.5 イージング

### Min, Max
`Min()` 関数は、与えられた引数の中の最小値を返します。`Max()` 関数は最大値を返します。

### 線形補間
あるベクトル A から別のベクトル B への線形補間は `A.lerp(B, t)` で計算できます。A と B の中間のベクトルは `A.lerp(B, 0.5)` で計算します。

![](https://github.com/Siv3D/siv3d.docs.images/blob/master/tutorial/3/5-0.gif?raw=true)

```C++ hl_lines="20 23"
# include <Siv3D.hpp>

void Main()
{
	Scene::SetBackground(ColorF(0.25));

	constexpr Vec2 begin(100, 300);
	constexpr Vec2 end(700, 300);

	Stopwatch stopwatch;

	while (System::Update())
	{
		if (MouseL.down())
		{
			stopwatch.restart();
		}

		// 移動の割合 0.0～1.0
		const double t = Min(stopwatch.sF(), 1.0);

		// begin と end の線形補間
		const Vec2 pos = begin.lerp(end, t);

		Circle(pos, 40).draw();
	}
}
```

### イージング
0.0 から 1.0 に一定の割合で値を増加させるだけでは単調な動きになってしまいます。はじめは少しずつ加速し、ゴールに近づくとゆっくりになるといったように、速度に変化を与えると、より洗練された視覚効果を実現できます。0.0 → 1.0 の単調増加を、特徴的なカーブに変換できる **イージング関数** を取り入れて、アニメーションの印象を改善しましょう。

![](https://github.com/Siv3D/siv3d.docs.images/blob/master/tutorial/3/5-1.gif?raw=true)

```C++ hl_lines="23 26"
# include <Siv3D.hpp>

void Main()
{
	Scene::SetBackground(ColorF(0.25));

	constexpr Vec2 begin(100, 300);
	constexpr Vec2 end(700, 300);

	Stopwatch stopwatch;

	while (System::Update())
	{
		if (MouseL.down())
		{
			stopwatch.restart();
		}

		// 移動の割合 0.0～1.0
		const double t = Min(stopwatch.sF(), 1.0);

		// イージング関数を適用
		const double e = EaseInOutExpo(t);

		// begin と end の線形補間
		const Vec2 pos = begin.lerp(end, e);

		Circle(pos, 40).draw();
	}
}
```

イージング関数は全部で約 30 種類用意されています。一覧は [Easing Functions Cheat Sheet](https://easings.net/) で確認できます。`EaseInOutExpo()` 以外にも、`EaseOutBounce()` や `EaseInOutBack()` など様々なイージング関数を試してみましょう。