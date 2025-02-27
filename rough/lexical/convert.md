# 🔗 How to convert Protocol??

To Integrate Lexical into EGP, we need to covert EGP Page Schema to Lexical JSON.

Let’s see both formats.

EGP Page Schema
```json
{
    "id": ...,
    "tags": [],
    "schema": {
        "encapsulateRootId": null,
        ...,
        "components": [
            {
                "id": "root",
                "name": null,
                "fields": null,
                "elements": [
                    {
                        "id": "root",
                        "name": null,
                        "tagName": "Layout",
                        "props": {
                            "className": "flex flex-col gap-[16px]\nself-center \nwhitespace-pre-wrap\nw-[100%] \np-[16px]\nmax-w-[680px] leading-[140%] \ntext-m-text-primary\nbg-m-background-primary \n[&_article]:text-[15px]\n[&_article]:leading-[140%]\n[&_article_a]:underline\n[&_article_h1]:text-[32px] \n[&_article_h1]:leading-[140%] \n[&_article_h1,h2,h3,h4]:font-[700] \n[&_article_h2]:text-[28px] \n[&_article_h2]:leading-[140%] \n[&_article_h3]:leading-[140%] \n[&_article_h4]:leading-[140%] \n[&_article_h3]:text-[20px] \n[&_article_h4]:text-[15px] \n[&_article_blockquote]:flex\n[&_article_blockquote]:!text-[17px]\n[&_article_blockquote]:p-[16px]\n[&_article_blockquote]:rounded-[4px] \n[&_article_blockquote]:bg-m-background-secondary\n[&_article_blockquote]:leading-[140%]\n[&_article_blockquote]:p-[16px]\n[&_article_blockquote]:font-[600]\n[&_article_hr]:border-m-background-secondary\n[&_article_hr]:border-t-[0px]\n[&_article_hr]:border-b-[2px]\n[&_article_hr]:border-solid\n[&_article_pre]:bg-m-background-accentThin\n[&_article_pre]:rounded-[4px] \n[&_article_pre]:p-[8px]\n[&_article_pre]:my-[8px]\n[&_article_kbd]:bg-m-background-secondary\n[&_article_kbd]:px-[6px]\n[&_article_kbd]:py-[2px]\n[&_article_kbd]:rounded-[4px]\n[&_article_ul]:m-[0]\n[&_article_p]:m-[0]\n[&_article_tr]:border-solid\n[&_article_tr]:border-[1px]\n[&_article_td]:p-[16px]\n[&_article_td]:text-center\n[&_article_th]:p-[16px]\n[&_article_tr]:border-m-border-primary\n[&_article_p]:leading-[140%]\n[&_article_ul]:leading-[50%]\n[&_article_ul]:whitespace-nowrap\n[&_article_li]:list-disc\n[&_article_li]:p-[0]\n[&_article_li]:text-[17px]\n[&_article_li]:ml-[16px]\n[&_article_li]:leading-[140%]\n[&_article_table]:border-collapse\n[&_article_table]:rounded-[4px]\n[&_article_table]:w-full\n[&_article_code]:bg-m-background-secondary\n[&_article_code]:rounded-sm\n[&_article_code]:p-[2px]\n[&_article_img]:w-[100%]\n[&_article_img]:rounded-[8px]\n[&_article_img]:my-[16px]\n[&_article_img]:bg-m-background-secondary",
                            "children": [
                                ":=element.the-first-markdown"
                            ]
                        },
                        "meta": null,
                        "intlProps": null
                    },
                    {
                        "id": "the-first-markdown",
                        "name": null,
                        "tagName": "Markdown",
                        "props": {
                            "content": "# Title\n content"
                        },
                        "meta": null,
                        "intlProps": null
                    }
                ]
            }
        ]
    }
}
```

Lexical JSON
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

The EGP Page Schema is somewhat complex because it is designed to accommodate various page formats (e.g., EGP Page, Card, Article). While EGP Pages and Card UIs can have multiple layers, the content in Articles is presented in a single line. Therefore, for Articles, we can simplify the schema to resemble Lexical JSON.

## Idea

Directly converting between schemas can be challenging. As mentioned earlier, we can simplify the EGP Page Schema, and creating a Universal Schema is one possible solution.

The conversion process can be visualized as follows:

- Lexical → Universal Schema → EGP Article

- EGP Article → Universal Schema → Lexical

### What Universal Schema is like??

Next, we need to determine what elements are essential for the Universal Schema.

The EGP Page Schema includes a className for each component, but if we define the EGP Article styling theme uniquely, we can eliminate the need to store this information in the Universal Schema.

The Universal Schema should only convey the following:

- What component is
  - (e.g., Markdown, Image, …)
- Where component is
- What styling component has
  - not className
  - (e.g., “h1”, “bold”, …)

### Universal Schema Prototype

As a prototype, I created following page schema, and survey the possibility.

```ts
type UniversalArticleElement =
  | {
      type: 'Markdown';
      content: string;
      elementId?: string;
    }
  | {
      type: 'Image';
      src: string;
      description: string;
      elementId: string;
    };
```

This schema only supports Markdown and Image.

Let’s see the implementation idea.

### EGP Page Schema → Lexical JSON

The workflow is following:

1. extract content data (which follows Universal Schema)
2. convert content to markdown string
   - h1 → #, Image → ![](), …
3. convert markdown string to Lexical JSON

In Lexical, it has TRANSFORMER and $convertFromMarkdownString function, so it can be easily done! (You can see in detail: https://mercari.atlassian.net/wiki/spaces/CF/pages/3587770300 )

### Lexical JSON → EGP Page Schema

The workflow is following:

1. convert content to markdown string (which follows Universal Schema)
   - In Lexical, content and styling is divided, so need to convert like below:
   ```ts
   if (node.type === 'heading') {
      let prefix = '';
      switch (node.tag) {
        case 'h1':
          prefix = '# ';
          break;
        case 'h2':
          prefix = '## ';
          break;
        case 'h3':
          prefix = '### ';
          break;
      }
      
      if (Array.isArray(node.children)) {
        for (const child of node.children) {
          if (child.type === 'text') {
            results.push({
              type: 'Markdown',
              content: prefix + (child.text ?? ''),
            });
          }
        }
      }
   }
   ```
2. convert markdown string to EGP Page Schema

For the conversion of step 2, we can utilize copy and paste functionality. In the EGP system, when a user copies a component and pastes it into the editor, the editor can interpret the copied JSON and convert it into the corresponding EGP component. Therefore, if we have the content information, we can easily transform it into the EGP Page Schema.

Here’s a rough outline of the code for this process:

```ts
const convertLexicalJsonToEgpPage = (json: any) => {
    const universalArticleElements = dehydrateLexicalData(json);

    // pop all elements from the root
    actions.updateElementProps({ elementId: 'root', props: { children: [] } });

    universalArticleElements.reverse().map((item) => {
      if (item.type === 'Markdown') {
        const markdownElement: Element = {
          id: nanoid(),
          tagName: 'Markdown',
          props: {
            content: item.content,
          },
        };
        actions.insertElementToTargetOrRoot({ element: markdownElement, targetId: 'root' });
      } else if (item.type === 'Image') {
        const imageElement: Element = {
          id: nanoid(),
          tagName: 'Image',
          props: {
            imageUrl: item.src,
            description: item.description,
            srcSet: generateSrcSet(item.src),
          },
        };
        actions.insertElementToTargetOrRoot({ element: imageElement, targetId: 'root' });
      }
    });

    return;
  };
```

