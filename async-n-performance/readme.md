# Foreword
# Preface
# Chapter 1: Asynchrony: Now & Later
## A Program in Chunks
### Async Console
* `console.*` methods [...] are not officially part of JavaScript, but are instead added to JS by the *hosting environment* [...] there are some browsers and (me: when a browser felt it needed to defer the console I/O to the background for performance reasons)(some conditions) [...] that `console.log(..)` does not [...] immediately output what it's given. The main reason this may happen is because I/O is a very slow and blocking part of `many programs` (`not just JS`).

* If you run into this rare scenario, the best option is to use `breakpoints` in your JS debugger.

## Event Loop
