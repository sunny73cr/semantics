# Programming tips / semantics
General advice for programming / software development.

No, there's no donate link.

## Minimising stack exchange; program correctness, code correctness

[Best practices on C Programming style; minimising stack exchange and memory acquisition](https://gist.github.com/sunny73cr/2a7cee5ca817faf110438b006b32ab62)

## Reducing the reverse-ability of code via static analysis

[Referential Gist on quietening your binary, and (somewhat) your stack](https://gist.github.com/sunny73cr/a2c8e9fa9e7e81e66692e5f831f6142d)

## and now; for [something completely different](https://www.youtube.com/watch?v=dO_vv3aRZRo):

[Referential Gist on "Memory Layout Hardening or '++ASLR'"](https://gist.github.com/sunny73cr/f6cd4b182024299bc9436ab3f9fc7488)

## hardening your compilation, assembly and linkage steps:

[Referential Gist on "Best Practices" for 'C' compilation, assembly and linkage](https://gist.github.com/sunny73cr/9d26b4efe479e8ff7ac7d6cf34300c73)

## Bad practices in the industry!

What the hell is wrong with sharing a public TLS certificate?

## Correcting... CPU's? (Nevermind. Wrong approach!)

[Analytical Gist on 'Intel 64-bit / IA-32 RORX Instruction'](https://gist.github.com/sunny73cr/e94e7e4a1c3cd3f4eb3f35cb53f5d1b4)

Update: correcting... myself, of course!

## Correcting... Stanford University? Kinda.

[Analytical Gist on 'Stanford's "Bit Twiddling Hacks"'](https://gist.github.com/sunny73cr/1b4a83e8ace5760cd6a06c5a847fe5c2)

Feel free to claim the reward for yourself.

## Correcting... Compilers.

Programming is F A R too declarative; even at the level ("closeness to hardware") of 'C'.

Take the following snippets as an example:

```
...
 return a_ip4cidr(addr, mask);
}
```

```
...
 ip4cidr* elem = a_ip4cidr(addr, mask);
 return elem;
}
```

```
...
 ip4cidr* elem = a_ip4cidr(addr, mask);
 return a_ip4cidr(addr, mask);
}
```

All three should have zero effect on my program's final 'functionality'; that is, the user experience: except for performance.

- I found that with the first version: it cannot parse
  addresses correctly. It always fails.

- I found that with the second version: it "works",
  but the address is wrong: it has still failed.

- I found that with the third version: it fails intermittently;
  out of 447 addresses... 14, 27, 51, 69, 70, 71, 85, etc... failed to parse,
  then the program finally fails with an assertion on some field that I know
  should absolutely be non-null. Keep in mind that it had JUST worked for
  addresses 1-13, 15-26... etc. I can only assume that the address is still
  incorrect, much like the first examples.

You might think that it is wasteful in developer time and performance to
be really absolutely certain of the value of an input; even if it was
generated hundreds of nanoseconds ago, from known-good data; by a 'child'
function call, or parent method: but the above example is precisely why
I still valdiate every input... and it is wasteful; but I find it is still
neccessary.

It is synchronous, imperative code. Do bitwise ands, ors and shifts not
work the way that I think they do on unsigned integers?

The code that I altered should not have such wild effects on the program.

I used MinGW (GCC 15.2) this time. Previously I was using GCC 14.2 on Debian... similar issues.

Which toolchain for C programming actually works?
