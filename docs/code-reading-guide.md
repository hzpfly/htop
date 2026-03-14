# Code Reading Guide

This guide is for developers who want to understand the `htop` source tree quickly.

## 1. Build once, then run under a debugger

Start with a local debug build so symbols and assertions are available.

```sh
./autogen.sh
./configure --enable-debug
make -j"$(nproc)"
```

Then run `./htop` from the repository root and keep a debugger session ready (`gdb ./htop`).

## 2. Learn the high-level architecture first

Use this rough mental model before diving into individual files:

- **Entry point and startup**: `htop.c`
- **Platform abstraction**: `Machine.*`, `ProcessTable.*`, plus OS-specific directories such as `linux/`, `freebsd/`, `darwin/`
- **Data model**: `Process.*`, `Row.*`, `Object.*`, `Vector.*`, `Hashtable.*`
- **UI framework**: `Panel.*`, `MainPanel.*`, `Header.*`, `ScreenManager.*`, `FunctionBar.*`
- **Meters and columns**: `Meter.*`, `DynamicMeter.*`, `DynamicColumn.*`, concrete meter implementations like `CPUMeter.*`, `MemoryMeter.*`
- **Configuration/state**: `Settings.*`, option/screen panel files (`*Panel.*`, `*Screen.*`)

## 3. Follow one end-to-end flow

A practical way to understand the code is to trace one interaction all the way through. Good first flows:

1. Start-up and initial screen draw.
2. Pressing a key in the main view.
3. Changing sort column.
4. Killing a selected process.

For each flow, map:

- Where input is captured.
- Where state changes happen.
- Where rendering is triggered.
- Which platform hooks are involved.

This quickly reveals the boundaries between UI, generic logic, and OS-specific collection code.

## 4. Use naming patterns to navigate faster

The project follows useful naming conventions:

- `Something.c` + `Something.h` pairs define a component.
- `*Meter.c` files render information widgets.
- `*Panel.c` and `*Screen.c` files implement interactive UI screens.
- OS directories (`linux/`, `openbsd/`, etc.) override or extend behavior per platform.

When you find a type in a header, jump to its matching `.c` file and then search call-sites with:

```sh
rg "TypeName|function_name"
```

## 5. Recommended reading order (first day)

1. `README.md` (build/run context)
2. `htop.c` (program entry and initialization)
3. `MainPanel.*` and `Panel.*` (core interaction model)
4. `ProcessTable.*` and `Process.*` (process collection/model)
5. Two concrete meters, e.g. `CPUMeter.*` and `MemoryMeter.*`
6. One OS-specific process backend, e.g. files in `linux/`
7. `Settings.*` and configuration-related panels

## 6. Recommended reading order (deeper pass)

Once you know the basics:

- Trace one meter from data source to rendering.
- Trace one action command (e.g. kill/renice) from key handling to system call.
- Compare generic code with one non-Linux backend to understand abstraction limits.

## 7. Practical debugging tips

- Build with `--enable-debug` during exploration.
- Use `rg` heavily instead of recursive grep.
- Keep notes of ownership boundaries (UI vs model vs platform).
- Prefer small, behavior-preserving refactors once you understand a flow.

## 8. Contributor checklist before changing code

- Reproduce behavior manually in `./htop`.
- Identify which layer your change belongs to (UI/model/platform).
- Validate on at least one supported platform backend (or ensure generic-only changes).
- Run relevant build/tests before sending a patch.

---

If you are brand new to the codebase, do not try to understand everything at once—pick one flow and own it end-to-end first.
