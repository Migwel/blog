---
layout: post
title: "Deserialize an inconsistent API with Jackson"
date: 2020-11-29 00:00:00
tags: programming
excerpt: "You can't predict the weather"
---

# Introduction

With the beginning of COVID-related lockdowns, chess has known a boom of popularity and the reference when you want to learn or play chess is [chess.com](https://www.chess.com). Even though their website is rather well done, their API has some flaws and one of the most annoying is the lack of consistency in the responses returned by their endpoints. Let's use the game participant as an example (referred as white and black). Following [their documentation](https://www.chess.com/news/view/published-data-api), if you want to get all the ongoing games for a player, here's what you'll get back:

```json
{
  "games": [
    {
      "white": "string", // URL of the white player's profile
      "black": "string", // URL of the black player's profile
      ...
    }
  ]
}
```

And if you want to get all the finished games for a player during a month, here's what you'll get:

```json
{
  "games": [
    {
      "white": { // details of the white-piece player:
          "username": "string", // the username
          "rating": 1492, // the player's rating at the start of the game
          "result": "string", // see "Game results codes" section
          "@id": "string", // URL of this player's profile
      },
      "black": { // details of the black-piece player:
          "username": "string", // the username
          "rating": 1942, // the player's rating at the start of the game
          "result": "string", // see "Game results codes" section
          "@id": "string", // URL of this player's profile
      },
      ...
    }
  ]
}
```

Ok, this is slightly annoying but not too big of a deal, we can simply deal with such situations by using different objects for different calls.

Where it becomes more irritating is that for some calls, both formats are used. Sometimes you'll get the string, sometimes the object. What can we do about that?

The first step is [to complain](https://www.chess.com/clubs/forum/view/inconsistency-in-white-black-format-in-tournamentroundgroup) because you'll feel good afterwards. The second one is to look into the possibilities provided by our deserialization framework.

# Introduction to Jackson

In my case, I'm using Jackson. If you don't know it, Jackson is one of the most used frameworks in the Java world to serialize objects from Java to JSON (or XML, or Avro, or protobuf, ...) or deserialize from JSON to Java. It's useful when you need to store some object in your database or when you interact with endpoint that return responses in JSON format.

Here's a very simple example:

```java
public class NameDeserializationTest {

    private static record Name(String firstName, String lastName){}

    @Test
    void testSerializeSimpleObject() throws JsonProcessingException {
        ObjectMapper objectMapper = new JsonMapper();
        Name name = new Name("Donald", "Duck");
        assertEquals("{\"firstName\":\"Donald\",\"lastName\":\"Duck\"}", objectMapper.writeValueAsString(name));
    }

    @Test
    void testDeserializeSimpleObject() throws JsonProcessingException {
        ObjectMapper objectMapper = new JsonMapper();
        String json = "{\"firstName\":\"Donald\",\"lastName\":\"Duck\"}";
        Name name = objectMapper.readValue(json, Name.class);
        assertEquals("Donald", name.firstName);
        assertEquals("Duck", name.lastName);

    }
}
```

Very good, Jackson works well and is easy to use but what can we do to handle inconsistencies in the received JSON? We can write our own deserialization method!

# Home-made deserialization

First, we need to create a child class of `StdDeserializer` and override the `deserialize` method.

```java
public class ParticipantDeserializer extends StdDeserializer<Participant> {

    private final static String URL_REGEXP = "https:\\/\\/api.chess.com\\/pub\\/player\\/(.+)";
    private final static Pattern urlPattern = Pattern.compile(URL_REGEXP);
...

    @Override
    public Participant deserialize(JsonParser jsonParser, DeserializationContext ctxt) throws IOException {
        String participantUrl = jsonParser.getValueAsString();
        if (participantUrl == null) {
            return jsonParser.readValueAs(Participant.class);
        }

        Matcher matcher = urlPattern.matcher(participantUrl);
        String username = participantUrl;
        if (matcher.find()) {
            username = matcher.group(1);
        }
        return new Participant(username, participantUrl);
    }
}
```

In our application, whatever we receive back from chess.com, we always want to create a Participant object which consists of a username and a URL. We first verify if what we received back is a String. If it's not the case, we don't need to do anything and we can delegate the deserialization to Jackson.

But if we did receive a string, we'll create a new instance of Participant "by hand". There's a bit of additional logic here because the string returned always seems to follow the format [`https://api.chess.com/pub/player/username`](https://api.chess.com/pub/player/username) so we try and get the username out of it. If this username extraction doesn't work, we give up and use the participantUrl as username (this is an arbitrary decision; we can easily imagine leaving the username empty in such situations).

Now that we have our custom deserializer, what's left is to add annotations to inform Jackson to use it on the white and black fields by using `@JsonDeserialize`.

```java
public record GroupGame(@JsonProperty("white") @JsonDeserialize(using = ParticipantDeserializer.class) Participant white,
                        @JsonProperty("black") @JsonDeserialize(using = ParticipantDeserializer.class) Participant black,
                        ...)
{}
```

(The `@JsonProperty` annotation let us define the field name in the JSON message. Another useful Jackson annotation!)

# Conclusion

I hope this has helped some of you who need to integrate with inconsistent APIs. The code used in this post is available on [Github](https://github.com/Migwel/ChessComJava/), and more specifically [this commit](https://github.com/Migwel/ChessComJava/commit/e682d6664f078ed171f1b3fad5b5583cb48b7831) created to fix this specific issue.
