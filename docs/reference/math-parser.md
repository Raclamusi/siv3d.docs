
!!! warning "This is the documentation for an old version"
	This is the documentation for an old version of Siv3D (v0.4.3). See [Siv3D Reference v0.6.0](https://zenn.dev/reputeless/books/siv3d-documentation-en) for the latest version.

# Math parser

## 数式の計算
`Eval()` に数式を渡すと、`double` 型の精度での計算結果を返します。`EvalOpt()` は戻り値の型が `Optional<double>` で、数式にエラーがある場合は `none` を返します。

![](https://github.com/Siv3D/siv3d.docs.images/blob/master/reference/math-parser/0.gif?raw=true)

```C++
# include <Siv3D.hpp>

void Main()
{
	Scene::SetBackground(ColorF(0.8, 0.9, 1.0));

	const Font font(30, Typeface::Bold);

	TextEditState tes;

	Optional<double> result = 0.0;

	while (System::Update())
	{
		// 数式を入力するテキストボックス
		if (SimpleGUI::TextBox(tes, Vec2(20, 20), 700))
		{
			// 結果を取得（エラーの場合 none）
			result = EvalOpt(tes.text);
		}

		if (result)
		{
			font(result.value()).draw(Rect(20, 100, 760, 500), ColorF(0.25));
		}
		else
		{
			font(U"Error").draw(20, 100, ColorF(0.25));
		}
	}
}
```


