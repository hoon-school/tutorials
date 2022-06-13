#   Urbit for Hackathons

Competitive programming requires developers to move quickly, be ready to employ common code patterns, and troubleshoot 
You have to move quickly.

For the competitive programmers, the Urbit platform offers you many helpful features for free:

- identity & associated state, cryptographically secured
- peer-to-peer end-to-end-encrypted apps
- persistent database
- software distribution without corporate oversight
- interfacing with traditional Web clients

If you have programmed with a scripting language like Lua inside of a modded game, for instance, you will be familiar with how the game serves as a platform providing correct primitives which speed your development.  Urbit does this for the operating system layer, serving as a platform for personal data management, social media, user apps, and blockchain apps.

However, Urbit development is couched in terms which are unfamiliar to most programmers unless they are versed in something like APL.  The Urbit programming language, Hoon, composes statements together in a way that behaves like functional programming but reads a bit more like imperative programming.

As you put the time into it, I expect that you will find Urbit to be refreshingly different from other platforms, while affording rapid application development and deployment.

This workshop is divided into three sections:

1. **The Hoon language**, which provides a minimum viable introduction to working with the language.
2. **The userspace app framework**, called Gall, which requires a standard application interface and in return gives you access to a rich operating environment, state management, and peer-to-peer software distribution.
3. **Front-end app development**, in which React-based browser sessions use the Urbit ship for identity, communication, and database while providing a conventional client-based user experience.

An Urbit instance is called a ship.  Regular ships are connected to the network, but are inconvenient for development.  Normally we compose code using fakeships, which don't have cryptographic authentication or network presence.  Go ahead and use the Urbit executable to spin up a new fakeship called `zod`.

```sh
$ urbit -F zod
```

Back it up in the same directory so you have a fresh copy in case you break something.  This is faster than booting from scratch every time.

```sh
$ cp -r zod zod-backup
```

This video will skip over a lot of conventions and conveniences in favor of showcasing how to work with Urbit very quickly.  Ask and observe to find smooth ways to work with Urbit in general.


##  The Hoon Language

While Urbit runs on a virtual machine using a language called Nock, Nock is too minimalist for humans to effectively write, rather like bytecode.  Instead, we compose in Hoon, which compiles to Nock.  Here we'll speedrun through some essentials of the language so you get the flavor of how it works, and so we can showcase more of what's possible on working on the Urbit platform in a short period of time.

### Syntax

Everything in Hoon is a noun, which means either a bare value (an _atom_ or unsigned integer) or a pair of values (a _cell_).  Metadata can attach to particular values to indicate to the system how a value should be interpreted; compare how the letter `a`, ASCII value `97`, and ASCII hexadecimal value `0x61` are all the same thing, just with a different interpretation applied to the value.

Hoon programs are built like a mathematical expression along a backbone of symbols called _runes_.  Runes are relational operators that describe how different parts of an expression relate to each other.  Since code is data and data is code, powerful Lisp-like introspection is possible.

Like a `+` or `=` in arithmetic, each rune expects a certain number of child expressions, until the entire program can be modeled as a tree.

Hoon allows the developer a lot of freedom in deciding how to display and format code, and we'll only scratch the surface here.

### Semantics

```hoon
=trial |%
+$  input  ?(@ud tape)
++  try
  |=  =input
  ~&  >  trial
  ?@  input  ~&  "atom"  !!
  ~&  "tape"
  !!
--
```

#### Types

##### Atoms
##### Cells

#### Lists

#### Text


Now that we have some fundamentals under our belt, let's implement a few common patterns that can come in handy for competitive programming.  Obviously I don't know what you'll need, but we can't go wrong with showing you some patterns for sorting, searching, 
Many of these you can treat as a black-box algorithm.

The Hoon standard library at the current time is heavily biased towards code parsing and building.  This means that some operations, like linear algebra or image manipulation, are relatively undersupported and you can expect to have to implement more primitive operations in those cases.  In addition, given the nature of competitive programming, I will prefer to point you at black-box solutions or quick-and-dirty solutions rather than comprehensive-but-involved solutions.  We're also going to steer away from slightly more exotic data structures like treaps and ordered maps in favor of straightforward `tree`/`list`/`map` implementations.

### Control Structures

### Cores

**You cannot grasp the nature of Hoon until you grasp cores.**

### Doors

A gate serves as the Hoon analogue of a function, subject to the requirement that all functions are _fruitful_:  they return something.  Gates are cores with a default `$` buc arm.  The door generalizes the concept of a gate, permitting many arms and an overall sample.  Hoon developers frequently employ gate-building arms as templates to build type-appropriate cores.  Since Hoon is a statically-typed language, this is an important feature to make it actually usable for flexible data structure implementations.

This accelerated tutorial will skip over producing doors, although a userspace Gall agent is a door.  We will, however, see how to use two of the most common and useful doors:  maps and sets.

