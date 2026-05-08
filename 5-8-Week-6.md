# Week 3
### What I Did

Week 6 means I should make some progress on my mid-quarter goal done. After a few weeks of debugging code on the `dbittman-e1000-driver` branch, I'm now trying, with the sage advice of Surendra, to just cherry-pick features from the branch off of a clean slate. So then, at least, I'm only getting errors from features that I'm trying to implement. I'm really thankful I wrote a detailed notebook entry during Week 3, because I kind of got lost in the sauce and totally forgot what research I had already done! So I'm going to try and do that again with this notebook:

`src/ports/femto/src/main.rs` is the starting point of it all, the program seems to panic when run in the terminal at line `269` in this function:
<details>
  <summary>main()</summary>
  
  ```rust
  fn main() {
      let mut editor = Editor::new();
  
      let mut args: Vec<String> = std::env::args().collect();
      if args.len() == 2 {
          editor.open(PathBuf::from(args.remove(1)));
      } else if args.len() > 2 {
          return println!("Error: too many arguments.\nusage: femto [FILE]");
      }
      // RIGHT BELOW HERE! <----------------------------------------------------
      let mut stdout = stdout().into_raw_mode().expect("Unsupported terminal.");
      write!(stdout, "{}", ToAlternateScreen).unwrap();
  
      loop {
          print_screen(&mut stdout, &mut editor);
          if handle_keys(&mut editor) {
              break;
          }
      }
  
      write!(stdout, "{}{}", termion::clear::All, Goto(1, 1)).unwrap();
      stdout.flush().unwrap();
  }
  ```
</details>

This is caused by the function `into_raw_mode()` being called on `stdout()`. This function is a custom function found in Termion:
`src/ports/termion/src/raw.rs`
<details>
  <summary>into_raw_mode()</summary>
  
```rust
pub trait IntoRawMode: Write + Sized {
    /// Switch to raw mode.
    ///
    /// Raw mode means that stdin won't be printed (it will instead have to be written manually by
    /// the program). Furthermore, the input isn't canonicalised or buffered (that is, you can
    /// read from stdin one byte of a time). The output is neither modified in any way.
    fn into_raw_mode(self) -> io::Result<RawTerminal<Self>>;
}

impl<W: Write> IntoRawMode for W {
    fn into_raw_mode(self) -> io::Result<RawTerminal<W>> {
        let mut ios = get_terminal_attr()?;
        let prev_ios = ios;

        raw_terminal_attr(&mut ios);

        set_terminal_attr(&ios)?;

        Ok(RawTerminal {
            prev_ios: prev_ios,
            output: self,
        })
    }
}
```
</details>

As visible in the code above, the function uses the functions `get_terminal_attr()`, `raw_terminal_attr()`, and `set_terminal_attr()`.
These functions are defined in:
`src/ports/termion/src/sys/unix/attr.rs`
<details>
  <summary>attr.rs</summary>
  
```rust
pub fn get_terminal_attr() -> io::Result<Termios> {
    extern "C" {
        pub fn tcgetattr(fd: c_int, termptr: *mut Termios) -> c_int;
    }
    unsafe {
        let mut termios = mem::zeroed();
        cvt(tcgetattr(1, &mut termios))?;
        Ok(termios)
    }
}

pub fn set_terminal_attr(termios: &Termios) -> io::Result<()> {
    extern "C" {
        pub fn tcsetattr(fd: c_int, opt: c_int, termptr: *const Termios) -> c_int;
    }
    cvt(unsafe { tcsetattr(1, 0, termios) }).and(Ok(()))
}

pub fn raw_terminal_attr(termios: &mut Termios) {
    extern "C" {
        pub fn cfmakeraw(termptr: *mut Termios);
    }
    unsafe { cfmakeraw(termios) }
}
```
</details>

This makes use of a smaller support function that is suppossed to convert `libc` return values to io errors, `cvt()`
`src/ports/termion/src/sys/unix/mod.rs`

<details>
  <summary>cvt()</summary>
  
```rust
fn cvt<T: IsMinusOne>(t: T) -> io::Result<T> {
    if t.is_minus_one() {
        Err(io::Error::last_os_error())
    } else {
        Ok(t)
    }
}
```
</details>

The `Trait`, `IsMinusOne` is defined in the file as well, it does what the name suggests.

Going back to `attr.rs`, we see our first instances of `libc` functions! This is where the code fails or rather returns an "Unsupported" Error Code, or `-1`, which is taken by `cvt()`. `cvt()` tries to find `errno`, but since it is never set, `errno` is still `0` (Success) and Twizzler outputs this  slightly confusing message on panic:
```json
Os { code: 0, kind: Other, message: "operation successful" }
```

These `libc` functions are `tcsetattr`, `tcgetattr`, and `cfmakeraw`. Heading over to Twizzler's `mlibc` submodule that it uses for `libc` syscalls, we see the following implementations for these functions:
`toolchain/src/mlibc/sysdeps/twizzler/sysdeps.cpp`
<details>
  <summary>sys_ functions</summary>
  
```cpp
int sys_tcgetattr(int fd, struct termios *attr) {
	return ENOSYS;
}

int sys_tcsetattr(int fd, int optional_action, const struct termios *attr) {
	return ENOSYS;
}

// No function for sys_cfmakeraw ;(
```
</details>

We also know that code `ENOSYS` is also defined in `twz_errno_generic` as `NO_SUCH_OPERATION`.
The `struct termios` is defined in:
`toolchain/src/mlibc/sysdeps/twizzler/include/abi-bits/termios.h`
which is a symlink to `toolchain/src/mlibc/abis/linux/termios.h`
<details>
  <summary>termios.h</summary>
  
```c
typedef unsigned char cc_t;
typedef unsigned int speed_t;
typedef unsigned int tcflag_t;

/* indices for the c_cc array in struct termios */
#define NCCS     32
// ...
struct termios {
	tcflag_t c_iflag; // Handles basic translation and filtering of input
	tcflag_t c_oflag; // Formats output so the terminal hardware or emulator displays it correctly
	tcflag_t c_cflag; // Controls the electrical/hardware aspects of the serial line (largely ignored)
	tcflag_t c_lflag; // Manages echoing, line editing, and signal generation (high-level)
	cc_t c_line; // "Line discipline"
	cc_t c_cc[NCCS]; // An array map of keystrokes to special actions
	speed_t c_ibaud; // Input baud rate
	speed_t c_obaud; // Output baud rate
};
```
</details>

### Troubles

### Goals and Aspirations
