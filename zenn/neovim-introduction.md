# NeoVim初めてみた
## はじめに
### 自己紹介
初めまして、@hrm1810884と申します。
現在東京大学大学院の修士一年生でフロントエンドを中心に絶賛勉強中のエンジニア学生です。
フロントエンドではTypeScript・React・Next.jsを用いたシステム開発を色々しています。
研究は主にHuman-AI Interactionの分野に興味があり、最近はAIを用いたシステム開発や展示会への出品などもしています。
### 本記事の概要
本記事は僕のzennデビュー記事になります。(お手柔らかに。。)
早速本題ですが、僕はこれまでの開発はVSCodeを用いて行ってきていたのですが、PC環境を整えていくうちにキーボードから手を離すことにストレスを感じる体になってしまいました。
そこで憧れのVimmerに転身することを決め、Vimエディタとして大きな人気を誇るNeovimの環境整備を行ったのでその行動記録を残すことに決めました。

Neovimにはさまざまなプラグインが存在し、多くの方がおすすめプラグインをまとめてくださっています。
ただ、その数の多さゆえに選定の認知負荷が高いのと、それぞれの細かい設定方法についてはプラグインごとのGitHubのReadmeを読むしかないなど、環境整備に苦労したので自分が選んだプラグインの選定理由と細かい設定configをここにまとめておこうと思います。
以上を踏まえ、この記事の対象読者は以下のようになります
- Neovimの導入を考えている人
- とりあえず最低限必要なプラグインを知りたい人
- 各用途のプラグインの違いを知りたい人
- よしなな設定を知りたい人

全くのNeovim初心者なのでこれからVimmerとして実装を進める中で他のプラグインに移行したり、新たなプラグインを追加していくことになると思いますが、とりあえずの初期整備で導入したものをまとめます。（随時更新していこうと思います）

## プラグイン・マネージャ
まずはプラグインを管理するマネージャになります。
プラグイン導入に必須のアイテムになります。
マネージャにも様々な種類がありますが調べてみるとPacker.nvimやLazy.nvimなどを利用している人が多い印象でした。
ここはかなり悩みましたが、多くの人に利用されていて、遅延読み込みの効率化に伴って起動時間の短縮を図るLazy.nvimを選びました、
起動時間が短いのも嬉しいのですが、なんとなくconfigファイルの書き方が好みだったのでそちらを選びました。Packer.nvimとLazy.nvimのconfigファイルの違いを以下に示します。

:::details Packer.nvimの設定ファイル例

```lua
require('packer').startup(function()
  -- Telescopeの設定
  use {
    'nvim-telescope/telescope.nvim',
    requires = { 'nvim-lua/plenary.nvim' },
    config = function()
      require('telescope').setup {
        defaults = {
          prompt_prefix = "> ",
          sorting_strategy = "ascending",
          layout_config = {
            horizontal = { width = 0.8, preview_width = 0.5 }
          },
        }
      }
      -- キーマッピングを設定
      local builtin = require('telescope.builtin')
      vim.keymap.set('n', '<leader>ff', builtin.find_files, { desc = 'Find Files' })
      vim.keymap.set('n', '<leader>fg', builtin.live_grep, { desc = 'Live Grep' })
    end
  }

  -- fzf拡張機能を読み込む
  use {
    'nvim-telescope/telescope-fzf-native.nvim',
    run = 'make',
    config = function()
      require('telescope').load_extension('fzf')
    end
  }
end)
```

:::details

:::details Lazy.nvimの設定ファイル例

