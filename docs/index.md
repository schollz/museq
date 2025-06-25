# museq 

**museq** is a simple, text-based interface for creating and playing music. It is designed to be easy to use and understand, even for those with no prior experience in music theory or programming.


## quickstart {#quickstart}

First install **museq**:

```bash
curl https://museq.schollz.com/install.sh | bash
```

Now create a simple file called `first.tli`:

```bash
echo "c4 d4 e4 f4" > first.tli
```

Then run the program:

```
museq first.tli 
```

This program will play the notes `c4`, `d4`, `e4`, and `f4` in sequence using the first available MIDI device. 


You can also specify a different MIDI device. Find MIDI devices by running

```bash
$ museq -midi
Available MIDI devices:
- Midi Through:Midi Through Port-0 14:0
- Virtual Raw MIDI 1-0:VirMIDI 1-0 20:0
```

Then you can add a MIDI device by editing the `first.tli` file and adding the `midi` command at the top with any part of the name (case insensitive). If you want to use a particular channel, you can add that too. For example, if you want to use the first virtual MIDI device, you can do this:

<pre class="shiny box">
midi virtual
channel 1
c4 d4 e4 f
</pre>



## tli {#tli}

**tli** is the core of museq.

**tli** means "text-limited interface". It is the syntax used to enter commands, or add collections of commands (called a "pattern"), or collections of patterns (called a "chain"). The commands are often notes, but they can also be modifiers that augment the way that a note is played.

**tli** also means "too little information". It is a style of syntax developed for producing oblique rhythms without music theory. **tli** at its core is a single line of letters or numbers separated by spaces. The tracker allocates time to each line, and subdivides the time equally among each entity on the line.


Lets start with some simple examples.

<h3 class="h2under">Example 1 (quarter notes)</h3>
<p class="shiny">c d e f</p>

You can type this out into the norns and then press *ctrl*+*s* to parse it/save it. Then to play it just do *ctrl*+*p*. In this example there are four notes so each is given 1/4 of the time allotted to the line. If the line is given one measure, then each note will be a quarter note.

Notice that the notes are supplied without octave information. In this case *zxcvbn* will try to guess the octave (using the last known octave or defaulting to octave 4).


<h3 class="h2under">Example 2 (triplets)</h3>
<p class="shiny">c4 e g c4 e g c4 e g c4 e g</p>

In this example there are twelve notes so each is given 1/12 of the time allocated to the line. If the line is given one measure, then this will sound as four triplets. In this example, the octave is specified to the "C" note. The octave is simply supplied as a number after the note.


<h3 class="h2under">Example 3 (rests)</h3>

Rests are important, as they also are an entity given time. If you want a rest between notes you put a *.*. In the example here there are quarter notes played on the first and the fourth beat.

<p class="shiny">c4 . . g4</p>


If instead you want the two notes to be eighth notes, you have to add in more rests to make the line have eight entities.

<p class="shiny">c4 . . . . . . . . g4</p>

Now the two notes play on the first and last eighth note of the series.



<h2 class="h2under">Example 4 (ties)</h2>

Ties are equally important as rests. If you use a tie, you can length the preceding note. So in Example 3, if we want a half note, followed by a rest and then a quarter-note you can use a tie signified by *-*. For example:

<p class="shiny">c4 - . g4</p>

Ties are special too, because they can continue onto the next line. If you want to play out a note for two measures, then you can simply use two lines to express it:


<pre class="shiny">c4
 -
</pre>



<h2 class="h2under" id="subdivisions">Example 5 (subdivisions)</h2>

Parentheses can be used to define subdivisions which can save space instead of entering many rests. Each enclosed set of parentheses is considered a single entity for the purposes of dividing by the number of entities per line. For example, consider the following:

<p class="shiny">(c d) e</p>

In this example, there are three notes, but two of the notes (`c` and `d`) are enclosed in a parentheses so they are considered a single entity. This means there are only two entities on that line (`(c d)` and `e`), and each is given half of the pulses (48 pulses). The parenthetical entity is then compiled with the remaining pulses, so that the remaining pulses are redivided, so the 48 pulses is split into 24 pulses for both the `c` and `d` notes.

The above incantation can be written using a tie:

<p class="shiny">c d e -</p>

In this case it might be simpler to use ties. But as you add more subdivisions, things become more complicated. For example:


<p class="shiny">((c d e f) g) a</p>

This is the same as:

<p class="shiny">c d e f  g - - - a - - - - - - </p>

The latter requires a lot more ties to get the same invocation.

<h2 class="h2under">Allocating time in pulses</h2>

In *zxcvbn*, time is discrete and allocated in **pulses**. The number of pulses per quarter-note is immutable - it is defined as **24 pulses per quarter note** (24 ppqn).

Everything in *zxcvbn* is defined by pulses. The TLI syntax lets you split time, but the amount of time to split is given by the definition of the number of pulses per line. This is under your control.

You can define the number of pulses anywhere. If you want each line to be "one measure", with 4 quarter notes per measure, then you can define the number of pulses to be 96 (24 * 4) using the [pulse](#pulse) ([*p*](#pulse)) command:

<p class="shiny">p96</p>

You can also change the number of pulses **per line**. So if you want the number of pulses to change from 4 beats per line to 3 beats per line, you can do that:

<pre class="shiny">c4 d4 e4 f4 p96
 c4 d4 e4 p72
</pre>

The "shortcodes" like the [pulse](#pulse) command ([*p*](#pulse)) are ignored for the purposes of counting subdivisions - only notes, rests, ties or parentheses-enclosed sections are counted. So in this case the first line has four notes with 96 pulses and the second line has three notes with 72 pulses, so this essentially is a 7/4 time signature.

There is nothing stopping you from defining prime numbers of pulses to get oblique rhythms.


<h2 class="h2under">Imperfections in division</h2>

Since the number of pulses is discrete, there can be times where the number of pulses will not evenly divide between notes. In these cases, an euclidean system is used to allocate the notes across the pulses. 

For example, if you define only 8 pulses, but have 3 notes, 
then the pulses will not evenly divide.

<p class="shiny">c4 d4 e4 p8</p>

In this case, an euclidean generator is used to allocate three items across eight slots. The result would look like this in TLI syntax:

<p class="shiny">c4 . . d4 . . e4 . p8</p>

There is a special [offset](#offset) command ([*o*](#offset)) which can be used to offset the result. So if you include *o1* you can offset by 1. For example, this line:

<p class="shiny">c4 d4 e4 p8 o1</p>


Would convert into this:

<p class="shiny">. c4 . . d4 . . e4  p8</p>



<h2 class="h2under">Traditional-music pulses</h2>

In traditional music we mostly define note lengths as power-of-2 fractions of a measure - with a measure being four notes. There is shorthand for this in the pulse syntax. You can write `m`, `h`, `q`, `s`, `e` for measures, half-notes, quarter-notes, sixteenth-notes respectively. The measure is defined as 96 pulses and everything is a division from that.

You can also use mathematical expressions with pulses. So if you want a line to be one quarter-note less than two measures you can write:

<p class="shiny">c4 d4 e4 f4 p2*m-q</p>

The `p2*m-q` is evaluated as `2*96-24` which is `168` pulses. Its important not to put any spaces when using this syntax.

## Installation {#installation}


# Contact {#contact}

Send feedback to: `example@domain.com`



```python
-# pattern_a*2 pattern_b

# pattern_a
c 
e 
g
d

# pattern_b
f4




```

> **_NOTE:_**  The note content.


<div class="box">

asdfjlk

</div>
