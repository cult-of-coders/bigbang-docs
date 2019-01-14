---
title: Testing
description: 'Start testing your react components'
disqusPage: 'React:Testing'
---

In this sample we are going to use [Enzyme](https://airbnb.io/enzyme/docs/api/) to test our react components:

```
meteor npm i --save-dev enzyme enzyme-adapter-react-16
```

You should already have setup your tests from the [previous chapter](../starterpack/testing.md).

```js
// src/__tests__/enzyme.js
import Enzyme from 'enzyme';
import Adapter from 'enzyme-adapter-react-16';

Enzyme.configure({ adapter: new Adapter() });
```

And import this file inside `src/__tests__/client.js`.

Now let's write a test for the previous component we built in the Form.

```js
// src/ui/components/Login/__tests__/index.js
import { expect } from 'chai';
import React from 'react';
import { mount } from 'enzyme';
import { AutoForm } from 'uniforms-antd';

import LoginPage from '../Login';

describe('Login', () => {
  it('Should render the form', () => {
    const wrapper = mount(<LoginPage />);
    expect(wrapper.find(AutoForm)).to.have.lengthOf(1);
  });
});
```

Now if you run `npm test` and open http://localhost:3050 you should see your test running!

Please refer to [Enzyme](https://airbnb.io/enzyme/docs/api/) documentation for more details about testing your components.