```lua
return {
  -- Telescopeの設定
  {
    'nvim-telescope/telescope.nvim',
    dependencies = { 'nvim-lua/plenary.nvim' }, -- 必要な依存プラグイン
    config = function()
      require('telescope').setup {
        defaults = {
          prompt_prefix = "> ",
          sorting_strategy = "ascending",
          layout_config = {
            horizontal = { width = 0.8, preview_width = 0.5 }
          },
        }
      }
      -- キーマッピングを設定
      local builtin = require('telescope.builtin')
      vim.keymap.set('n', '<leader>ff', builtin.find_files, { desc = 'Find Files' })
      vim.keymap.set('n', '<leader>fg', builtin.live_grep, { desc = 'Live Grep' })
    end
  },
  -- fzf拡張機能を読み込む
  {
    'nvim-telescope/telescope-fzf-native.nvim',
    build = 'make',
    config = function()
      require('telescope').load_extension('fzf')
    end
  }
}
```　
:::details

Neovimの設定ファイルはluaを用いて書いていきます。Lazy.luaでは各プラグインの設定ごとにファイルを分けることができるので、今後開発を進める中でプラグインを再比較し、乗り換えを行うことを想定して、各用途ごとに以下のようにファイル分けを行いました。

~/.config/nvim
├── lua
│   ├── config
│   │   └── lazy.lua
│   └── plugins
│       ├── filer.lua
|       ├── lsp.lua
│       ├── utils.lua
│       └── github.lua
└── init.lua

プラグインごとのluaファイルとは別にnvim全体の設定(キーバインドなど)を書くinit.lua、Lazy.luaに関する基本設定を書くlazy.luaを作成します。各ファイルは以下のように設定しました。
init.luaの基本設定は食パン🍞さんが紹介してくださっているファイルを参考にさせていただきました🙇
https://zenn.dev/fukakusa_kadoma/articles/4d48fb4e67c945

:::details init.lua

