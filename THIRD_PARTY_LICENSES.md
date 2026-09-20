# Third-party licenses

`dist/megane-rdkit.wasm` statically links the libraries below. Their licence
terms apply to every redistribution of that binary, alongside this package's
own `LICENSE`. The versions are the ones pinned in `scripts/build.sh`.

## RDKit (`RDKIT_TAG`, https://github.com/rdkit/rdkit)

The bulk of the binary. BSD 3-Clause; the notice below must accompany binary
redistributions.

```
BSD 3-Clause License

Copyright (c) 2006-2015, Rational Discovery LLC, Greg Landrum, and Julie Penzotti and others
All rights reserved.

Redistribution and use in source and binary forms, with or without
modification, are permitted provided that the following conditions are met:

1. Redistributions of source code must retain the above copyright notice, this
   list of conditions and the following disclaimer.

2. Redistributions in binary form must reproduce the above copyright notice,
   this list of conditions and the following disclaimer in the documentation
   and/or other materials provided with the distribution.

3. Neither the name of the copyright holder nor the names of its
   contributors may be used to endorse or promote products derived from
   this software without specific prior written permission.

THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS"
AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE
IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE
DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT HOLDER OR CONTRIBUTORS BE LIABLE
FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL
DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR
SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER
CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY,
OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE
OF THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.
```

## Boost (`BOOST_VERSION`, https://www.boost.org)

Header-only use (`property_tree` in the wrapper, plus what RDKit includes).
Boost Software License 1.0. The licence's notice requirement applies to source
distributions only, not to machine-executable object code such as this
binary. Text: https://www.boost.org/LICENSE_1_0.txt

## zlib (`ZLIB_TAG`, https://zlib.net)

Linked for RDKit's `FileParsers` (PNG metadata reader). zlib License; no
notice is required for binary distributions. Text:
https://github.com/madler/zlib/blob/develop/LICENSE

## Eigen (`EIGEN_TAG`, https://eigen.tuxfamily.org)

Header-only, used by RDKit's ETKDG and force-field code. Mozilla Public
License 2.0 (file-level copyleft on Eigen's own sources, which are unmodified
and publicly available at the URL above); binaries built against Eigen carry
no further obligation. Text: https://www.mozilla.org/MPL/2.0/

## Emscripten runtime

The JavaScript glue in `dist/megane-rdkit.mjs` and the runtime support
compiled into the binary come from Emscripten (`EMSDK_VERSION`), MIT /
University of Illinois-NCSA dual licence:
https://github.com/emscripten-core/emscripten/blob/main/LICENSE
