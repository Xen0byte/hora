<div align="center">
  <img src="asset/logo.svg" width="70%"/>
</div>

# Hora

**[[Homepage](https://horasearch.com/)]** **[[Document](https://horasearch.com/doc)]** **[[Examples](https://horasearch.com/doc/example.html)]**

***Hora Search Everywhere!***

Hora, a approximate nearest neighbor search algorithm library. We implement all code in `Rust🦀` for reliability, high level abstraction and high speed comparable to `C++`.

Hora, **`ほら`** in Japanese, sound like `[hōlə]`, means `Wow`, `You see!` or `Look at that!`. The name is inspired by a famous Japanese song **`「小さな恋のうた」`**.

# Key Features

* **Performant** ⚡️
  * **SIMD-Accelerated ([packed_simd](https://github.com/rust-lang/packed_simd))**
  * **Stable algorithm implementation**
  * **Multiple threads design**

* **Multiple Languages Support** ☄️
  * `Python`
  * `Javascript`
  * `Java`
  * `Go` (WIP)
  * `Ruby` (WIP)
  * `Swift` (WIP)
  * `R` (WIP)
  * `Julia` (WIP)
  * **also can serve as a service**

* **Multiple Indexes Support** 🚀
  * `Hierarchical Navigable Small World Graph Index(HNSW)` ([detail](https://arxiv.org/abs/1603.09320))
  * `Satellite System Graph (SSG)` ([detail](https://arxiv.org/abs/1907.06146))
  * `Product Quantization Inverted File(PQIVF)` ([detail](https://lear.inrialpes.fr/pubs/2011/JDS11/jegou_searching_with_quantization.pdf))
  * `Random Projection Tree(RPT)` (LSH, WIP)
  * `BruteForce` (naive implementation with SIMD)

* **Portable** 💼
  * Support `no_std` (WIP, partial)
  * Support `Windows`, `Linux` and `OS X`
  * Support `IOS` and `Android` (WIP)
  * **No** heavy dependency, such as `BLAS`

* **Reliability** 🔒
  * `Rust` compiler secure all code
  * Memory managed by `Rust` for all language libs such as `Python lib`
  * Broad testing coverage

* **Multiple Distances Support** 🧮
  * `Dot Product Distance`
    * ![equation](https://latex.codecogs.com/gif.latex?D%28x%2Cy%29%20%3D%20%5Csum%7B%28x*y%29%7D)
  * `Euclidean Distance`
    * ![equation](https://latex.codecogs.com/gif.latex?D%28x%2Cy%29%20%3D%20%5Csqrt%7B%5Csum%7B%28x-y%29%5E2%7D%7D)
  * `Manhattan Distance`
    * ![equation](https://latex.codecogs.com/gif.latex?D%28x%2Cy%29%20%3D%20%5Csum%7B%7C%28x-y%29%7C%7D)
  * `Cosine Similarity`
    * ![equation](https://latex.codecogs.com/gif.latex?D%28x%2Cy%29%20%3D%20%5Cfrac%7Bx%20*y%7D%7B%7C%7Cx%7C%7C*%7C%7Cy%7C%7C%7D)

* **Productive** ⭐
  * Well documented
  * Elegant and simple API, easy to learn

# Installation

### `Rust`

in `Cargo.toml`

```toml
[dependencies]
hora = "0.1.0"
```

### `Python`

```Bash
$ pip install hora
```

### `Building from source`

```bash
$ git clone https://github.com/hora-search/hora
$ cargo build
```

# Benchmark
<img src="asset/fashion-mnist-784-euclidean_10_euclidean.png"/>

by `aws t2.medium (CPU: Intel(R) Xeon(R) CPU E5-2686 v4 @ 2.30GHz)`

# Examples

**`Rust` example**

```Rust
use hora::core::ann_index::ANNIndex;
use rand::{thread_rng, Rng};
use rand_distr::{Distribution, Normal};

pub fn demo() {
    let n = 1000;
    let dimension = 64;

    // make sample points
    let mut samples = Vec::with_capacity(n);
    let normal = Normal::new(0.0, 10.0).unwrap();
    for _i in 0..n {
        let mut sample = Vec::with_capacity(dimension);
        for _j in 0..dimension {
            sample.push(normal.sample(&mut rand::thread_rng()));
        }
        samples.push(sample);
    }

    let mut index = hora::index::hnsw_idx::HNSWIndex::<f32, usize>::new(
        dimension,
        &hora::index::hnsw_params::HNSWParams::<f32>::default(),
    ); // init index
    for (i, sample) in samples.iter().enumerate().take(n) {
        index.add(sample, i).unwrap(); // add point
    }
    index.build(hora::core::metrics::Metric::Euclidean).unwrap();

    let mut rng = thread_rng();
    let target: usize = rng.gen_range(0..n);
    println!(
        "{:?} has neighbors: {:?}",
        target,
        index.search(&samples[target], 10) // search for k nearest neighbors
    ); // 523 has neighbors: [523, 762, 364, 268, 561, 231, 380, 817, 331, 246]
}
```

**`Python` exmaple**

```Python
import numpy as np
from hora import HNSWIndex

dimension = 50
n = 1000
index = HNSWIndex(dimension, "usize")
samples = np.float32(np.random.rand(n, dimension))
for i in range(0, len(samples)):
    index.add(np.float32(samples[i]), i)
index.build("euclidean")
target = np.random.randint(0, n)
print("{} has neighbors: {}".format(target, index.search(samples[target], 10))) # 631 has neighbors: [631, 656, 113, 457, 584, 179, 586, 979, 619, 976]

```

# Roadmap

- [ ] Full Coverage Test
- [ ] implement [EFANNA](http://arxiv.org/abs/1609.07228) algo to achieve faster KNN graph building
- [ ] Swift Support and also IOS/Mac OS deployment example
- [ ] Support `R` 
- [ ] support `mmap`

# Related Projects and Comparison

* [Faiss](https://github.com/facebookresearch/faiss), [Annoy](https://github.com/spotify/annoy), [ScaNN](https://github.com/google-research/google-research/tree/master/scann): 
  * **`Hora`'s implementation is strongly inspired by these lib.**
  * `Faiss` focus more on the GPu scenerio, and `Hora` is lighter than Faiss, such as **no heavy dependency**.
  * `Hora` expects to support more language, and everything related to performance shall be implemented by Rust🦀.
  * `Annoy` only support `LSH(Random Projection)` algorithm.
  * `ScaNN` and `Faiss` are less user-friendly, such as lack of document.
  * Hora is **ALL IN RUST** 🦀.

* [Milvus](https://github.com/milvus-io/milvus), [Vald](https://github.com/vdaas/vald), [Jina AI](https://github.com/jina-ai/jina)
  * `Milvus` and `Vald` also support multiple languages, but serve as a service instead of a lib
  * `Milvus` is built upon some libs such as `Faiss`, while `Hora` is an algorithm lib with all the algo implemented by itself

# Contribute

we are pretty gald to have you to participate, any contributions is welcome, including the documentations and tests.
you can do the  `Pull Requests`, `Issue` on the github, and we will review it as soon as possible.

We use GitHub issues for tracking suggestions and bugs.

To install for development:

#### clone the repo

```bash
git clone https://github.com/hora-search/hora
```

#### build

```bash
cargo build
```

#### test

```bash
cargo test --lib
```

#### try the changes

```bash
cd exmaples
cargo run
```

# License

The entire repo is under [Apache License](https://github.com/hora-search/hora/blob/main/LICENSE).
