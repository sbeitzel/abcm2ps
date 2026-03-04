# abcm2ps Extensions

This file documents formatting directives that are implemented in this fork of
abcm2ps but are not part of the [ABC standard v2.2](https://abcnotation.com/wiki/abc:standard:v2.2).

---

## `%%stemalignment`

**Syntax:** `%%stemalignment <integer>`

**Type:** integer
**Default:** `0` (disabled)
**Scope:** global (tune header or file preamble), or voice-level (tune body)

### Description

Forces all note stems to end at a fixed vertical position relative to the
centre line of the staff, rather than at the default length computed from the
note head position. The integer argument encodes both the direction of the
stems and the target endpoint:

- A **negative** value forces stems **downward**; the target endpoint is
  `|value|` diatonic steps *below* the staff centre.
- A **positive** value forces stems **upward**; the target endpoint is
  `|value|` diatonic steps *above* the staff centre.
- `0` disables the directive (default behaviour).

One *diatonic step* equals 3 PostScript points in abcm2ps's internal
coordinate system.

### Staff centre reference

The staff centre is the middle line of a standard five-line staff. For common
clefs, the note on the centre line is:

| Clef   | Centre-line note |
|--------|-----------------|
| Treble | B4              |
| Alto   | C4 (middle C)   |
| Tenor  | A3              |
| Bass   | D3              |

### Examples

#### Global: stems down to middle C (treble clef)

Middle C (C4) is 6 diatonic steps below the treble-clef centre line (B4).

```abc
X:1
T:Stems aligned at middle C
M:4/4
L:1/4
%%stemalignment -6
K:C treble
GABC | DEFG | ABCD | EFGc |
```

All stems are forced downward. The stem tips align at the C4 ledger line
below the staff.

#### Global: stems up, six steps above centre

```abc
X:2
T:Stems aligned at A5
M:4/4
L:1/4
%%stemalignment 6
K:C treble
CDEF | GABc | cBAG | FEDC |
```

All stems are forced upward. The stem tips align at A5, six diatonic steps
above B4.

#### Voice-level override

When `%%stemalignment` appears inside the tune body (after a `V:` switch), it
overrides the global value for that voice only. The directive must be placed
on its own line, immediately after the `V:<name>` line.

```abc
X:3
T:Per-voice stem alignment
M:4/4
L:1/4
K:C treble
V:1
%%stemalignment -4
GABc | cBAG |
V:2
%%stemalignment 4
GABc | cBAG |
```

Voice 1 uses `%%stemalignment -4` (stems down to E4, the bottom staff line);
voice 2 uses `%%stemalignment 4` (stems up to F5, the top staff line). Setting
a voice-level value does not change the global `%%stemalignment`; other voices
continue to use the global value (or their own voice-level override, if any).

#### Beamed groups

`%%stemalignment` applies to beamed notes as well as individual notes. Because
all stem tips in a beamed group are set to the same fixed position, the beam
will be rendered horizontally.

```abc
X:4
T:Beamed notes with stemalignment
M:4/4
L:1/8
%%stemalignment -6
K:C treble
cdef gabc' | c'bag fedc |
```

### Behaviour when the note head reaches or crosses the target

If a note head is already at or below (for stem-down) or at or above (for
stem-up) the alignment target, the stem is drawn with a minimum length of one
diatonic step (3 pt) beyond the note head. This prevents zero-length or
inverted stems while still indicating that the alignment constraint could not
be satisfied for that note.

```abc
X:5
T:Notes crossing the alignment target
M:2/4
L:1/8
%%stemalignment -6
K:C treble
cBAG | FEDC | B,2 z2 |
```

Notes from c5 down to D4 align at C4. At C4 itself the stem is drawn with
the minimum stub. B3 (below the target) also receives a minimum stub.

### Interaction with other directives

- **`%%stemdir` / `V: up` / `V: down`**: `%%stemalignment` overrides the stem
  *length* calculation but operates after the stem *direction* has been set.
  When used alongside explicit direction overrides, the alignment target is
  applied in the forced direction.
- **Stemless notes** (`!stemless!` decoration or `%%stemless`): Stemless notes
  are unaffected; `%%stemalignment` is ignored for them.
- **Grace notes**: Grace-note stems are not affected by `%%stemalignment`.

### Resetting

To revert to default stem-length behaviour within a tune, set the value back
to zero:

```abc
%%stemalignment 0
```

A voice-level reset of `%%stemalignment 0` restores that voice to the global
setting.
