---
title: "ミニマリズムを追求した Next.js 15 + MDX ブログテンプレート next-minimal-blog を作りました！"
emoji: "🎅"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["nextjs", "tailwindcss", "shadcnui"]
published: true
---

## この記事の要約 (リポジトリだけ見たい人向け)

Next.js (15.2.1) + MDX + shadcn/ui + Tailwind CSS で「**公式ライク & ミニマルなブログテンプレート**」を作成しました！

こちらのリポジトリで公開しています！ぜひ見ていってください！

https://github.com/cakegaly/next-minimal-blog

## はじめに

Next.js は、公式でも紹介されているように、SSR / SG (SSG) / ISR / API Routes / Middleware / RSC など、「Web アプリケーションに関することなら俺に任せろ！なんでもできるぜ！」フレームワークに進化しています。

一方で、こんな声もよく耳にするようになりました。

- 「できることが多すぎて、どれを選べばいいかわからない！」
- 「ライブラリ、書き方のパターンが多すぎて迷う！」
- 「進化が速すぎて追いきれない！気づけば古いバージョンのドキュメントを見てた；；」
- 「シンプルに始めたいのに、最初から複雑すぎる」

筆者自身も、「Next.js を使ってサクッとシンプルなサイトを作りたいだけなのに、気づけば依存や設定ファイルだらけ」ということがよくありました。

そこで、 **「ミニマリストアプローチ！公式の Next.js / Vercel 製だけでどこまでできる！？」** をテーマに、 **Next.js 15 (App Router) + MDX + shadcn/ui + Tailwind CSS ブログテンプレート** を作ってみました。

## next-minimal-blog の紹介

### 技術構成

- Next.js 15.2.1 (App Router) / React 19.0.0
- MDX
- Tailwind CSS + shadcn/ui

### おすすめしたい人

- Next.js / Vercel を学びたい初心者
- ヘッドレス CMS / API 連携などのコンテンツ管理を検討中だけど、とりあえずフロントエンドを組んでみたい人
- シンプルで軽量なサイトを持ちたい人

### 機能一覧

- MDX による記事コンテンツ管理
- RSS XML (Atom) の自動生成
- リンクカード (OGP fetch)
- サイトマップ / Robots.txt 自動生成
- shadcn/ui + Tailwind CSS + rehype-pretty-code によるスタイリング
- ダークモード切り替え
- ページネーション
- lucide-react ベースのアイコン管理

Zenn ライクに「記事のテーマが一目でわかる」デザインを意識して、
記事ごとに icon を指定 すると、ブログカード (記事一覧) のアイキャッチとして表示されるようにしました。

```mdx
---
title: Next.js についての記事
date: 2025-03-09
icon: nextjs
---
```

