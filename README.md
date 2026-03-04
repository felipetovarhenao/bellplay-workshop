# Introduction to algorithmic audio with _bellplay~_

## Setup

To install _bellplay~_ for this workshop, follow these steps:

1. Download the [bellplay~ pre-release version](https://github.com/felipetovarhenao/bellplay/releases/dev)—this is the version we will be using for the workshop, not the version available in the website below.
2. Download [Visual Studio Code](https://code.visualstudio.com/)
3. Please follow the instructions in [this website](https://bellplay.net/docs/installation), with the only difference being that the _bellplay~_ version should be the pre-release you downloaded above, instead of the one it links to in the website. Make sure you also follow the [instructions](https://bellplay.net/docs/installation/#text-editor-setup-recommended) on how to setup [Visual Studio Code](https://code.visualstudio.com/).

---

## Documentation

Always refer to the [official online documentation](https://bellplay.net) for any _bellplay~_ related question.

---

## _bell_ programming language primer

### Basics

- **Comments**: Use `##` for single-line comments and `#( ... )#` for multi-line blocks.
- **Printing**: Use `print("message")` to see output in the console.
- **Semicolons**: Use `;` to **separate** (or concatenate) statements. You do **not** need one on the very last line of a block of code.

### Variables

- **Local Variables**: Start with `$` (e.g., `$pitch = c5`). They exist only within the current script.
- **Global Variables**: No prefix (e.g., `Tempo = 120`). They persist across scripts for the current session.
- **Assignment**: Use `=` to store values. Unassigned variables default to `null`.

### Data types

| Type         | Description                                                                            | Examples                                                                                                    |
| ------------ | -------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------- |
| **Integer**  | Whole numbers                                                                          | `60`, `-7`                                                                                                  |
| **Float**    | Floating-point (or decimal) numbers                                                    | `440.0`, `.5`, `1.` `1e-3`                                                                                  |
| **Rational** | Fractional numbers                                                                     | `1/4`, `3/2`                                                                                                |
| **Pitch**    | [Scientific pitch notation](https://en.wikipedia.org/wiki/Scientific_pitch_notation)   | `C5` (Middle C), `F#4`, `Bb3`, `Gq3`                                                                        |
| **Symbol**   | The _bell_ version of a _string_ in other programming languages. Useful for text data. | `"hello world"` (double-quotes) `'hello world'` (single quotes) `` `hello `` (back-tick, no spaces allowed) |

- **Pitch Accidentals**: `#` (sharp), `b` (flat), `q` (quarter-sharp), `d` (quarter-flat), `^` (eighth-sharp), `v` (eighth-flat)
- **MIDI Cents**: bell represents pitches internally as MIDI cents (Middle C `C5` = `6000`, `C#` = `6100`, `C#q` = `6150`).

### Lisp-like linked lists (_llll_)

The **llll** is bell's core data structure, which allows combining all data types above.

- **Creation**: Separate items with spaces (no commas): `$chord = C5 E5 G5`.
- **Nesting**: Use `[ ]` to group lists: `$progression = [C5 E5 G5] [F5 A5 C6]`.
- **Flatting**: The `flat($list)` function removes nesting levels.

### Common operators

- **Arithmetic**: `+`, `-`, `*`, `/`, `%` (remainder), `**` (power).
- **Range**: `1...4` creates the list `1 2 3 4`.
- **Repeat**: `(C4 D4) :* 2` results in `C4 D4 C4 D4`.
- **Access**: `$list:1` gets the first item (1-based indexing).

### List operations

_bell_ automatically applies operations across lists, saving you from writing many loops:

- **One-to-Many**: `$chord + 100` transposes every note in the list up by 1 semitone.
- **Many-to-Many**: `(6000 6400) + (100 200)` results in `6100 6600`.

### Control flow

- **Conditionals**: `if <condition> then <action> else <action>`.
- **Loops**:
- `for $item in $list collect ( $item + 12 )` (executes and collects the result of each iteration).
- `for $item in $list do ( print($item) )` (executes and keeps the result of the very last iteration).

---

## _bellplay~_ programming language primer

### The workflow loop

Every _bellplay~_ script generally follows this three-step sequence:

1. **Generation**: Create one or more buffers via synthesis (e.g., `cycle()` function) or sampling (`importaudio()` or `ezsampler()` functions).
2. **Transcription**: Schedule buffers onto the timeline using `transcribe(...)`.
3. **Rendering**: Render all transcribed buffers into a single one with `render()`.

### Loading & creating audio

- **Import**: `$buffer = importaudio('guitar.wav');`.
- **Oscillator functions**: `cycle()`, `saw()`, `tri()`, `rect()`, `noise()`, `simplefm()`.
- **Sampling** (more advanced): `$pluck = ezsampler(c5, 1000)`
- **Attributes**: Most oscillator functions use `@frequency` (Hz) and `@duration` (ms).
- _Example_: `$sound = cycle(@frequency 440 @duration 1000);`.

### The _bellplay~_ timeline

Buffers must be _transcribed_ to be heard. The `transcribe()` function places a buffer on a global timeline.

- **Common Attributes**:
- `@onset`: Start time in ms (default is `0`).
- `@gain`: Linear gain (`0.0` to `1.0`) or an envelope.
- `@pan`: Stereo position (`0` = _left_, `0.5` = _center_, `1` = _right_).
- _Example_: `transcribe(@buffer $b @onset 500 @gain 0.5 @pan 0.2);`.

### Buffer processing operations

Use `process()` to apply effects to a buffer. Process takes a buffer and a list of operations, which are generated via functions (think of them as recipes for `process` to know what to do to the buffer).

- **Common processes**: `normalize()`, `gain()`, `window()`, `freeverb()`, `resample()`, `overdrive()`, `biquad()`.
- **Example**: `$b.process(gain(0.5) window())`.

### Audio analysis

Analyze buffers to extract musical data.

- **Functions**: `onsets()`, `larm()` (long-term loudness), `pitchmelodia()`, `envmaxtime()`.
- **Retrieving Data**: Use `getkey($buffer, 'keyname')`.
- _Example_: `$markers = $b.analyze(onsets()).getkey('onsets');`.

### The render call

The `render()` function executes the transcription and can apply processing to the resulting buffer.

- **Common attributes**:
- `@play 1`: Automatically plays the result.
- `@process`: Optional process to apply to the resulting buffer—e.g., `normalize()` or `freeverb()`.
- `@reset 1`: Clears the timeline after rendering. Useful when doing batch rendering, or multi-pass rendering.
- _Example_: `$finalbuf = render(@play 1 @process normalize(-3) freeverb())`.

---

## Tips and common mistakes

- **Missing semi-colons**: always check for missing semicolons in your code, which may lead to unexpected behavior or errors.
- **Mispelled functions**: make sure the functions you're using actually exist.
- **Namespacing** _bellplay~_ functions like `saw()` and `render()` are not reserved keywords; if you name a variable `render = 10`, you will "break" the function and won't be able to use it until you restart.
- **Multi-buffer transciption**: The `transcribe` buffer can only transcribe one buffer at a time. To transcribe multiple buffer, make use of `for` or `while` loops.
- **Misusing functions**: When in doubt, always check the [reference documentation](https://bellplay.net/docs/reference/buffer-utilities/transcribe) and make sure you understand which arguments a function expects to behave as expected.

---

## Resources

- [**Bell programming tutorials**](https://felipe-tovar-henao.com/bell-tutorials/)
- [**Replay: Algorithmic music puzzles**](https://felipe-tovar-henao.com/replay/)
