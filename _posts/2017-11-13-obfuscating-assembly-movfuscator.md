---
published: false
---
#### Introduction
Reverse Engineering

#### the mov instruction


#### What is a Turing Machine?


#### Turing completeness of mov
Logically, you would say "how would a mov instruction be turing-complete? it's illogical". Fortunately, Stephen Dolan, a researcher at Cambridge University has published a [paper](http://www.cl.cam.ac.uk/~sd601/papers/mov.pdf) proving how could mov be turing-complete. He


#### Movfuscator's concept
Following the release of Dolan's paper, it inspired Chris Domas, a security researcher, to write his own compiler that compiles the entire C program into mov instructions only.


#### Example code




#### References:
[mov is Turing-complete, by Stephen Dolan](http://www.cl.cam.ac.uk/~sd601/papers/mov.pdf)

[Movfuscator's repository on Github](https://github.com/xoreaxeaxeax/movfuscator/)

[A presentation by Domas explaining the concept and the design of Movfuscator (Video)](https://www.youtube.com/watch?v=R7EEoWg6Ekk)

