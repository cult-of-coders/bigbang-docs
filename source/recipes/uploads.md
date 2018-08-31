---
title: Uploads
description: 'Let\'s upload a file using the form!'
disqusPage: 'Recipes:Uploads'
---

# Uploads

Let's create a simple function that allows us to store our uploads in our temp directory:

```
meteor npm i -S shortid
```

```js
// src/api/uploads/storeFS.js
import os from 'os';
import fs from 'fs';
import shortid from 'shortid';

export default ({ stream, filename }) => {
  const id = shortid.generate();
  const path = `${os.tmpdir()}/${id}-${filename}`;
  return new Promise((resolve, reject) =>
    stream
      .on('error', error => {
        if (stream.truncated)
          // Delete the truncated file
          fs.unlinkSync(path);
        reject(error);
      })
      .pipe(fs.createWriteStream(path))
      .on('error', error => reject(error))
      .on('finish', () => resolve({ id, path }))
  );
};
```

Now let's define the GraphQL module:

```js
// src/api/uploads/index.js
import storeFS from './storeFS';

export default {
  typeDefs: `
    type Mutation {
      uploadFile(file: Upload!): Boolean
    }
  `,
  resolvers: {
    Mutation: {
      async uploadFile(_, args) {
        const { file } = args;
        const { stream, filename } = await file;

        const { id, path } = await storeFS({ stream, filename });

        console.log(`File was saved here: `, path);

        return true;
      },
    },
  },
};
```

Before we define our component, we need to define a field that we can connect to uniforms for uploading, like this:

```js
import React from 'react';
import connectField from 'uniforms/connectField';
import { Upload, Button, Icon } from 'antd';

const UploadField = ({ label, onChange }) => {
  const uploadProps = {
    onChange({ file }) {
      if (file.status === 'done') {
        onChange(file.originFileObj);
      }
    },
  };

  return (
    <Upload {...uploadProps}>
      <Button>
        <Icon type="upload" /> {label}
      </Button>
    </Upload>
  );
};

export default connectField(UploadField);
```

Perfect, now our Upload Form will look like this:

```js
import React from 'react';
import { Mutation } from 'react-apollo';
import gql from 'graphql-tag';
import { AutoForm } from 'uniforms-antd';
import SimpleSchema from 'simpl-schema';
import UploadField from './UploadField';

const FileUploadSchema = new SimpleSchema({
  file: {
    type: Object,
    blackbox: true,
    optional: true,
    // Yes you can specify uniforms function directly in the schema!
    uniforms: {
      label: 'Upload your file',
      component: UploadField,
    },
  },
});

const UPLOAD_FILE = gql`
  mutation UPLOAD_FILE($file: Upload!) {
    uploadFile(file: $file)
  }
`;

const ItemForm = ({ onSubmit }) => (
  <AutoForm schema={FileUploadSchema} onSubmit={onSubmit} id="file-upload" />
);

export default () => (
  <Mutation mutation={UPLOAD_FILE}>
    {mutate => {
      return (
        <ItemForm
          onSubmit={({ file }) =>
            mutate({ variables: { file } }).then(response =>
              console.log(`I received`, response)
            )
          }
        />
      );
    }}
  </Mutation>
);
```

That's it, you can now upload files in your forms with ease!
