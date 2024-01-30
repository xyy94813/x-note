# CSS BEM

**BEM（Block Element Modifiers）** 是 CSS 命名规范的一种，可以很好的帮助创建可复用“组件”。

## 核心概念

BEM 如字面一样由三部分组成。

- Block。一个有意义的独立的实体
- Element。Block 中的一部分，与 Block 紧密相关，无法独立存在
- Modifiers。标记 Block 或 Element，用于改变外观或行为。

命名格式为：`Block__Element--Modifier`

## Example

```html
<!-- 一个卡片组件 -->
<div className="card">
  <div className="card__picture"></div>
  <div className="card__title"></div>
  <div className="card__desc"></div>
  <button className="card__btn card__btn--active"></button>
</div>
```

## 参考

- [You Probably Need BEM CSS in Your Life (Tutorial)](https://www.youtube.com/watch?v=er1JEDuPbZQ)