![ブログカードの表示 (ライトモード)](https://storage.googleapis.com/zenn-user-upload/ab86db9b4af4-20250310.png)
_ブログカードの表示 (ライトモード)_

### ディレクトリ構成

```sh
src/
├── actions/        # Server Actions (例: OGP メタデータ fetch)
├── app/            # App Router ページ群 + RSS, Sitemap, Robots
├── components/     # UI コンポーネント
├── content/        # 記事コンテンツ (MDX)
├── config/         # サイト設定 / タグ管理
├── lib/            # 各種ユーティリティ (MDX, OGP, pagination)
├── styles/         # サイト全体 & MDX スタイル
└── types/          # 型定義
```

開発者向けの設定として、私が Next.js で開発するときの ESLint, Prettier, VSCode の設定もリポジトリにまとめています。

散らかりがちな `components/` 以下は、用途別に分類してディレクトリを分けています。`shadcn/ui` コンポーネントを管理するディレクトリについては、よりわかりやすくするために、デフォルトの `components/ui/` から `components/shadcn-ui/` にリネームしています。

```sh
components/
├── content/      # MDX コンテンツの表示に関するもの
├── icons/        # アイコン素材 (svg)
├── layout/       # レイアウト関連 (site-header, site-footer, mode-switch など)
├── shared/       # 汎用 UI コンポーネント (callout, pagination など)
└── shadcn-ui/    # shadcn/ui コンポーネント
```

### 利用しているライブラリ一覧

```sh
dependencies:
@mdx-js/mdx, @radix-ui/react-slot, separator, switch, lucide-react, next, react, react-dom
tailwindcss, tailwind-scrollbar, tailwindcss-animate, rehype-pretty-code, remark-gfm
gray-matter, class-variance-authority, clsx, tailwind-merge, next-themes

devDependencies:
@next/eslint-plugin-next, @types, prettier, prettier-plugin-tailwindcss
eslint, eslint-plugin-import, eslint-plugin-react, typescript-eslint, postcss, autoprefixer
```

ESLint, Prettier, Tailwind に関連するもの以外は、「公式 > 公式推奨 > 必要最低限の外部ライブラリ」の順に厳選しました。

コードブロックのシンタックスハイライト用途には、公式推奨の rehype-pretty-code (shiki) + remark-gfm を利用しています。

`@radix-ui` 関連は、呼び出し元の shadcn/ui コンポーネントで最低限必要なものだけに絞っています。

### 主な自作機能

**「とりあえず動かしたいけど、余計なライブラリは増やしたくない」** を実現するために、以下の機能は、**Next.js 標準の API / RSC / route handler** だけで完結させました。

| 機能                     | 説明                                                 | 使っているもの                                       |
| ------------------------ | ---------------------------------------------------- | ---------------------------------------------------- |
| **MDX コンテンツ管理**   | シンプルな記事ファイル (MDX) からデータ取得          | `fs`, `@mdx-js/mdx` によるヘルパー関数を追加して対応 |
| **OGP メタデータ fetch** | リンクカード (リンクプレビュー) のための OG タグ取得 | `fetch` (公式 API だけ)                              |
| **RSS フィード出力**     | 記事一覧から RSS XML を自動生成                      | App Router + `Response` API                          |
| **ページネーション**     | ページ分割表示                                       | 自作 paginate 関数 (hooks なし)                      |

#### MDX コンテンツ管理

ファイルシステムからの読み込み、`gray-matter` による Frontmatter パース処理などをヘルパー関数としてまとめて実現しています。`compileMDX` なしで、RSC 向けにも柔軟に対応できるようにしました。

```ts:lib/mdx.ts
import fs from "fs";
import matter from "gray-matter";

export async function getAllBlogPosts() {
  const files = await fs.promises.readdir("src/content/blog");
  return await Promise.all(
    files.map(async (file) => {
      const raw = await fs.promises.readFile(
        `src/content/blog/${file}`,
        "utf-8"
      );
      const { data, content } = matter(raw);
      return {
        metadata: data,
        slug: file.replace(/\.mdx$/, ""),
        rawContent: content,
      };
    })
  );
}
```

ヘルパー関数といっても中身はとてもシンプルで、コンテンツを管理しているディレクトリ (今回の場合、`src/content/blog/`) の対象ファイルを `fs` で読み込んでいるだけです！

```ts:lib/mdx.ts
export async function getBlogPostBySlug(slug: string) {
  return getBlogPost((post) => post.slug === slug);
}

async function getBlogPost(
  predicate: (post: BlogPost) => boolean
): Promise<BlogPost | undefined> {
  const posts = await getAllBlogPosts();
  return posts.find(predicate);
}

async function getMDXData<T>(dir: string): Promise<MDXData<T>[]> {
  const files = await getMDXFiles(dir);
  return Promise.all(files.map((file) => readMDXFile<T>(path.join(dir, file))));
}

async function getMDXFiles(dir: string): Promise<string[]> {
  return (await fs.promises.readdir(dir)).filter(
    (file) => path.extname(file) === '.mdx'
  );
}

async function readMDXFile<T>(filePath: string): Promise<MDXData<T>> {
  const rawContent = await fs.promises.readFile(filePath, 'utf-8');

  const { data, content } = matter(rawContent);

  return {
    metadata: data as Frontmatter<T>,
    slug: path.basename(filePath, path.extname(filePath)),
    rawContent: content,
  };
}
```

この考え方は、Vercel の Leerob さんの見解がとても参考になりました。

https://archive.leerob.io/blog/2023

MDX コンテンツの表示も、カスタムコンポーネント (mdx-components.tsx) で柔軟に対応できるようにしました。スタイル定義は、Tailwind Typography (`prose`) を導入せず、コンポーネント側ですべて対応しています。

```tsx:src/components/content/mdx-components.tsx
export const components = {
  h2: (props) => <h2 className="mt-12 text-2xl font-bold" {...props} />,
  a: (props) => <a className="text-primary underline" {...props} />,
  pre: (props) => <pre className="bg-muted/30 rounded-lg p-4" {...props} />,
  Callout,
  LinkPreview,
  // ...
};
```

シンタックスハイライトは、公式の `@mdx-js/mdx` で `evaluate()` するときのオプションに `rehype-pretty-code`, `remark-gfm` を指定して反映させています。

```tsx:components/content/custom-mdx.tsx
import { evaluate, type EvaluateOptions } from "@mdx-js/mdx";
import * as React from "react";
import * as runtime from "react/jsx-runtime";
import rehypePrettyCode from "rehype-pretty-code";
import remarkGfm from "remark-gfm";

import { components } from "@/components/content/mdx-components";

interface CustomMDXProps {
  source: string;
  additionalComponents?: Record<string, React.ComponentType<any>>;
}

const rehypePrettyCodeOptions = {
  theme: "github-dark",
  keepBackground: true,
  defaultLang: "plaintext",
};

export async function CustomMDX({
  source,
  additionalComponents,
}: CustomMDXProps) {
  try {
    const options: EvaluateOptions = {
      ...runtime,
      remarkPlugins: [remarkGfm],
      rehypePlugins: [[rehypePrettyCode, rehypePrettyCodeOptions]],
    };

    const { default: MDXContent } = await evaluate(source, options);

    const mergedComponents = {
      ...components,
      ...(additionalComponents || {}),
    };

    return <MDXContent components={mergedComponents} />;
  } catch (error) {
    console.error("Error rendering MDX:", error);
    return (
      <div className="border-destructive/50 bg-destructive/10 text-destructive rounded-md border p-4">
        An error occurred while rendering the content.
      </div>
    );
  }
}
```

OGP メタデータの fetch と合わせて、URL を指定すれば、リンクカードのプレビューも表示できます。

例として、

```mdx
GitHub リポジトリは、こちらです。

<LinkPreview url="https://github.com/cakegaly/next-minimal-blog" />
```

とすると、以下のように表示されます。

![リンクカードの表示 (ライトモード)](https://storage.googleapis.com/zenn-user-upload/b926a3870dc0-20250310.png)
_リンクカードの表示 (ライトモード)_

#### OGP メタデータ fetch (リンクプレビュー)

リンクカード (リンクプレビュー) 用の OGP メタデータは、Server Action で fetch して取得しています。

```ts:actions/fetch-og-metadata.ts
"use server";

import { cache } from "react";

interface OGData {
  title: string;
  description: string;
  image: string;
  url: string;
}

export const getOGData = cache(
  async (url: string): Promise<Partial<OGData>> => {
    try {
      const response = await fetch(url, {
        headers: {
          "User-Agent": "bot",
        },
      });

      const html = await response.text();

      const getMetaContent = (property: string): string | undefined => {
        const regex = new RegExp(
          `<meta[^>]+(?:property|name)="${property}"[^>]+content="([^"]+)"`,
          "i"
        );
        return regex.exec(html)?.[1];
      };

      const titleMatch = /<title>(.*?)<\/title>/i.exec(html);

      return {
        title: getMetaContent("og:title") || titleMatch?.[1] || "",
        description:
          getMetaContent("og:description") ||
          getMetaContent("description") ||
          "",
        image: getMetaContent("og:image") || "",
        url,
      };
    } catch (error) {
      console.error("Error fetching OG data:", error);
      return { url };
    }
  }
);
```

#### RSS / Sitemap 自動生成

next-sitemap や外部パッケージを使わずに、App Router の route handler だけで完結させています。

RSS (Atom で出力する例):

```ts:app/rss.xml/route.ts
import { siteConfig } from "@/config/site";
import { getAllBlogPosts } from "@/lib/mdx";
import { NextResponse } from "next/server";