```lua
-- 文字エンコーディングの設定
vim.scriptencoding = 'utf-8'         -- スクリプトエンコーディングをUTF-8に設定
vim.opt.encoding = 'utf-8'           -- 内部エンコーディングをUTF-8に設定
vim.opt.fileencoding = 'utf-8'       -- ファイルのエンコーディングをUTF-8に設定

-- 行番号の表示設定
vim.opt.number = true                 -- 行番号を表示
vim.wo.number = true                  -- ウィンドウごとに行番号を表示
vim.wo.relativenumber = false         -- 相対行番号を無効化

-- マウス操作の設定
vim.opt.mouse = 'a'                   -- マウス操作を全モードで有効化

-- ウィンドウタイトルの設定
vim.opt.title = true                  -- ウィンドウタイトルを有効化

-- インデントの設定
vim.opt.autoindent = true             -- 自動インデントを有効化
vim.opt.smartindent = true            -- スマートインデントを有効化

-- 検索の設定
vim.opt.hlsearch = true               -- 検索結果をハイライト表示

-- バックアップファイルの設定
vim.opt.backup = false                -- バックアップファイルを作成しない

-- コマンド表示の設定
vim.opt.showcmd = true                -- 入力中のコマンドを表示
vim.opt.cmdheight = 2                 -- コマンドラインの高さを2行に設定

-- ステータスラインの設定
vim.opt.laststatus = 2                -- 常にステータスラインを表示

-- タブとスペースの設定
vim.opt.expandtab = true              -- タブをスペースに変換
vim.opt.shiftwidth = 2                -- 自動インデント時のスペース数を2に設定
vim.opt.tabstop = 2                   -- タブ幅を2スペースに設定
vim.opt.smarttab = true               -- スマートタブを有効化

-- スクロール設定
vim.opt.scrolloff = 10                -- カーソルの上下に常に10行の余白を保持

-- インクリメンタルサーチの設定
vim.opt.inccommand = 'split'           -- インクリメンタルサーチ時に分割ウィンドウで結果を表示

-- 検索時の大文字小文字無視設定
vim.opt.ignorecase = true             -- 検索時に大文字小文字を無視
vim.opt.smartcase = true              -- 検索語に大文字が含まれている場合は大文字小文字を区別

-- 折り返しの設定
vim.opt.wrap = true                   -- 行の折り返しを有効化
vim.opt.breakindent = true            -- 折り返し行にインデントを適用

-- ヘルプ言語の設定
vim.opt.helplang = {'ja', 'en'}       -- ヘルプの言語を日本語と英語に設定

-- 更新待機時間の設定
vim.opt.updatetime = 300              -- 更新待機時間を300ミリ秒に設定

-- タブラインの表示設定
vim.opt.showtabline = 2                -- 常にタブラインを表示

-- クリップボードの設定
vim.opt.clipboard = 'unnamedplus'      -- クリップボードとレジスタを共有

-- ターミナルカラーの設定
vim.opt.termguicolors = true           -- True Colorを有効化

-- サインカラムの設定
vim.opt.signcolumn = 'yes'             -- サインカラムを常に表示

-- バッファの設定
vim.opt.hidden = true                  -- バッファを隠しても変更を保持
vim.opt.swapfile = false               -- スワップファイルを生成しない

-- キーマッピングの設定
local keymap = vim.keymap

-- 画面分割用のキーマッピング
keymap.set('n', 'ss', ':split<Return><C-w>w', { silent = true })  -- 水平分割して新しいウィンドウに移動
keymap.set('n', 'sv', ':vsplit<Return><C-w>w', { silent = true }) -- 垂直分割して新しいウィンドウに移動

-- アクティブウィンドウの移動用キーマッピング
keymap.set('n', 'sh', '<C-w>h')                  -- 左のウィンドウに移動
keymap.set('n', 'sk', '<C-w>k')                  -- 上のウィンドウに移動
keymap.set('n', 'sj', '<C-w>j')                  -- 下のウィンドウに移動
keymap.set('n', 'sl', '<C-w>l')                  -- 右のウィンドウに移動

-- Emacs風のキーマッピング
keymap.set('i', '<C-f>', '<Right>')              -- 挿入モードでCtrl+fで右にカーソル移動

-- jjでEscするキーマッピング
keymap.set('i', 'jj', '<Esc>')                   -- 挿入モードでjjを押すとノーマルモードに戻る

-- 設定ファイルを開くキーマッピング
keymap.set('n', '<F1>', ':edit $MYVIMRC<CR>')    -- ノーマルモードでF1を押すと設定ファイルを開く

-- d でブラックホールレジスタに削除
keymap.set("n", "d", '"_d', { noremap = true, silent = true })
keymap.set("v", "d", '"_d', { noremap = true, silent = true })

-- _d で通常の削除（デフォルトレジスタに保存）
keymap.set("n", "_d", "d", { noremap = true, silent = true })
keymap.set("v", "_d", "d", { noremap = true, silent = true })

require("config.lazy")

```
:::details

:::details lazy.lua

```lua
-- Bootstrap lazy.nvim
local lazypath = vim.fn.stdpath("data") .. "/lazy/lazy.nvim"
if not (vim.uv or vim.loop).fs_stat(lazypath) then
  local lazyrepo = "https://github.com/folke/lazy.nvim.git"
  local out = vim.fn.system({ "git", "clone", "--filter=blob:none", "--branch=stable", lazyrepo, lazypath })
  if vim.v.shell_error ~= 0 then
    vim.api.nvim_echo({
      { "Failed to clone lazy.nvim:\n", "ErrorMsg" },
      { out, "WarningMsg" },
      { "\nPress any key to exit..." },
    }, true, {})
    vim.fn.getchar()
    os.exit(1)
  end
end
vim.opt.rtp:prepend(lazypath)

-- Make sure to setup `mapleader` and `maplocalleader` before
-- loading lazy.nvim so that mappings are correct.
-- This is also a good place to setup other settings (vim.opt)
vim.g.mapleader = " "
vim.g.maplocalleader = "\\"

-- Setup lazy.nvim
require("lazy").setup({
  spec = {
    -- import your plugins
    { import = "plugins" },
  },
  -- Configure any other settings here. See the documentation for more details.
  -- colorscheme that will be used when installing plugins.
  install = { colorscheme = { "habamax" } },
  -- automatically check for plugin updates
  checker = { enabled = true },
})
```
:::details

