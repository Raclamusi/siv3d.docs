
!!! warning "This is the documentation for an old version"
	This is the documentation for an old version of Siv3D (v0.4.3). See [Siv3D Reference v0.6.0](https://zenn.dev/reputeless/books/siv3d-documentation-en) for the latest version.

# kd-tree

## 半径による探索と k 近傍法による探索

<video src="https://github.com/Siv3D/siv3d.docs.images/blob/master/reference/kd-tree/0.mp4?raw=true" autoplay loop muted></video>

```C++
# include <Siv3D.hpp>

struct Unit
{
	Circle circle;

	ColorF color;

	void draw() const
	{
		circle.draw(color);
	}
};

// Unit を KDTree で扱えるようにするためのアダプタ
struct UnitAdapter : KDTreeAdapter<Array<Unit>, Vec2, double, 2>
{
	static const element_type* GetPointer(const point_type& point)
	{
		return &point.x;
	}

	static element_type GetElement(const dataset_type& dataset, size_t index, size_t dim)
	{
		return dataset[index].circle.center.elem(dim);
	}

	static element_type DistanceSq(const dataset_type& dataset, size_t index, const element_type* other)
	{
		return dataset[index].circle.center.distanceFromSq(Vec2(other[0], other[1]));
	}
};

void Main()
{
	// 4000 個の Unit を生成
	Array<Unit> units;
	for (size_t i = 0; i < 4000; ++i)
	{
		Unit unit;
		unit.circle = Circle(RandomVec2(Circle(0, 0, 100)), 0.25);
		unit.color = RandomColorF();
		units << unit;
	}

	// kd-tree を構築
	KDTree<UnitAdapter> kdTree(units);

	// 探索の種類（ラジオボタンのインデックス）
	size_t searchTypeIndex = 0;

	// radius search する際の探索距離
	double searchDistance = 4.0;

	// 2D カメラ
	Camera2D camera(Vec2(0, 0), 24);

	while (System::Update())
	{
		// 2D カメラの更新
		camera.update();

		// 画面内のユニットだけ処理するための基準の長方形
		const RectF viewRect = camera.getRegion();
		const RectF viewRectScaled = viewRect.scaledAt(viewRect.center(), 1.2);
		{
			auto t = camera.createTransformer();

			const Vec2 cursorPos = Cursor::PosF();

			if (searchTypeIndex == 0) // radius search
			{
				Circle(cursorPos, searchDistance).draw(ColorF(1.0, 0.2));

				// searchDistance 以内の距離にある Unit のインデックスを取得
				for (auto index : kdTree.radiusSearch(cursorPos, searchDistance))
				{
					Line(cursorPos, units[index].circle.center).draw(0.1);
				}
			}
			else // k-NN search
			{
				const size_t k = (searchTypeIndex == 1) ? 1 : 5;

				// 最も近い k 個の Unit のインデックスを取得
				for (auto index : kdTree.knnSearch(k, cursorPos))
				{
					Line(cursorPos, units[index].circle.center).draw(0.1);
				}
			}

			// ユニットを描画
			for (const auto& unit : units)
			{
				// 描画負荷削減のため、画面内 (viewRectScaled) に無ければスキップ
				if (!unit.circle.center.intersects(viewRectScaled))
				{
					continue;
				}

				unit.draw();
			}
		}

		SimpleGUI::RadioButtons(searchTypeIndex, { U"radius", U"k-NN (k=1)", U"k-NN (k=5)" }, Vec2(20, 20));
		SimpleGUI::Slider(U"searchDistance", searchDistance, 0.0, 20.0, Vec2(180, 20), 160, 120, (searchTypeIndex == 0));
		if (SimpleGUI::Button(U"Move units", Vec2(180, 60)))
		{
			// Unit をランダムに移動
			for (auto& unit : units)
			{
				unit.circle.moveBy(RandomVec2(0.5));
			}

			// Unit の座標が更新されたので kd-tree を再構築
			kdTree.rebuildIndex();
		}

		camera.draw(Palette::Orange);
	}
}
```