export async function GET() {
  const baseUrl = process.env.NEXT_PUBLIC_APP_URL || siteConfig.url;
  const posts = await getAllBlogPosts();

  const rssXml = `
    <rss version="2.0" xmlns:atom="http://www.w3.org/2005/Atom">
      <channel>
        <title>Your Blog Name</title>
        <link>${baseUrl}</link>
        <description>Your blog description here</description>
        <language>ja</language>
        <lastBuildDate>${new Date().toUTCString()}</lastBuildDate>
        <atom:link href="${baseUrl}/rss.xml" rel="self" type="application/rss+xml"/>
        ${posts
          .map((post) => {
            return `
              <item>
                <title><![CDATA[${post.metadata.title}]]></title>
                <link>${baseUrl}/blog/${post.slug}</link>
                <guid isPermaLink="true">${baseUrl}/blog/${post.slug}</guid>
                <pubDate>${new Date(post.metadata.date).toUTCString()}</pubDate>
                <description><![CDATA[${
                  post.metadata.description || ""
                }]]></description>
                ${
                  post.metadata.tags
                    ? post.metadata.tags
                        .map((tag) => `<category>${tag}</category>`)
                        .join("")
                    : ""
                }
              </item>
            `;
          })
          .join("")}
      </channel>
    </rss>
  `.trim();

  return new NextResponse(rssXml, {
    headers: {
      "Content-Type": "application/xml",
      "Cache-Control": "public, max-age=3600, s-maxage=3600",
    },
  });
}
```

サイトマップ (トップページと記事ページを出力する例):

```ts:app/sitemap.ts
import { siteConfig } from "@/config/site";
import { getAllBlogPosts } from "@/lib/mdx";
import { MetadataRoute } from "next";

