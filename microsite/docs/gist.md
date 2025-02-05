---
layout: docs
title: Gist API
permalink: gist
---

# Gist API

Github4s supports the [Gist API](https://developer.github.com/v3/gists/). As a result,
with Github4s, you can:

- [Create a gist](#create-a-gist)
- [Get a single gist or specific revision of a gist](#get-a-single-gist-or-specific-revision-of-a-gist)
- [Edit a gist](#edit-a-gist)

The following examples assume the following code:

```scala mdoc:silent
import cats.effect.IO
import github4s.Github
import org.http4s.client.{Client, JavaNetClientBuilder}

val httpClient: Client[IO] = JavaNetClientBuilder[IO].create // You can use any http4s backend

val accessToken = sys.env.get("GITHUB_TOKEN")
val gh = Github[IO](httpClient, accessToken)
```

## Create a gist

You can create a gist using `newGist`; it takes as arguments:

- the gist description.
- whether it is public or private.
- an association of file names and file contents where the contents are wrapped in
[GistFile][gist-scala]s.

To create a gist:

```scala mdoc:compile-only
import github4s.domain.GistFile
val gistfiles = Map(
  "token.scala" -> GistFile("val accessToken = sys.env.get(\"GITHUB_TOKEN\")"),
  "gh4s.scala"  -> GistFile("val gh = Github(accessToken)")
)
val newGist = gh.gists.newGist("Github4s entry point", public = true, gistfiles)
newGist.flatMap(_.result match {
  case Left(e)  => IO.println(s"Something went wrong: ${e.getMessage}")
  case Right(r) => IO.println(r)
})
```

The `result` on the right is the created [Gist][gist-scala].

See [the API doc](https://developer.github.com/v3/gists/#create-a-gist) for full reference.

## Get a single gist or specific revision of a gist

You can create a gist using `getGist`; it takes as arguments:

- the gist id (obtained via [creation of a gist](#create-a-gist), for ex.).
- optional sha of the gist revision.

To get a single gist:

```scala mdoc:compile-only
val singleGist = gh.gists.getGist("aa5a315d61ae9438b18d")
singleGist.flatMap(_.result match {
  case Left(e)  => IO.println(s"Something went wrong: ${e.getMessage}")
  case Right(r) => IO.println(r)
})
```

Similarly, to get a specific revision of a gist:

```scala mdoc:compile-only
val sepcificRevisionGist = gh.gists.getGist("aa5a315d61ae9438b18d", Some("4e481528046a016fc11d6e7d8d623b55ea11e372"))
sepcificRevisionGist.flatMap(_.result match {
  case Left(e)  => IO.println(s"Something went wrong: ${e.getMessage}")
  case Right(r) => IO.println(r)
})
```

The `result` on the right is the requested [Gist][gist-scala].

See [the API doc](https://developer.github.com/v3/gists/#get-a-single-gist) for full reference.

## Edit a gist

You can edit a gist using `editGist`; it takes as arguments:

- the gist id (obtained via [creation of a gist](#create-a-gist), for ex.).
- the gist description.
- an association of file names and optional file contents where the contents are wrapped in
[EditGistFile][gist-scala]s, if a new file name required, then it must be provided.

To edit a gist (change description, update content of _token.scala_, rename _gh4s.scala_ and remove _token.class_ file):

```scala mdoc:compile-only
import github4s.domain.EditGistFile
val editfiles = Map(
  "token.scala" -> Some(EditGistFile("lazy val accessToken = sys.env.get(\"GITHUB_TOKEN\")")),
  "gh4s.scala"  -> Some(EditGistFile("val gh = Github(accessToken)", Some("GH4s.scala"))),
  "token.class"  -> None
)

val updatedGist = gh.gists.editGist("aa5a315d61ae9438b18d", "Updated github4s entry point", editfiles)
updatedGist.flatMap(_.result match {
  case Left(e)  => IO.println(s"Something went wrong: ${e.getMessage}")
  case Right(r) => IO.println(r)
})
```

The `result` on the right is the updated [Gist][gist-scala].

See [the API doc](https://developer.github.com/v3/gists/#edit-a-gist) for full reference.

As you can see, a few features of the gist endpoint are missing. As a result, if you'd like to see a
feature supported, feel free to create an issue and/or a pull request!

[gist-scala]: https://github.com/47degrees/github4s/blob/main/github4s/src/main/scala/github4s/domain/Gist.scala
