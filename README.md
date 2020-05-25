# Homework 06: processor components

*These exercises are for a Homework 06 that is due
next Wednesday before lecture.*

You should already have downloaded a copy of
[LogiSim](http://www.cburch.com/logisim/) which allows you to draw
digital circuit diagrams and test their circuit's logic and
behavior. In this folder I have several LogiSim files 
that are "starter circuits," ones that you'll edit or 
mimic to complete the exercises. There is not starter circuit
for Exercise 4.

• `alu1.circ`: this is a circuit you'll reference and play with
to help you complete Exercise 1.

• `alu2.circ`: this is a circuit you'll edit to complete the 
for work for Exercise 2.

• `2to4decoder.circ`: this is a circuit you'll complete for 
Exercise 3.

• `up-down-counter.circ`: this you'll want to copy and mimic to build
the finite state machine that behaves according to Exercise 5. I'll
be showing how this circuit works in Friday's and Monday's lectures.

The folder also contains several other circuits that we built in
lecture. These all are related to the work in exercises 1 and 2, and
are the following:

• `4bit-full-add.circ`: This is a full adder that will add two
4-bit integers according to two's complement encoding. So it
computes the operation `z := x + y`.

• `flipper.circ`: This circuit either outputs its 4 bits of
input, or complements them, depending on whether the `flip?`
input bit is 0 or 1. So, when this bit is set to 1, it computes
the operation `z := !x` Here, we are using C's bit-wise complement
notation `!`. It uses a 2:1 MUX to select whether it performs
`z := x` or performs `z := !x`.

• `4bit-subtract.circ`: This is a full adder that will compute
the difference of two 4-bit integers according to two's complement
encoding. It relies on the full adder and on complementation,
and it also sets the `carry in` of the full adder. It computes
the operation `z := x - y`. In class, we saw that, in two's complement,
the negation of y is just the complement of y's bits, then incremented.
And so we have that `x - y` is just `x + (!y + 1)`. This is the key to
the subtraction circuit's design

• `4bit-add-or-subtract.circ`: This is a combination of the addition
and subtraction circuits. It uses a `subtract?` input to set the 
`carry in` and to control a "flipper" which complements the bits of y.
To add, we set both to 0. To subtract, we set both to 1.


---

Exercise 1: control an arithmetic unit
------------------------------------------

This exercise asks you to edit a file called `alu1.md` by working
with the circuit `alu1.circ`. Rather than submitting a circuit for
the exercise, you'll instead be submitting this `alu1.md` text file.

Take a look at the circuit built in `alu1.circ`. It is a complete
circuit that is made up of 4 components:

* a 4-bit full adder that takes two inputs `x` and `y` and produces
an output `z`

* a flipper that allows you to invert each of the 4 bits of `x` before 
they're fed into the adder. This is controlled by the line `flip x?`.

* a component that allows you to "zero out" the 4 bits of `y`. This is
controlled by the line `zero y?`

* a flipper that takes those `y` bits (either preserved or zeroed out)
and inverts them, or keeps them the same. These are fed into the adder.
This is controlled by the line `flip y?`. 

* an `add 1` line that is 0 or 1, and is fed into the `carry in` of the
adder.

Your goal in this exercise is to figure out the effects of controlling
these four lines in order to perform different calculations on `x` 
and `y` with the adder. For example, if `flip x?` is 0, `zero y?` is 1,
and `flip y?` is `, and `add 1?` is 0, then the circuit will add `x` to
`1111`. Since "all ones" is the two's complement encoding of -1, then,
in that case, the circuit behaves like:

    z := x - 1

There are 32 different combinations of these five control lines. So we
just pointed out that

     flip x? | zero y? | flip y? | add 1? || circuit
       (fx)  |   (zx)  |   (fy)  |  (a1)  || behavior
    ---------+---------+---------+--------++------------
         0   |     1   |     1   |    0   || z := x-1

Our goal in this and the next exercise is to build an **Arithmetic Unit**
that can be "instructed" to compute one of 8 different functions on the
inputs `x` and `y`. One of these is the function that computes `x - 1`.
We just remarked that setting the control lines `fx:zx:fy:a1` to the
pattern `0:1:1:0` makes the `alu1.circ` behave like that function.

This and the 7 other functions are listed as a table in the file `alu1.md`.
Figure out how the four control lines should be set so that the circuit
instead computes each of those functions. I've already filled out the table
row for the `z := x-1` function. Fill out the other 7 rows. You can ignore
the "instruction" column at the far left, for now. We'll be using this in
Exercise 2.

Make sure you test the circuit's behavior on different values of `x` and `y`
for a fixed setting of the control lines. Make sure the circuit computes the
correct output for each function specified, when those control lines are
set in that way.

Exercise 2. instructing the arithmetic unit
-------------------------------------------

**Note:** This exercise asks you to edit a circuit called `alu2.circ`
using the info you wrote up in `alu1.md` as your guide.

Now that you've figured out how to get the circuit to behave in these 8
different ways, let's now build a circuit that uses only three bits to
control the arithmetic unit, rather than those five bits. This makes 
sense: since we are only interested in having the circuit perform 8
different functions, we only need 3 bits to "select" which of the 8
functions we want the arithmetic unit to compute on `x` and `y`.

Open up the circuit `alu2.circ`. It contains `alu1.circ` as a subcircuit
named `ex1`. The `main` circuit has three inputs: a 4-bit input `x`,
a 4-bit input `y`, and an *instruction code* input as the three bits
`i2:i1:i0`. Our goal is to *decode* those three instruction bits
and use them to determine the arithmetic unit's controls `fx:zx:fy:a1`.

Here is a summary of the instruction codes, and the function each 
code indicates:

     instruction || circuit 
      (i2:i1:i0) || behavior
    -------------++------------
        0  0  0  || z := x+y
        0  0  1  || z := x
        0  1  0  || z := -x
        0  1  1  || z := x-y
        1  0  0  || z := x+1
        1  0  1  || z := x-1
        1  1  0  || z := -x-y-1
        1  1  1  || z := x+y+1

Your job is to devise the logic for four subcircuits:

* `fx(i2,i1,i0)` : what `flip x?` should be set to 
for that instruction

* `zy(i2,i1,i0)` : what `zero y?` should be set to 
for that instruction

* `fy(i2,i1,i0)` : what `flip y?` should be set to
for that instruction

* `a1(i2,i1,i0)` : what `add 1` should be set to
for that instruction

Using only `AND`, `OR`, and `NOT` gates, build the
subcircuits for the logic of each of these arithmetic 
unit control lines, based on the input lines `i2:i1:i0`.
The subcircuits should be in SOP form.

To help you think about this, what we're actually 
doing is treating the first 7 columns as a truth
table for a circuit that takes the input lines `i2:i1:i0`
and produces the output lines `fx:zx:fy:a1`. You build
a circuit for each of these four output columns as
you've specified them in this `alu1.md` truth table.

---

Exercise 3: 2:4 decoder
-----------------------

A *decoder* circuit takes *n* inputs and produces *2^n* outputs. For
example, there are 1:2, 2:4, 3:8, etc decoder circuits. A decoder
takes an *n*-bit binary code as input. It's *2^n* output lines are
labelled with each possible *n*-bit binary code.  When fed an input
code, exactly one of those outputs is 1---namely, the one with that
code---and all the other outputs are 0.

For example, a 3:8 decoder would have outputs labelled `o000`, `o001`,
`o010`, `o011`, `o100`, `o101`, `o110`, `o111`. It would have the 3
input lines `i2`:`i1`:`i0`.  If the inputs are set as `0`:`1`:`1`,
then `o011` will output 1. The rest output 0.  If instead the inputs
are set as `1`:`0`:`0` then `o100` outputs 1.

Build a circuit using `2to4decoder.circ` for the 2:4 decoder using only
AND, OR, and NOT gates in SOP form. You'll need to develop a boolean
expression for each of the 4 outputs. *Hint:* you won't need any `OR`
gates in this circuit, if you apply the truth table method outlined
above.  Each sum of products will just be a single product term.

---

Exercise 4. comparison
----------------------

Consider the comparison of two integers *x* and *y* encoded in binary
with 3 bits `x2`:`x1`:`x0` and `y1`:`y1`:`y0`. Build a circuit
that outputs a 1 when *x* is less than *y*, and 0 otherwise. You probably
don't want to write out a full truth table for this circuit. Instead,
think about how, when reading the bits of each input *simultaneously*,
from left to right, you can identify when one value is smaller than the
other.

Name the circuit file `less.circ`.

If you get stuck figuring out the logic, instead try to build the
circuit for 2-bit inputs and see what's at play. Or even just 1-bit
inputs. What is the condition on the bits that make the one number
less than the other?

---

Exercise 5. next state logic
----------------------------

**Note:** *You may want to wait until after Friday's and Monday's lectures to complete 
this exercise.*

Build a sequential circuit named `3saturate.circ` that has 3-bits of
state representing the integers from **0** to **7**. When its external
"count down" input is set to `0`, the circuit should count from its
current state up to **7**, with one incrementation per clock
tick. Once at **7**, it should stay at **7**.  This is its "saturated"
count up state (i.e.  the state bits of `111`).

When instead its external `down?` input is set to `1`, the circuit 
should count down from its state to **0**. Once at **0**, it should
stay at **0**. This is its "unsaturated" count state. (i.e. the
state bits of `000`).

The circuit should mimic the 2-bit counter circuit we'll describe in
lecture, included as the circuit file `up-down-counter.circ`. However,
it should instead have three registers to keep track of three bits of
state. And then the 7-segment LED display should display the digits
for **0** thru **7**. This means you'll build a 4-input 3-output
combinatorial circuit for the next state logic, and you'll build a
3-input 7-output combinatorial circuit for the display logic.