## アイコン
いきなりアイコン？！と思われるかもしれませんが、一番初めにセットアップしておくといいかと思います。というのも様々なUIをリッチにするプラグインをこれから導入していくことになるかと思いますが、
使用するアイコンもプラグインとして導入するので最もベースのプラグインとなりそうです。
今回は多くのUIプラグインで依存プラグインとして参照されているnvim-web-deviconsを選定しました。
アイコンプラグインはカスタマイズも可能なので@ありあなさんの記事を参考にカスタマイズした例を挙げておきます。
https://zenn.dev/alliana_ab2m/articles/a8c68a3a5933c8

:::details icon.lua
```lua
-- plugins/devicons.lua
return {
  {
    "nvim-tree/nvim-web-devicons",
    config = function()
      require('nvim-web-devicons').setup({
        override = {
          ts = {
            icon = 'ﯤ', -- Nerd Fontでコピーしたアイコン
            color = '#3178C6' -- TypeScriptの色
          }
        }
      })
    end,
  }
}
```
:::details

## ファイラー
ファイラーはNeovimのプラグインとして重要なアイテムになります。
システム開発において重要なファイル選択のUXを向上させるため様々なプラグインが存在しています。
多くのファイラー・プラグインの中で最終的に二つの候補に絞りました、
### [nvim-tree.lua](https://github.com/nvim-tree/nvim-tree.lua)
一つはnvim-tree.luaです。
お馴染みのVSCodeライクなファイルツリー形式で表示してくれ、様々なオプションを設定可能です。
### [oil.nvim](https://github.com/stevearc/oil.nvim)
二つ目はoil.nvimです。
こちらはバッファを編集することでvimのコード編集と同じようにファイルの作成やファイル名の編集などを行うことができます。
正直、vimmerを目指すに当たって突き詰めたVim環境をセットしたいと考えていたので、最初はoil.nvimを選定していたのですが、さすがにVSCodeの慣れがあったのと、特にフロントエンドではUIの構造に紐づいたディレクトリ構成を行う関係上各ファイルをツリー形式で見られた方が嬉しいと感じたのではじめはnvim-tree.luaにすることにしました。
（前述しましたが、このように各用途でかなり悩んで決めているので、今後乗り換えることも想定して各用途に紐づいた名前で以下のようにluaを書いています）

:::details filer.lua

