# 📝 Json Import / Export

As mentioned in https://mercari.atlassian.net/wiki/x/cglg1Q , Lexical stores its contents in JSON format.

To edit EGP Articles using Lexical, you will need to represent the content in JSON format according to the protocol provided by Lexical, and then import it into Lexical. Similarly, to store the content created in Lexical as EGP Articles in the database, you will need to export it in JSON format and then convert it to comply with EGP’s protocol.

I will elaborate on the details of the protocol conversion in a separate document. For now, I will compile resources to assist you in customizing the import and export processes as needed.

## Import

In Lexical, there are several import methods that allow you to import content in formats such as Markdown, HTML, and more. In this section, only the Markdown format is supported. If you want to explore other formats, please refer to the documentation, such as https://lexical.dev/docs/packages/lexical-html.

### Markdown to Lexical Json

Fortunately, Lexical provides converter from Markdown string to Lexical json. So simply you only need to prepare markdown string you want to import, and code like below

```ts
useEffect(() => {
    editor.update(() => {
      $convertFromMarkdownString(markdownString);
    });
  }, [editor, markdownString]);
```

Although importing using the method described above is straightforward, if you want to integrate a custom component—which you will likely want to do in most cases—there are additional steps you need to take.

### Custom Transformer

In Lexical Json import, there is an important concept, which is “Transformer”.

To integrate a custom node reader in $convertFromMarkdownString, you need to define a custom transformer as shown below.

```ts
export const imageMarkdownTransformer: TextMatchTransformer = {
  type: 'text-match',
  dependencies: [ImageNode],
  // regExp: ![]()
  importRegExp: /!\[([^\]]*)\]\(([^)]+)\)/,
  regExp: /!\[([^\]]*)\]\(([^)]+)\)/,
  // trigger text-match by ')'
  trigger: ')',
  replace: (textNode: TextNode, match: RegExpMatchArray): void => {
    const id = match[1].split('/')[0];
    const altText = match[1].split('/')[1];
    const src = match[2];

    const imageNode = $createImageNode({ src, id, altText });

    textNode.replace(imageNode);
  },
  export: (node, _exportChildren) => {
    if (node.getType() === 'image') {
      const imageNode = node as ImageNode;
      const src = imageNode.getSrc();
      const altText = imageNode.getAltText();
      return `![${altText}](${src})`;
    }
    return null;
  },
};
```

In this example, transformer for custom ImageNode (you can see how to create it in https://mercari.atlassian.net/wiki/spaces/CF/pages/3587835799 ) is defined. To define the transformer, you first need to specify how it is expressed in Markdown.

In this case, ImageNode is represented as ![${alt text}](${id}/${src}) . Using a text matcher, Lexical detects the custom expression and creates a corresponding Node.

Lexical provides a default transformer, so after defining your custom transformer, you need to merge it as shown below.

```ts
export const CUSTOM_TRANSORMERS = [imageMarkdownTransformer, ...TRANSFORMERS];
```

Finally, you can update the previous import code as follows.

```ts
useEffect(() => {
    editor.update(() => {
      $convertFromMarkdownString(markdownString, CUSTOM_TRANSORMERS);
    });
  }, [editor, markdownString]);
```

### Page to Markdown

As you see above. in order to use this import function in EGP, we need to convert Page contents to a Markdown string. How to achieve this involves converting the page protocol, so I intend to summarize the details in https://mercari.atlassian.net/wiki/x/yArZ1Q.

## Export 

In Lexical, you can export JSON from page contents. Similar to the import process, exporting JSON can be done easily using an oneline command. However, when dealing with custom nodes, there are several additional steps involved.

### Page contents to JSON

As shown below, Lexical provides a method for exporting content in JSON format, making it easy for us to obtain the desired JSON structure.

```ts
const [editor] = useLexicalComposerContext();
const handleExport = useCallback(() => {
  const json = editor.getEditorState().toJSON();
  return json
}, [editor]);
```

As mentioned earlier, if you want to integrate a custom node into this export method, you will need to follow the steps outlined below.

### Export Method Registration

As mentioned in https://mercari.atlassian.net/wiki/spaces/CF/pages/3587835799 , we can define several methods in a custom node, including the `exportJSON` method. This can be set up as follows:

```ts
export class ImageNode extends DecoratorNode<React.ReactNode> {
  __src: string;
  __id: string;
  __altText: string;
  __width: 'inherit' | number;
  __height: 'inherit' | number;
  __maxWidth: number;
  __showCaption: boolean;
  __caption: LexicalEditor;
  __captionsEnabled: boolean;
 
 {/** Other methods definition  **/ }
 
  exportJSON(): SerializedImageNode {
    return {
      ...super.exportJSON(),
      id: this.__id,
      altText: this.getAltText(),
      caption: this.__caption.toJSON(),
      height: this.__height === 'inherit' ? 0 : this.__height,
      maxWidth: this.__maxWidth,
      showCaption: this.__showCaption,
      src: this.getSrc(),
      width: this.__width === 'inherit' ? 0 : this.__width,
    };
  }
  
  {/** Other methods definition  **/ }
};
```

With this definition, `exportJSON` is executed internally when `editor.getEditorState().toJSON()` is triggered.

### Protocol Conversion

After exporting, you may want to convert the JSON data into the EGP Protocol to ensure data persistence. With this topic, there are several aspects to discuss and evaluate, so I will summarize these points in https://mercari.atlassian.net/wiki/spaces/CF/pages/3587771080 
