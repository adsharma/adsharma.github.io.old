## Fquery Improvements

A few months ago, I published some code as open source with minimal unit tests and without much of an announcement. This is an attempt to describe what the code does with some pointers to other similar projects.

### Similar projects

You may be familiar with projects like [Kotlin's Async Flow](https://kotlinlang.org/docs/reference/coroutines/flow.html) or [Typegraphql](http://typegraphql.com).

These projects present an object graph as asynchrounous generators with a friendly UI. All you do is a mostly framework free plain-old objects with some decorators thrown in. Typegraphql uses `@Entity`, `@Field` decorators for example.

Like Kotlin async flow, this package implements a bunch of operators, which can be combined with python dataclasses and some decorators to express queries. The generators themselves are lazy/cold. No work is done unless requested. Like Kotlin's `collect`, we have `to_json()` and `as_dict()` as terminal operators.


### What Does fquery do?

fquery is built on top of `aioitertool` and python's async generators. It provides a few intermediate and terminal operators so you can create a type safe query builder in python. It provides a few decorators that you can use to annotate your dataclasses. Lastly, it can serialize the query to SQL (experimental feature) or you can serialize to an expression that can be transported on the wire to a backend for remote execution.

### Specifying objects

```
@dataclass
class MockUser(ViewModel):
    name: str
    age: int

    @edge
    async def friends(self) -> List["MockUser"]:
        yield [MockUser.get(m) for m in range(3 * self.id, 3 * self.id + 3)]

    @edge
    async def reviews(self) -> List["MockReview"]:
        yield [
            MockReview.get(m) for m in range(3 * self.id + 300, 3 * self.id + 300 + 5)
        ]

    @staticmethod
    def get(id: int) -> "MockUser":
        ...

@dataclass
class MockReview(ViewModel):
    business: str
    rating: int
    author: MockUser

    @edge
    async def author(self) -> MockUser:
        yield self.author

    @staticmethod
    def get(id: int) -> "MockReview":
        ...
```

Super simple isn't it? For each of the types, you'll need to implement a `Resolver` object. These objects are typically named as `MockUserQuery` or `MockReviewQuery` and provide a `resolve_obj()` method. This way, all the details of how to fetch an object given it's ID and how to resolve an edge is abstracted out from the actual query execution mechanism.


### Internals

To understand the internals, you can look at `walk.py`. `materialize_walk()` and `materialize_walk_obj()` are key interfaces. They're described by included unit tests.

### External Interface

The supported operators are documented in `test_operators.py`. Sample query:

```
    @async_test
    async def test_edge_project_other_kind(self):
        resp = (
            await UserQuery(range(1, 5))
            .edge("reviews")
            .project(["business", "rating", ":id"])
            .take(3)
            .to_json()
        )
        ...
```

### Future Improvements

- The `MockUser` class inherits from `ViewModel` class. We can move that into decorators. 
- Support more complex SQL queries
- Serialization to other like minded query languages
- Support for `graphql` on top of this infra
- Tighter integration into Django and Flask
