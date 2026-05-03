# Macros


<a id="c-evocative"></a>
## Input syntax is evocative of the output (C-EVOCATIVE)

Rust macros let you dream up practically whatever input syntax you want. Aim to
keep input syntax familiar and cohesive with the rest of your users' code by
mirroring existing Rust syntax where possible. Pay attention to the choice and
placement of keywords and punctuation.

A good guide is to use syntax, especially keywords and punctuation, that is
similar to what will be produced in the output of the macro.

For example if your macro declares a struct with a particular name given in the
input, preface the name with the keyword `struct` to signal to readers that a
struct is being declared with the given name.

```rust
// Prefer this...
bitflags! {
    struct S: u32 { /* ... */ }
}

// ...over no keyword...
bitflags! {
    S: u32 { /* ... */ }
}

// ...or some ad-hoc word.
bitflags! {
    flags S: u32 { /* ... */ }
}
```

Another example is semicolons vs commas. Constants in Rust are followed by
semicolons so if your macro declares a chain of constants, they should likely be
followed by semicolons even if the syntax is otherwise slightly different from
Rust's.

```rust
// Ordinary constants use semicolons.
const A: u32 = 0b000001;
const B: u32 = 0b000010;

// So prefer this...
bitflags! {
    struct S: u32 {
        const C = 0b000100;
        const D = 0b001000;
    }
}

// ...over this.
bitflags! {
    struct S: u32 {
        const E = 0b010000,
        const F = 0b100000,
    }
}
```

Macros are so diverse that these specific examples won't be relevant, but think
about how to apply the same principles to your situation.


<a id="c-macro-attr"></a>
## Item macros compose well with attributes (C-MACRO-ATTR)

Macros that produce more than one output item should support adding attributes
to any one of those items. One common use case would be putting individual items
behind a cfg.

```rust
bitflags! {
    struct Flags: u8 {
        #[cfg(windows)]
        const ControlCenter = 0b001;
        #[cfg(unix)]
        const Terminal = 0b010;
    }
}
```

Macros that produce a struct or enum as output should support attributes so that
the output can be used with derive.

```rust
bitflags! {
    #[derive(Default, Serialize)]
    struct Flags: u8 {
        const ControlCenter = 0b001;
        const Terminal = 0b010;
    }
}
```


<a id="c-anywhere"></a>
## Item macros work anywhere that items are allowed (C-ANYWHERE)

Rust allows items to be placed at the module level or within a tighter scope
like a function. Item macros should work equally well as ordinary items in all
of these places. The test suite should include invocations of the macro in at
least the module scope and function scope.

```rust
#[cfg(test)]
mod tests {
    test_your_macro_in_a!(module);

    #[test]
    fn anywhere() {
        test_your_macro_in_a!(function);
    }
}
```

As a simple example of how things can go wrong, this macro works great in a
module scope but fails in a function scope.

```rust
macro_rules! broken {
    ($m:ident :: $t:ident) => {
        pub struct $t;
        pub mod $m {
            pub use super::$t;
        }
    }
}

broken!(m::T); // okay, expands to T and m::T

fn g() {
    broken!(m::U); // fails to compile, super::U refers to the containing module not g
}
```


<a id="c-macro-vis"></a>
## Item macros support visibility specifiers (C-MACRO-VIS)

Follow Rust syntax for visibility of items produced by a macro. Private by
default, public if `pub` is specified.

```rust
bitflags! {
    struct PrivateFlags: u8 {
        const A = 0b0001;
        const B = 0b0010;
    }
}

bitflags! {
    pub struct PublicFlags: u8 {
        const C = 0b0100;
        const D = 0b1000;
    }
}
```


<a id="c-macro-ty"></a>
## Type fragments are flexible (C-MACRO-TY)

If your macro accepts a type fragment like `$t:ty` in the input, it should be
usable with all of the following:

- Primitives: `u8`, `&str`
- Relative paths: `m::Data`
- Absolute paths: `::base::Data`
- Upward relative paths: `super::Data`
- Generics: `Vec<String>`

As a simple example of how things can go wrong, this macro works great with
primitives and absolute paths but fails with relative paths.

```rust
macro_rules! broken {
    ($m:ident => $t:ty) => {
        pub mod $m {
            pub struct Wrapper($t);
        }
    }
}

broken!(a => u8); // okay

broken!(b => ::std::marker::PhantomData<()>); // okay

struct S;
broken!(c => S); // fails to compile
```
# Debuggability


<a id="c-debug"></a>
## All public types implement `Debug` (C-DEBUG)

If there are exceptions, they are rare.


<a id="c-debug-nonempty"></a>
## `Debug` representation is never empty (C-DEBUG-NONEMPTY)

Even for conceptually empty values, the `Debug` representation should never be
empty.

```rust
let empty_str = "";
assert_eq!(format!("{:?}", empty_str), "\"\"");

let empty_vec = Vec::<bool>::new();
assert_eq!(format!("{:?}", empty_vec), "[]");
```
# Summary

[About](about.md)
[Checklist](checklist.md)

- [Naming](naming.md)
- [Interoperability](interoperability.md)
- [Macros](macros.md)
- [Documentation](documentation.md)
- [Predictability](predictability.md)
- [Flexibility](flexibility.md)
- [Type safety](type-safety.md)
- [Dependability](dependability.md)
- [Debuggability](debuggability.md)
- [Future proofing](future-proofing.md)
- [Necessities](necessities.md)

[External links](external-links.md)
# External links