export default async function sitemap(): Promise<MetadataRoute.Sitemap> {
  const baseUrl = process.env.NEXT_PUBLIC_APP_URL || siteConfig.url;

  const posts = await getAllBlogPosts();

  const blogEntries = posts.map((post) => ({
    url: `${baseUrl}/blog/${post.slug}`,
    lastModified: new Date(post.metadata.date),
    changeFrequency: "weekly" as const,
    priority: 0.8,
  }));

  const staticPages = [
    {
      url: baseUrl,
      lastModified: new Date(),
      changeFrequency: "daily" as const,
      priority: 1.0,
    },
  ];

  return [...staticPages, ...blogEntries];
}
```

#### ページネーション生成

ページ数の合計をもとに、前後のページリンクを取得するだけのシンプルな処理で対応しました。

```ts:lib/pagination.ts
export interface PaginationResult<T> {
  items: T[];
  currentPage: number;
  totalPages: number;
  totalItems: number;
}

export function paginateItems<T>(
  items: T[],
  page = 1,
  pageSize = 10
): PaginationResult<T> {
  const totalItems = items.length;
  const totalPages = Math.ceil(totalItems / pageSize);
  const currentPage = Math.max(1, Math.min(page, totalPages));

  const startIndex = (currentPage - 1) * pageSize;
  const endIndex = Math.min(startIndex + pageSize, totalItems);

  return {
    items: items.slice(startIndex, endIndex),
    currentPage,
    totalPages,
    totalItems,
  };
}
```

shadcn/ui の Pagination コンポーネントを導入しても (より柔軟なページネーションを) 実現できますが、ブログ用途であればそこまでページ数もかさまないし、hooks なしの関数ベースにしたほうが楽に管理できそうと判断しました。

:::message
microCMS, Notion API など、外部のヘッドレス CMS と連携する場合は、MDX ヘルパー関数 lib/mdx.ts を、CMS からデータフェッチするヘルパー関数 (例: lib/microcms.ts) に置き換えれば、このテンプレートの構成のままで対応できます！
:::

## おわりに

> 公式機能だけで、どこまで「ちょうどいい」ブログが作れるかな！？

そんな思い立ちから生まれたテンプレートの紹介でした。この記事を読んでいただき、ありがとうございました！

シンプルでミニマルなテンプレートを作ったつもりが、気づけば長い記事になっちゃいましたね（笑）興味がある方はぜひ Fork, Star してみてください！

https://github.com/cakegaly/next-minimal-blog
