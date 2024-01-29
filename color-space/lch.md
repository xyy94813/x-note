# LCH

LCH 色彩空间，也称为 HCL、CIELAB、LAB、CIELUV 或 CIELCH。

**其目标是以更类似于人眼感知颜色的方式表示颜色。**
这种被称为感知均匀性的品质，使 LCH 在 UI 设计中成为绝佳选择。

[lch playground](https://lch.oklch.com/)

## 历史背景

LCH 由国际照明委员会(CIE) 于 1976 年首次定义的色彩空间，因此命名为 CIELAB。
主要目标是实现感知一致性，这意味着它的行为取决于我们的眼睛感知颜色的方式。

但是，2005 年 Sarifuddin 注意到 CIELAB 缺乏蓝色色调一致性。

## 色彩空间组成

LCH 由亮度（Lightness）、色度（Chroma）和色调（Hue）三个方向组成。

- Hue 从红色开始，逐渐经过绿色、蓝色，最后回到红色。
- Lightness 取值范围通常为 0 到 100
- Chroma 的范围从 0 到大约 131

![LCH 坐标系](./images/color-space-LCH-coordinate-system.jpeg)

## 对比其它色彩空间

与 HSL 相比：

- LCH 感知均匀性、设备无关性、色彩范围更广
- HSL 颜色空间更符合人类对颜色的感知，包括色相、饱和度和亮度这三个直观的属性。也因此更易于调整

![LCH 和 HSL 亮度上的差异](./images/the-same-colour-in-HSL-and-LCH-but-brightness-changing.webp)
the-difference-between-LCH-Chroma-and-HSL-Saturation

![LCH 和 HUE 在 HUE 上的差异](./images/the-differences-in-hue-between-LCH-and-HUE.webp)

![LCH 色度与 HSL 饱和度的不同](./images/LCH-Chromaticity-vs-HSL-Saturation.webp)

与 sRGB 对比：

- LCH 感知均匀性、设备无关性、色彩范围更广
- sRGB 基于历史原因，设备兼容性非常良好，色彩表示相对直观

## 衍生色彩空间 -- OKLCH

okLCH 是对 LCH 的改进和扩展，主要解决了 LCH 蓝色色调一致性问题（在 270 和 的色调值之间 330）

[oklch playground](https://oklch.com/)

## 参考资料

- [Understanding the CIE L*C*h Color Space](https://sensing.konicaminolta.us/us/blog/understanding-the-cie-lch-color-space/)
- [LCH is the best color space!](https://atmos.style/blog/lch-color-space)
- [wikipedia: HCL color space](https://en.wikipedia.org/wiki/HCL_color_space)
- [OKLCH in CSS: why we moved from RGB and HSL](https://evilmartians.com/chronicles/oklch-in-css-why-quit-rgb-hsl)
