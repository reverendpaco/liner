Fork Notice
====

This is fork of "github.com/peterh/liner" with breaking changes.  I'm using it for some exploration
and will provide no feedback/support.

This module adds the ability to use a Ctrl-X followed by a rune to end the primary Prompt/PromptWithSuggestion
API method.  As such, these two methods no longer pass back a string but a struct carrying
the context of what CtrlX successor was hit.

This allows the caller to take the contents of the line so far and do with it as it wishes (call external
program, parse and reform, etc.).  Without this, there is no way for the user/caller to signify that
the current contents of the line should be processed within a different context.  'Enter' is general
evaluation, and the tab completion provides some hooks, but a more general method to provide the user/caller
with an 'evaluate thusly' is missing.

I had tried using the tab completion API method with this but was struggling with the fact that the
`nexter` goroutine still was blocking on ReadRune() and thus I could not have complete control of Stdio 
(I might have given up too early though).

I am labeling this with my own v1.0.0 and have deleted the tags of the previous "github.com/peterh/liner"
repo from which I cloned this.  I am not sure if this is exactly correct with the semver golang standard
but I didn't want to imply that this was in anyway related to the repository from which I forked.

The following shows a basic usage where `Ctrl-X c` capitalizes the current contents and `Ctrl-X l` lowercases 
them.
```go
package main

import (
	"log"
	"os"
	"path/filepath"
	"strings"

	"github.com/reverendpaco/liner"
)

var (
	history_fn = filepath.Join(os.TempDir(), ".liner_example_history")
	names      = []string{"john", "james", "mary", "nancy"}
)

func main() {
	line := liner.NewLiner()
	defer line.Close()

	line.SetCtrlCAborts(true)

	line.SetCompleter(func(line string) (c []string) {
		for _, n := range names {
			if strings.HasPrefix(n, strings.ToLower(line)) {
				c = append(c, n)
			}
		}
		return
	})

	if f, err := os.Open(history_fn); err == nil {
		line.ReadHistory(f)
		f.Close()
	}

	for {
		currentContents := ""
	restartHere:
		if lineStruct, err := line.PromptWithSuggestion("What is your name? ", currentContents, len(currentContents)); err == nil {
			name := lineStruct.Contents
			if lineStruct.CtrlXActivated {
				if lineStruct.NextLetter == 'c' {
					currentContents = strings.ToUpper(name)
					goto restartHere
				} else if lineStruct.NextLetter == 'l' {
					currentContents = strings.ToLower(name)
					goto restartHere
				} else {
					currentContents = name
					goto restartHere
				}
			} else {
				log.Print("Evaluating: ", name, " using 'ENTER'")
			}
			line.AppendHistory(name)
		} else if err == liner.ErrPromptAborted {
			log.Print("Aborted")
			break
		} else {
			log.Print("Error reading line: ", err)
		}
	}

	if f, err := os.Create(history_fn); err != nil {
		log.Print("Error writing history file: ", err)
	} else {
		line.WriteHistory(f)
		f.Close()
	}
}

```

Liner
=====



Liner is a command line editor with history. It was inspired by linenoise;
everything Unix-like is a VT100 (or is trying very hard to be). If your
terminal is not pretending to be a VT100, change it. Liner also support
Windows.

Liner is released under the X11 license (which is similar to the new BSD
license).

Line Editing
------------

The following line editing commands are supported on platforms and terminals
that Liner supports:

Keystroke    | Action
---------    | ------
Ctrl-A, Home | Move cursor to beginning of line
Ctrl-E, End  | Move cursor to end of line
Ctrl-B, Left | Move cursor one character left
Ctrl-F, Right| Move cursor one character right
Ctrl-Left, Alt-B    | Move cursor to previous word
Ctrl-Right, Alt-F   | Move cursor to next word
Ctrl-D, Del  | (if line is *not* empty) Delete character under cursor
Ctrl-D       | (if line *is* empty) End of File - usually quits application
Ctrl-C       | Reset input (create new empty prompt)
Ctrl-L       | Clear screen (line is unmodified)
Ctrl-T       | Transpose previous character with current character
Ctrl-H, BackSpace | Delete character before cursor
Ctrl-W, Alt-BackSpace | Delete word leading up to cursor
Alt-D        | Delete word following cursor
Ctrl-K       | Delete from cursor to end of line
Ctrl-U       | Delete from start of line to cursor
Ctrl-P, Up   | Previous match from history
Ctrl-N, Down | Next match from history
Ctrl-R       | Reverse Search history (Ctrl-S forward, Ctrl-G cancel)
Ctrl-Y       | Paste from Yank buffer (Alt-Y to paste next yank instead)
Tab          | Next completion
Shift-Tab    | (after Tab) Previous completion

Getting started
-----------------

```go
package main

import (
	"log"
	"os"
	"path/filepath"
	"strings"

	"github.com/peterh/liner"
)

var (
	history_fn = filepath.Join(os.TempDir(), ".liner_example_history")
	names      = []string{"john", "james", "mary", "nancy"}
)

func main() {
	line := liner.NewLiner()
	defer line.Close()

	line.SetCtrlCAborts(true)

	line.SetCompleter(func(line string) (c []string) {
		for _, n := range names {
			if strings.HasPrefix(n, strings.ToLower(line)) {
				c = append(c, n)
			}
		}
		return
	})

	if f, err := os.Open(history_fn); err == nil {
		line.ReadHistory(f)
		f.Close()
	}

	if name, err := line.Prompt("What is your name? "); err == nil {
		log.Print("Got: ", name)
		line.AppendHistory(name)
	} else if err == liner.ErrPromptAborted {
		log.Print("Aborted")
	} else {
		log.Print("Error reading line: ", err)
	}

	if f, err := os.Create(history_fn); err != nil {
		log.Print("Error writing history file: ", err)
	} else {
		line.WriteHistory(f)
		f.Close()
	}
}
```

For documentation, see http://godoc.org/github.com/peterh/liner
