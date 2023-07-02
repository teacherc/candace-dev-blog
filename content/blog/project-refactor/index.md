---
title: How I Would Refactor the Project That Got Me My First Software Engineering Role
date: "2023-07-03"
description: This is a blog post about how I would refactor my first public Python project to improve its robustness.
---

# What is Spheri?

Four days after being laid off in October 2022, I decided to teach myself how to
code. As a former K-12 educator, I knew that the best way for me to learn would
be to jump into a project right away. After reading the first few chapters of
[Automate the Boring Stuff with Python](https://automatetheboringstuff.com/), I came up with the idea to create Python code that took in a user's zipcode and returned the weather paired
with a Spotify song recommendation.

# What went well?

## I got a job!

When I first started to code, I only told my wife, my best friend, and Andrew Lee (the guy who
encouraged me to start coding) about my goals. After I had a published project,
it was easy for me to slap together a resume. I posted about my journey on
LinkedIn. From that post, I got a few introductions that lead to interviews. I
applied to a Product Management role at Inferex on Wellfound. When I got to the
interview, Greg, the CEO, asked me about my coding projects and what I really
wanted to do at Inferex. I floated the idea of a dual-PM + Jr Software Eng role.
Greg ended up hiring me to be a Junior Software Engineer. If I hadn't of had
this proof of work on GitHub, I could have never gotten this role or the other
interviews.

## I learned a lot in a short amount of time

Going from `print("Hello, Candace!)` to a deployed web app was a major leap for
me. I started working on the project in the first week of November, and posted
it to r/Python on December 2, 2022. When I started the project, I thought I would just create a .p
file. Then, I realized that to share this idea, I needed to put it on the web. That lead to me learning about web frameworks (I chose Flask), routing and HTTP requests, containerization via
Docker, and cloud deployments. 

## The r/Python community gave me excellent feedback and support

After I posted about the app on r/Python (it got 500+ upvotes!), I created a
project in the repo and made an issue card for each piece of feedback. Over
time, I refactored the app. For example, I changed my deployment from Google App
Engine to Docker + Google Cloud Run. I improved things like naming, and tried to
air out some of the code smells.

## Documentation

My README, [release documentation](), and project board not only helped me keep track of
the context of my work, but helped me tell the story of the project, build
community, and solicit feedback.

# What didn't go well?

A lot! It takes a long time for our skills to catch up with our taste. I knew a
lot of things could be better about my project. When I look at the code now, I
feel equal parts pride and "yikes!" 

I thought about refactoring Spheri, but my interests have evolved. I'm obsessed with developer experience and CLIs. Also, I work in the ML/AI space and need to stay up on advancements in the field. Right now, I'm working on a CLI that leverages LangChain and ChatGPT to help people who are learning how to code.

When I started this new project, I spent a week domain modeling and designing. It was useful to look back at Spheri and make a list of things I would do better in this new project. The rest of this blog post is that list. My understanding of Python, computer science, system design, and domain modeling is increasing by the hour. I'll probably look back at this blog post in semi-shame in a few months. 

## Then: Spaghetti dinner. Now: Objects, layered architecture, and dependency-injector.

### Then 
Almost all of the code, from Flask routes to business logic and some of the
presentation code, is in
[app/main.py](https://github.com/teacherc/spheri-app/blob/main/app/main.py).

Spheri tree:
```bash
.
├── .dockerignore
├── .env
├── .git
├── .github
│   └── workflows
├── .gitignore
├── Dockerfile
├── LICENSE
├── README.md
├── app
│   ├── __init__.py
│   ├── constants.py
│   ├── main.py
│   ├── static
│   └── templates
├── assets
│   └── images
├── cloudbuild.yaml
├── docker-compose.yml
├── env.sample
├── lib
├── requirements.txt
├── tests
│   └── test_main.py
└── venv
```

There is no separation of concerns and a lot of spaghetti code. If I wanted to change the APIs I used or change to a different framework, I would have to refactor most of the code instead of just dropping in the new elements I needed and deleting the old information.

### Now

Before I write a line of code, I make a series [C4 Domain Diagrams](https://c4model.com/) so I have a clear idea of the domain. Domain design helps me construct loosely coupled, highly-cohesive objects. I am still experimenting with different design patterns, but I tend to start with a layered architecture and go from there. If I refactored Spheri, there would be presentation, business, service, and persistence layers.

The tree for my new CLI project looks like this:


```bash
.
├── __init__.py
├── __main__.py
├── cli
│   ├── __init__.py
│   ├── app.py
│   └── command
│       ├── __init__.py
│       ├── assess.py
│       └── load.py
├── config.py
├── containers.py
├── errors.py
├── model
│   ├── __init__.py
│   ├── chat_history.py
│   ├── database.py
│   ├── embedding.py
│   ├── jsonschema.py
│   ├── llmschema.py
│   ├── output.py
│   ├── prompt_dto.py
│   └── repo.py
├── persistence
│   ├── __init__.py
│   └── database.db
├── prompts
│   ├── annotations.py
│   ├── cicd.py
│   ├── code_docs.py
│   ├── oop.py
│   ├── project_docs.py
│   ├── shared.py
│   └── testing.py
├── service
│   ├── __init__.py
│   ├── embedding.py
│   ├── inputhandler.py
│   ├── llmapihandler.py
│   ├── openai_worker.py
│   ├── outputparsing.py
│   ├── project.py
│   ├── repo.py
│   ├── terminal.py
│   └── validation.py
└── util
    ├── __init__.py
    ├── logging.py
    └── tmpcontext.py
```


## Then: Tuples all the way down. Now: Validation via Pydantic.

### Then

Here is the assign_valence function from my main.py:
```python
def assign_valence(weather_data):
    # This function uses the 'weather_code' to set the 'valence' parameter
    from constants import WEATHERSTACK_CODES, SPOTIFY_VALENCE
    weather_code = int(weather_data["weather"]["current"]["weather_code"])
    valence = [SPOTIFY_VALENCE[k] for k, v in WEATHERSTACK_CODES.items() if weather_code in v][0]
    min_valence = valence[0]
    max_valence = valence[-1]

    return min_valence, max_valence
```

Notice how I'm parsing dictionaries and returning tuples. Using Pydantic would
help me avoid key errors because I could validate data. Also, creating custom
data objects allows me to handle types more easily in my code.

### Now
This is an example Pydantic model from my new project. It's helps me handle
documents and split texts that I will use in the embedding process:


```python
from typing import Optional, List, Iterable
from pydantic import BaseModel
from langchain.schema import Document


class DocumentModel(BaseModel):
    docs: Optional[Iterable[Document]] = None
    split_texts: Optional[List[Document]] = None

```

## Then: Limited error handling. Now: Experimenting with returned error types and structural pattern matching.

### Then

I'm glad I took the time to validate the zipcode my user enters in the app:

```python
def validate_zipcode(zipcode):
    return bool(re.search(r"^[0-9]{5}(-[0-9]{4})?$", zipcode))

# Later 

@app.route("/")
def index():
    zipcode = request.args.get("zipcode", "")
    min_valence = ""
    max_valence = ""
    weather_genre = ""
    token = []
    final_recommendation = {}
    spotify_data = {}

    if zipcode:
        from constants import ERROR_MESSAGES
        if not validate_zipcode(zipcode):
            return render_template(
                "error.html", error_message=str(ERROR_MESSAGES["INVALID_ZIPCODE"])
            )
```
Noble attempt, but it's clear this approach is lacking.

### Now 
Now that I understand how to separate my concerns and make custom error objects,
there are better ways to handle my errors. I know that I can raise or return
errors. In most code I write now, I annotate my methods to return specific error
types. I know that some consider it more Pythonic to raise errors, but for now
this approach helps me account for different error situations ahead of time. I
also have more freedom at the application level to decide how errors are
handled. Since I like to design user-facing products, this also helps me tailor
the error messages with a reason, suggestion, and other pertinent information.

Here is an excerpt of my errors.py:

```python
from typing import Optional
from pydantic import BaseModel


class PblError(BaseModel):
    reason: str
    suggestion: Optional[str] = None
    cause: Optional["PblError"] = None


class IndexingError(PblError):
    """Error occurred while indexing git repo documents."""


class EmbeddingError(PblError):
    """Could not embed your repo files in the vector store."""


class DatabaseError(PblError):
    """An error occurred while committing data to the database."""


class EnvError(PblError):
    """Valid .env not found."""

```
Here's an example of how I handle errors in my Click functions (this is
unfinished code - I need to have it parse out the reason and other info):

```python
    with console.status(Spinner("aesthetic", text="[bold blue] Loading your project.")):
        if path:
            repo_service.set_path(path)
            maybe_embedded_repo = embedding_service.embedded_files(name)
            if isinstance(
                maybe_embedded_repo, (RepoError, RootError, IndexingError, LoadError)
            ):
                error = maybe_embedded_repo
            else:
                message = f"Project loaded successfully. Run 'pbl assess {name}' next."

        elif url:
            maybe_load_url = embedding_service.load_project_url_to_db(name)
            if isinstance(maybe_load_url, DatabaseError):
                error = maybe_load_url
            else:
                message = f"URL: {url} and project name {name} have been loaded. Run 'pbl assess {name}'."

    if error:
        console.print(error)
    elif message:
        console.print(message)

    return

```

## Then: No testing. Now: Pytest + TDD-lite.

### Then

You'll notice there is a lot wrong with these tests. You can't have multiple
tests with the same name. The True/False arguments are unnecessary. Since my
code is sphagetti, it is impossible to isolate the behavior
of specific functions or methods from the behavior of other pieces of code. It's
also telling that I wrote these right before I deployed my code for the first
time.

from 'spheri-app/tests/test_main.py'

```python
import unittest
from app.main import main

class TestMain(unittest.TestCase):
#Test validate_zipcode
    def test_validate_zipcode(self):
        result = main.validate_zipcode("98023")
        self.assertTrue(result, True)
    
    def test_validate_zipcode(self):
        result = main.validate_zipcode("cat")
        self.assertFalse(result, False) # Result should be true
```

### Now
 
I plan my testing strategy before I start a project. For my CLI project, I
decided to chunk my work by useable functionality, so that I could write an
integration test before I writing any code. This integration test models what the
end state would look like and how the pieces would fit together.

In a perfect world, I would also do TDD by the book. Sometimes, I use red-green-refactor.
Other work is nonlinear, so it is harder to fit the tests into this model. No
matter what, I make sure I understand what the input types and output types are
for each function or method and write type annotations in the
method signature before I write anything else. This means that I have an
explicit understanding of what each method must accomplish before I start
writing code.

Since I'm using dependency-injector and have a very good idea of what each
method should return (in terms of types), I'm able to use Magic Mocks to mock
the behavior of my services.

Here is the test tree for my current project. As of now, I have a test for each
service, as well as integration tests for each piece of grouped functionality.

```bash
.
├── __init__.py
├── integration
│   ├── test_commands.py
│   ├── test_database.db
│   └── test_project_db.py
└── unit
    ├── test_embedding_service.py
    ├── test_env_file.py
    ├── test_input_handler.py
    ├── test_llmapihandler.py
    ├── test_project_service.py
    └── test_reposervice.py
```

Example unit test:
```python
def test_embedded_files_returns_indexing_error_on_indexed_files_failure(
    service_wrapper,
):
    # Arrange
    embedding_service = service_wrapper.embedding_service

    # Simulate that repo_dir and loaded_files return None, and indexed_files returns IndexingError
    embedding_service.repo_dir = MagicMock(return_value=None)
    embedding_service.loaded_files = MagicMock(return_value=None)
    embedding_service.indexed_files = MagicMock(
        return_value=IndexingError(reason="Indexing error")
    )

    # Act
    result = embedding_service.embedded_files("project_name")

    # Assert
    assert isinstance(result, IndexingError)
```

After reading [PEP 634](https://peps.python.org/pep-0634/) and [PEP 636](https://peps.python.org/pep-0636/), I'm also experimenting with structural pattern matching for my returned errors. 

### Resource
[Growing Object-Oriented Software, Guided by Tests](https://learning.oreilly.com/library/view/growing-object-oriented-software/9780321574442/) by Steve Freeman and Nat Pryce

## Then: What are type annotations? Now: Annotations everywhere.

### Then

If someone wanted to extend this project, they would have to parse the (terrible) code in
the function itself to figure out what the inputs and outputs are. 

```python
def assign_recommendations(spotify_data, weather_data):
    # This function stores the recommendation in variables
    artist_name = spotify_data["spotify_response"]["tracks"][spotify_data["random_song"]]["artists"][0]["name"]
    name_of_song = spotify_data["spotify_response"]["tracks"][spotify_data["random_song"]]["name"]
    web_address = spotify_data["spotify_response"]["tracks"][spotify_data["random_song"]]["external_urls"]["spotify"]
    album_cover_image = spotify_data["spotify_response"]["tracks"][spotify_data["random_song"]]["album"]["images"][0]["url"]
    short_mp3_link = spotify_data["spotify_response"]["tracks"][spotify_data["random_song"]]["preview_url"]
    feelslike = weather_data["weather"]["current"]["feelslike"]
    current_conditions = str(weather_data["weather"]["current"]["weather_descriptions"]).strip("['']")
    current_conditions = current_conditions.lower()
    city_name = weather_data["weather"]["location"]["name"]

    final_recommendation = {
        "name_of_song": name_of_song,
        "artist_name": artist_name,
        "web_address": web_address,
        "album_cover_image": album_cover_image,
        "short_mp3_link": short_mp3_link,
        "feelslike": feelslike,
        "current_conditions": current_conditions,
        "city_name": city_name,
    }

    return final_recommendation
```

## Now

Before I wrote the guts of this method, I would write something like this:

```python
class Recommendation:
    def parse_recommendation(
        self, spotify_result: SpotifyResult, weather_report: Weather_Report
    ) -> None | ReportError:
        pass
```
This way, Pyright will complain if I create something that doesn't produce
`None` or 'ReportError'.

'ReportError would have a pydantic baseclass I've defined in 'errors.py'.

## Then: Couldn't get CI/CD to work. Now: GitHub Actions, flake8, Pyright, and Black.

### Then

I'm proud of myself for deploying my app to Google Cloud Run with a Docker
container. I tried to configure Github actions but couldn't figure out how to do
it.


### Now

Now, I use Github Actions for my CI/CD. Here is an example Github workflow file for my tests:

```yaml
name: Tests

on:
  push:
    branches:
      - main
  pull_request:
    branches:
      - main

jobs:
  test:
    runs-on: ubuntu-latest

    steps:
    - name: Set up Python
      uses: actions/setup-python@v2
      with:
        python-version: 3.x  # Specify the desired Python version

    - name: Checkout code
      uses: actions/checkout@v2

    - name: Install Poetry
      run: |
        curl -sSL https://install.python-poetry.org | python3 -

    - name: Install dependencies
      run: poetry install  # Install project dependencies using Poetry

    - name: Run tests
      run: poetry run pytest  # Run tests using Poetry
```

Now that I'm writing unit and integration tests consistently, as well as using
flake8, Pyright, and Black for formatting, type checking, and linting, it is
easier for me to enforce a consistent approach to types and formatting.

## Then: requirements.txt. Now: Poetry.

### Then
I'm glad I figured out how to use venv and generate a requirements.txt.

### Now

Since I'm using ML-related libraries and models, dependency management has
become even more important. I use Poetry because it helps me with dependency
management, virtual environment management, and versioning. In the future, it'll
even help me publish packages to PyPi.

## Then: Pushing to main all the time. Now: Branches, pull requests, and code review.

### Then

I used branches every once in awhile, but you'll notice most commits went
directly to main.

### Now

Since I've gone through the code review and merge process at my job, I know the
importance of feature branches and pull requests. It's also great how [Linear](https://linear.app/) helps me link my branches to our internal issues (I also use Linear for my personal projects). I try to use the [Conventional Commits](https://www.conventionalcommits.org/en/v1.0.0/) spec for my commit messages.

After a project becomes public, I try to commit to branches.

## Then: VSCode + life on the desktop. Now: LunarVim + life in the terminal.

### Then

On my first day of work, I noticed that our most senior engineers used Vim. I
was impressed by how quickly they could navigate, read, and edit code using
motions. After [reading this incredible blog post](https://www.ssp.sh/blog/why-using-neovim-data-engineer-and-writer-2023/) about the mental model needed to adopt vim, I downloaded NeoVim and started learning it. After trying a few different configs, I've settled on LunarVim with my own tweaks.

Although I'd consider myself to be a Vim novice, I can navigate code files very
quickly and use LSP features to make my life easier (find, rename, diagnostics, etc). I've configured flake8, Black, and Pyright, so my files are linted and type checked as they are saved. I still have a lot to learn, but each week I gain a little speed. I've also noticed that I live more in the terminal in general. I'm more likely to look for CLIs or terminal commands to do my work.  

## Then: Rubber ducking and troubleshooting with Google. Now: PEPs, documentation, source code, and ChatGPT4. 

### Then

I didn't have ChatGPT when writing the code for Spheri. I had to teach myself
how to Google around for answers, read books (I got myself an O'Reilly
subscription), and copy-paste error messages into StackOverflow and Google.

### Now

I am much more likely to read source code directly. This was a big jump in my
development. I've even started to edit the sourcecode of open source projects to
fit my needs in my projects. I also look to package documentation and PEPs instead of blog posts
and YouTube videos. I read books for more wholistic computer science and system
design concepts.

# What's next?

At work, I'm trying to soak in as much as I can from our senior devs. My role
has evolved to that of a Product Engineer. I spend half of my time coding and
the other half talking to users, researching, and prototyping new product ideas. 

At night and on the weekends, I've been working on my CLI project. I hope to open source my CLI by the end of the summer.

I'm no expert programmer, but if you're learning how to code and want someone to
bounce project-based learning ideas off of, please reach out to me. 