```lua
local function open_nvim_tree()
    require("nvim-tree.api").tree.open()
end

vim.api.nvim_create_autocmd({ "VimEnter" }, { callback = open_nvim_tree })

return {
  "nvim-tree/nvim-tree.lua",
  version = "*",
  lazy = false,
  dependencies = {
    "nvim-tree/nvim-web-devicons",
  },
  keys = {
    {mode = "n", "<C-n>", "<cmd>NvimTreeToggle<CR>", desc = "NvimTreeをトグルする"},
    {mode = "n", "<C-m>", "<cmd>NvimTreeFocus<CR>", desc = "NvimTreeにフォーカス"},
  },
  config = function()
    require("nvim-tree").setup {
      git = {
        enable = true,
        ignore = true,
      }
    }
  end,
}
```
:::details
アイコン表示においての注意事項はアイコン対応のフォントでないと表示がされませんので [Nerd Font](https://www.nerdfonts.com/font-downloads)などのアイコン対応フォントをダウンロードしてターミナル上で設定する必要があります。

## fzf
fzf (fuzzy finder)とはfuzzy検索(あいまい検索)によって「こんな感じのファイル名」のようなあいまいな入力内容からファイル検索などが行える優れものです。
fzf系のプラグインも調べてみるとかなりあったのですが、ある程度利用率が高く、LSPなどとの連携も申し分なさそうだったので、[telescope.nvim](https://github.com/nvim-telescope/telescope.nvim)を利用することにしました

:::details fzf.lua
```lua
return {
  {
    'nvim-telescope/telescope.nvim',
    dependencies = {
      'nvim-lua/plenary.nvim',
      {
        'nvim-telescope/telescope-fzf-native.nvim',
        build = 'make'
      }
    },
    config = function()
      require('telescope').setup {
        extensions = {
          fzf = {
            fuzzy = true,                    -- ファジーマッチングを有効化
            override_generic_sorter = true,  -- ジェネリックソーターを上書き
            override_file_sorter = true,     -- ファイルソーターを上書き
            case_mode = "smart_case",        -- 大文字小文字のモード
          }
        }
      }
      
      -- fzf拡張機能をロード
      require('telescope').load_extension('fzf')

      -- キーマッピング設定
      local builtin = require('telescope.builtin')
      vim.keymap.set('n', '<leader>ff', builtin.find_files, { desc = 'Telescope find files' })
      vim.keymap.set('n', '<leader>fg', builtin.live_grep, { desc = 'Telescope live grep' })
      vim.keymap.set('n', '<leader>fb', builtin.buffers, { desc = 'Telescope buffers' })
      vim.keymap.set('n', '<leader>fh', builtin.help_tags, { desc = 'Telescope help tags' })
    end
  }
}
:::details

```
## LSP
次にLSP (Language Server Protocol)のプラグインを紹介します。
LSPは言語に応じて補完機能を実現するためのもので、オリジナルのNeovimの補完機能(こちらはryoppippiさんが詳しくまとめてくださっています)を強化し、より良い開発環境を目指す上でなくてはならないものになります。
https://zenn.dev/ryoppippi/articles/8aeedded34c914
LSPに関してはどの記事を読んでもおおかた同じようなセットアップがされていたので悩まずに選定ができました。
中でもbotamotchさんが解説してくださっているこちらの記事が非常に参考になったので、ご紹介していただいているセットアップを参考に以下のように設定をしました。
https://zenn.dev/botamotch/articles/21073d78bc68bf
- [nvim-lspconfig](https://github.com/neovim/nvim-lspconfig)
  - LSPの設定を簡単に行えるよう、wrappingしてくれるプラグインです
- [mason.nvim](https://github.com/williamboman/mason.nvim)
  - パッケージ・マネージャとしてプラグインのインストールを管理してくれます
- [nvim-cmp](https://github.com/hrsh7th/nvim-cmp)
  - カスタマイズ性の高い補完プラグインです
以上のプラグインをまとめたluaファイルが以下になります
:::details LSP.lua
```lua
return {
  -- Package manager
  {
    "williamboman/mason.nvim",
    config = function()
      require("mason").setup()
    end
  },
  {
    "williamboman/mason-lspconfig.nvim",
    dependencies = { "williamboman/mason.nvim" },
    config = function()
      require("mason-lspconfig").setup_handlers({ function(server)
        local opt = {
          capabilities = require('cmp_nvim_lsp').default_capabilities(
            vim.lsp.protocol.make_client_capabilities()
          )
        }
        require('lspconfig')[server].setup(opt)
      end })
    end
  },
  {
    "neovim/nvim-lspconfig",
    dependencies = { "williamboman/mason-lspconfig.nvim" },
    config = function()
      -- LSP-related keymaps
      vim.keymap.set('n', 'K',  '<cmd>lua vim.lsp.buf.hover()<CR>')
      vim.keymap.set('n', 'gf', '<cmd>lua vim.lsp.buf.format()<CR>')
      vim.keymap.set('n', 'gr', '<cmd>lua vim.lsp.buf.references()<CR>')
      vim.keymap.set('n', 'gd', '<cmd>lua vim.lsp.buf.definition()<CR>')
      vim.keymap.set('n', 'gD', '<cmd>lua vim.lsp.buf.declaration()<CR>')
      vim.keymap.set('n', 'gi', '<cmd>lua vim.lsp.buf.implementation()<CR>')
      vim.keymap.set('n', 'gt', '<cmd>lua vim.lsp.buf.type_definition()<CR>')
      vim.keymap.set('n', 'gn', '<cmd>lua vim.lsp.buf.rename()<CR>')
      vim.keymap.set('n', 'ga', '<cmd>lua vim.lsp.buf.code_action()<CR>')
      vim.keymap.set('n', 'ge', '<cmd>lua vim.diagnostic.open_float()<CR>')
      vim.keymap.set('n', 'g]', '<cmd>lua vim.diagnostic.goto_next()<CR>')
      vim.keymap.set('n', 'g[', '<cmd>lua vim.diagnostic.goto_prev()<CR>')

      -- LSP handlers
      vim.lsp.handlers["textDocument/publishDiagnostics"] = vim.lsp.with(
        vim.lsp.diagnostic.on_publish_diagnostics, { virtual_text = false }
      )
      -- Reference highlight
      vim.cmd [[
        set updatetime=500
        highlight LspReferenceText  cterm=underline ctermfg=1 ctermbg=8 gui=underline guifg=#A00000 guibg=#104040
        highlight LspReferenceRead  cterm=underline ctermfg=1 ctermbg=8 gui=underline guifg=#A00000 guibg=#104040
        highlight LspReferenceWrite cterm=underline ctermfg=1 ctermbg=8 gui=underline guifg=#A00000 guibg=#104040
        augroup lsp_document_highlight
          autocmd!
          autocmd CursorHold,CursorHoldI * lua vim.lsp.buf.document_highlight()
          autocmd CursorMoved,CursorMovedI * lua vim.lsp.buf.clear_references()
        augroup END
      ]]
    end
  },
  -- Completion setup
  {
    "hrsh7th/nvim-cmp",
    dependencies = {
      "hrsh7th/cmp-nvim-lsp",
      "hrsh7th/vim-vsnip"
    },
    config = function()
      local cmp = require("cmp")
      cmp.setup({
        snippet = {
          expand = function(args)
            vim.fn["vsnip#anonymous"](args.body)
          end,
        },
        sources = {
          { name = "nvim_lsp" },
          -- { name = "buffer" },
          -- { name = "path" },
        },
        mapping = cmp.mapping.preset.insert({
          ["<C-p>"] = cmp.mapping.select_prev_item(),
          ["<C-n>"] = cmp.mapping.select_next_item(),
          ['<C-l>'] = cmp.mapping.complete(),
          ['<C-e>'] = cmp.mapping.abort(),
          ["<CR>"] = cmp.mapping.confirm { select = true },
        }),
        experimental = {
          ghost_text = true,
        },
      })

      cmp.setup.cmdline(":", {
        completion = {
          completeopt = "menu,menuone,noinsert,noselect",
        },
      })

    end
  }
}
```

特にnvim-cmpの以下の設定は表示される候補の一つ目にフォーカスを当てておくための設定です。
```lua
cmp.setup.cmdline(":", {
  completion = {
    completeopt = "menu,menuone,noinsert,noselect",
  },
})
```
## github
開発にあたってなくてはならないGitHubに関連したプラグインもいくつか導入しました。
- copilot.nvim
  - luaを作成後、:Copilot setupのコマンドで出てきたOTPでGitHubアカウントの紐付けをブラウザ上で行えばすぐに導入できます 
- gitsigns.nvim
- git-blame.nvim
- diffview.nvim

## utils
便利プラグイン系でいくつか初期に導入したものがあったのでまとめておきます。ここら辺は実際に開発を進めながら需要に合わせて随時探していこうと思っています。
## おまけ
ここからはおまけとして自分が行うフロントエンドの開発に必要なものをいくつかピックアップしてまとめておきます
