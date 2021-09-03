
## Tutorial for using django + fquery

[fquery](https://github.com/adsharma/fquery) is primarily a graph query engine. It can also be thought of as an explicit query plan for declarative query languages such as SQL or opencypher. In that sense it's multi-model (can support relational, document or graph) and can be enhanced with additional operators as needed.

In this tutorial we'll describe how to use fquery with django and create a graphql interface.

## Django Polls App

Start with the standard django [tutorial](https://docs.djangoproject.com/en/3.2/intro/tutorial01/) and once you have finished the tutorial and created the polls app, move on to the [next page](https://docs.djangoproject.com/en/3.2/intro/tutorial02/). This is where we do things differently.

The tutorial tells you to do this:

```
from django.db import models


class Question(models.Model):
    question_text = models.CharField(max_length=200)
    pub_date = models.DateTimeField('date published')


class Choice(models.Model):
    question = models.ForeignKey(Question, on_delete=models.CASCADE)
    choice_text = models.CharField(max_length=200)
    votes = models.IntegerField(default=0)
```

But instead with fquery, you'd write the same model as:

```
from datetime import datetime
from fquery.django import model
from dataclasses import dataclass
from fquery.query import query
from fquery.view_model import edge, node


@model
@dataclass
@node
class Question:
    question_text: str
    pub_date: datetime


@model
@dataclass
@node
class Choice:
    choice_text: str
    votes: int

    @edge
    async def question(self) -> Question:
        return self._question

@query
class QuestionQuery:
    TYPE = Question

@query
class ChoiceQuery:
    TYPE = Choice

```              

run the migrations and follow the rest of the tutorial. If something doesn't work as before, please file an issue with the github repo linked.

### Pros and Cons

Pros: you write strongly typed, compact python dataclasses. Rich data types like date/time and uuids are supported. Things like `duration` and `URL` can be added trivially to bring a safe and familiar programming model to the DDL.

Cons: you lose some low level control over how fields are represented in the database. This can be added back later in the form of dataclass field metadata and declaratively mapped to the underlying database.

In addition to what's in the tutorial, you get access to a simpler query syntax that encapsulates the underlying relational model.

Compare:

Django:

```
Choice.objects.filter(question__pub_date__year=current_year)
```

fquery:

```
ChoiceQuery.edge("question").where(pub_date=current_year)
```

## Using graphql

Much of the hard work here is done by the strawberry-graphql project. If it so happens that your domain objects and the database schema map one to one, you can simply add the `@graphql` decorator on the dataclass and generate a graphql type object.

```
# polls/graphql_question_query.py
from fquery.fgraphql import obj, field, graphql, root
from polls import Question, Choice, QuestionQuery, ChoiceQuery

@obj
class GraphQLQuestion(Question):
    @field
    async def choices(self) -> List["GraphQLChoice"]:
        ...

@obj
class GraphQLChoice(Choice):
    @field
    async def question(self) -> List["GraphQLQuestion"]:
        ...

@root
@graphql
class GraphQLQuestionQuery:
    @field
    def question(id: int) -> GraphQLMockQuestion:
        return QuestionQuery.TYPE.get(id)

@graphql
class GraphQLChoiceQuery:
    @field
    def choice(id: int) -> GraphQLMockChoice:
        return ChoiceQuery.TYPE.get(id)
```

Install strawberry graphql

```
pip3 install strawberry-graphql --user
```

and run it

```
PYTHONPATH=. ~/.local/bin/strawberry server polls.graphql_question_query
```

Navigate to: http://localhost:8000/ and run some queries. Enjoy!
### More complex mappings

What if your view model doesn't map 1:1 to the database model? This is a fallacy a lot of the "expose your database as a graphql" solutions fall prey to. In those cases, you'd write a view model and database model separately and use async methods to perform the mapping. In the future we could make it more declarative.

```
@model
@dataclass
class Question:
    question_text: str
    pub_date: datetime

@node
@dataclass
class QuestionView(Question):
    question_text: str
    pub_date: datetime

    async def location(self):
        # some logic to fetch location from somewhere else

```

The use of inheritance to auto-map a few fields to the database could be problematic. There are other potential [schemes](https://github.com/adsharma/flattools/commit/486e8bbfeceec55e5b8dc904474488585805a240) to alleviate it.


### Pros and Cons

Pros: You get nested query results which is something Django's query model doesn't support. When writing views, you need to make a bunch of flat queries and manually convert the result to a nested data structure. Here things fall into place more naturally.

Cons: there is a bit of an implementation problem - there is more repetition than what is strictly necessary. Perhaps `GraphQLQuestion` and `GraphQLQuestionQuery` can be auto-generated via decorator based meta-programming.

## Future Enhancements

* An edge at a time query engine backing up most python graphql implementations including strawberry-graphql can be enhanced with whole query optimization modes. In other words, an entire graphql can be synthesized into a `fquery` which can be mapped back to SQL or another language the storage engine supports.

* `fquery` can be serialized efficiently into a network representation. Zero-copy is a major concern.

* Incremental/streaming mode of execution

Please leave any comments/feedback [here](https://github.com/adsharma/fquery/discussions/)
