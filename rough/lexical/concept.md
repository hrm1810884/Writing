# 🎰 How Lexical Works??

In this docs, let’s see how Lexical Playground works and check Lexical’s main concepts. (source code is [here](https://github.com/facebook/lexical/tree/main/packages/lexical-playground/src))

## 🎨 Theme

With Lexical, we can set global theme like UI components utils library.

Global theme styling is described by css, 

```css
// PlaygroundEditorTheme.css

.PlaygroundEditorTheme__h1 {
  font-size: 24px;
  color: rgb(5, 5, 5);
  font-weight: 400;
  margin: 0;
}
```
and we can set them by TypeScript like below.

```ts
// PlaygroundEditorTheme.ts

import type {EditorThemeClasses} from 'lexical';

import './PlaygroundEditorTheme.css';

const theme: EditorThemeClasses = {
  heading: {
    h1: 'PlaygroundEditorTheme__h1',
  },
};
```

like above, we can apply any styling that CSS can describe, and we can also use any CSS utility library that maps class names directly to styles.

## 🌴 Nodes

In Lexical, nodes play an important role.

Other than rich text components, we can set customize components like Image Node.

Nodes are defined as class objects, allowing you to configure their behavior when exporting to JSON files and freely specify any props you wish to pass.

ImageNode example is [here](https://github.com/facebook/lexical/blob/main/packages/lexical-playground/src/nodes/ImageNode.tsx)

## 🔌 Plugins

Another important aspect of Lexical is its plugin system. 

Plugins enable you to extend and customize the editor's functionality without altering its core. With plugins, you can add features like toolbars, auto-complete, syntax highlighting, or even integration with external services. 

Additionally, plugins enable you to integrate the editor with custom nodes. Once your custom nodes are created, you can develop custom plugins to define how to trigger these nodes (e.g., via shortcut commands) or to enhance their functionality (e.g., with onDragStart or onResize events).

ImagePlugins example is [here](https://github.com/facebook/lexical/blob/main/packages/lexical-playground/src/plugins/ImagesPlugin/index.tsx)

## 📝 Json

In Lexical, as in other text editors (e.g., Google Docs, Notion, etc.), content is internally stored in JSON format.

Content can be both exported and imported in JSON format, and even for custom components, the methods for JSON export and import can be finely configured—ensuring that customizability is not an issue.

Let's take a look at the actual JSON. 

```md
# Title
content

![https://placehold.jp/150x150.png](placehold image)
```

An article with the structure described above is represented as follows. The content is arranged flatly under the root, resulting in a very simple configuration.

```json
{
    "root": {
        "children": [
            {
                "children": [
                    {
                        "detail": 0,
                        "format": 0,
                        "mode": "normal",
                        "style": "",
                        "text": "Title",
                        "type": "text",
                        "version": 1
                    }
                ],
                "direction": "ltr",
                "format": "",
                "indent": 0,
                "type": "heading",
                "version": 1,
                "tag": "h1"
            },
            {
                "children": [
                    {
                        "detail": 0,
                        "format": 0,
                        "mode": "normal",
                        "style": "",
                        "text": " content",
                        "type": "text",
                        "version": 1
                    }
                ],
                "direction": "ltr",
                "format": "",
                "indent": 0,
                "type": "paragraph",
                "version": 1,
                "textFormat": 0,
                "textStyle": ""
            },
            {
                "children": [
                    {
                        "type": "image",
                        "version": 1,
                        "altText": "",
                        "caption": {
                            "editorState": {
                                "root": {
                                    "children": [],
                                    "direction": null,
                                    "format": "",
                                    "indent": 0,
                                    "type": "root",
                                    "version": 1
                                }
                            }
                        },
                        "height": 0,
                        "maxWidth": 500,
                        "showCaption": false,
                        "src": "https://placehold.jp/150x150.png",
                        "width": 0
                    }
                ],
                "direction": null,
                "format": "",
                "indent": 0,
                "type": "paragraph",
                "version": 1,
                "textFormat": 0,
                "textStyle": ""
            }
        ],
        "direction": "ltr",
        "format": "",
        "indent": 0,
        "type": "root",
        "version": 1
    }
}
```