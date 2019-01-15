---
title: Styling
description: 'Setup styles in a proper intuitive manner'
disqusPage: 'Starterpack:Styling'
---

# Styling

Oh the many possibilities, the many ways to do this. I agree, there are many ways and the problem is that there are many good ways.

However, `BigBang` is opinionated and tries to enforce certain paradigms to simplify the life, so this is our approach:

```
meteor add fourseven:scss
meteor remove standard-minifier-css
meteor add seba:minifiers-autoprefixer
```

Now, even if we are in `src` all `scss` will get eagerly loaded if we don't provide them with the `_` prefix (`buttons.scss` vs `_buttons_.scss`)

So, because we don't want any eager loading, we'll have a `main.scss` and the rest will be imported from there.

```
mkdir -p src/ui/styles
touch src/ui/styles/main.scss
```

We love and recommend [Ant Design](https://ant.design/docs/react/introduce). Let's see how we can add it quickly:

```
meteor npm i -S antd
```

```js
cd src/ui/styles
ln -s ../../../node_modules/antd/dist/antd.min.css _antd.scss
```

```scss
// src/ui/styles/main.scss
@import '_antd';
@import '_variables';
```

Spin-up some variables:

```scss
// src/ui/styles/_variables.scss
$primaryColor: #333;
```

We recommend that you only write scss for presentational components which lie in `src/ui/components` something like this:

```js
// src/ui/components/Button/index.js
export default ({ children }) => {
  return <button className="c-Button">{children}</button>;
};
```

```scss
// src/ui/components/Button/_style.scss
.c-Button {
  padding: 15px;
}
```

```scss
// src/ui/components/_index.scss
@import './Button/_style.scss';
@import './Input/_style.scss';
@import './Form/_style.scss';
```

And finally import it in your `main.scss`:

```scss
@import '_antd';
@import '_variables';
@import '../components/_index';
```

Note: keep all your presentational components unique so you won't have any css class collisions.

It's important that you follow guidelines and good styling principles: https://sass-guidelin.es/

Enjoy!
