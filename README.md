<div align="center">
  <h1>Voy</h1>
  <strong>A WASM vector similarity search engine written in Rust</strong>
</div>

![voy: a vector similarity search engine in WebAssembly][demo]

[![npm version](https://badge.fury.io/js/voy-search.svg)](https://badge.fury.io/js/voy-search)

- **Tiny**: 75KB gzipped, 69KB brotli.
- **Fast**: Create the best search experience for the users. Voy uses [k-d tree][k-d-tree] to index and provide fast search
- **Tree Shakable**: Optimize bundle size and enable asynchronous capabilities for modern Web API, such as [Web Workers](https://developer.mozilla.org/en-US/docs/Web/API/Web_Workers_API/Using_web_workers).
- **Resumable**: Generate portable embeddings index anywhere, anytime.
- **Worldwide**: Designed to deploy and run on CDN edge servers.

> **🚜 Work in Progress**
>
> Voy is under active development. As a result, the API is not stable. Please be aware that there might be breaking changes before the upcoming 1.0 release.
>
> A sneak peek of what we are working on:
>
> - [ ] Built-in text transformation in WebAssembly: As of now, voy relies on JavaScript libraries like [`web-ai`][web-ai] to generate text embeddings. See [Usage](#usage) for more detail.
> - [ ] Index update: Currently it's required to [re-build the index](#indexinput-resource-serializedindex) when a resource update occurs.
> - [x] TypeScript support: Due to the limitation of WASM tooling, complex data types are not auto-generated.

## Installation

```bash
# with npm
npm i voy-search

# with Yarn
yarn add voy-search

# with pnpm
pnpm add voy-search
```

## APIs

#### `index(input: Resource): SerializedIndex`

**Parameters**

```ts
interface Resource {
  embeddings: Array<{
    id: string; // id of the resource
    title: string; // title of the resource
    url: string; // path to the resource
    embeddings: number[]; // embeddings of the resource
  }>;
}
```

**Return**

```ts
type SerializedIndex = string; // serialized k-d tree
```

#### `search(index: SerializedIndex, query: Query, k: NumberOfResult): Nearests`

**Parameter**

```ts
type SerializedIndex = string; // serialized k-d tree

type Query = Float32Array; // embeddings of the search query

type NumberOfResult = number; // K top results to return
```

**Return**

```ts
type Nearests = Array<{
  id: string; // id of the nearest resource
  title: string; // title of the nearest resource
  url: string; // path of the nearest resource
  embeddings: number[]; // embeddings of the nearest resource
}>;
```

## Usage

As of now, voy relies on libraries like [`web-ai`][web-ai] to generate embeddings for text:

```js
import { TextModel } from "@visheratin/web-ai";

const voy = await import("voy-search");

const phrases = [
  "That is a very happy Person",
  "That is a Happy Dog",
  "Today is a sunny day",
];
const query = "That is a happy person";

// Create text embeddings
const model = await (await TextModel.create("gtr-t5-quant")).model;
const processed = await Promise.all(phrases.map((q) => model.process(q)));

// Index embeddings with voy
const data = processed.map(({ result }, i) => ({
  id: String(i),
  title: phrases[i],
  url: `/path/${i}`,
  embeddings: result,
}));
const input = { embeddings: data };
const index = voy.index(input);

// Perform similarity search for a query embeddings
const q = await model.process(query);
const result = voy.search(index, q.result, 1);

// Display search result
result.neighbors.forEach((result) =>
  console.log(`✨ voy similarity search result: "${result.title}"`)
);
```

## License

Licensed under either of

- Apache License, Version 2.0, ([LICENSE-APACHE](LICENSE-APACHE) or http://www.apache.org/licenses/LICENSE-2.0)
- MIT license ([LICENSE-MIT](LICENSE-MIT) or http://opensource.org/licenses/MIT)

at your option.

## Sponsor

<a href="https://reflect.app" target="_blank"><img src="https://avatars.githubusercontent.com/u/73365487?s=64&v=4"></a>
<a href="https://github.com/markhughes" target="_blank"><img src="https://avatars.githubusercontent.com/u/1357323?s=64&v=4"></a>

### Contribution

Unless you explicitly state otherwise, any contribution intentionally
submitted for inclusion in the work by you, as defined in the Apache-2.0
license, shall be dual licensed as above, without any additional terms or
conditions.

[demo]: ./voy.gif "voy demo"
[web-ai]: https://github.com/visheratin/web-ai
[k-d-tree]: https://en.wikipedia.org/wiki/K-d_tree
