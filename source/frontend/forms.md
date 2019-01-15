---
title: Forms
description: 'Easily create and configure forms'
disqusPage: 'React:Forms'
---

# Forms

One of the nicest package out there for forms is `uniforms`.

In this chapter we'll explore how to use uniforms together with Antd design.

```js
meteor npm i -S uniforms uniforms-antd antd simpl-schema
```

Let's create our first form:

```js
// src/ui/pages/Login/index.js
import React from 'react';
import SimpleSchema from 'simpl-schema';
import { AutoForm, AutoField, ErrorField } from 'uniforms-antd';
import { Button, notification } from 'antd';

class Login extends React.Component {
  onSubmit = data => {
    const { history } = this.props;

    console.log(data);

    notification.open({
      message: 'Form submitted, check your console!',
    });
  };

  render() {
    return (
      <div className="c-Login">
        <AutoForm schema={LoginSchema} onSubmit={this.onSubmit} id="login">
          <AutoField
            name="email"
            placeholder="Enter your email address"
            label={false}
          />
          <ErrorField name="email" />

          <AutoField
            name="password"
            placeholder="Password"
            label={false}
            type="password"
          />
          <ErrorField name="password" />

          <Button type="primary" htmlType="submit">
            Login
          </Button>
        </AutoForm>
      </div>
    );
  }
}

// This contains the validation logic for the form
// Refer to https://www.npmjs.com/package/simpl-schema
const LoginSchema = new SimpleSchema({
  email: {
    type: String,
    regEx: SimpleSchema.RegEx.Email,
  },
  password: { type: String },
});

export default Login;
```

Don't forget to add it as a route:

```js
// src/ui/main/routes.js
import Login from '../../pages/Login';

const routes = [
  {
    path: '/login',
    component: Login,
  },
];
```

Now open up your browser: http://localhost:3000/login and enjoy your first form.

Please refer to uniforms documentation to find out more:
https://github.com/vazco/uniforms/blob/master/INTRODUCTION.md