- [RFC 199] - Ownership naming conventions
- [RFC 344] - Naming conventions
- [RFC 430] - Naming conventions
- [RFC 505] - Doc conventions
- [RFC 1574] - Doc conventions
- [RFC 1687] - Crate-level documentation
- [Elegant Library APIs in Rust](https://deterministic.space/elegant-apis-in-rust.html)
- [Rust Design Patterns](https://rust-unofficial.github.io/patterns/)

[RFC 344]: https://github.com/rust-lang/rfcs/blob/master/text/0344-conventions-galore.md
[RFC 430]: https://github.com/rust-lang/rfcs/blob/master/text/0430-finalizing-naming-conventions.md
[RFC 1687]: https://github.com/rust-lang/rfcs/pull/1687
[RFC 505]: https://github.com/rust-lang/rfcs/blob/master/text/0505-api-comment-conventions.md
[RFC 1105]: https://github.com/rust-lang/rfcs/blob/master/text/1105-api-evolution.md
[RFC 1574]: https://github.com/rust-lang/rfcs/blob/master/text/1574-more-api-documentation-conventions.md
[RFC 199]: https://github.com/rust-lang/rfcs/blob/master/text/0199-ownership-variants.md
# Future proofing


<a id="c-sealed"></a>
## Sealed traits protect against downstream implementations (C-SEALED)

Some traits are only meant to be implemented within the crate that defines them.
In such cases, we can retain the ability to make changes to the trait in a
non-breaking way by using the sealed trait pattern.

```rust
/// This trait is sealed and cannot be implemented for types outside this crate.
pub trait TheTrait: private::Sealed {
    // Zero or more methods that the user is allowed to call.
    fn ...();

    // Zero or more private methods, not allowed for user to call.
    #[doc(hidden)]
    fn ...();
}

// Implement for some types.
impl TheTrait for usize {
    /* ... */
}

mod private {
    pub trait Sealed {}

    // Implement for those same types, but no others.
    impl Sealed for usize {}
}
```

The empty private `Sealed` supertrait cannot be named by downstream crates, so
we are guaranteed that implementations of `Sealed` (and therefore `TheTrait`)
only exist in the current crate. We are free to add methods to `TheTrait` in a
non-breaking release even though that would ordinarily be a breaking change for
traits that are not sealed. Also we are free to change the signature of methods
that are not publicly documented.

Note that removing a public method or changing the signature of a public method
in a sealed trait are still breaking changes.

To avoid frustrated users trying to implement the trait, it should be documented
in rustdoc that the trait is sealed and not meant to be implemented outside of
the current crate.

### Examples

- [`serde_json::value::Index`](https://docs.serde.rs/serde_json/value/trait.Index.html)
- [`byteorder::ByteOrder`](https://docs.rs/byteorder/1.1.0/byteorder/trait.ByteOrder.html)


<a id="c-struct-private"></a>
## Structs have private fields (C-STRUCT-PRIVATE)

Making a field public is a strong commitment: it pins down a representation
choice, _and_ prevents the type from providing any validation or maintaining any
invariants on the contents of the field, since clients can mutate it arbitrarily.

Public fields are most appropriate for `struct` types in the C spirit: compound,
passive data structures. Otherwise, consider providing getter/setter methods and
hiding fields instead.


<a id="c-newtype-hide"></a>
## Newtypes encapsulate implementation details (C-NEWTYPE-HIDE)

A newtype can be used to hide representation details while making precise
promises to the client.

For example, consider a function `my_transform` that returns a compound iterator
type.

```rust
use std::iter::{Enumerate, Skip};

pub fn my_transform<I: Iterator>(input: I) -> Enumerate<Skip<I>> {
    input.skip(3).enumerate()
}
```

We wish to hide this type from the client, so that the client's view of the
return type is roughly `Iterator<Item = (usize, T)>`. We can do so using the
newtype pattern:

```rust
use std::iter::{Enumerate, Skip};

pub struct MyTransformResult<I>(Enumerate<Skip<I>>);

impl<I: Iterator> Iterator for MyTransformResult<I> {
    type Item = (usize, I::Item);

    fn next(&mut self) -> Option<Self::Item> {
        self.0.next()
    }
}

pub fn my_transform<I: Iterator>(input: I) -> MyTransformResult<I> {
    MyTransformResult(input.skip(3).enumerate())
}
```

Aside from simplifying the signature, this use of newtypes allows us to promise
less to the client. The client does not know _how_ the result iterator is
constructed or represented, which means the representation can change in the
future without breaking client code.

Rust 1.26 also introduces the [`impl Trait`][] feature, which is more concise
than the newtype pattern but with some additional trade offs, namely with `impl
Trait` you are limited in what you can express.  For example, returning an
iterator that impls `Debug` or `Clone` or some combination of the other iterator
extension traits can be problematic.  In summary `impl Trait` as a return type
is probably great for internal APIs and may even be appropriate for public APIs,
but probably not in all cases.  See the ["`impl Trait` for returning complex
types with ease"][impl-trait-2] section of the Edition Guide for more details.

[`impl Trait`]: https://github.com/rust-lang/rfcs/blob/master/text/1522-conservative-impl-trait.md
[impl-trait-2]: https://rust-lang.github.io/edition-guide/rust-2018/trait-system/impl-trait-for-returning-complex-types-with-ease.html

```rust
pub fn my_transform<I: Iterator>(input: I) -> impl Iterator<Item = (usize, I::Item)> {
    input.skip(3).enumerate()
}
```


<a id="c-struct-bounds"></a>
## Data structures do not duplicate derived trait bounds (C-STRUCT-BOUNDS)

Generic data structures should not use trait bounds that can be derived or do
not otherwise add semantic value. Each trait in the `derive` attribute will be
expanded into a separate `impl` block that only applies to generic arguments
that implement that trait.

```rust
// Prefer this:
#[derive(Clone, Debug, PartialEq)]
struct Good<T> { /* ... */ }

// Over this:
#[derive(Clone, Debug, PartialEq)]
struct Bad<T: Clone + Debug + PartialEq> { /* ... */ }
```

Duplicating derived traits as bounds on `Bad` is unnecessary and a
backwards-compatibiliity hazard. To illustrate this point, consider deriving
`PartialOrd` on the structures in the previous example:

```rust
// Non-breaking change:
#[derive(Clone, Debug, PartialEq, PartialOrd)]
struct Good<T> { /* ... */ }

// Breaking change:
#[derive(Clone, Debug, PartialEq, PartialOrd)]
struct Bad<T: Clone + Debug + PartialEq + PartialOrd> { /* ... */ }
```

Generally speaking, adding a trait bound to a data structure is a breaking
change because every consumer of that structure will need to start satisfying
the additional bound. Deriving more traits from the standard library using the
`derive` attribute is not a breaking change.

The following traits should never be used in bounds on data structures:

- `Clone`
- `PartialEq`
- `PartialOrd`
- `Debug`
- `Display`
- `Default`
- `Error`
- `Serialize`
- `Deserialize`
- `DeserializeOwned`

There is a grey area around other non-derivable trait bounds that are not
strictly required by the structure definition, like `Read` or `Write`. They may
communicate the intended behavior of the type better in its definition but also
limits future extensibility. Including semantically useful trait bounds on data
structures is still less problematic than including derivable traits as bounds.

### Exceptions

There are three exceptions where trait bounds on structures are required:

1. The data structure refers to an associated type on the trait.
1. The bound is `?Sized`.
1. The data structure has a `Drop` impl that requires trait bounds.
Rust currently requires all trait bounds on the `Drop` impl are also present
on the data structure.

### Examples from the standard library

- [`std::borrow::Cow`] refers to an associated type on the `Borrow` trait.
- [`std::boxed::Box`] opts out of the implicit `Sized` bound.
- [`std::io::BufWriter`] requires a trait bound in its `Drop` impl.

[`std::borrow::Cow`]: https://doc.rust-lang.org/std/borrow/enum.Cow.html
[`std::boxed::Box`]: https://doc.rust-lang.org/std/boxed/struct.Box.html
[`std::io::BufWriter`]: https://doc.rust-lang.org/std/io/struct.BufWriter.html
# Dependability


<a id="c-validate"></a>
## Functions validate their arguments (C-VALIDATE)

Rust APIs do _not_ generally follow the [robustness principle]: "be conservative
in what you send; be liberal in what you accept".

[robustness principle]: http://en.wikipedia.org/wiki/Robustness_principle

Instead, Rust code should _enforce_ the validity of input whenever practical.

Enforcement can be achieved through the following mechanisms (listed in order of
preference).

### Static enforcement

Choose an argument type that rules out bad inputs.

For example, prefer

```rust
fn foo(a: Ascii) { /* ... */ }
```

over

```rust
fn foo(a: u8) { /* ... */ }
```

where `Ascii` is a _wrapper_ around `u8` that guarantees the highest bit is
zero; see newtype patterns ([C-NEWTYPE]) for more details on creating typesafe
wrappers.

Static enforcement usually comes at little run-time cost: it pushes the costs to
the boundaries (e.g. when a `u8` is first converted into an `Ascii`). It also
catches bugs early, during compilation, rather than through run-time failures.

On the other hand, some properties are difficult or impossible to express using
types.

[C-NEWTYPE]: type-safety.html#c-newtype

### Dynamic enforcement

Validate the input as it is processed (or ahead of time, if necessary). Dynamic
checking is often easier to implement than static checking, but has several
downsides:

1. Runtime overhead (unless checking can be done as part of processing the
   input).
2. Delayed detection of bugs.
3. Introduces failure cases, either via `panic!` or `Result`/`Option` types,
   which must then be dealt with by client code.

#### Dynamic enforcement with `debug_assert!`

Same as dynamic enforcement, but with the possibility of easily turning off
expensive checks for production builds.

#### Dynamic enforcement with opt-out

Same as dynamic enforcement, but adds sibling functions that opt out of the
checking.

The convention is to mark these opt-out functions with a suffix like
`_unchecked` or by placing them in a `raw` submodule.

The unchecked functions can be used judiciously in cases where (1) performance
dictates avoiding checks and (2) the client is otherwise confident that the
inputs are valid.


<a id="c-dtor-fail"></a>
## Destructors never fail (C-DTOR-FAIL)

Destructors are executed while panicking, and in that context a failing
destructor causes the program to abort.

Instead of failing in a destructor, provide a separate method for checking for
clean teardown, e.g. a `close` method, that returns a `Result` to signal
problems. If that `close` method is not called, the `Drop` implementation
should do the teardown and ignore or log/trace any errors it produces.


<a id="c-dtor-block"></a>
## Destructors that may block have alternatives (C-DTOR-BLOCK)

Similarly, destructors should not invoke blocking operations, which can make
debugging much more difficult. Again, consider providing a separate method for
preparing for an infallible, nonblocking teardown.
# Flexibility


<a id="c-intermediate"></a>
## Functions expose intermediate results to avoid duplicate work (C-INTERMEDIATE)

Many functions that answer a question also compute interesting related data. If
this data is potentially of interest to the client, consider exposing it in the
API.

### Examples from the standard library

- [`Vec::binary_search`] does not return a `bool` of whether the value was
  found, nor an `Option<usize>` of the index at which the value was maybe found.
  Instead it returns information about the index if found, and also the index at
  which the value would need to be inserted if not found.

- [`String::from_utf8`] may fail if the input bytes are not UTF-8. In the error
  case it returns an intermediate result that exposes the byte offset up to
  which the input was valid UTF-8, as well as handing back ownership of the
  input bytes.

- [`HashMap::insert`] returns an `Option<T>` that returns the preexisting value
  for a given key, if any. For cases where the user wants to recover this value
  having it returned by the insert operation avoids the user having to do a second
  hash table lookup.

[`Vec::binary_search`]: https://doc.rust-lang.org/std/vec/struct.Vec.html#method.binary_search
[`String::from_utf8`]: https://doc.rust-lang.org/std/string/struct.String.html#method.from_utf8
[`HashMap::insert`]: https://doc.rust-lang.org/stable/std/collections/struct.HashMap.html#method.insert


<a id="c-caller-control"></a>
## Caller decides where to copy and place data (C-CALLER-CONTROL)

If a function requires ownership of an argument, it should take ownership of the
argument rather than borrowing and cloning the argument.

```rust
// Prefer this:
fn foo(b: Bar) {
    /* use b as owned, directly */
}

// Over this:
fn foo(b: &Bar) {
    let b = b.clone();
    /* use b as owned after cloning */
}
```

If a function *does not* require ownership of an argument, it should take a
shared or exclusive borrow of the argument rather than taking ownership and
dropping the argument.

```rust
// Prefer this:
fn foo(b: &Bar) {
    /* use b as borrowed */
}

// Over this:
fn foo(b: Bar) {
    /* use b as borrowed, it is implicitly dropped before function returns */
}
```

The `Copy` trait should only be used as a bound when absolutely needed, not as a
way of signaling that copies should be cheap to make.


<a id="c-generic"></a>
## Functions minimize assumptions about parameters by using generics (C-GENERIC)

The fewer assumptions a function makes about its inputs, the more widely usable
it becomes.

Prefer

```rust
fn foo<I: IntoIterator<Item = i64>>(iter: I) { /* ... */ }
```

over any of

```rust
fn foo(c: &[i64]) { /* ... */ }
fn foo(c: &Vec<i64>) { /* ... */ }
fn foo(c: &SomeOtherCollection<i64>) { /* ... */ }
```

if the function only needs to iterate over the data.

More generally, consider using generics to pinpoint the assumptions a function
needs to make about its arguments.

### Advantages of generics

* _Reusability_. Generic functions can be applied to an open-ended collection of
  types, while giving a clear contract for the functionality those types must
  provide.

* _Static dispatch and optimization_. Each use of a generic function is
  specialized ("monomorphized") to the particular types implementing the trait
  bounds, which means that (1) invocations of trait methods are static, direct
  calls to the implementation and (2) the compiler can inline and otherwise
  optimize these calls.

* _Inline layout_. If a `struct` and `enum` type is generic over some type
  parameter `T`, values of type `T` will be laid out inline in the
  `struct`/`enum`, without any indirection.

* _Inference_. Since the type parameters to generic functions can usually be
  inferred, generic functions can help cut down on verbosity in code where
  explicit conversions or other method calls would usually be necessary.

* _Precise types_. Because generics give a _name_ to the specific type
  implementing a trait, it is possible to be precise about places where that
  exact type is required or produced. For example, a function

  ```rust
  fn binary<T: Trait>(x: T, y: T) -> T
  ```

  is guaranteed to consume and produce elements of exactly the same type `T`; it
  cannot be invoked with parameters of different types that both implement
  `Trait`.

### Disadvantages of generics

* _Code size_. Specializing generic functions means that the function body is
  duplicated. The increase in code size must be weighed against the performance
  benefits of static dispatch.

* _Homogeneous types_. This is the other side of the "precise types" coin: if
  `T` is a type parameter, it stands for a _single_ actual type. So for example
  a `Vec<T>` contains elements of a single concrete type (and, indeed, the
  vector representation is specialized to lay these out in line). Sometimes
  heterogeneous collections are useful; see [trait objects][C-OBJECT].

* _Signature verbosity_. Heavy use of generics can make it more difficult to
  read and understand a function's signature.

[C-OBJECT]: #c-object

### Examples from the standard library

- [`std::fs::File::open`] takes an argument of generic type `AsRef<Path>`. This
  allows files to be opened conveniently from a string literal `"f.txt"`, a
  [`Path`], an [`OsString`], and a few other types.

[`std::fs::File::open`]: https://doc.rust-lang.org/std/fs/struct.File.html#method.open
[`Path`]: https://doc.rust-lang.org/std/path/struct.Path.html
[`OsString`]: https://doc.rust-lang.org/std/ffi/struct.OsString.html


<a id="c-object"></a>
## Traits are object-safe if they may be useful as a trait object (C-OBJECT)

Trait objects have some significant limitations: methods invoked through a trait
object cannot use generics, and cannot use `Self` except in receiver position.

When designing a trait, decide early on whether the trait will be used as an
object or as a bound on generics.

If a trait is meant to be used as an object, its methods should take and return
trait objects rather than use generics.

A `where` clause of `Self: Sized` may be used to exclude specific methods from
the trait's object. The following trait is not object-safe due to the generic
method.

```rust
trait MyTrait {
    fn object_safe(&self, i: i32);

    fn not_object_safe<T>(&self, t: T);
}
```

Adding a requirement of `Self: Sized` to the generic method excludes it from the
trait object and makes the trait object-safe.

```rust
trait MyTrait {
    fn object_safe(&self, i: i32);

    fn not_object_safe<T>(&self, t: T) where Self: Sized;
}
```

### Advantages of trait objects

* _Heterogeneity_. When you need it, you really need it.
* _Code size_. Unlike generics, trait objects do not generate specialized
  (monomorphized) versions of code, which can greatly reduce code size.

### Disadvantages of trait objects

* _No generic methods_. Trait objects cannot currently provide generic methods.
* _Dynamic dispatch and fat pointers_. Trait objects inherently involve
  indirection and vtable dispatch, which can carry a performance penalty.
* _No Self_. Except for the method receiver argument, methods on trait objects
  cannot use the `Self` type.

### Examples from the standard library

- The [`io::Read`] and [`io::Write`] traits are often used as objects.
- The [`Iterator`] trait has several generic methods marked with `where Self:
  Sized` to retain the ability to use `Iterator` as an object.

[`io::Read`]: https://doc.rust-lang.org/std/io/trait.Read.html
[`io::Write`]: https://doc.rust-lang.org/std/io/trait.Write.html
[`Iterator`]: https://doc.rust-lang.org/std/iter/trait.Iterator.html
# Interoperability


<a id="c-common-traits"></a>
## Types eagerly implement common traits (C-COMMON-TRAITS)

Rust's trait system does not allow _orphans_: roughly, every `impl` must live
either in the crate that defines the trait or the implementing type.
Consequently, crates that define new types should eagerly implement all
applicable, common traits.

To see why, consider the following situation:

* Crate `std` defines trait `Display`.
* Crate `url` defines type `Url`, without implementing `Display`.
* Crate `webapp` imports from both `std` and `url`,

There is no way for `webapp` to add `Display` to `Url`, since it defines
neither. (Note: the newtype pattern can provide an efficient, but inconvenient
workaround.)

The most important common traits to implement from `std` are:

- [`Copy`](https://doc.rust-lang.org/std/marker/trait.Copy.html)
- [`Clone`](https://doc.rust-lang.org/std/clone/trait.Clone.html)
- [`Eq`](https://doc.rust-lang.org/std/cmp/trait.Eq.html)
- [`PartialEq`](https://doc.rust-lang.org/std/cmp/trait.PartialEq.html)
- [`Ord`](https://doc.rust-lang.org/std/cmp/trait.Ord.html)
- [`PartialOrd`](https://doc.rust-lang.org/std/cmp/trait.PartialOrd.html)
- [`Hash`](https://doc.rust-lang.org/std/hash/trait.Hash.html)
- [`Debug`](https://doc.rust-lang.org/std/fmt/trait.Debug.html)
- [`Display`](https://doc.rust-lang.org/std/fmt/trait.Display.html)
- [`Default`](https://doc.rust-lang.org/std/default/trait.Default.html)

Note that it is common and expected for types to implement both
`Default` and an empty `new` constructor. `new` is the constructor
convention in Rust, and users expect it to exist, so if it is
reasonable for the basic constructor to take no arguments, then it
should, even if it is functionally identical to `default`.


<a id="c-conv-traits"></a>
## Conversions use the standard traits `From`, `AsRef`, `AsMut` (C-CONV-TRAITS)

The following conversion traits should be implemented where it makes sense:

- [`From`](https://doc.rust-lang.org/std/convert/trait.From.html)
- [`TryFrom`](https://doc.rust-lang.org/std/convert/trait.TryFrom.html)
- [`AsRef`](https://doc.rust-lang.org/std/convert/trait.AsRef.html)
- [`AsMut`](https://doc.rust-lang.org/std/convert/trait.AsMut.html)

The following conversion traits should never be implemented:

- [`Into`](https://doc.rust-lang.org/std/convert/trait.Into.html)
- [`TryInto`](https://doc.rust-lang.org/std/convert/trait.TryInto.html)

These traits have a blanket impl based on `From` and `TryFrom`. Implement those
instead.

### Examples from the standard library

- `From<u16>` is implemented for `u32` because a smaller integer can always be
  converted to a bigger integer.
- `From<u32>` is *not* implemented for `u16` because the conversion may not be
  possible if the integer is too big.
- `TryFrom<u32>` is implemented for `u16` and returns an error if the integer is
  too big to fit in `u16`.
- [`From<Ipv6Addr>`] is implemented for [`IpAddr`], which is a type that can
  represent both v4 and v6 IP addresses.

[`From<Ipv6Addr>`]: https://doc.rust-lang.org/std/net/struct.Ipv6Addr.html
[`IpAddr`]: https://doc.rust-lang.org/std/net/enum.IpAddr.html


<a id="c-collect"></a>
## Collections implement `FromIterator` and `Extend` (C-COLLECT)

[`FromIterator`] and [`Extend`] enable collections to be used conveniently with
the following iterator methods:

[`FromIterator`]: https://doc.rust-lang.org/std/iter/trait.FromIterator.html
[`Extend`]: https://doc.rust-lang.org/std/iter/trait.Extend.html

- [`Iterator::collect`](https://doc.rust-lang.org/std/iter/trait.Iterator.html#method.collect)
- [`Iterator::partition`](https://doc.rust-lang.org/std/iter/trait.Iterator.html#method.partition)
- [`Iterator::unzip`](https://doc.rust-lang.org/std/iter/trait.Iterator.html#method.unzip)

`FromIterator` is for creating a new collection containing items from an
iterator, and `Extend` is for adding items from an iterator onto an existing
collection.

### Examples from the standard library

- [`Vec<T>`] implements both `FromIterator<T>` and `Extend<T>`.

[`Vec<T>`]: https://doc.rust-lang.org/std/vec/struct.Vec.html


<a id="c-serde"></a>
## Data structures implement Serde's `Serialize`, `Deserialize` (C-SERDE)

Types that play the role of a data structure should implement [`Serialize`] and
[`Deserialize`].

[`Serialize`]: https://docs.serde.rs/serde/trait.Serialize.html
[`Deserialize`]: https://docs.serde.rs/serde/trait.Deserialize.html

There is a continuum of types between things that are clearly a data structure
and things that are clearly not, with gray area in between. [`LinkedHashMap`]
and [`IpAddr`] are data structures. It would be completely reasonable for
somebody to want to read in a `LinkedHashMap` or `IpAddr` from a JSON file, or
send one over IPC to another process. [`LittleEndian`] is not a data structure.
It is a marker used by the `byteorder` crate to optimize at compile time for
bytes in a particular order, and in fact an instance of `LittleEndian` can never
exist at runtime. So these are clear-cut examples; the #rust or #serde IRC
channels can help assess more ambiguous cases if necessary.

[`LinkedHashMap`]: https://docs.rs/linked-hash-map/0.4.2/linked_hash_map/struct.LinkedHashMap.html
[`IpAddr`]: https://doc.rust-lang.org/std/net/enum.IpAddr.html
[`LittleEndian`]: https://docs.rs/byteorder/1.0.0/byteorder/enum.LittleEndian.html

If a crate does not already depend on Serde for other reasons, it may wish to
gate Serde impls behind a Cargo cfg. This way downstream libraries only need to
pay the cost of compiling Serde if they need those impls to exist.

For consistency with other Serde-based libraries, the name of the Cargo cfg
should be simply `"serde"`. Do not use a different name for the cfg like
`"serde_impls"` or `"serde_serialization"`.

The canonical implementation looks like this when not using derive:

```toml
[dependencies]
serde = { version = "1.0", optional = true }
```

```rust
pub struct T { /* ... */ }

#[cfg(feature = "serde")]
impl Serialize for T { /* ... */ }

#[cfg(feature = "serde")]
impl<'de> Deserialize<'de> for T { /* ... */ }
```

And when using derive:

```toml
[dependencies]
serde = { version = "1.0", optional = true, features = ["derive"] }
```

```rust
#[cfg_attr(feature = "serde", derive(Serialize, Deserialize))]
pub struct T { /* ... */ }
```


<a id="c-send-sync"></a>
## Types are `Send` and `Sync` where possible (C-SEND-SYNC)

[`Send`] and [`Sync`] are automatically implemented when the compiler determines
it is appropriate.

[`Send`]: https://doc.rust-lang.org/std/marker/trait.Send.html
[`Sync`]: https://doc.rust-lang.org/std/marker/trait.Sync.html

In types that manipulate raw pointers, be vigilant that the `Send` and `Sync`
status of your type accurately reflects its thread safety characteristics. Tests
like the following can help catch unintentional regressions in whether the type
implements `Send` or `Sync`.

```rust
#[test]
fn test_send() {
    fn assert_send<T: Send>() {}
    assert_send::<MyStrangeType>();
}

#[test]
fn test_sync() {
    fn assert_sync<T: Sync>() {}
    assert_sync::<MyStrangeType>();
}
```


<a id="c-good-err"></a>
## Error types are meaningful and well-behaved (C-GOOD-ERR)

An error type is any type `E` used in a `Result<T, E>` returned by any public
function of your crate. Error types should always implement the
[`std::error::Error`] trait which is the mechanism by which error handling
libraries like [`error-chain`] abstract over different types of errors, and
which allows the error to be used as the [`source()`] of another error.

[`std::error::Error`]: https://doc.rust-lang.org/std/error/trait.Error.html
[`error-chain`]: https://docs.rs/error-chain
[`source()`]: https://doc.rust-lang.org/std/error/trait.Error.html#method.source

Additionally, error types should implement the [`Send`] and [`Sync`] traits. An
error that is not `Send` cannot be returned by a thread run with
[`thread::spawn`]. An error that is not `Sync` cannot be passed across threads
using an [`Arc`]. These are common requirements for basic error handling in a
multithreaded application.

[`Send`]: https://doc.rust-lang.org/std/marker/trait.Send.html
[`Sync`]: https://doc.rust-lang.org/std/marker/trait.Sync.html
[`thread::spawn`]: https://doc.rust-lang.org/std/thread/fn.spawn.html
[`Arc`]: https://doc.rust-lang.org/std/sync/struct.Arc.html

`Send` and `Sync` are also important for being able to package a custom error
into an IO error using [`std::io::Error::new`], which requires a trait bound of
`Error + Send + Sync`.

[`std::io::Error::new`]: https://doc.rust-lang.org/std/io/struct.Error.html#method.new

One place to be vigilant about this guideline is in functions that return Error
trait objects, for example [`reqwest::Error::get_ref`]. Typically `Error +
Send + Sync + 'static` will be the most useful for callers. The addition of
`'static` allows the trait object to be used with [`Error::downcast_ref`].

[`reqwest::Error::get_ref`]: https://docs.rs/reqwest/0.7.2/reqwest/struct.Error.html#method.get_ref
[`Error::downcast_ref`]: https://doc.rust-lang.org/std/error/trait.Error.html#method.downcast_ref-2

Never use `()` as an error type, even where there is no useful additional
information for the error to carry.

- `()` does not implement `Error` so it cannot be used with error handling
  libraries like `error-chain`.
- `()` does not implement `Display` so a user would need to write an error
  message of their own if they want to fail because of the error.
- `()` has an unhelpful `Debug` representation for users that decide to
  `unwrap()` the error.
- It would not be semantically meaningful for a downstream library to implement
  `From<()>` for their error type, so `()` as an error type cannot be used with
  the `?` operator.

Instead, define a meaningful error type specific to your crate or to the
individual function. Provide appropriate `Error` and `Display` impls. If there
is no useful information for the error to carry, it can be implemented as a unit
struct.

```rust
use std::error::Error;
use std::fmt::Display;

// Instead of this...
fn do_the_thing() -> Result<Wow, ()>

// Prefer this...
fn do_the_thing() -> Result<Wow, DoError>

#[derive(Debug)]
struct DoError;

impl Display for DoError { /* ... */ }
impl Error for DoError { /* ... */ }
```

The error message given by the `Display` representation of an error type should
be lowercase without trailing punctuation, and typically concise.

[`Error::description()`] should not be implemented. It has been deprecated and users should
always use `Display` instead of `description()` to print the error.

[`Error::description()`]: https://doc.rust-lang.org/std/error/trait.Error.html#tymethod.description

### Examples from the standard library

- [`ParseBoolError`] is returned when failing to parse a bool from a string.

[`ParseBoolError`]: https://doc.rust-lang.org/std/str/struct.ParseBoolError.html

### Examples of error messages

- "unexpected end of file"
- "provided string was not \`true\` or \`false\`"
- "invalid IP address syntax"
- "second time provided was later than self"
- "invalid UTF-8 sequence of {} bytes from index {}"
- "environment variable was not valid unicode: {:?}"


<a id="c-num-fmt"></a>
## Binary number types provide `Hex`, `Octal`, `Binary` formatting (C-NUM-FMT)

- [`std::fmt::UpperHex`](https://doc.rust-lang.org/std/fmt/trait.UpperHex.html)
- [`std::fmt::LowerHex`](https://doc.rust-lang.org/std/fmt/trait.LowerHex.html)
- [`std::fmt::Octal`](https://doc.rust-lang.org/std/fmt/trait.Octal.html)
- [`std::fmt::Binary`](https://doc.rust-lang.org/std/fmt/trait.Binary.html)

These traits control the representation of a type under the `{:X}`, `{:x}`,
`{:o}`, and `{:b}` format specifiers.

Implement these traits for any number type on which you would consider doing
bitwise manipulations like `|` or `&`. This is especially appropriate for
bitflag types. Numeric quantity types like `struct Nanoseconds(u64)` probably do
not need these.

<a id="c-rw-value"></a>
## Generic reader/writer functions take `R: Read` and `W: Write` by value (C-RW-VALUE)

The standard library contains these two impls:

```rust
impl<'a, R: Read + ?Sized> Read for &'a mut R { /* ... */ }

impl<'a, W: Write + ?Sized> Write for &'a mut W { /* ... */ }
```

That means any function that accepts `R: Read` or `W: Write` generic parameters
by value can be called with a mut reference if necessary.

In the documentation of such functions, briefly remind users that a mut
reference can be passed. New Rust users often struggle with this. They may have
opened a file and want to read multiple pieces of data out of it, but the
function to read one piece consumes the reader by value, so they are stuck. The
solution would be to leverage one of the above impls and pass `&mut f` instead
of `f` as the reader parameter.

### Examples

- [`flate2::read::GzDecoder::new`]
- [`flate2::write::GzEncoder::new`]
- [`serde_json::from_reader`]
- [`serde_json::to_writer`]

[`flate2::read::GzDecoder::new`]: https://docs.rs/flate2/0.2/flate2/read/struct.GzDecoder.html#method.new
[`flate2::write::GzEncoder::new`]: https://docs.rs/flate2/0.2/flate2/write/struct.GzEncoder.html#method.new
[`serde_json::from_reader`]: https://docs.serde.rs/serde_json/fn.from_reader.html
[`serde_json::to_writer`]: https://docs.serde.rs/serde_json/fn.to_writer.html
# Rust API Guidelines Checklist

<!-- Read CONTRIBUTING.md before writing new guidelines -->

- **Naming** *(crate aligns with Rust naming conventions)*
  - [ ] Casing conforms to RFC 430 ([C-CASE])
  - [ ] Ad-hoc conversions follow `as_`, `to_`, `into_` conventions ([C-CONV])
  - [ ] Getter names follow Rust convention ([C-GETTER])
  - [ ] Methods on collections that produce iterators follow `iter`, `iter_mut`, `into_iter` ([C-ITER])
  - [ ] Iterator type names match the methods that produce them ([C-ITER-TY])
  - [ ] Feature names are free of placeholder words ([C-FEATURE])
  - [ ] Names use a consistent word order ([C-WORD-ORDER])
- **Interoperability** *(crate interacts nicely with other library functionality)*
  - [ ] Types eagerly implement common traits ([C-COMMON-TRAITS])
    - `Copy`, `Clone`, `Eq`, `PartialEq`, `Ord`, `PartialOrd`, `Hash`, `Debug`,
      `Display`, `Default`
  - [ ] Conversions use the standard traits `From`, `AsRef`, `AsMut` ([C-CONV-TRAITS])
  - [ ] Collections implement `FromIterator` and `Extend` ([C-COLLECT])
  - [ ] Data structures implement Serde's `Serialize`, `Deserialize` ([C-SERDE])
  - [ ] Types are `Send` and `Sync` where possible ([C-SEND-SYNC])
  - [ ] Error types are meaningful and well-behaved ([C-GOOD-ERR])
  - [ ] Binary number types provide `Hex`, `Octal`, `Binary` formatting ([C-NUM-FMT])
  - [ ] Generic reader/writer functions take `R: Read` and `W: Write` by value ([C-RW-VALUE])
- **Macros** *(crate presents well-behaved macros)*
  - [ ] Input syntax is evocative of the output ([C-EVOCATIVE])
  - [ ] Macros compose well with attributes ([C-MACRO-ATTR])
  - [ ] Item macros work anywhere that items are allowed ([C-ANYWHERE])
  - [ ] Item macros support visibility specifiers ([C-MACRO-VIS])
  - [ ] Type fragments are flexible ([C-MACRO-TY])
- **Documentation** *(crate is abundantly documented)*
  - [ ] Crate level docs are thorough and include examples ([C-CRATE-DOC])
  - [ ] All items have a rustdoc example ([C-EXAMPLE])
  - [ ] Examples use `?`, not `try!`, not `unwrap` ([C-QUESTION-MARK])
  - [ ] Function docs include error, panic, and safety considerations ([C-FAILURE])
  - [ ] Prose contains hyperlinks to relevant things ([C-LINK])
  - [ ] Cargo.toml includes all common metadata ([C-METADATA])
    - authors, description, license, homepage, documentation, repository,
      keywords, categories
  - [ ] Release notes document all significant changes ([C-RELNOTES])
  - [ ] Rustdoc does not show unhelpful implementation details ([C-HIDDEN])
- **Predictability** *(crate enables legible code that acts how it looks)*
  - [ ] Smart pointers do not add inherent methods ([C-SMART-PTR])
  - [ ] Conversions live on the most specific type involved ([C-CONV-SPECIFIC])
  - [ ] Functions with a clear receiver are methods ([C-METHOD])
  - [ ] Functions do not take out-parameters ([C-NO-OUT])
  - [ ] Operator overloads are unsurprising ([C-OVERLOAD])
  - [ ] Only smart pointers implement `Deref` and `DerefMut` ([C-DEREF])
  - [ ] Constructors are static, inherent methods ([C-CTOR])
- **Flexibility** *(crate supports diverse real-world use cases)*
  - [ ] Functions expose intermediate results to avoid duplicate work ([C-INTERMEDIATE])
  - [ ] Caller decides where to copy and place data ([C-CALLER-CONTROL])
  - [ ] Functions minimize assumptions about parameters by using generics ([C-GENERIC])
  - [ ] Traits are object-safe if they may be useful as a trait object ([C-OBJECT])
- **Type safety** *(crate leverages the type system effectively)*
  - [ ] Newtypes provide static distinctions ([C-NEWTYPE])
  - [ ] Arguments convey meaning through types, not `bool` or `Option` ([C-CUSTOM-TYPE])
  - [ ] Types for a set of flags are `bitflags`, not enums ([C-BITFLAG])
  - [ ] Builders enable construction of complex values ([C-BUILDER])
- **Dependability** *(crate is unlikely to do the wrong thing)*
  - [ ] Functions validate their arguments ([C-VALIDATE])
  - [ ] Destructors never fail ([C-DTOR-FAIL])
  - [ ] Destructors that may block have alternatives ([C-DTOR-BLOCK])
- **Debuggability** *(crate is conducive to easy debugging)*
  - [ ] All public types implement `Debug` ([C-DEBUG])
  - [ ] `Debug` representation is never empty ([C-DEBUG-NONEMPTY])
- **Future proofing** *(crate is free to improve without breaking users' code)*
  - [ ] Sealed traits protect against downstream implementations ([C-SEALED])
  - [ ] Structs have private fields ([C-STRUCT-PRIVATE])
  - [ ] Newtypes encapsulate implementation details ([C-NEWTYPE-HIDE])
  - [ ] Data structures do not duplicate derived trait bounds ([C-STRUCT-BOUNDS])
- **Necessities** *(to whom they matter, they really matter)*
  - [ ] Public dependencies of a stable crate are stable ([C-STABLE])
  - [ ] Crate and its dependencies have a permissive license ([C-PERMISSIVE])


[C-CASE]: naming.html#c-case
[C-CONV]: naming.html#c-conv
[C-GETTER]: naming.html#c-getter
[C-ITER]: naming.html#c-iter
[C-ITER-TY]: naming.html#c-iter-ty
[C-FEATURE]: naming.html#c-feature
[C-WORD-ORDER]: naming.html#c-word-order

[C-COMMON-TRAITS]: interoperability.html#c-common-traits
[C-CONV-TRAITS]: interoperability.html#c-conv-traits
[C-COLLECT]: interoperability.html#c-collect
[C-SERDE]: interoperability.html#c-serde
[C-SEND-SYNC]: interoperability.html#c-send-sync
[C-GOOD-ERR]: interoperability.html#c-good-err
[C-NUM-FMT]: interoperability.html#c-num-fmt
[C-RW-VALUE]: interoperability.html#c-rw-value

[C-EVOCATIVE]: macros.html#c-evocative
[C-MACRO-ATTR]: macros.html#c-macro-attr
[C-ANYWHERE]: macros.html#c-anywhere
[C-MACRO-VIS]: macros.html#c-macro-vis
[C-MACRO-TY]: macros.html#c-macro-ty

[C-CRATE-DOC]: documentation.html#c-crate-doc
[C-EXAMPLE]: documentation.html#c-example
[C-QUESTION-MARK]: documentation.html#c-question-mark
[C-FAILURE]: documentation.html#c-failure
[C-LINK]: documentation.html#c-link
[C-METADATA]: documentation.html#c-metadata
[C-HTML-ROOT]: documentation.html#c-html-root
[C-RELNOTES]: documentation.html#c-relnotes
[C-HIDDEN]: documentation.html#c-hidden

[C-SMART-PTR]: predictability.html#c-smart-ptr
[C-CONV-SPECIFIC]: predictability.html#c-conv-specific
[C-METHOD]: predictability.html#c-method
[C-NO-OUT]: predictability.html#c-no-out
[C-OVERLOAD]: predictability.html#c-overload
[C-DEREF]: predictability.html#c-deref
[C-CTOR]: predictability.html#c-ctor

[C-INTERMEDIATE]: flexibility.html#c-intermediate
[C-CALLER-CONTROL]: flexibility.html#c-caller-control
[C-GENERIC]: flexibility.html#c-generic
[C-OBJECT]: flexibility.html#c-object

[C-NEWTYPE]: type-safety.html#c-newtype
[C-CUSTOM-TYPE]: type-safety.html#c-custom-type
[C-BITFLAG]: type-safety.html#c-bitflag
[C-BUILDER]: type-safety.html#c-builder

[C-VALIDATE]: dependability.html#c-validate
[C-DTOR-FAIL]: dependability.html#c-dtor-fail
[C-DTOR-BLOCK]: dependability.html#c-dtor-block

[C-DEBUG]: debuggability.html#c-debug
[C-DEBUG-NONEMPTY]: debuggability.html#c-debug-nonempty

[C-SEALED]: future-proofing.html#c-sealed
[C-STRUCT-PRIVATE]: future-proofing.html#c-struct-private
[C-NEWTYPE-HIDE]: future-proofing.html#c-newtype-hide
[C-STRUCT-BOUNDS]: future-proofing.html#c-struct-bounds

[C-STABLE]: necessities.html#c-stable
[C-PERMISSIVE]: necessities.html#c-permissive
# Documentation


<a id="c-crate-doc"></a>
## Crate level docs are thorough and include examples (C-CRATE-DOC)

See [RFC 1687].

[RFC 1687]: https://github.com/rust-lang/rfcs/pull/1687


<a id="c-example"></a>
## All items have a rustdoc example (C-EXAMPLE)

Every public module, trait, struct, enum, function, method, macro, and type
definition should have an example that exercises the functionality.

This guideline should be applied within reason.

A link to an applicable example on another item may be sufficient. For example
if exactly one function uses a particular type, it may be appropriate to write a
single example on either the function or the type and link to it from the other.

The purpose of an example is not always to show *how to use* the item. Readers
can be expected to understand how to invoke functions, match on enums, and other
fundamental tasks. Rather, an example is often intended to show *why someone
would want to use* the item.

```rust
// This would be a poor example of using clone(). It mechanically shows *how* to
// call clone(), but does nothing to show *why* somebody would want this.
fn main() {
    let hello = "hello";

    hello.clone();
}
```


<a id="c-question-mark"></a>
## Examples use `?`, not `try!`, not `unwrap` (C-QUESTION-MARK)

Like it or not, example code is often copied verbatim by users. Unwrapping an
error should be a conscious decision that the user needs to make.

A common way of structuring fallible example code is the following. The lines
beginning with `#` are compiled by `cargo test` when building the example but
will not appear in user-visible rustdoc.

```
/// ```rust
/// # use std::error::Error;
/// #
/// # fn main() -> Result<(), Box<dyn Error>> {
/// your;
/// example?;
/// code;
/// #
/// #     Ok(())
/// # }
/// ```
```


<a id="c-failure"></a>
## Function docs include error, panic, and safety considerations (C-FAILURE)

Error conditions should be documented in an "Errors" section. This applies to
trait methods as well -- trait methods for which the implementation is allowed
or expected to return an error should be documented with an "Errors" section.

For example in the standard library, Some implementations of the
[`std::io::Read::read`] trait method may return an error.

[`std::io::Read::read`]: https://doc.rust-lang.org/std/io/trait.Read.html#tymethod.read

```
/// Pull some bytes from this source into the specified buffer, returning
/// how many bytes were read.
///
/// ... lots more info ...
///
/// # Errors
///
/// If this function encounters any form of I/O or other error, an error
/// variant will be returned. If an error is returned then it must be
/// guaranteed that no bytes were read.
```

Panic conditions should be documented in a "Panics" section. This applies to
trait methods as well -- traits methods for which the implementation is allowed
or expected to panic should be documented with a "Panics" section.

In the standard library the [`Vec::insert`] method may panic.

[`Vec::insert`]: https://doc.rust-lang.org/std/vec/struct.Vec.html#method.insert

```
/// Inserts an element at position `index` within the vector, shifting all
/// elements after it to the right.
///
/// # Panics
///
/// Panics if `index` is out of bounds.
```

It is not necessary to document all conceivable panic cases, especially if the
panic occurs in logic provided by the caller. For example documenting the
`Display` panic in the following code seems excessive. But when in doubt, err on
the side of documenting more panic cases.

```rust
/// # Panics
///
/// This function panics if `T`'s implementation of `Display` panics.
pub fn print<T: Display>(t: T) {
    println!("{}", t.to_string());
}
```

Unsafe functions should be documented with a "Safety" section that explains all
invariants that the caller is responsible for upholding to use the function
correctly.

The unsafe [`std::ptr::read`] requires the following of the caller.

[`std::ptr::read`]: https://doc.rust-lang.org/std/ptr/fn.read.html

```
/// Reads the value from `src` without moving it. This leaves the
/// memory in `src` unchanged.
///
/// # Safety
///
/// Beyond accepting a raw pointer, this is unsafe because it semantically
/// moves the value out of `src` without preventing further usage of `src`.
/// If `T` is not `Copy`, then care must be taken to ensure that the value at
/// `src` is not used before the data is overwritten again (e.g. with `write`,
/// `zero_memory`, or `copy_memory`). Note that `*src = foo` counts as a use
/// because it will attempt to drop the value previously at `*src`.
///
/// The pointer must be aligned; use `read_unaligned` if that is not the case.
```


<a id="c-link"></a>
## Prose contains hyperlinks to relevant things (C-LINK)

Regular links can be added inline with the usual markdown syntax of
`[text](url)`. Links to other types can be added by marking them with
``[`text`]``, then adding the link target in a new line at the end of
the docstring with ``[`text`]: <target>``, where `<target>` is
described below.

Link targets to methods within the same type usually look like this:

```md
[`serialize_struct`]: #method.serialize_struct
```

Link targets to other types usually look like this:

```md
[`Deserialize`]: trait.Deserialize.html
```

Link targets may also point to a parent or child module:

```md
[`Value`]: ../enum.Value.html
[`DeserializeOwned`]: de/trait.DeserializeOwned.html
```

This guideline is officially recommended by RFC 1574 under the heading ["Link
all the things"].

["Link all the things"]: https://github.com/rust-lang/rfcs/blob/master/text/1574-more-api-documentation-conventions.md#link-all-the-things


<a id="c-metadata"></a>
## Cargo.toml includes all common metadata (C-METADATA)

The `[package]` section of `Cargo.toml` should include the following
values:

- `authors`
- `description`
- `license`
- `repository`
- `keywords`
- `categories`

In addition, there are two optional metadata fields:

- `documentation`
- `homepage`

By default, *crates.io* links to documentation for the crate on [*docs.rs*]. The
`documentation` metadata only needs to be set if the documentation is hosted
somewhere other than *docs.rs*, for example because the crate links against a
shared library that is not available in the build environment of *docs.rs*.

[*docs.rs*]: https://docs.rs

The `homepage` metadata should only be set if there is a unique website for the
crate other than the source repository or API documentation. Do not make
`homepage` redundant with either the `documentation` or `repository` values. For
example, serde sets `homepage` to *https://serde.rs*, a dedicated website.

<a id="c-relnotes"></a>
## Release notes document all significant changes (C-RELNOTES)

Users of the crate can read the release notes to find a summary of what
changed in each published release of the crate. A link to the release notes,
or the notes themselves, should be included in the crate-level documentation
and/or the repository linked in Cargo.toml.

Breaking changes (as defined in [RFC 1105]) should be clearly identified in the
release notes.

If using Git to track the source of a crate, every release published to
*crates.io* should have a corresponding tag identifying the commit that was
published. A similar process should be used for non-Git VCS tools as well.

```bash
# Tag the current commit
GIT_COMMITTER_DATE=$(git log -n1 --pretty=%aD) git tag -a -m "Release 0.3.0" 0.3.0
git push --tags
```

Annotated tags are preferred because some Git commands ignore unannotated tags
if any annotated tags exist.

[RFC 1105]: https://github.com/rust-lang/rfcs/blob/master/text/1105-api-evolution.md

### Examples

- [Serde 1.0.0 release notes](https://github.com/serde-rs/serde/releases/tag/v1.0.0)
- [Serde 0.9.8 release notes](https://github.com/serde-rs/serde/releases/tag/v0.9.8)
- [Serde 0.9.0 release notes](https://github.com/serde-rs/serde/releases/tag/v0.9.0)
- [Diesel change log](https://github.com/diesel-rs/diesel/blob/master/CHANGELOG.md)


<a id="c-hidden"></a>
## Rustdoc does not show unhelpful implementation details (C-HIDDEN)

Rustdoc is supposed to include everything users need to use the crate fully and
nothing more. It is fine to explain relevant implementation details in prose but
they should not be real entries in the documentation.

Especially be selective about which impls are visible in rustdoc -- all the ones
that users would need for using the crate fully, but no others. In the following
code the rustdoc of `PublicError` by default would show the `From<PrivateError>`
impl. We choose to hide it with `#[doc(hidden)]` because users can never have a
`PrivateError` in their code so this impl would never be relevant to them.

```rust
// This error type is returned to users.
pub struct PublicError { /* ... */ }

// This error type is returned by some private helper functions.
struct PrivateError { /* ... */ }

// Enable use of `?` operator.
#[doc(hidden)]
impl From<PrivateError> for PublicError {
    fn from(err: PrivateError) -> PublicError {
        /* ... */
    }
}
```

[`pub(crate)`] is another great tool for removing implementation details from
the public API. It allows items to be used from outside of their own module but
not outside of the same crate.

[`pub(crate)`]: https://github.com/rust-lang/rfcs/blob/master/text/1422-pub-restricted.md
# Predictability


<a id="c-smart-ptr"></a>
## Smart pointers do not add inherent methods (C-SMART-PTR)

For example, this is why the [`Box::into_raw`] function is defined the way it
is.

[`Box::into_raw`]: https://doc.rust-lang.org/std/boxed/struct.Box.html#method.into_raw

```rust
impl<T> Box<T> where T: ?Sized {
    fn into_raw(b: Box<T>) -> *mut T { /* ... */ }
}

let boxed_str: Box<str> = /* ... */;
let ptr = Box::into_raw(boxed_str);
```

If this were defined as an inherent method instead, it would be confusing at the
call site whether the method being called is a method on `Box<T>` or a method on
`T`.

```rust
impl<T> Box<T> where T: ?Sized {
    // Do not do this.
    fn into_raw(self) -> *mut T { /* ... */ }
}

let boxed_str: Box<str> = /* ... */;

// This is a method on str accessed through the smart pointer Deref impl.
boxed_str.chars()

// This is a method on Box<str>...?
boxed_str.into_raw()
```


<a id="c-conv-specific"></a>
## Conversions live on the most specific type involved (C-CONV-SPECIFIC)

When in doubt, prefer `to_`/`as_`/`into_` to `from_`, because they are more
ergonomic to use (and can be chained with other methods).

For many conversions between two types, one of the types is clearly more
"specific": it provides some additional invariant or interpretation that is not
present in the other type. For example, [`str`] is more specific than `&[u8]`,
since it is a UTF-8 encoded sequence of bytes.

[`str`]: https://doc.rust-lang.org/std/primitive.str.html

Conversions should live with the more specific of the involved types. Thus,
`str` provides both the [`as_bytes`] method and the [`from_utf8`] constructor
for converting to and from `&[u8]` values. Besides being intuitive, this
convention avoids polluting concrete types like `&[u8]` with endless conversion
methods.

[`as_bytes`]: https://doc.rust-lang.org/std/primitive.str.html#method.as_bytes
[`from_utf8`]: https://doc.rust-lang.org/std/str/fn.from_utf8.html


<a id="c-method"></a>
## Functions with a clear receiver are methods (C-METHOD)

Prefer

```rust
impl Foo {
    pub fn frob(&self, w: widget) { /* ... */ }
}
```

over

```rust
pub fn frob(foo: &Foo, w: widget) { /* ... */ }
```

for any operation that is clearly associated with a particular type.

Methods have numerous advantages over functions:

* They do not need to be imported or qualified to be used: all you need is a
  value of the appropriate type.
* Their invocation performs autoborrowing (including mutable borrows).
* They make it easy to answer the question "what can I do with a value of type
  `T`" (especially when using rustdoc).
* They provide `self` notation, which is more concise and often more clearly
  conveys ownership distinctions.


<a id="c-no-out"></a>
## Functions do not take out-parameters (C-NO-OUT)

Prefer

```rust
fn foo() -> (Bar, Bar)
```

over

```rust
fn foo(output: &mut Bar) -> Bar
```

for returning multiple `Bar` values.

Compound return types like tuples and structs are efficiently compiled and do
not require heap allocation. If a function needs to return multiple values, it
should do so via one of these types.

The primary exception: sometimes a function is meant to modify data that the
caller already owns, for example to re-use a buffer:

```rust
fn read(&mut self, buf: &mut [u8]) -> io::Result<usize>
```


<a id="c-overload"></a>
## Operator overloads are unsurprising (C-OVERLOAD)

Operators with built in syntax (`*`, `|`, and so on) can be provided for a type
by implementing the traits in [`std::ops`]. These operators come with strong
expectations: implement `Mul` only for an operation that bears some resemblance
to multiplication (and shares the expected properties, e.g. associativity), and
so on for the other traits.

[`std::ops`]: https://doc.rust-lang.org/std/ops/index.html#traits


<a id="c-deref"></a>
## Only smart pointers implement `Deref` and `DerefMut` (C-DEREF)

The `Deref` traits are used implicitly by the compiler in many circumstances,
and interact with method resolution. The relevant rules are designed
specifically to accommodate smart pointers, and so the traits should be used
only for that purpose.

### Examples from the standard library

- [`Box<T>`](https://doc.rust-lang.org/std/boxed/struct.Box.html)
- [`String`](https://doc.rust-lang.org/std/string/struct.String.html) is a smart
  pointer to [`str`](https://doc.rust-lang.org/std/primitive.str.html)
- [`Rc<T>`](https://doc.rust-lang.org/std/rc/struct.Rc.html)
- [`Arc<T>`](https://doc.rust-lang.org/std/sync/struct.Arc.html)
- [`Cow<'a, T>`](https://doc.rust-lang.org/std/borrow/enum.Cow.html)


<a id="c-ctor"></a>
## Constructors are static, inherent methods (C-CTOR)

In Rust, "constructors" are just a convention. There are a variety of
conventions around constructor naming, and the distinctions are often
subtle.

A constructor in its most basic form is a `new` method with no arguments.

```rust
impl<T> Example<T> {
    pub fn new() -> Example<T> { /* ... */ }
}
```

Constructors are static (no `self`) inherent methods for the type that they
construct. Combined with the practice of fully importing type names, this
convention leads to informative but concise construction:

```rust
use example::Example;

// Construct a new Example.
let ex = Example::new();
```

The name `new` should generally be used for the primary method of instantiating
a type. Sometimes it takes no arguments, as in the examples above. Sometimes it
does take arguments, like [`Box::new`] which is passed the value to place in the
`Box`.

Some types' constructors, most notably I/O resource types, use distinct naming
conventions for their constructors, as in [`File::open`], [`Mmap::open`],
[`TcpStream::connect`], and [`UdpSocket::bind`]. In these cases names are chosen
as appropriate for the domain.

Often there are multiple ways to construct a type. It's common in these cases
for secondary constructors to be suffixed `_with_foo`, as in
[`Mmap::open_with_offset`]. If your type has a multiplicity of construction
options though, consider the builder pattern ([C-BUILDER]) instead.

Some constructors are "conversion constructors", methods that create a new type
from an existing value of a different type. These typically have names beginning
with `from_` as in [`std::io::Error::from_raw_os_error`]. Note also though the
`From` trait ([C-CONV-TRAITS]), which is quite similar. There are three
distinctions between a `from_`-prefixed conversion constructor and a `From<T>`
impl.

- A `from_` constructor can be unsafe; a `From` impl cannot. One example of this
  is [`Box::from_raw`].
- A `from_` constructor can accept additional arguments to disambiguate the
  meaning of the source data, as in [`u64::from_str_radix`].
- A `From` impl is only appropriate when the source data type is sufficient to
  determine the encoding of the output data type. When the input is just a bag
  of bits like in [`u64::from_be`] or [`String::from_utf8`], the conversion
  constructor name is able to identify their meaning.

[`Box::from_raw`]: https://doc.rust-lang.org/std/boxed/struct.Box.html#method.from_raw
[`u64::from_str_radix`]: https://doc.rust-lang.org/std/primitive.u64.html#method.from_str_radix
[`u64::from_be`]: https://doc.rust-lang.org/std/primitive.u64.html#method.from_be
[`String::from_utf8`]: https://doc.rust-lang.org/std/string/struct.String.html#method.from_utf8

Note that it is common and expected for types to implement both `Default` and a
`new` constructor. For types that have both, they should have the same behavior.
Either one may be implemented in terms of the other.

[C-BUILDER]: type-safety.html#c-builder
[C-CONV-TRAITS]: interoperability.html#c-conv-traits

### Examples from the standard library

- [`std::io::Error::new`] is the commonly used constructor for an IO error.
- [`std::io::Error::from_raw_os_error`] is a conversion constructor
  based on an error code received from the operating system.
- [`Box::new`] creates a new container type, taking a single argument.
- [`File::open`] opens a file resource.
- [`Mmap::open_with_offset`] opens a memory-mapped file, with additional options.

[`File::open`]: https://doc.rust-lang.org/stable/std/fs/struct.File.html#method.open
[`Mmap::open`]: https://docs.rs/memmap/0.5.2/memmap/struct.Mmap.html#method.open
[`Mmap::open_with_offset`]: https://docs.rs/memmap/0.5.2/memmap/struct.Mmap.html#method.open_with_offset
[`TcpStream::connect`]: https://doc.rust-lang.org/stable/std/net/struct.TcpStream.html#method.connect
[`UdpSocket::bind`]: https://doc.rust-lang.org/stable/std/net/struct.UdpSocket.html#method.bind
[`std::io::Error::new`]: https://doc.rust-lang.org/std/io/struct.Error.html#method.new
[`std::io::Error::from_raw_os_error`]: https://doc.rust-lang.org/std/io/struct.Error.html#method.from_raw_os_error
[`Box::new`]: https://doc.rust-lang.org/stable/std/boxed/struct.Box.html#method.new

# Rust API Guidelines

This is a set of recommendations on how to design and present APIs for the Rust
programming language. They are authored largely by the Rust library team, based
on experiences building the Rust standard library and other crates in the Rust
ecosystem.

These are only guidelines, some more firm than others. In some cases they are
vague and still in development. Rust crate authors should consider them as a set
of important considerations in the development of idiomatic and interoperable
Rust libraries, to use as they see fit. These guidelines should not in any way
be considered a mandate that crate authors must follow, though they may find
that crates that conform well to these guidelines integrate better with the
existing crate ecosystem than those that do not.

This book is organized in two parts: the concise [checklist] of all individual
guidelines, suitable for quick scanning during crate reviews; and topical
chapters containing explanations of the guidelines in detail.

If you are interested in contributing to the API guidelines, check out
[contributing.md] and join our [Gitter channel].

[checklist]: checklist.html
[contributing.md]: https://github.com/rust-lang/api-guidelines/blob/master/CONTRIBUTING.md
[Gitter channel]: https://gitter.im/rust-impl-period/WG-libs-guidelines
# Naming


<a id="c-case"></a>
## Casing conforms to RFC 430 (C-CASE)

Basic Rust naming conventions are described in [RFC 430].

In general, Rust tends to use `UpperCamelCase` for "type-level" constructs (types and
traits) and `snake_case` for "value-level" constructs. More precisely:

| Item | Convention |
| ---- | ---------- |
| Crates | [unclear](https://github.com/rust-lang/api-guidelines/issues/29) |
| Modules | `snake_case` |
| Types | `UpperCamelCase` |
| Traits | `UpperCamelCase` |
| Enum variants | `UpperCamelCase` |
| Functions | `snake_case` |
| Methods | `snake_case` |
| General constructors | `new` or `with_more_details` |
| Conversion constructors | `from_some_other_type` |
| Macros | `snake_case!` |
| Local variables | `snake_case` |
| Statics | `SCREAMING_SNAKE_CASE` |
| Constants | `SCREAMING_SNAKE_CASE` |
| Type parameters | concise `UpperCamelCase`, usually single uppercase letter: `T` |
| Lifetimes | short `lowercase`, usually a single letter: `'a`, `'de`, `'src` |
| Features | [unclear](https://github.com/rust-lang/api-guidelines/issues/101) but see [C-FEATURE] |

In `UpperCamelCase`, acronyms and contractions of compound words count as one word: use `Uuid` rather than `UUID`, `Usize` rather than `USize` or `Stdin` rather than `StdIn`. In `snake_case`, acronyms and contractions are lower-cased: `is_xid_start`.

In `snake_case` or `SCREAMING_SNAKE_CASE`, a "word" should never consist of a
single letter unless it is the last "word". So, we have `btree_map` rather than
`b_tree_map`, but `PI_2` rather than `PI2`.

Crate names should not use `-rs` or `-rust` as a suffix or prefix. Every crate
is Rust! It serves no purpose to remind users of this constantly.

[RFC 430]: https://github.com/rust-lang/rfcs/blob/master/text/0430-finalizing-naming-conventions.md
[C-FEATURE]: #c-feature

### Examples from the standard library

The whole standard library. This guideline should be easy!


<a id="c-conv"></a>
## Ad-hoc conversions follow `as_`, `to_`, `into_` conventions (C-CONV)

Conversions should be provided as methods, with names prefixed as follows:

| Prefix | Cost | Ownership |
| ------ | ---- | --------- |
| `as_` | Free | borrowed -\> borrowed |
| `to_` | Expensive | borrowed -\> borrowed<br>borrowed -\> owned (non-Copy types)<br>owned -\> owned (Copy types) |
| `into_` | Variable | owned -\> owned (non-Copy types) |

For example:

- [`str::as_bytes()`] gives a view of a `str` as a slice of UTF-8 bytes, which
  is free. The input is a borrowed `&str` and the output is a borrowed `&[u8]`.
- [`Path::to_str`] performs an expensive UTF-8 check on the bytes of an
  operating system path. The input and output are both borrowed. It would not be
  correct to call this `as_str` because this method has nontrivial cost at
  runtime.
- [`str::to_lowercase()`] produces the Unicode-correct lowercase equivalent of a
  `str`, which involves iterating through characters of the string and may
  require memory allocation. The input is a borrowed `&str` and the output is an
  owned `String`.
- [`f64::to_radians()`] converts a floating point quantity from degrees to
  radians. The input is `f64`. Passing a reference `&f64` is not warranted
  because `f64` is cheap to copy. Calling the function `into_radians` would be
  misleading because the input is not consumed.
- [`String::into_bytes()`] extracts the underlying `Vec<u8>` of a `String`,
  which is free. It takes ownership of a `String` and returns an owned
  `Vec<u8>`.
- [`BufReader::into_inner()`] takes ownership of a buffered reader and extracts
  out the underlying reader, which is free. Data in the buffer is discarded.
- [`BufWriter::into_inner()`] takes ownership of a buffered writer and extracts
  out the underlying writer, which requires a potentially expensive flush of any
  buffered data.

[`str::as_bytes()`]: https://doc.rust-lang.org/std/primitive.str.html#method.as_bytes
[`Path::to_str`]: https://doc.rust-lang.org/std/path/struct.Path.html#method.to_str
[`str::to_lowercase()`]: https://doc.rust-lang.org/std/primitive.str.html#method.to_lowercase
[`f64::to_radians()`]: https://doc.rust-lang.org/std/primitive.f64.html#method.to_radians
[`String::into_bytes()`]: https://doc.rust-lang.org/std/string/struct.String.html#method.into_bytes
[`BufReader::into_inner()`]: https://doc.rust-lang.org/std/io/struct.BufReader.html#method.into_inner
[`BufWriter::into_inner()`]: https://doc.rust-lang.org/std/io/struct.BufWriter.html#method.into_inner

Conversions prefixed `as_` and `into_` typically _decrease abstraction_, either
exposing a view into the underlying representation (`as`) or deconstructing data
into its underlying representation (`into`). Conversions prefixed `to_`, on the
other hand, typically stay at the same level of abstraction but do some work to
change from one representation to another.

When a type wraps a single value to associate it with higher-level semantics,
access to the wrapped value should be provided by an `into_inner()` method. This
applies to wrappers that provide buffering like [`BufReader`], encoding or
decoding like [`GzDecoder`], atomic access like [`AtomicBool`], or any similar
semantics.

[`BufReader`]: https://doc.rust-lang.org/std/io/struct.BufReader.html#method.into_inner
[`GzDecoder`]: https://docs.rs/flate2/0.2.19/flate2/read/struct.GzDecoder.html#method.into_inner
[`AtomicBool`]: https://doc.rust-lang.org/std/sync/atomic/struct.AtomicBool.html#method.into_inner

If the `mut` qualifier in the name of a conversion method constitutes part of
the return type, it should appear as it would appear in the type. For example
[`Vec::as_mut_slice`] returns a mut slice; it does what it says. This name is
preferred over `as_slice_mut`.

[`Vec::as_mut_slice`]: https://doc.rust-lang.org/std/vec/struct.Vec.html#method.as_mut_slice

```rust
// Return type is a mut slice.
fn as_mut_slice(&mut self) -> &mut [T];
```

##### More examples from the standard library

- [`Result::as_ref`](https://doc.rust-lang.org/std/result/enum.Result.html#method.as_ref)
- [`RefCell::as_ptr`](https://doc.rust-lang.org/std/cell/struct.RefCell.html#method.as_ptr)
- [`slice::to_vec`](https://doc.rust-lang.org/std/primitive.slice.html#method.to_vec)
- [`Option::into_iter`](https://doc.rust-lang.org/std/option/enum.Option.html#method.into_iter)


<a id="c-getter"></a>
## Getter names follow Rust convention (C-GETTER)

With a few exceptions, the `get_` prefix is not used for getters in Rust code.

```rust
pub struct S {
    first: First,
    second: Second,
}

impl S {
    // Not get_first.
    pub fn first(&self) -> &First {
        &self.first
    }

    // Not get_first_mut, get_mut_first, or mut_first.
    pub fn first_mut(&mut self) -> &mut First {
        &mut self.first
    }
}
```

The `get` naming is used only when there is a single and obvious thing that
could reasonably be gotten by a getter. For example [`Cell::get`] accesses the
content of a `Cell`.

[`Cell::get`]: https://doc.rust-lang.org/std/cell/struct.Cell.html#method.get

For getters that do runtime validation such as bounds checking, consider adding
unsafe `_unchecked` variants. Typically those will have the following
signatures.

```rust
fn get(&self, index: K) -> Option<&V>;
fn get_mut(&mut self, index: K) -> Option<&mut V>;
unsafe fn get_unchecked(&self, index: K) -> &V;
unsafe fn get_unchecked_mut(&mut self, index: K) -> &mut V;
```

The difference between getters and conversions ([C-CONV](#c-conv)) can be subtle
and is not always clear-cut. For example [`TempDir::path`] can be understood as
a getter for the filesystem path of the temporary directory, while
[`TempDir::into_path`] is a conversion that transfers responsibility for
deleting the temporary directory to the caller. Since `path` is a getter, it
would not be correct to call it `get_path` or `as_path`.

[`TempDir::path`]: https://docs.rs/tempdir/0.3.5/tempdir/struct.TempDir.html#method.path
[`TempDir::into_path`]: https://docs.rs/tempdir/0.3.5/tempdir/struct.TempDir.html#method.into_path

### Examples from the standard library

- [`std::io::Cursor::get_mut`](https://doc.rust-lang.org/std/io/struct.Cursor.html#method.get_mut)
- [`std::pin::Pin::get_mut`](https://doc.rust-lang.org/std/pin/struct.Pin.html#method.get_mut)
- [`std::sync::PoisonError::get_mut`](https://doc.rust-lang.org/std/sync/struct.PoisonError.html#method.get_mut)
- [`std::sync::atomic::AtomicBool::get_mut`](https://doc.rust-lang.org/std/sync/atomic/struct.AtomicBool.html#method.get_mut)
- [`std::collections::hash_map::OccupiedEntry::get_mut`](https://doc.rust-lang.org/std/collections/hash_map/struct.OccupiedEntry.html#method.get_mut)
- [`<[T]>::get_unchecked`](https://doc.rust-lang.org/std/primitive.slice.html#method.get_unchecked)


<a id="c-iter"></a>
## Methods on collections that produce iterators follow `iter`, `iter_mut`, `into_iter` (C-ITER)

Per [RFC 199].

For a container with elements of type `U`, iterator methods should be named:

```rust
fn iter(&self) -> Iter             // Iter implements Iterator<Item = &U>
fn iter_mut(&mut self) -> IterMut  // IterMut implements Iterator<Item = &mut U>
fn into_iter(self) -> IntoIter     // IntoIter implements Iterator<Item = U>
```

This guideline applies to data structures that are conceptually homogeneous
collections. As a counterexample, the `str` type is slice of bytes that are
guaranteed to be valid UTF-8. This is conceptually more nuanced than a
homogeneous collection so rather than providing the
`iter`/`iter_mut`/`into_iter` group of iterator methods, it provides
[`str::bytes`] to iterate as bytes and [`str::chars`] to iterate as chars.

[`str::bytes`]: https://doc.rust-lang.org/std/primitive.str.html#method.bytes
[`str::chars`]: https://doc.rust-lang.org/std/primitive.str.html#method.chars

This guideline applies to methods only, not functions. For example
[`percent_encode`] from the `url` crate returns an iterator over percent-encoded
string fragments. There would be no clarity to be had by using an
`iter`/`iter_mut`/`into_iter` convention.

[`percent_encode`]: https://docs.rs/url/1.4.0/url/percent_encoding/fn.percent_encode.html
[RFC 199]: https://github.com/rust-lang/rfcs/blob/master/text/0199-ownership-variants.md

### Examples from the standard library

- [`Vec::iter`](https://doc.rust-lang.org/std/vec/struct.Vec.html#method.iter)
- [`Vec::iter_mut`](https://doc.rust-lang.org/std/vec/struct.Vec.html#method.iter_mut)
- [`Vec::into_iter`](https://doc.rust-lang.org/std/vec/struct.Vec.html#method.into_iter)
- [`BTreeMap::iter`](https://doc.rust-lang.org/std/collections/struct.BTreeMap.html#method.iter)
- [`BTreeMap::iter_mut`](https://doc.rust-lang.org/std/collections/struct.BTreeMap.html#method.iter_mut)


<a id="c-iter-ty"></a>
## Iterator type names match the methods that produce them (C-ITER-TY)

A method called `into_iter()` should return a type called `IntoIter` and
similarly for all other methods that return iterators.

This guideline applies chiefly to methods, but often makes sense for functions
as well. For example the [`percent_encode`] function from the `url` crate
returns an iterator type called [`PercentEncode`][PercentEncode-type].

[PercentEncode-type]: https://docs.rs/url/1.4.0/url/percent_encoding/struct.PercentEncode.html

These type names make the most sense when prefixed with their owning module, for
example [`vec::IntoIter`].

[`vec::IntoIter`]: https://doc.rust-lang.org/std/vec/struct.IntoIter.html

### Examples from the standard library

* [`Vec::iter`] returns [`Iter`][slice::Iter]
* [`Vec::iter_mut`] returns [`IterMut`][slice::IterMut]
* [`Vec::into_iter`] returns [`IntoIter`][vec::IntoIter]
* [`BTreeMap::keys`] returns [`Keys`][btree_map::Keys]
* [`BTreeMap::values`] returns [`Values`][btree_map::Values]

[`Vec::iter`]: https://doc.rust-lang.org/std/vec/struct.Vec.html#method.iter
[slice::Iter]: https://doc.rust-lang.org/std/slice/struct.Iter.html
[`Vec::iter_mut`]: https://doc.rust-lang.org/std/vec/struct.Vec.html#method.iter_mut
[slice::IterMut]: https://doc.rust-lang.org/std/slice/struct.IterMut.html
[`Vec::into_iter`]: https://doc.rust-lang.org/std/vec/struct.Vec.html#method.into_iter
[vec::IntoIter]: https://doc.rust-lang.org/std/vec/struct.IntoIter.html
[`BTreeMap::keys`]: https://doc.rust-lang.org/std/collections/struct.BTreeMap.html#method.keys
[btree_map::Keys]: https://doc.rust-lang.org/std/collections/btree_map/struct.Keys.html
[`BTreeMap::values`]: https://doc.rust-lang.org/std/collections/struct.BTreeMap.html#method.values
[btree_map::Values]: https://doc.rust-lang.org/std/collections/btree_map/struct.Values.html


<a id="c-feature"></a>
## Feature names are free of placeholder words (C-FEATURE)

Do not include words in the name of a [Cargo feature] that convey zero meaning,
as in `use-abc` or `with-abc`. Name the feature `abc` directly.

[Cargo feature]: http://doc.crates.io/manifest.html#the-features-section

This arises most commonly for crates that have an optional dependency on the
Rust standard library. The canonical way to do this correctly is:

```toml
# In Cargo.toml

[features]
default = ["std"]
std = []
```

```rust
// In lib.rs
#![no_std]

#[cfg(feature = "std")]
extern crate std;
```

Do not call the feature `use-std` or `with-std` or any creative name that is not
`std`. This naming convention aligns with the naming of implicit features
inferred by Cargo for optional dependencies. Consider crate `x` with optional
dependencies on Serde and on the Rust standard library:

```toml
[package]
name = "x"
version = "0.1.0"

[features]
std = ["serde/std"]

[dependencies]
serde = { version = "1.0", optional = true }
```

When we depend on `x`, we can enable the optional Serde dependency with
`features = ["serde"]`. Similarly we can enable the optional standard library
dependency with `features = ["std"]`. The implicit feature inferred by Cargo for
the optional dependency is called `serde`, not `use-serde` or `with-serde`, so
we like for explicit features to behave the same way.

As a related note, Cargo requires that features are additive so a feature named
negatively like `no-abc` is practically never correct.


<a id="c-word-order"></a>
## Names use a consistent word order (C-WORD-ORDER)

Here are some error types from the standard library:

- [`JoinPathsError`](https://doc.rust-lang.org/std/env/struct.JoinPathsError.html)
- [`ParseBoolError`](https://doc.rust-lang.org/std/str/struct.ParseBoolError.html)
- [`ParseCharError`](https://doc.rust-lang.org/std/char/struct.ParseCharError.html)
- [`ParseFloatError`](https://doc.rust-lang.org/std/num/struct.ParseFloatError.html)
- [`ParseIntError`](https://doc.rust-lang.org/std/num/struct.ParseIntError.html)
- [`RecvTimeoutError`](https://doc.rust-lang.org/std/sync/mpsc/enum.RecvTimeoutError.html)
- [`StripPrefixError`](https://doc.rust-lang.org/std/path/struct.StripPrefixError.html)

All of these use verb-object-error word order. If we were adding an error to
represent an address failing to parse, for consistency we would want to name it
in verb-object-error order like `ParseAddrError` rather than `AddrParseError`.

The particular choice of word order is not important, but pay attention to
consistency within the crate and consistency with similar functionality in the
standard library.
# Type safety


<a id="c-newtype"></a>
## Newtypes provide static distinctions (C-NEWTYPE)

Newtypes can statically distinguish between different interpretations of an
underlying type.

For example, a `f64` value might be used to represent a quantity in miles or in
kilometers. Using newtypes, we can keep track of the intended interpretation:

```rust
struct Miles(pub f64);
struct Kilometers(pub f64);

impl Miles {
    fn to_kilometers(self) -> Kilometers { /* ... */ }
}
impl Kilometers {
    fn to_miles(self) -> Miles { /* ... */ }
}
```

Once we have separated these two types, we can statically ensure that we do not
confuse them. For example, the function

```rust
fn are_we_there_yet(distance_travelled: Miles) -> bool { /* ... */ }
```

cannot accidentally be called with a `Kilometers` value. The compiler will
remind us to perform the conversion, thus averting certain [catastrophic bugs].

[catastrophic bugs]: http://en.wikipedia.org/wiki/Mars_Climate_Orbiter


<a id="c-custom-type"></a>
## Arguments convey meaning through types, not `bool` or `Option` (C-CUSTOM-TYPE)

Prefer

```rust
let w = Widget::new(Small, Round)
```

over

```rust
let w = Widget::new(true, false)
```

Core types like `bool`, `u8` and `Option` have many possible interpretations.

Use a deliberate type (whether enum, struct, or tuple) to convey interpretation
and invariants. In the above example, it is not immediately clear what `true`
and `false` are conveying without looking up the argument names, but `Small` and
`Round` are more suggestive.

Using custom types makes it easier to expand the options later on, for example
by adding an `ExtraLarge` variant.

See the newtype pattern ([C-NEWTYPE]) for a no-cost way to wrap existing types
with a distinguished name.

[C-NEWTYPE]: #c-newtype


<a id="c-bitflag"></a>
## Types for a set of flags are `bitflags`, not enums (C-BITFLAG)

Rust supports `enum` types with explicitly specified discriminants:

```rust
enum Color {
    Red = 0xff0000,
    Green = 0x00ff00,
    Blue = 0x0000ff,
}
```

Custom discriminants are useful when an `enum` type needs to be serialized to an
integer value compatibly with some other system/language. They support
"typesafe" APIs: by taking a `Color`, rather than an integer, a function is
guaranteed to get well-formed inputs, even if it later views those inputs as
integers.

An `enum` allows an API to request exactly one choice from among many. Sometimes
an API's input is instead the presence or absence of a set of flags. In C code,
this is often done by having each flag correspond to a particular bit, allowing
a single integer to represent, say, 32 or 64 flags. Rust's [`bitflags`] crate
provides a typesafe representation of this pattern.

[`bitflags`]: https://github.com/bitflags/bitflags

```rust
use bitflags::bitflags;

bitflags! {
    struct Flags: u32 {
        const FLAG_A = 0b00000001;
        const FLAG_B = 0b00000010;
        const FLAG_C = 0b00000100;
    }
}

fn f(settings: Flags) {
    if settings.contains(Flags::FLAG_A) {
        println!("doing thing A");
    }
    if settings.contains(Flags::FLAG_B) {
        println!("doing thing B");
    }
    if settings.contains(Flags::FLAG_C) {
        println!("doing thing C");
    }
}

fn main() {
    f(Flags::FLAG_A | Flags::FLAG_C);
}
```


<a id="c-builder"></a>
## Builders enable construction of complex values (C-BUILDER)

Some data structures are complicated to construct, due to their construction
needing:

* a large number of inputs
* compound data (e.g. slices)
* optional configuration data
* choice between several flavors

which can easily lead to a large number of distinct constructors with many
arguments each.

If `T` is such a data structure, consider introducing a `T` _builder_:

1. Introduce a separate data type `TBuilder` for incrementally configuring a `T`
   value. When possible, choose a better name: e.g. [`Command`] is the builder
   for a [child process], [`Url`] can be created from a [`ParseOptions`].
2. The builder constructor should take as parameters only the data _required_ to
   make a `T`.
3. The builder should offer a suite of convenient methods for configuration,
   including setting up compound inputs (like slices) incrementally. These
   methods should return `self` to allow chaining.
4. The builder should provide one or more "_terminal_" methods for actually
   building a `T`.

[`Command`]: https://doc.rust-lang.org/std/process/struct.Command.html
[child process]: https://doc.rust-lang.org/std/process/struct.Child.html
[`Url`]: https://docs.rs/url/1.4.0/url/struct.Url.html
[`ParseOptions`]: https://docs.rs/url/1.4.0/url/struct.ParseOptions.html

The builder pattern is especially appropriate when building a `T` involves side
effects, such as spawning a task or launching a process.

In Rust, there are two variants of the builder pattern, differing in the
treatment of ownership, as described below.

### Non-consuming builders (preferred)

In some cases, constructing the final `T` does not require the builder itself to
be consumed. The following variant on [`std::process::Command`] is one example:

[`std::process::Command`]: https://doc.rust-lang.org/std/process/struct.Command.html

```rust
// NOTE: the actual Command API does not use owned Strings;
// this is a simplified version.

pub struct Command {
    program: String,
    args: Vec<String>,
    cwd: Option<String>,
    // etc
}

impl Command {
    pub fn new(program: String) -> Command {
        Command {
            program: program,
            args: Vec::new(),
            cwd: None,
        }
    }

    /// Add an argument to pass to the program.
    pub fn arg(&mut self, arg: String) -> &mut Command {
        self.args.push(arg);
        self
    }

    /// Add multiple arguments to pass to the program.
    pub fn args(&mut self, args: &[String]) -> &mut Command {
        self.args.extend_from_slice(args);
        self
    }

    /// Set the working directory for the child process.
    pub fn current_dir(&mut self, dir: String) -> &mut Command {
        self.cwd = Some(dir);
        self
    }

    /// Executes the command as a child process, which is returned.
    pub fn spawn(&self) -> io::Result<Child> {
        /* ... */
    }
}
```

Note that the `spawn` method, which actually uses the builder configuration to
spawn a process, takes the builder by shared reference. This is possible because
spawning the process does not require ownership of the configuration data.

Because the terminal `spawn` method only needs a reference, the configuration
methods take and return a mutable borrow of `self`.

#### The benefit

By using borrows throughout, `Command` can be used conveniently for both
one-liner and more complex constructions:

```rust
// One-liners
Command::new("/bin/cat").arg("file.txt").spawn();

// Complex configuration
let mut cmd = Command::new("/bin/ls");
if size_sorted {
    cmd.arg("-S");
}
cmd.arg(".");
cmd.spawn();
```

### Consuming builders

Sometimes builders must transfer ownership when constructing the final type `T`,
meaning that the terminal methods must take `self` rather than `&self`.

```rust
impl TaskBuilder {
    /// Name the task-to-be.
    pub fn named(mut self, name: String) -> TaskBuilder {
        self.name = Some(name);
        self
    }

    /// Redirect task-local stdout.
    pub fn stdout(mut self, stdout: Box<io::Write + Send>) -> TaskBuilder {
        self.stdout = Some(stdout);
        self
    }

    /// Creates and executes a new child task.
    pub fn spawn<F>(self, f: F) where F: FnOnce() + Send {
        /* ... */
    }
}
```

Here, the `stdout` configuration involves passing ownership of an `io::Write`,
which must be transferred to the task upon construction (in `spawn`).

When the terminal methods of the builder require ownership, there is a basic
tradeoff:

* If the other builder methods take/return a mutable borrow, the complex
  configuration case will work well, but one-liner configuration becomes
  impossible.

* If the other builder methods take/return an owned `self`, one-liners continue
  to work well but complex configuration is less convenient.

Under the rubric of making easy things easy and hard things possible, all
builder methods for a consuming builder should take and return an owned
`self`. Then client code works as follows:

```rust
// One-liners
TaskBuilder::new("my_task").spawn(|| { /* ... */ });

// Complex configuration
let mut task = TaskBuilder::new();
task = task.named("my_task_2"); // must re-assign to retain ownership
if reroute {
    task = task.stdout(mywriter);
}
task.spawn(|| { /* ... */ });
```

One-liners work as before, because ownership is threaded through each of the
builder methods until being consumed by `spawn`. Complex configuration, however,
is more verbose: it requires re-assigning the builder at each step.
# Necessities


<a id="c-stable"></a>
## Public dependencies of a stable crate are stable (C-STABLE)

A crate cannot be stable (>=1.0.0) without all of its public dependencies being
stable.

Public dependencies are crates from which types are used in the public API of
the current crate.

```rust
pub fn do_my_thing(arg: other_crate::TheirThing) { /* ... */ }
```

A crate containing this function cannot be stable unless `other_crate` is also
stable.

Be careful because public dependencies can sneak in at unexpected places.

```rust
pub struct Error {
    private: ErrorImpl,
}

enum ErrorImpl {
    Io(io::Error),
    // Should be okay even if other_crate isn't
    // stable, because ErrorImpl is private.
    Dep(other_crate::Error),
}

// Oh no! This puts other_crate into the public API
// of the current crate.
impl From<other_crate::Error> for Error {
    fn from(err: other_crate::Error) -> Self {
        Error { private: ErrorImpl::Dep(err) }
    }
}
```


<a id="c-permissive"></a>
## Crate and its dependencies have a permissive license (C-PERMISSIVE)

The software produced by the Rust project is dual-licensed, under either the
[MIT] or [Apache 2.0] licenses. Crates that simply need the maximum
compatibility with the Rust ecosystem are recommended to do the same, in the
manner described herein. Other options are described below.

These API guidelines do not provide a detailed explanation of Rust's license,
but there is a small amount said in the [Rust FAQ]. These guidelines are
concerned with matters of interoperability with Rust, and are not comprehensive
over licensing options.

[MIT]: https://github.com/rust-lang/rust/blob/master/LICENSE-MIT
[Apache 2.0]: https://github.com/rust-lang/rust/blob/master/LICENSE-APACHE
[Rust FAQ]: https://github.com/dtolnay/rust-faq#why-a-dual-mitasl2-license

To apply the Rust license to your project, define the `license` field in your
`Cargo.toml` as:

```toml
[package]
name = "..."
version = "..."
authors = ["..."]
license = "MIT OR Apache-2.0"
```

Then add the files `LICENSE-APACHE` and `LICENSE-MIT` in the repository root,
containing the text of the licenses (which you can obtain, for instance, from
choosealicense.com, for [Apache-2.0](https://choosealicense.com/licenses/apache-2.0/)
and [MIT](https://choosealicense.com/licenses/mit/)).

And toward the end of your README.md:

```
## License

Licensed under either of

 * Apache License, Version 2.0
   ([LICENSE-APACHE](LICENSE-APACHE) or <http://www.apache.org/licenses/LICENSE-2.0>)
 * MIT license
   ([LICENSE-MIT](LICENSE-MIT) or <http://opensource.org/licenses/MIT>)

at your option.

## Contribution

Unless you explicitly state otherwise, any contribution intentionally submitted
for inclusion in the work by you, as defined in the Apache-2.0 license, shall be
dual licensed as above, without any additional terms or conditions.
```

Besides the dual MIT/Apache-2.0 license, another common licensing approach used
by Rust crate authors is to apply a single permissive license such as MIT or
BSD. This license scheme is also entirely compatible with Rust's, because it
imposes the minimal restrictions of Rust's MIT license.

Crates that desire perfect license compatibility with Rust are not recommended
to choose only the Apache license. The Apache license, though it is a permissive
license, imposes restrictions beyond the MIT and BSD licenses that can
discourage or prevent their use in some scenarios, so Apache-only software
cannot be used in some situations where most of the Rust runtime stack can.

The license of a crate's dependencies can affect the restrictions on
distribution of the crate itself, so a permissively-licensed crate should
generally only depend on permissively-licensed crates.
