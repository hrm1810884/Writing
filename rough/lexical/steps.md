# 🪜 Integration Steps

Here are the logs from when I conducted research on our integration strategy as a internship task.

## 🏗️ Decentralizing Global Styles: Moving Class Definitions to Local Files

In Lexical Playground, node and editor styles are defined in an "index.css" file located in the project's root directory. This file is approximately 2,000 lines long, making it challenging to decipher how Lexical functions. Additionally, the class names are deeply nested, as shown in the example below🤮

```css
.toolbar i.chevron-down,
.toolbar-item i.chevron-down {
  margin-top: 3px;
  width: 16px;
  height: 16px;
  display: flex;
  user-select: none;
}
```

However, this approach may make it challenging to integrate other CSS libraries and to fully understand how Lexical works.

To address this issue, we identified the specific styling scope and moved the CSS into files located in the same directory as their related components. This approach divides responsibilities and improves both readability and maintainability.

## 📉 Narrow Down Functionalities: Focus on EGP Article Capabilities

While Lexical's exceptional customizability and flexibility are compelling advantages, they can pose challenges during integrations. For instance, when linking protocols, Lexical's inherent flexibility might lead to representations that conventional protocols cannot fully accommodate.

As a first step, we decided to focus only on RichText, Image and ItemList which is mainly used in current EGP Article.

As mentioned in https://mercari.atlassian.net/wiki/x/cglg1Q, we can create custom nodes and plugins. Building on this foundation, our approach is to implement only the necessary custom nodes and plugins to replicate and enhance the existing functionality of EGP Article.

## 📝 Protocol Integration: Converting Lexical to EGP Article

Bridging protocols becomes crucial during integration. To maximize consistency, it's essential that the same content is displayed regardless of whether the conventional EGP Article or Lexical is used.

EGP Protocols offer a wide range of components (e.g., Layout, When, etc.) that, when arranged in a nested layer architecture, enable the creation of complex and varied page UIs. Within that context, Article, despite offering various content such as ItemList and Image, presents them in a flat layout without any hierarchical structuring. 

Due to the simplicity of its structure, if a converter is provided for Article that can communicate just two key aspects—

1. the sequence of content from top to bottom, and 
2. the applied styling

—it is believed that protocol bridging can be easily achieved.

Fortunately, Lexical is equipped with a feature that outputs content in JSON format, and it also allows you to configure the JSON representation for custom components. Therefore, the conversion mentioned above should be achievable without any issues.