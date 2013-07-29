# frak

frak transforms collections of strings into regular expressions for
matching those strings. The primary goal of this library is to
generate regular expressions from a known set of inputs which avoid
backtracking as much as possible.

## "Installation"

Add frak as a dependency to your `project.clj` file.

```clojure
[frak "0.1.1"]
```

## Usage

```clojure
user> (require 'frak)
nil
user> (frak/pattern ["foo" "bar" "baz" "quux"])
#"(?:ba(?:r|z)|foo|quux)"
user> (frak/pattern ["Clojure" "Clojars" "ClojureScript"])
#"Cloj(?:ure(?:Script)?|ars)"
```

## How?

A frak pattern is constructed from a very stupid trie of characters.
As words are added to it, meta data is stored in it's branches. The
meta data contains information such as which branches are terminal and
a record of characters which have "visited" the branch.

During the rendering process frak will prefer branch characters that
have "visited" the most. In the example above, you will notice the
`ba(?:r|z)` branch takes precedence over `foo` even though `"foo"` was
the first to enter the trie. This is because the character `\b` has
frequented the branch more than `\f` and `\q`. The example below
illustrates this behavior on the second character of each input.

```clojure
user> (frak/pattern ["bit" "bat" "ban" "bot" "bar" "box"])
#"b(?:a(?:t|n|r)|o(?:t|x)|it)"
```

## Why?

[Here's](https://github.com/guns/vim-clojure-static/blob/249328ee659190babe2b14cd119f972b21b80538/syntax/clojure.vim#L91-L92)
why. Also because.

Let's build a regular expression for matching any word in
`/usr/share/dict/words` (hint: no). 

```clojure
user> (require '[clojure.java.io :as io])
nil
user> (def words
           (-> (io/file "/usr/share/dict/words")
               io/reader
               line-seq))
#'user/words
user> (def word-re (frak/pattern words))
#'user/word-re
user> (every? #(re-matches word-re %) words) 
true
```

You can view the full expression
[here](https://gist.github.com/noprompt/6106573/raw/fcb683834bb2e171618ca91bf0b234014b5b957d/word-re.clj)
(it's approximately `1.5M`!).
