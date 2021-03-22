# 在管理面板中创建一个新的 Field

在本指南中，我们将了解如何为管理面板创建新的 Field。

## 引言

对于这个例子，我们将看到如何在 **`内容管理器`** 插件中通过创建一个新的插件来改变带有 [CKEditor](https://ckeditor.com/ckeditor-5/) 的所见即所得(WYSIWYG) ，这个插件将在你的应用程序中添加一个新的 **Field**。

## 设置

1. 创建一个新项目:

:::: tabs

::: tab yarn

```
# Create an application using SQLite and prevent the server from starting automatically as we will create a plugin
# right after the project generation
yarn create strapi-app my-app --quickstart --no-run
```

:::

::: tab npx

```
# Create an application using SQLite and prevent the server from starting automatically as we will create a plugin
# right after the project generation
npx create-strapi-app my-app --quickstart --no-run
```

:::

::::

2. 生成一个插件:

:::: tabs

::: tab yarn

```
cd my-app
yarn strapi generate:plugin wysiwyg
```

:::

::: tab npm

```
cd my-app
npm run strapi generate:plugin wysiwyg
```

:::

::: tab strapi

```
cd my-app
strapi generate:plugin wysiwyg
```

:::

::::

3. 安装所需的依赖项:

:::: tabs

::: tab yarn

```
cd plugins/wysiwyg
yarn add @ckeditor/ckeditor5-react @ckeditor/ckeditor5-build-classic
```

:::

::: tab npm

```
cd plugins/wysiwyg
npm install @ckeditor/ckeditor5-react @ckeditor/ckeditor5-build-classic
```

:::

::::

4. 使用前端开发模式启动应用程序:

:::: tabs

::: tab yarn

```
# Go back to to strapi root folder
cd ../..
yarn develop --watch-admin
```

:::

::: tab npm

```
# Go back to to strapi root folder
cd ../..
npm run develop -- --watch-admin
```

:::

::: tab strapi

```
# Go back to to strapi root folder
cd ../..
strapi develop --watch-admin
```

:::

::::

一旦这一步完成，我们需要做的就是创建我们的新所见即所得，这将取代默认的 **内容管理器** 插件之一。

### 创建 WYSIWYG

在这一部分，我们将创建三个组件:

- MediaLib 它将用于在编辑器中插入媒体
- Wysiwyg 它将用一个标签和错误包装 CKEditor
- CKEditor 这将是新的 WYSIWYG 的实现

### 创建 MediaLib

**Path —** `./plugins/wysiwyg/admin/src/components/MediaLib/index.js`

```js
import React, { useEffect, useState } from 'react';
import { useStrapi, prefixFileUrlWithBackendUrl } from 'strapi-helper-plugin';
import PropTypes from 'prop-types';

const MediaLib = ({ isOpen, onChange, onToggle }) => {
  const {
    strapi: {
      componentApi: { getComponent },
    },
  } = useStrapi();
  const [data, setData] = useState(null);
  const [isDisplayed, setIsDisplayed] = useState(false);

  useEffect(() => {
    if (isOpen) {
      setIsDisplayed(true);
    }
  }, [isOpen]);

  const Component = getComponent('media-library').Component;

  const handleInputChange = data => {
    if (data) {
      const { url } = data;

      setData({ ...data, url: prefixFileUrlWithBackendUrl(url) });
    }
  };

  const handleClosed = () => {
    if (data) {
      onChange(data);
    }

    setData(null);
    setIsDisplayed(false);
  };

  if (Component && isDisplayed) {
    return (
      <Component
        allowedTypes={['images', 'videos', 'files']}
        isOpen={isOpen}
        multiple={false}
        noNavigation
        onClosed={handleClosed}
        onInputMediaChange={handleInputChange}
        onToggle={onToggle}
      />
    );
  }

  return null;
};

MediaLib.defaultProps = {
  isOpen: false,
  onChange: () => {},
  onToggle: () => {},
};

MediaLib.propTypes = {
  isOpen: PropTypes.bool,
  onChange: PropTypes.func,
  onToggle: PropTypes.func,
};

export default MediaLib;
```

#### 创建 WYSIWYG Wrapper

**Path —** `./plugins/wysiwyg/admin/src/components/Wysiwyg/index.js`

```js
import React, { useState } from 'react';
import PropTypes from 'prop-types';
import { isEmpty } from 'lodash';
import { Button } from '@buffetjs/core';
import { Label, InputDescription, InputErrors } from 'strapi-helper-plugin';
import Editor from '../CKEditor';
import MediaLib from '../MediaLib';

const Wysiwyg = ({
  inputDescription,
  errors,
  label,
  name,
  noErrorsDescription,
  onChange,
  value,
}) => {
  const [isOpen, setIsOpen] = useState(false);
  let spacer = !isEmpty(inputDescription) ? <div style={{ height: '.4rem' }} /> : <div />;

  if (!noErrorsDescription && !isEmpty(errors)) {
    spacer = <div />;
  }

  const handleChange = data => {
    if (data.mime.includes('image')) {
      const imgTag = `<p><img src="${data.url}" caption="${data.caption}" alt="${data.alternativeText}"></img></p>`;
      const newValue = value ? `${value}${imgTag}` : imgTag;

      onChange({ target: { name, value: newValue } });
    }

    // Handle videos and other type of files by adding some code
  };

  const handleToggle = () => setIsOpen(prev => !prev);

  return (
    <div
      style={{
        marginBottom: '1.6rem',
        fontSize: '1.3rem',
        fontFamily: 'Lato',
      }}
    >
      <Label htmlFor={name} message={label} style={{ marginBottom: 10 }} />
      <div>
        <Button color="primary" onClick={handleToggle}>
          MediaLib
        </Button>
      </div>
      <Editor name={name} onChange={onChange} value={value} />
      <InputDescription
        message={inputDescription}
        style={!isEmpty(inputDescription) ? { marginTop: '1.4rem' } : {}}
      />
      <InputErrors errors={(!noErrorsDescription && errors) || []} name={name} />
      {spacer}
      <MediaLib onToggle={handleToggle} isOpen={isOpen} onChange={handleChange} />
    </div>
  );
};

Wysiwyg.defaultProps = {
  errors: [],
  inputDescription: null,
  label: '',
  noErrorsDescription: false,
  value: '',
};

Wysiwyg.propTypes = {
  errors: PropTypes.array,
  inputDescription: PropTypes.oneOfType([
    PropTypes.string,
    PropTypes.func,
    PropTypes.shape({
      id: PropTypes.string,
      params: PropTypes.object,
    }),
  ]),
  label: PropTypes.oneOfType([
    PropTypes.string,
    PropTypes.func,
    PropTypes.shape({
      id: PropTypes.string,
      params: PropTypes.object,
    }),
  ]),
  name: PropTypes.string.isRequired,
  noErrorsDescription: PropTypes.bool,
  onChange: PropTypes.func.isRequired,
  value: PropTypes.string,
};

export default Wysiwyg;
```

#### Implementing CKEditor

**Path —** `./plugins/wysiwyg/admin/src/components/CKEditor/index.js`

```js
import React from 'react';
import PropTypes from 'prop-types';
import { CKEditor } from '@ckeditor/ckeditor5-react';
import ClassicEditor from '@ckeditor/ckeditor5-build-classic';
import styled from 'styled-components';

const Wrapper = styled.div`
  .ck-editor__main {
    min-height: 200px;
    > div {
      min-height: 200px;
    }
  }
`;

const configuration = {
  toolbar: [
    'heading',
    '|',
    'bold',
    'italic',
    'link',
    'bulletedList',
    'numberedList',
    '|',
    'indent',
    'outdent',
    '|',
    'blockQuote',
    'insertTable',
    'mediaEmbed',
    'undo',
    'redo',
  ],
};

const Editor = ({ onChange, name, value }) => {
  return (
    <Wrapper>
      <CKEditor
        editor={ClassicEditor}
        config={configuration}
        data={value}
        onChange={(event, editor) => {
          const data = editor.getData();
          onChange({ target: { name, value: data } });
        }}
      />
    </Wrapper>
  );
};

Editor.propTypes = {
  onChange: PropTypes.func.isRequired,
  name: PropTypes.string.isRequired,
  value: PropTypes.string,
};

export default Editor;
```

在这一点上，我们只是创建了一个新的插件，这是挂载在我们的项目，但我们的自定义 **Field** 尚未注册。

### 注册我们的新 `Field`

由于我们的目标是我们的插件覆盖当前 `WYSIWYG` ，我们不希望它显示在管理面板，但我们需要它注册我们的新 **Field** 。为了这样做，我们将简单地 **modify** 我们的插件的前端入口点。

此文件已经存在。请用以下内容替换此文件的内容:

**Path —** `./plugins/wysiwyg/admin/src/index.js`

```js
import pluginPkg from '../../package.json';
import Wysiwyg from './components/Wysiwyg';
import pluginId from './pluginId';

export default strapi => {
  const pluginDescription = pluginPkg.strapi.description || pluginPkg.description;

  const plugin = {
    blockerComponent: null,
    blockerComponentProps: {},
    description: pluginDescription,
    icon: pluginPkg.strapi.icon,
    id: pluginId,
    initializer: () => null,
    injectedComponents: [],
    isReady: true,
    isRequired: pluginPkg.strapi.required || false,
    mainComponent: null,
    name: pluginPkg.strapi.name,
    preventComponentRendering: false,
    settings: null,
    trads: {},
  };

  strapi.registerField({ type: 'wysiwyg', Component: Wysiwyg });

  return strapi.registerPlugin(plugin);
};
```

最后，你将不得不重建 strapi，使新的插件正确加载

:::: tabs

::: tab yarn

```
yarn build
```

:::

::: tab npm

```
npm run build
```

:::

::: tab strapi

```
strapi build
```

:::

::::

::: tip
如果插件仍然没有出现，你可能也应该清空 `.cache` 文件夹。
:::

如果您创建了一个新的 `collectionType` 或带有 `richtext` 字段的 `singleType`，那么您将看到 [CKEditor](https://ckeditor.com/ckeditor-5/) 的实现，而不是默认的 `WYSIWYG`。
