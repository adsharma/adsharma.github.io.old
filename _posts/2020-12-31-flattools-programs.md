---
comments:
  host: fosstodon.org
  username: seeker
  id: 105477542909151026
---

## Algorithms + Data Structures = Programs

So said Niklaus Wirth in 1976. But there is a debate 40+ years later about how important
each of them is for the most common use cases.

On one end of the spectrum are people claiming that the most
important thing is the database (the idea being the relational
model is the one true data structure). Once you get it right,
that's 90% of the program. The rest of it is an exercise left to the
student. On the other end of the spectrum is the view that software
at it's core is procedural and layers of object oriented abstraction
are obscuring the logic. Jim Kelly, the VP of R&D at quantcast has a
[blog](https://www.quantcast.com/blog/death-by-1000-layers-the-perils-of-over-abstraction-in-java/)
on the topic.

In this post, we explore this topic in a bit more detail. Is it 90-10, 50-50 or 10-90?

## How do you write software?

Before we quantify where you should invest your efforts, perhaps its worth walking through the
software development process. Do you write your algorithms first or data structures first?

Back in the 90s, there was the Entity-Relationship diagram (ER diagram)
and UML (Unified modeling language) diagram and people used to debate which one
was more important. ER diagram primarily described the database schema and UML
was about "business logic". The feelings were so fierce that unless you
got rid of the other one, your preferred one wouldn't survive.

Then came Java/C#, Object relational mappers (ORMs) and the
disillusionment with the complexity. The backlash took two forms: NoSQL
on the database side and anti-Object Oriented programming campaign in
the programming language community.

## No Code, Low Code

An interesting twist in the programming landscape is the trend towards visual programming. The idea
is that someone without a programming background can describe their business processes visually using
packaged concepts (like `Customer` and `Order`) and with minimal customization describe their business
processes and get the job done.

When you look under the hood how these things are working, you hear terms like `Ontology` and `RDF`.

## Overly complex technologies

Much of this happened in the mid-90s where there was effort to build the "semantic web" using something
called `RDF`. This has lead to a slew of technologies (`JSON-LD`, triples, quads) and languages like `SPARQL`.

While there is nothing wrong with technologies per-se, they are not easy to read, understand and query
by inexperienced programmers.

These same programers are much happier to use JSON, Protocol Buffers and simple, ad-hoc SQL.

More recent trends in programming - ActivityStream 2.0, a standardization effort to create interoperable
social networks (`Fediverse`) have suffered because no one can read the specs and
understand what they're doing.

Telegram, a popular chat app uses something that `TL Language` to describe the types. It takes someone a
good 2-3 days to understand the language before they can grok concepts like threads and messages.

Thankfully, there are tools to convert these complex technologies into simple ones like [flatbuffers IDL](
https://adsharma.github.io/flattools/) and the difference is stark.

1. [Telegram TL Schema](https://raw.githubusercontent.com/adsharma/tdlib.fbs/45d205684f1f19476762babc4e0758ee81983387/telegram_api.tl)
2. [Translated Flatbuffer Schema](https://raw.githubusercontent.com/adsharma/tdlib.fbs/45d205684f1f19476762babc4e0758ee81983387/core.fbs)

## What is Business Logic?

A recent [thread](https://news.ycombinator.com/item?id=25594323) about nim programming language
had a user describe this exchange on a chat thread:

```
<Facebook F8 tech talk about Flux>

Araq: meh, I cannot watch these hipsters talk

Araq: as soon as I hear “business logic” I stop listening
```

and how that lead to a negative experience and their decision not to use the language.

So why is this term so controversial? What is business logic anyway?

The reason why it's controversial is that writing any app by hand using low level primitives
involves decoding storage data, network protocols, shuffling bits and bytes and intefacing
between modules written by other engineers in a large project. Some people think this is business
logic, because they're coding in a certain layer of the stack. I tend
to think of it as just plumbing, not business logic.

Business logic can be thought of as code a product manager or someone without a software background would
write if they are given the ideal no-code platform. It truly describes the characteristics of a program as
perceived by the end user and enforces certain policies (Govt regulation or company commitments about data
usage among other things).

Ideally, engineers spend a majority of their time writing the business
logic and a small fraction of the time dealing with plumbing and
frameworks. But in practice, I've seen the opposite.

## Can Functional programming and Object Oriented (OO) Programming Coexist?

Languages such as Julia and Rust removed OO syntax completely from
the language. Another pioneering language OCaml calls itself an
object-oriented, imperative, functional programming language with
a smiley.

Being a moderate, I tend to be in the OCaml camp. Use OO-style for
encapsulation. Use inheritance very very carefully and write side-effect
free functions when possible.

## Can Types and SQL coexist?

They make excellent friends! There are many programming language technologies where the language ecosystem
implements a semantic SQL parser to make sure that the generated data (usually a Tuple/Row) is semantically
compatibile with the type system. Android Rooms and Nim's Ormin are two such efforts.

This allows programmers to exercise control over data fetching, caching policies by using raw SQL and without
sacrificing type safety, crucial to secure programming.

## So what is it?

In summary, I think getting your high level types right and then mapping them to storage/database
types is 70% of the effort. The other 30% involves writing the actual procedural logic. This is
just a ballpark figure. For a particular app it may be 90-10 or 80-20.

Once you have the high level and storage types figured out, run them through a code generator such as
flattools (supports 4 popular type safe languages!) and write your business logic in terms of those types.

So if this is actually a thing, doesn't it make sense to create a repository of high level types
without using complex technologies? That is the idea behind [this github meta repo](http://github.com/adsharma/fbs-schemas).
