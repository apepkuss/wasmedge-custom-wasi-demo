# wasmedge-custom-wasi-demo

This demo repo consists of two submodules:

- `WasmEdge` is the repo of WasmEdge WebAssembly runtime, which includes `wasmedge-sys`. We define a `CustomWasiModule` struct and wasi host functions in `wasmedge-sys`. For now, 6 wasi host functions are implemented, which are `wasi_args_get`, `wasi_args_sizes_get`, `wasi_environ_get`, `wasi_environ_sizes_get`, `fd_write` and `proc_exit`.

- `wasmedge-wasi` defines the two crates `wasmedge-wasi` and `wasmedge-wasi-example`, which implement the WASI specification.

## Clone the repo

Run the following command in the terminal program to clone the repo:

  ```bash
  git clone --recurse-submodules git@github.com:apepkuss/wasmedge-custom-wasi-demo.git
  ```

## Run the test cases for the wasi host functions

The test cases for the wasi host functions are located in `WasmEdge/bindings/rust/wasmedge-sys/src/instance/custom_wasi_module.rs`. For now, we have 6 test cases for the 6 wasi host functions, respectively. To run the test cases, you can follow the steps below:

- Install `rustup` and `Rust`

  Go to the [official Rust webpage](https://www.rust-lang.org/tools/install) and follow the instructions to install `rustup` and `Rust`.

  > It is recommended to use Rust 1.63 or above in the stable channel.

  Then, add `wasm32-wasi` target to the Rustup toolchain:

  ```bash
  rustup target add wasm32-wasi
  ```

- Install `libwasmedge 0.11.2`

  Refer to the [Quick Install](https://wasmedge.org/book/en/quick_start/install.html#quick-install) section of WasmEdge Runtime Book to install `libwasmedge`. Or, use the following command directly

  ```bash
  // The command will create a directory `$HOME/.wasmedge`
  ./install_libwasmedge.sh -v 0.11.2

  source $HOME/.wasmedge/env
  ```

- Run the test cases

  Take the test case `test_wasi_fd_write` as an example. To run the test case, run the two commands below:

  ```bash
  cd wasmedge-custom-wasi-demo/WasmEdge/bindings/rust/

  cargo test -p wasmedge-sys test_wasi_fd_write -- --nocapture
  ```

  After running the commands, you will see the following output:

  ```bash
  >>> wasi_fd_write begins
  in WasiEnviron::fd_write
  Hello, world!This is a test.
  <<< wasi_fd_write ends
  ```

## Add new wasi host functions

The wasi host functions are defined in `WasmEdge/bindings/rust/wasmedge-sys/src/instance/custom_wasi_module.rs`. To add new wasi host functions, you should follow the steps below:

- Define a new wasi host function.

  Take the `wasi_fd_write` function as an example. Define the wasi host   function in `custom_wasi_module.rs`, and it looks like the below:
  
  ```rust
  fn wasi_fd_write(cf: CallingFrame, args: Vec<WasmValue>) ->   Result<Vec<WasmValue>, HostFuncError> {
      ...
  }
  ```rust

- Add the new wasi host function to the `CustomWasiModule::init` method.

  ```rust
  impl CustomWasiModule {
    ...

    pub fn init(
        &mut self,
        args: Option<Vec<&str>>,
        envs: Option<Vec<(&str, &str)>>,
        preopened_dirs: Option<Vec<(cap_std::fs::Dir, &std::path::Path)>>,
    ) -> WasmEdgeResult<()> {
        ...

        // * add wasi host functions

        
        // `fd_write`
        let ty = FuncType::create(
            vec![ValType::I32, ValType::I32, ValType::I32, ValType::I32],
            vec![ValType::I32],
        )?;
        self.add_func(
            "fd_write",
            Function::create(&ty, Box::new(wasi_fd_write), 0)?,
        );

        ...
    }

    ...
  }
  ```
