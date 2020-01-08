# xxi
## A modern X11 library.

## Index

 - [Brief](#Brief)
 - [Examples](#Examples)

## Brief

An X11 protocol library written from scratch in Rust.

### Goals

 - No xorg/X11 dependency.
 - Async and sync support.
 - As performant as possible while promoting a very friendly API.

## Examples

```rust
extern crate xxi;

use {
    std::{io, cmp::max},
    xxi::{XAttributes, XSession, EventType::{KeyPress, ButtonPress, MotionNotify, ButtonRelease}},
};

// The "50 line window manager"
// http://incise.org/tinywm.html

fn main() -> io::Result {
    let mut session = xxi::XSession::new();
    let display = session.open_display(0x00)?;

    let mut attrs = XAttributes::empty();
    let mut start: (usize, usize, usize) = (0, 0, 0);

    for event in session.event_stream()? {
        if event.type_ == KeyPress {
        if let Some(subwindow) = event.xkey.subwindow {
            session.raise_window(subwindow);
            continue;
        }}

        if event.type_ == ButtonPress {
        if let Some(subwindow) = event.xbutton.subwindow {
            session.grab_pointer(
                subwindow,
                true,
                PointerMotionMask | ButtonReleaseMask,
                GrabModeAsync,
                GrabModeAsync,
                None, None, 0
            )?;

            attrs = subwindow.get_attributes()?;
            start = (event.x_root, event.y_root, event.button);
            continue;
        }}

        match (event.type_, start) {
            (MotionNotify, (x_root, y_root, button)) => {
                let base = session
                    .check_typed_event(EventType::MotionNotify)?
                    .last()
                    .unwrap();

                let xdiff = base.xbutton.x_root - x_root;
                let ydiff = base.xbutton.y_root - y_root;

                session.move_resize_window(
                    base.xmotion.window,
                    attrs.x + (if start_button == 1 { xdiff } else { 0 }),
                    attrs.y + (if start_button == 1 { ydiff } else { 0 }),
                    max(1, attr.width + (if start_button == 3 { xdiff } else { 0 }),
                    max(1, attr.height + (if start_button == 3 { ydiff } else { 0 }),
                )?;
            }

            (ButtonRelease, _) => session.ungrab_pointer()?,
            _ => (),
        }
    }
}
```
