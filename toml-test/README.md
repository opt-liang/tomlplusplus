# Encoder and Decoder for [toml-test]

> ℹ&#xFE0F; _The contents of this directory relate to testing with the [toml-test] suite, **not** anything to do with running the library's unit-tests.
> For information about running the unit-tests see [CONTRIBUTING](../CONTRIBUTING.md)._
>
> ℹ&#xFE0F; _All command snippets in this document assume the working directory is the toml++ repository root._

<br>

## Prequisites
For this document to make sense, you will need to:
1. Follow the installation instructions from the [toml-test] README to compile the `toml-test` runner
2. Add `toml-test` as an alias or have it on the system PATH
3. Clone the toml++ repository's [nlohmann/json] submodule:
```bash
git submodule update --init --depth 1 external/json
```
4. **Linux only:** Install `ninja` and `meson`:
```bash
sudo apt update && sudo apt install -y locales python3 python3-pip ninja-build
sudo pip3 install meson
```

<br>

## Building and Testing the Encoder and Decoder

### Windows with Visual Studio
Open `toml++.sln` and build the two projects in the `toml-test` solution folder. They'll be compiled in some target-specific subfolder under `/bin` in the repo root. Then run `toml-test` against them:
```bash
toml-test ./bin/win64_vc143_Release_Application/tt_decoder.exe
toml-test ./bin/win64_vc143_Release_Application/tt_encoder.exe --encoder
```

### Linux (and WSL)
```bash
# create the meson build target folder (first time only)
meson build_tt --buildtype=release -Dbuild_tt_encoder=true -Dbuild_tt_decoder=true -Dgenerate_cmake_config=false

# build and run
cd build_tt
ninja && toml-test ./toml-test/tt_decoder && toml-test ./toml-test/tt_encoder --encoder
```



[toml-test]: https://github.com/BurntSushi/toml-test
[CONTRIBUTING]: ../CONTRIBUTING.md
[nlohmann/json]: https://github.com/nlohmann/json
[meson]: https://mesonbuild.com/