Generally speaking, a door has a gate-building arm which is called using [`%~` censig](https://urbit.org/docs/hoon/reference/rune/cen#-censig), in its irregular form `~()`.

#### Maps

  
In general terms, a map is a pattern from a key to a value.  You can think of a dictionary, or an index, or a data table.  Essentially it scans for a particular key, then returns the data associated with that key (which may be any noun).

| Key         | Value      |
| ----------- | ---------- |
| 'Mazda'     | 'RX-8'     |
| 'Dodge'     | 'Viper'    |
| 'Ford'      | 'Mustang'  |
| 'Chevrolet' | 'Chevelle' |
| 'Porsche'   | 'Boxster'  |
| 'Bugatti'   | 'Type 22'  |

While `map` is the mold or type of the value, the door which affords `map`-related functionality is named `++by`.  (This felicitously affords us a way to read `map` operations in an English-friendly phrasing.)

In Urbit, all values are static and never change.  (This is why we “overwrite” or replace the values in a limb to change it with `%=` centis.)  This means that when we build a `map`, we often rather awkwardly replace it with its modified value explicitly.

We'll build a color `map`, from a `@tas` of a [color's name](https://en.wikipedia.org/wiki/List_of_Crayola_crayon_colors) to its HTML hexadecimal representation as a `@ux` hex value.

We can produce a `map` from a `list` of key-value cells using the [`++malt`](https://urbit.org/docs/hoon/reference/stdlib/2l#malt) gate:

```hoon
> =colors (malt `(list [@tas @ux])`~[[%red 0xed.0a3f] [%yellow 0xfb.e870] [%green 0x1.a638] [%blue 0x66ff]])
```

To insert one key-value pair at a time, we use `put`.  We need to either pin it into the subject for Dojo or modify it with `=/` tisfas.

```hoon
> =colors (~(put by colors) [%orange 0xff.8833])
> =colors (~(put by colors) [%violet 0x83.59a3])
> =colors (~(put by colors) [%black 0x0])
```

Note the pattern here:  there is a `++put` arm of `++by` which builds a gate to modify `colors` by inserting a value.

What happens if we try to add something that doesn't match the type?

```hoon
> =colors (~(put by colors) [%cerulean '#02A4D3'])
```

We'll see a `mull-grow`, a `mull-nice`, and a `nest-fail`.  Essentially these are all flavors of mold-matching errors.

(As an aside, `++put:by` is also how you'd replace a key's value.)

The point of a `map` is to make it easy to retrieve data values given their appropriate key.  Use `++get:by`:

```hoon
> (~(get by colors) %orange)
[~ 0xff.8833]
```

What is that cell?  Wasn't the value stored as `0xff.8833`?  Well, one fundamental problem that a `map` needs to solve is to allow us to distinguish an _empty_ result (or failure to locate a value) from a _zero_ result (or an answer that's actually zero).  To this end, the `unit` was introduced, a type union of a `~` (for no result) and `[~ item]` (for when a result exists).

`unit`s are common enough that they have their own syntax and set of operational functions.  Most of the time you'll simply use [`?~` wutsig](https://urbit.org/docs/hoon/reference/rune/wut#-wutsig) to check whether a value was returned:

```hoon
> (~(get by colors) %brown)
~

> =/  answer  (~(get by colors) %brown)
  ?~  answer
    %.n
  %.y
```

It's often appropriate to crash on a null result:

```hoon
> =/  answer  (~(get by colors) %brown)
  ?~  answer  !!
  %.y
```

([`++got:by`](https://urbit.org/docs/hoon/reference/stdlib/2i#gotby) returns the value without the `unit` wrapper, but crashes on failure to locate.  I recommend just using `++get` and extracting the tail of the resulting cell after confirming it isn't null with `?~` wutsig.  See also [`++gut:by`](https://urbit.org/docs/hoon/reference/stdlib/2i#gutby) which allows a default in case of failure to locate.)

You can check whether a key is present using `++has:by`:

```hoon
> (~(has by colors) %teal)
%.n
> (~(has by colors) %green)
%.y
```

You can get the `set` of all keys with `++key:by`:

```hoon
> ~(key by colors)
{%black %red %blue %violet %green %yellow %orange}
```

You can apply a gate to each value using `++run:by`.  For instance, these gates will break the color hexadecimal value into red, green, and blue components:

```hoon
> =red |=(a=@ux ^-(@ux (cut 2 [4 2] a)))
> =green |=(a=@ux ^-(@ux (cut 2 [2 2] a)))
> =blue |=(a=@ux ^-(@ux (cut 2 [0 2] a)))
> (~(run by colors) blue)
{ [p=%black q=0x0]  
  [p=%red q=0x3f]  
  [p=%blue q=0xff]  
  [p=%violet q=0xa3]  
  [p=%green q=0x38]  
  [p=%yellow q=0x70]  
  [p=%orange q=0x33]  
}
```

#### Sets

A `set` is like a `list` except that each entry can only occur once and it doesn't end in a `~` sig null value.  `set`s don't have an order, so they're basically a bag of unique values.  As with a `map`, a `set` is associated with a particular type, such as `(set @ud)` for a collection of decimal values.

`set` operations are provided by the `++in` core.  Most names are similar to `map`/`++by` operations when applicable.

[`++silt`](https://urbit.org/docs/hoon/reference/stdlib/2l#silt) produces a `set` from a `list`:

```hoon
> =primes (silt ~[2 3 5 7 11 13])
```

`++put:in` adds a value to a `set` (and null-ops when the value is already present):

```hoon
> =primes (~(put in primes) 17)
> =primes (~(put in primes) 13)
```

`++del:in` removes a value from a `set`:

```hoon
> =primes (~(put in primes) 18)
> =primes (~(del in primes) 18)
```

`++has:in` checks for existence:

```hoon
> (~(has in primes) 15)
%.n
> (~(has in primes) 17)
%.y
```

`++tap:in` yields a `list` of the values:

```hoon
> ~(tap in primes)  
~[3 2 7 5 11 13 17]  
> (sort ~(tap in primes) lth)  
~[2 3 5 7 11 13 17]
```

`++run:in` applies a function across all values:

```hoon
> (~(run in primes) dec)  
{10 6 12 1 2 16 4}
```


##  Examples

Let's take a quick look at implementations for some common tasks in Hoon.  I assume that you want to use library tools whenever possible, but you can delve into the source code itself if you want to know more.

### Sorting

- Given a `list` of values, sort the values according to a given rule and produce a `list`.

The standard `++sort` gate implements a quicksort.  You need to provide a comparison function as a gate as well.

```hoon
> =/  my-list  `(list @ud)`~[10 9 8 1 2 3 7 6 4 5]
  (sort my-list gth)
~[10 9 8 7 6 5 4 3 2 1]

> =/  my-list  `(list @ud)`~[10 9 8 1 2 3 7 6 4 5]
  (sort my-list lth)
~[1 2 3 4 5 6 7 8 9 10]
```

If you are sorting something more complicated than atoms, you'll need a comparison gate that handles pairwise values and returns truthness.

E.g. given a data structure like `[]`, a cell which we wish to sort on the tail, we could sort like this:

```hoon
> =/  my-list  `(list [@ud @tas])`~[[1 %a] [2 %c] [3 %b] [4 %d]]
  (sort my-list |=([a=[@ud @tas] b=[@ud @tas]] (lth +.a +.b)))
~[[1 %a] [3 %b] [2 %c] [4 %d]]
```

- Given a `set` of values, sort the values according to a given rule and produce a `list`.

If something isn't a `list`, the easiest way to sort it is to convert it to a `list` first and then sort that.

```hoon
> =/  my-set  (silt ~[1 10 10 1 2 3 3 2 4 9 8 5 7 6 8 9])
  =/  my-list  ~(tap in my-set)
  (sort my-list lth)
~[1 2 3 4 5 6 7 8 9 10]
```

### Bitwise Operations

Bitwise operations are necessary when you are closely packing data into binary formats or otherwise dealing with binary data.  Basically there are three scenarios:

1. Using packed binary data, like [NaN-boxing](https://anniecherkaev.com/the-secret-life-of-nan).  Urbit is reasonably good at this.
2. Working with MIME-type data.  Urbit has full standard library support for these.
3. Directly processing particular kinds of data streams, like audio or video data.  You'll be building these from scratch or interfacing with an external service.

#### Binary Operations

If you are working with packed binary data, you'll typically print the atom data with a `@ux` unsigned hexadecimal aura.

We'll use `'Wild Hearts Can\'t Be Broken'` as our source atom.  As `@ux` the ASCII components are clearly visible.

```hoon
> `@ux`'Wild Hearts Can\'t Be Broken'  
0x6e.656b.6f72.4220.6542.2074.276e.6143.2073.7472.6165.4820.646c.6957  
```

Here are a few ways to slice and dice binary atom data.

[`++rip`](https://urbit.org/docs/hoon/reference/stdlib/2c#rip) disassembles an atom `b` into `2^a`-sized chunks as a `list`.

```hoon
> =/  text  'Wild Hearts Can\'t Be Broken'  
 (rip 3 text)  
~[87 105 108 100 32 72 101 97 114 116 115 32 67 97 110 39 116 32 66 101 32 66 114 111 107 101 110]  

> =/  text  'Wild Hearts Can\'t Be Broken'  
 `(list @ux)`(rip 3 text)  
~[0x57 0x69 0x6c 0x64 0x20 0x48 0x65 0x61 0x72 0x74 0x73 0x20 0x43 0x61 0x6e 0x27 0x74 0x20 0x42 0x65 0x20 0x42 0x72 0x6f 0x6b 0x65 0x6e]

> =/  text  'Wild Hearts Can\'t Be Broken'  
 `(list @ux)`(rip 6 text)  
~[0x6165.4820.646c.6957 0x276e.6143.2073.7472 0x6f72.4220.6542.2074 0x6e.656b]
```

[`++rap`](https://urbit.org/docs/hoon/reference/stdlib/2c#rap) is the opposite operation, producing an atom from a list of atoms `b` with a block size `2^a`.

```hoon
> `(list @ux)`(rip 3 'Mars')
~[0x4d 0x61 0x72 0x73]

> `@t`(rap 3 ~[0x4d 0x61 0x72 0x73])
'Mars'

> `@ux`(rap 3 ~[0x4d 0x61 0x72 0x73])
0x7372.614d

> `@ux`(rap 6 ~[0x4d 0x61 0x72 0x73])
0x73.0000.0000.0000.0072.0000.0000.0000.0061.0000.0000.0000.004d
```

[`++cut`](https://urbit.org/docs/hoon/reference/stdlib/2c#cut) slices `2^a`-sized chunks from `b` to `c` out of atom `d`.

```hoon
> =/  text  'Wild Hearts Can\'t Be Broken'
 `@ux`(cut 3 [0 4] text)
0x646c.6957

> =/  text  'Wild Hearts Can\'t Be Broken'
 `@t`(cut 3 [0 4] text)
'Wild'
```

(There are many more of these types of operations available as well.)

Standard bitwise logical operations are available:

- `OR` is [`++con`](https://urbit.org/docs/hoon/reference/stdlib/2d#con):

    ```hoon
    > `@ub`(con 0b10.0001.0110 0b11.1101.1011)
    0b11.1101.1111
    ```

- `AND` is [`++dis`](https://urbit.org/docs/hoon/reference/stdlib/2d#dis):

    ```hoon
    > `@ub`(dis 0b10.0001.0110 0b11.1101.1011)
    0b10.0001.0010
    ```

- `XOR` is [`++mix`](https://urbit.org/docs/hoon/reference/stdlib/2d#mix):

    ```hoon
    > `@ub`534
    0b10.0001.0110

    > `@ub`987
    0b11.1101.1011

    > `@ub`(mix 534 987)
    0b1.1100.1101
    ```

- `NOT` is [`++not`](https://urbit.org/docs/hoon/reference/stdlib/2d#not); it requires a block size because leading zeroes are always stripped in Hoon (and thus implicitly based on block size):

    ```hoon
    > `@ub`(not 2 1 0b1011)
    0b100

    > `@ub`(not 3 1 0b1011)
    0b1111.0100
    
    > `@ub`(not 4 1 0b1011)
    0b1111.1111.1111.0100
    ```

#### MIME Data Operations

[MIME data types](https://en.wikipedia.org/wiki/MIME) allow HTTP communications to include rich content.  The `++html` core in the standard library provides quite a few operations for encoding and decoding MIME-typed values.

Data transmitted as bytes ([“octets”](https://en.wikipedia.org/wiki/Octet_%28computing%29)) are decoded using `++as-octs:mimes:html`.  This is transparent for ASCII text data:

```hoon
> (as-octs:mimes:html '<html><body>')
[p=12 q=19.334.824.924.412.244.887.090.784.316]

> `[@ @ux]`(as-octs:mimes:html '<html><body>')
[12 0x3e79.646f.623c.3e6c.6d74.683c]

> `[@ @t]`(as-octs:mimes:html '<html><body>')
[12 '<html><body>']
```

Perhaps surprisingly, many text conversion operations live here.  To parse a hexadecimal value as a string, for instance, use `++de:base16:mimes:html`:

```hoon
> (de:base16:mimes:html 'deadbeef')
[~ [p=4 q=3.735.928.559]]

> `(unit [@ @ux])`(de:base16:mimes:html 'deadbeef')
[~ [4 0xdead.beef]]
```

There are operations for [Base58](https://en.wikipedia.org/wiki/Binary-to-text_encoding#Base58) and XML/HTML as well.  We will discuss JSON operations subsequently.

Urbit furthermore has a notion of _jammed noun_, which is essentially a serialization (marshaling, pickling) of a noun into an atom.

### Primes and Factors

### Pragmatic Input/Output


While Hoon has a sophisticated text parsing library, the primitives are rather low-level and we won't assume that you want to implement a parser using them.

### Functional Operators

`++turn` etc.

### Floating-Point Operations


I picked the above set of examples after perusing the excellent book [Antti Laaksonen (2017) _Guide to Competitive Programming:  Learning and Improving Algorithms Through Contests_](https://link.springer.com/book/10.1007/978-3-319-72547-5) (which available for free as a PDF or EPUB).

### Debugging and Troubleshooting

Static typing with compile-time type checking turns out to be a secret strength of Hoon.

There are three basic things that tend to go wrong:

1. Syntax error, general (just typing things out wrong, for instance in a way Dojo would prevent)
2. Syntax error mismatching rune daughters (due to `ace`/`gap` or miscounting children)
3. Type issues (`need`/`have`, notoriously)

This last case can be handled with a couple of expedients:

- Frequent use of `^-` kethep/`^+` ketlus to make sure that types match at various points in the code.

    This has two benefits:  it verifies your expectations about what values are being passed around, and it means that mismatches raise an error more closely to the source of the error.

    (Since Hoon checks type at build time, this does not incur a computational cost when the program is running.)

- Use of assertions to enforce type constraints.  Assertions are a form of `?` wut rune which check the structure of a value.  Ultimately they all reduce back to `?:` wutcol, but are very useful in this sugar form:

    - [`?>` wutgar](https://urbit.org/docs/hoon/reference/rune/wut#-wutgar) is a positive assertion, that a condition _must_ be true.
    - [`?<` wutgal](https://urbit.org/docs/hoon/reference/rune/wut#-wutgal) is a negative assertion, that a condition _must_ be false.
    - [`?~` wutsig](https://urbit.org/docs/hoon/reference/rune/wut#-wutsig) is a branch on null.

    For instance, some operations require a `lest`, a `list` guaranteed to be non-null (that is, `^-  (list)  ~` is excluded).

    ```hoon
    > =/  list=(list @)  ~[1 2 3]
     i.list
    -find.i.list
    find-fork
    dojo: hoon expression failed
    ```

    `?~` wutsig solves the problem for this case:

    ```hoon
    > =/  list=(list @)  ~[1 2 3]
     ?~  list  !!
     i.list
    1
    ```
    
    In general, if you see an error like `find.fork`, it means that the type system is confused by your use of a too general of a type for a particular case.  Use the assertion runes to correct its assumption.

##  Back-End App Development

I will use agent and app interchangeably, although more properly an agent is an instantiation of an app on a particular ship.

### A Gall Agent's Structure

Gall defines a standard application interface, like if you were writing code to use Microsoft .NET or as a Chrome extension.  A Gall agent is a core which has ten standard arms to handle known kinds of events, such as initialization, subscription, and data request.

#### Minimal Working Example #1

This means that our minimal working example is a bit more complicated than a simple “Hello, World” program, but at this point you have seen many of the runes which will allow you to interpret this program.

This is the simplest possible Gall agent.  It has no state and does nothing.  Any messages sent to it will cause a crash.  But it lets us examine the structure quickly.

**/app/alfa.hoon**:

```hoon
|_  =bowl:gall
++  on-init   `..on-init
++  on-save   !>(~)
++  on-load   |=(vase `..on-init)
++  on-poke   |=(cage !!)
++  on-watch  |=(path !!)
++  on-leave  |=(path `..on-init)
++  on-peek   |=(path ~)
++  on-agent  |=([wire sign:agent:gall] !!)
++  on-arvo   |=([wire sign-arvo] !!)
++  on-fail   |=([term tang] `..on-init)
--
```

You should create a new desk for each agent and clear out the agents from the copy:

```hoon
> |merge %alfa our %base

> |mount %alfa
```

```sh
$ cd zod/alfa
$ echo "~[%alfa]" > desk.bill
```

and save the above file to `alfa/app/alfa.hoon`.

```hoon
> |commit %alfa

> |install our %alfa
```

Line by line:

```hoon
|_  =bowl:gall
...
--
```

A Gall agent is a particular kind of core called a _door_.  The main thing to know is that we can provide the `bowl`, which is a data structure containing the attested ship identity, the timestamp, entropy for randomness, and some other system information.

```hoon
++  on-init   `..on-init
```

The `++on-init` arm returns the whole agent (the parent of the current arm) prepended with a `~` sig null value.

```hoon
++  on-save   !>(~)
++  on-load   |=(vase `..on-init)
```

The `++on-save` and `++on-load` arms are used to pack the agent's state ahead of an upgrade, and then to unpack and possibly update the structure of the agent's state after the upgrade.

```hoon
++  on-poke   |=(cage !!)
++  on-peek   |=(path ~)
```

A poke is a one-off message, like a data status request.  Pokes initiate some kind of well-defined action by the agent, often either triggering an event or requesting a data return of some kind.  Pokes are useful as single-instance requests.

A peek represents a direct look into the agent state.  A peek (or “scry”) can't change anything in the state, but it's a quick way to directly get information.

```hoon
++  on-watch  |=(path !!)
++  on-leave  |=(path `..on-init)
```

Gall agents utilize a dataflow computing model, wherein changes in state are indicated to subscribers who then modify or react in turn to the flow of data arriving from this agent.  A subscription is a data-reactive standing request for changes.  For instance, one can watch a database agent for any changes to the database.  Whenever a change occurs, the agent notifies all subscribers, who then act as they should in the event of a message being received (e.g. from a particular ship).  Subscriptions are probably the most complicated part of writing agents, so we'll spend some time in a moment looking at them in more detail.

`++on-watch` registers a new subscriber.  `++on-leave` registers a dropped subscriber.

```hoon
++  on-agent  |=([wire sign:agent:gall] !!)
++  on-arvo   |=([wire sign-arvo] !!)
++  on-fail   |=([term tang] `..on-init)
--
```

These three arms deal with system events.  The `++on-agent` and `++on-fail` arms are called in certain circumstances (i.e. as an update to a subscription to another agent or cleanup after a `%poke` crash). We can leave them as boilerplate for now.

The `++on-arvo` arm handles information passed back from the Arvo operating function (or system kernel).  For instance, if we set a timer for an event, the wake-up call will be handled here.

#### Minimal Working Example #2

Let's upgrade to something that looks more like a complete Gall agent.

**/app/bravo.hoon**

```hoon
/+  default-agent, dbug
|%
+$  versioned-state
  $%  state-0
  ==
+$  state-0
  $:  [%0 values=(list @)]
  ==
+$  card  card:agent:gall
--
%-  agent:dbug
=|  state-0
=*  state  -
^-  agent:gall
|_  =bowl:gall
+*  this      .
    default   ~(. (default-agent this %|) bowl)
++  on-init
  ^-  (quip card _this)
  ~&  >  '%bravo initialized successfully'
  =.  state  [%0 *(list @)]
  `this
++  on-save   on-save:default
++  on-load   on-load:default
++  on-poke   on-poke:default
++  on-arvo   on-arvo:default
++  on-watch  on-watch:default
++  on-leave  on-leave:default
++  on-peek   on-peek:default
++  on-agent  on-agent:default
++  on-fail   on-fail:default
--
```

```hoon
> |merge %bravo our %base

> |mount %bravo
```

```sh
$ cd zod/bravo
$ echo "~[%bravo]" > desk.bill
```

and save the above file to `bravo/app/bravo.hoon`.

```hoon
> |commit %bravo

> |install our %bravo
```

This agent still does nothing, but now it does nothing in a more sophisticated way.  It also stores a list of values as internal state, although we can't change it yet.

Line-by-line again, only looking at the new parts:

```hoon
/+  default-agent, dbug
```

The first line imports two libraries:
- `/lib/default-agent` provides some standard arms we can use as placeholders for parts of the agent we don't want or need to create.
- `/lib/dbug` allows us to directly peek inside the agent and grab its state.

```hoon
|%
+$  versioned-state
  $%  state-0
  ==
::
+$  state-0
  $:  [%0 values=(list @)]
  ==
+$  card  card:agent:gall
--
```

We then create a `|%` barcen core which defines some molds or types we'll need later.  Most saliently, every real Gall agent has a versioned state which allows you to define upgrade paths for subsequent releases.  Here we call the particular version `+$state-0`; it contains only a list of atoms we can use for any purpose.  We also commonly alias the Gall message `card` for easier reference.

```hoon
%-  agent:dbug
=|  state-0
=*  state  -
^-  agent:gall
```

We wrap the whole core in the `%dbug` agent so we can peek at the state directly, then we give a face to the state as `state-0` and return something that fits the Gall agent mold.

```hoon
|_  =bowl:gall
+*  this      .
    default   ~(. (default-agent this %|) bowl)
::
++  on-init
  ^-  (quip card _this)
  ~&  >  '%bravo initialized successfully'
  =.  state  [%0 *(list @)]
  `this
++  on-save   on-save:default
++  on-load   on-load:default
++  on-poke   on-poke:default
++  on-arvo   on-arvo:default
++  on-watch  on-watch:default
++  on-leave  on-leave:default
++  on-peek   on-peek:default
++  on-agent  on-agent:default
++  on-fail   on-fail:default
--
```

Finally we have our main agent again, the rest being helpful boilerplate.  Most of the lines have been explicitly replaced with the `/lib/default` version of the arms, which is a completely satisfactory solution to pass and do nothing.

The two main part of interest now are the aliases we pin and the initialization.

1. We pin `this` to refer to the current agent subject, and `default` to refer to the library.  Commonly we may also refer to a helper core which will sit beside the agent and provide a cleaner way to process data.
2. We then initialize the state with the bunt, or empty value, of `(list @)` and return the unit of that state as before.  We also emit a side effect message indicating that the agent `++on-init` arm has successfully completed.  Notice that the return type of this and most arms is a `(quip card _this)`.  For the most part you can crib from existing code examples rather than needing to deeply parse the mold here.

If we run this agent, we can see it initialize successfully, but we can't interact with it yet.  The docket file warning has to do with registration for release and we can ignore it for now.

```hoon
> |commit %bravo
+ /~zod/bravo/3/app/bravo/hoon

> |install our %bravo
kiln: installing %bravo locally
gall: installing %bravo
>   '%bravo initialized successfully'
[%docket %no-docket-file-for %bravo]

> :bravo +dbug
>   ~
```

#### Pokes & Peeks

The next agent we'll take a look at will allow two ships to poke each other and peek at state.  Each agent will maintain a list of numbers, but allow other agents to append numbers to the list (or pop them off).  (We won't filter for agent permissions right now, which would be critical for a real-world application.)

We will start to use a helper core, an auxiliary core which will define data operations that are inconvenient to include in the main agent.  Think of it as a library we define after the main core to perform necessary operations.

We will introduce marks at this point as well.  Marks serve the same role as file types:  they mark a set of data as having a particular structure.  They are like molds, but they include transformation rules as well, like how to turn an HTML data blob into a plaintext data blob.  Pokes are constrained to match the data structure indicated by the appropriate mark, then handled first by type then content.

**/app/charlie.hoon**

```hoon
/-  *charlie
/+  default-agent, dbug
|%
+$  versioned-state
  $%  state-0
  ==
+$  state-0
  $:  [%0 values=(list @)]
  ==
+$  card  card:agent:gall
--
%-  agent:dbug
=|  state-0
=*  state  -
^-  agent:gall
|_  =bowl:gall
+*  this     .
    default  ~(. (default-agent this %|) bowl)
++  on-init   on-init:default
++  on-save   !>(state)
++  on-load
  |=  old=vase
  ^-  (quip card _this)
  `this(state !<(state-0 old))
++  on-poke
  |=  [=mark =vase]
  ^-  (quip card _this)
  ?>  ?=(%charlie-action mark)
  =/  act  !<(action vase)
  ?-    -.act
      %push
    ?:  =(our.bowl target.act)
      `this(values [value.act values])
    ?>  =(our.bowl src.bowl)
    :_  this
    [%pass /pokes %agent [target.act %charlie] %poke mark vase]~
  ::
      %pop
    ?:  =(our.bowl target.act)
      `this(values ?~(values ~ t.values))
    ?>  =(our.bowl src.bowl)
    :_  this
    [%pass /pokes %agent [target.act %charlie] %poke mark vase]~
  ==
::
++  on-peek
  |=  =path
  ^-  (unit (unit cage))
  ?+  path  (on-peek:default path)
    [%x %values ~]  ``noun+!>(values)
  ==
++  on-arvo   on-arvo:default
++  on-watch  on-watch:default
++  on-leave  on-leave:default
++  on-agent  on-agent:default
++  on-fail   on-fail:default
--
```

**`/sur/charlie.hoon`**

```hoon
|%
+$  action
  $%  [%push target=@p value=@]
      [%pop target=@p]
  ==
--
```

**`/mar/charlie/action.hoon`**

```hoon
/-  *charlie
|_  act=action
++  grow
  |%
  ++  noun  act
  --
++  grab
  |%
  ++  noun  action
  --
++  grad  %noun
--
```

Set up the desk as usual:

```hoon
> |merge %charlie our %base

> |mount %charlie
```

```sh
$ cd zod/charlie
$ echo "~[%charlie]" > desk.bill
```

and save the above files to `charlie/`.

```hoon
> |commit %charlie

> |install our %charlie
```

Once that's launching correctly, two fakeships on the same local machine running `%charlie` can poke each other.  (The next ship after ~zod is ~nec; start one of those up now as well.)

```hoon
::  Pokes pass through the action mark.
:charlie &charlie-action [%push ~zod 100]
:charlie &charlie-action [%push ~nec 50]
:charlie &charlie-action [%pop ~zod]
:charlie &charlie-action [%pop ~nec]

::  Peeks need the return value specified.
.^((list @) %gx /=charlie=/values/noun)
```

Pokes pass through the action mark, which validates that they match the correct structure.  Here we provide them very explicitly at the command line, but we'll be able to trigger them using Javascript from a client session.

Peeks are done using the Nock command `.^` dotket.  You have to specify the mold of the return type, the `%gx` care, and the path that the peek takes place on.  These are convenient but we'll prefer pokes for most one-off actions.

Because Urbit is a functional programming language with no side effects, to modify the state of an agent we have to replace the agent with appropriate changes made.  The Gall platform takes care of much of this for us, but we have to explicitly return a modified `this`, and sometimes getting the state back out can be a bit tricky until you get the hang of the type system's expectations.

#### Subscriptions

The basic unit of a subscription is the _path_.  An agent will typically define a number of subscription paths in its `++on-watch` arm, and other local or remote agents can subscribe to those paths.  The agent will then send out updates called `%fact`s on one or more of its paths, and _all_ subscribers to those paths will receive them.  An agent cannot send out updates to specific subscribers, it can only target a paths.  An agent can kick subscribers from its paths, and subscribers can unsubscribe from any paths.  Paths can be static or dynamic.

Agents talk to each other by means of `card`s, which are messages requesting or replying to other agents or parts of Arvo.  We'll introduce that mechanism for agents to talk to each other here as well.  For this agent, we will retain the peek/poke behavior from before and introduce a new subscriber behavior, wherein every follower is notified when the stack `values` changes.  (We don't have to track this manually, as Gall will do it for us.)  We'll build a separate `delta-follower` agent as well which will listen for subscription updates, but won't store any state itself.

**`/app/delta.hoon`**

```hoon
/-  *delta
/+  default-agent, dbug
|%
+$  versioned-state
  $%  state-0
  ==
+$  state-0
  $:  [%0 values=(list @)]
  ==
+$  card  card:agent:gall
--
%-  agent:dbug
=|  state-0
=*  state  -
^-  agent:gall
|_  =bowl:gall
+*  this     .
    default  ~(. (default-agent this %|) bowl)
++  on-init   on-init:default
++  on-save   !>(state)
++  on-load
  |=  old=vase
  ^-  (quip card _this)
  `this(state !<(state-0 old))
++  on-poke
  |=  [=mark =vase]
  ^-  (quip card _this)
  ?>  ?=(%delta-action mark)
  =/  act  !<(action vase)
  ?-    -.act
      %push
    ?:  =(our.bowl target.act)
      :_  this(values [value.act values])
      [%give %fact ~[/values] %delta-update !>(`update`act)]~
    ?>  =(our.bowl src.bowl)
    :_  this
    [%pass /pokes %agent [target.act %delta] %poke mark vase]~
  ::
      %pop
    ?:  =(our.bowl target.act)
      :_  this(values ?~(values ~ t.values))
      [%give %fact ~[/values] %delta-update !>(`update`act)]~
    ?>  =(our.bowl src.bowl)
    :_  this
    [%pass /pokes %agent [target.act %delta] %poke mark vase]~
  ==
::
++  on-peek
  |=  =path
  ^-  (unit (unit cage))
  ?+  path  (on-peek:default path)
    [%x %values ~]  ``noun+!>(values)
  ==
++  on-watch
  |=  =path
  ^-  (quip card _this)
  ?>  ?=([%values ~] path)
  :_  this
  [%give %fact ~ %delta-update !>(`update`[%init values])]~
++  on-arvo   on-arvo:default
++  on-leave  on-leave:default
++  on-agent  on-agent:default
++  on-fail   on-fail:default
--
```

**`/sur/delta.hoon`**

```hoon
|%
+$  action
  $%  [%push target=@p value=@]
      [%pop target=@p]
  ==
+$  update
  $%  [%init values=(list @)]
      action
  ==
--
```

**`/mar/delta/action.hoon`**

```hoon
/-  *delta
|_  act=action
++  grow
  |%
  ++  noun  act
  --
++  grab
  |%
  ++  noun  action
  --
++  grad  %noun
--
```

**`/mar/delta/update.hoon`**

```hoon
/-  *delta
|_  upd=update
++  grow
  |%
  ++  noun  upd
  --
++  grab
  |%
  ++  noun  update
  --
++  grad  %noun
--
```

**`/app/delta-follower.hoon`**

```hoon
/-  *delta
/+  default-agent, dbug
|%
+$  card  card:agent:gall
--
%-  agent:dbug
^-  agent:gall
|_  =bowl:gall
+*  this     .
    default  ~(. (default-agent this %|) bowl)
++  on-poke
  |=  [=mark =vase]
  ^-  (quip card _this)
  ?>  ?=(%noun mark)
  =/  action  !<(?([%sub =@p] [%unsub =@p]) vase)
  ?-    -.action
      %sub
    :_  this
    [%pass /values-wire %agent [p.action %delta] %watch /values]~
  ::
      %unsub
    :_  this
    [%pass /values-wire %agent [p.action %delta] %leave ~]~
  ==
::
++  on-agent
  |=  [=wire =sign:agent:gall]
  ^-  (quip card _this)
  ?>  ?=([%values-wire ~] wire)
  ?+    -.sign  (on-agent:default wire sign)
      %watch-ack
    ?~  p.sign
      ((slog '%delta-follower: subscribe succeeded!' ~) `this)
    ((slog '%delta-follower: subscribe failed!' ~) `this)
  ::
      %kick
    %-  (slog '%delta-follower: Got kick, resubscribing...' ~)
    :_  this
    [%pass /values-wire %agent [src.bowl %delta] %watch /values]~
  ::
    %fact
    ~&  >>  fact+p.cage.sign
    ?>  ?=(%delta-update p.cage.sign)
    ~&  >>  !<(update q.cage.sign)
    `this
  ==
++  on-watch  on-watch:default
++  on-peek   on-peek:default
++  on-init   on-init:default
++  on-save   on-save:default
++  on-load   on-load:default
++  on-arvo   on-arvo:default
++  on-leave  on-leave:default
++  on-fail   on-fail:default
--
```

Mount and make this desk as usual:

```hoon
> |merge %delta our %base

> |mount %delta
```

```sh
$ cd zod/delta
$ echo "~[%delta %delta-follower]" > desk.bill
```

and save the above file to `delta/app/delta.hoon`.

```hoon
> |commit %delta

> |install our %delta
```

It works like this.  You can interact with `%delta` the same way as `%charlie`, but you can also subscribe to updates in `%delta` with `%delta-follower`.

```hoon
> :delta &delta-action [%push ~zod 30.000]

> :delta-follower [%sub ~zod]
>=
%delta-follower: subscribe succeeded!
>>  [%fact %delta-update]
>>  [%init values=~[30.000]]

> :delta &delta-action [%push ~zod 10.000]
>=
>>  [%fact %delta-update]
>>  [%push target=~zod value=10.000]
```

The main thing to note is the use of `%fact` cards.  These are messages sent to all subscribers by the originator.  A subscriber subscribes to a `path` (which depends on the thing it is subscribing to) with a `wire` (which is a unique ID local to the subscriber).

So much for basic Gall agents.  You can implement anything you like using them, but right now you are limited to command-line interactions for the user interface.  Let's take a quick look at how to upgrade to a front-end.


##  Front-End App Development

Front-end development instruments a browser client session to talk to Urbit.  Since Urbit provides a built-in server vane, we communicate via `GET`/`PUT` HTTP calls.

Most people working with Urbit use React as the framework for their front end, but you can make other arrangements if you're willing to figure out how to make things join up correctly.

We're going to move fast and light, and if you want to dig deeper into making an Urbit app with a front-end, you should check out the [Full-Stack Walkthrough](https://urbit.org/docs/userspace/full-stack/1-intro).

We will implement a version of our stack app that runs on one ship and doesn't track communications with other ships.  Although creating such functionality is straightforward, it would take longer than we have in this one-hour walkthrough.

### Talking to a Ship

We don't need to change much in our current Gall agent, but we do need to register endpoints and validate received input using a cookie.

Eyre is the HTTP server vane of Arvo.  It will receive and handle `GET` and `PUT` messages.  The web client will produce a unique channel ID and connect that channel to the agent to monitor subscriptions and acks.

A cookie may be requested using the ship's web login code (by default for ~zod, `lidlut-tabwed-pillex-ridrup`).

### JSON Manipulation

The Urbit Visor Chrome extension produced by dcSpark bridges Web 2.0 applications to web3 on Urbit.  A UV integration would be an excellent hackathon-size project.

### Updates to `%delta`

Let's take `%delta` from our back-end work and update it to have a small front-end which displays its state and allows the front-end to push and pop values. We can leave the `/sur` file and agent the same, and just add JSON encoding & decoding functions to our mark files:

**`/mar/delta/action.hoon`**

```hoon
/-  *delta
|_  act=action
++  grow
  |%
  ++  noun  act
  --
++  grab
  |%
  ++  noun  action
  ++  json
    =,  dejs:format
    |=  jon=json
    ^-  action
    %.  jon
    %-  of
    :~  [%push (ot ~[target+(se %p) value+ni])]
        [%pop (se %p)]
    ==
  --
++  grad  %noun
--
```

**`/mar/delta/update.hoon`**

```hoon
/-  *delta
|_  upd=update
++  grow
  |%
  ++  noun  upd
  ++  json
    =,  enjs:format
    ^-  ^json
    ?-    -.upd
      %pop   (frond 'pop' s+(scot %p target.upd))
      %init  (frond 'init' a+(turn values.upd numb))
      %push  %+  frond  'push'
             %-  pairs
             :~  ['target' s+(scot %p target.upd)]
                 ['value' (numb value.upd)]
    ==       ==
  --
++  grab
  |%
  ++  noun  update
  --
++  grad  %noun
--
```

Commit these changes to the `%delta` desk:

```hoon
> |commit %delta
```

The CLI tools still work like they did in `%charlie` and the old `%delta`, but now we're primarily interested in the front end.

The front-end itself will be served from the ship by the standard `%docket` agent available on every Urbit ship.  While you can construct HTTP requests directly, most of the time you'll just use the `@urbit/http-api` library functions.

We will build the React front-end using Node.js.  Install the library `npm i @urbit/http-api`.  Set the `homepage` to `/apps/delta` in `package.json`.  There are a couple of other small tweaks documented in the [React app setup](https://urbit.org/docs/userspace/full-stack/7-react-setup) page as well.

We need to define a few functions in our `App.js` file:

**`init()`**

```js
init = () => {
  this.getStack()
  .then(
    (result) => {
      this.handleUpdate(result);
      this.setState({latestUpdate: result.time}); 
      this.subscribe()
    },
    (err) => {
      this.setErrorMsg("Connection failed");
      this.setState({status: "err"})
    }
  )
};
```

**`getStack()`**

```js
getStack = async () => {
  const {entries: e} = this.state;
  const path = `/values`;
  return window.urbit.scry({
    app: "echo",
    path: path
  })
};
```

**`moreStack()`**

```js
moreStack = () => {
  this.getStack()
  .then(
    (result) => {this.handleUpdate(result)},
    (err) => {this.setErrorMsg("Fetching more entries failed")}
  )
};
```

**`subscribe()`**

```js
subscribe = () => {
  try {
    window.urbit.subscribe({
      app: "echo",
      path: "/values",
      event: this.handleUpdate,
      err: ()=>this.setErrorMsg("Subscription rejected"),
      quit: ()=>this.setErrorMsg("Kicked from subscription")
    })
  } catch {
    this.setErrorMsg("Subscription failed")
  }
};
```

**`push()`**

```js
push = (value) => {
  window.urbit.poke({
    app: "echo",
    mark: "echo-action",
    json: {"push": {"value":value}},
    onError: ()=>this.setErrorMsg("Pop rejected")
  })
  this.setState({
        ...rmModalShow: false, entryToDelete: null})
        ...(toE && { values: values }),
  })
};
```

**`pop()`**

```js
pop = (value) => {
  window.urbit.poke({
    app: "echo",
    mark: "echo-action",
    json: {"pop": ~},
    onError: ()=>this.setErrorMsg("Pop rejected")
  })
  this.setState({rmModalShow: false, entryToDelete: null})
};
```

**`handleUpdate()`**

```js
handleUpdate = (upd) => {
  const { entries, drafts, results, latestUpdate } = this.state;
  if (upd.time !== latestUpdate) {
    if ("values" in upd) {
      this.setState({ values: values.concat(upd.values) });
    } else if ("push" in upd) {
      const { time, push } = upd;
      const eInd = this.spot(add.id, values);
      const rInd = this.spot(add.id, results);
      const toE =
        entries.length === 0 || add.id > entries[entries.length - 1].id;
      const toR = this.inSearch(add.id, time);
      toE && entries.splice(eInd, 0, add);
      toR && results.splice(rInd, 0, add);
      this.setState({
        ...(toE && { entries: entries }),
        ...(toR && { results: results }),
        latestUpdate: time,
      });
    } else if ("pop" in upd) {
      const { time, pop } = upd;
      const eInd = entries.findIndex((e) => e.id === edit.id);
      const rInd = results.findIndex((e) => e.id === edit.id);
      const toE = eInd !== -1;
      const toR = rInd !== -1 && this.inSearch(edit.id, time);
      if (toE) entries[eInd] = edit;
      if (toR) results[rInd] = edit;
      (toE || toR) && delete drafts[edit.id];
      this.setState({
        ...(toE && { entries: entries }),
        ...(toR && { results: results }),
        ...((toE || toR) && { drafts: drafts }),
        latestUpdate: time,
      });
    }
  }
};
```

**`submitNew()`**

```js
submitNew = (value) => {
  window.urbit.poke({
    app: "echo",
    mark: "echo-action",
    json: { push: { value: value } },
    onSuccess: () => this.setState({ newValue: {} }),
    onError: () => this.setErrorMsg("New value rejected"),
  });
};
```

**`delete`**

```js
delete = (id) => {
  window.urbit.poke({
    app: "echo",
    mark: "echo-action",
    json: {"pop": ~ },
    onError: ()=>this.setErrorMsg("Deletion rejected")
  })
  this.setState({rmModalShow: false, entryToDelete: null})
};
```

**`printEntry()`**

```js
printEntry = ({id, txt}) => {
  const {drafts} = this.state;
  const edit = (id in drafts);
  const draft = (id in drafts) ? drafts[id] : null; 
  const reg = /(?:\r?\n[ \t]*){2,}(?!\s*$)/;
  let d = new Date(id);         
  return (
    <Card key={id}>
      <Card.Header
        className="fs-4 d-flex align-items-center justify-content-between"
      >
        {d.toLocaleString()}
        <CloseButton
          className="fs-6"
          onClick={() => (edit) ? this.cancelEdit(id) : this.openRmModal(id)}
        />
      </Card.Header>
      <Card.Body onClick={()=>this.setEdit(id)}>
        {(edit)
         ? this.editBox(id, txt, draft)
         : txt.split(reg).map((e, ind) => <p key={ind}>{e}</p>)
        }
      </Card.Body>
      {(edit) &&
        <Button
          variant="outline-primary"
          className="mx-3 mb-3"
          onClick={()=>this.submitEdit(id, draft)}
        >
          Submit
        </Button>
      }
    </Card>
  )
};

```

**`render()`**

```js
function NumberList(props) {
  const numbers = props.numbers;
  const listItems = numbers.map((number) =>
    <li key={number.toString()}>{number}</li>
  );
  return (
    <ul>{listItems}</ul>
  );
}

const root = ReactDOM.createRoot(document.getElementById('root'));
root.render();

render() {
  return (
    <React.Fragment>
      {this.rmModal()}
      <div className="m-3 d-flex justify-content-center">
        <Card style={{maxWidth: "50rem", width: "100%"}}>
          <Tabs defaultActiveKey="echo" className="fs-2">
            <Tab eventKey="echo" title="Echo">
              <BottomScrollListener onBottom={()=>this.values()}>
                {(scrollRef) =>
                  <Stack gap={5} className="m-3 d-flex">
                    {this.newValue()}
                    {this.state.entries.map(e => this.printValue(e))}
                  </Stack>
                  <NumberList numbers={values} />
                }
              </BottomScrollListener>
            </Tab>
          </Tabs>
          {this.status()}
        </Card>
      </div>
    </React.Fragment>
  )
}
```

The webpage itself is built using Node.js.  We need to build the React app using `npm run build`, then after installing the desk we can upload the file glob to the `%docket` server app.

We provide the following files on the desk:

```
ourfiles
├── desk.bill
├── sys.kelvin
├── desk.docket-0
├── app
│   └── echo.hoon
├── lib
│   └── echo.hoon
├── mar
│   ├── bill.hoon
│   ├── docket-0.hoon
│   ├── kelvin.hoon
│   └── echo
│       ├── action.hoon
│       └── update.hoon
└── sur
    └── echo.hoon
```

The last thing we need to do is publish our app so other users can install it from our ship.  To do that, we just run the following command in the dojo:

```hoon
:treaty|publish %echo
```

Let's take a quick look at the front-end functionality now.

##  Conclusion

Hoon is a lot of fun to develop in, and Urbit's platform guarantees make it easy to implement serious software quickly.  I hope this whirlwind tour gives you a sense of the elements of Urbit development.

A few final points:

- Don't be shy about looking up runes.  There are a lot of irregular syntax expressions in Hoon, sugar syntax that does reduce to standard syntax.  If you see a confusing and dense expression, check for irregular syntax first.
- Refer to the standard library implementations if you need to tweak something's behavior, like implementing a different `++sort`.  Search in `/sys` by `++  arm-name` to locate the code.  You'll run into some terse and cryptic code, but be bold and thus favored by fortune!
- Any rune that starts with `~` sig is a runtime hint.  You can treat these like C `#pragma`s and ignore them when understanding the operation of code.
