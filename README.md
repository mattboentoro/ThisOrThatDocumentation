# This or That Application

## What exactly is this app doing?
This app enables users to create multiple comparison rooms where votes determine the rankings. Users can set up a room, list items or people for comparison, and invite others to vote. It is inspired by Mark Zuckerberg's [Facemash application](https://thesocialnetwork.fandom.com/wiki/Facemash).

## Functionalities currently supported
- A user can create a room and list the items or people they want others to vote on.
- A user can edit a room, insert new items or people (let's call it `Object of Comparison`), or delete existing items or people.
- A user can get a room, vote, and have the `rating score` updated on the backend.
- A user can see the leaderboard of that particular room, which contains a list of active `Object of Comparison` sorted in descending order of the `rating score`.

## What exactly is this `rating score`?
This `rating score` is [Elo-rating](https://en.wikipedia.org/wiki/Elo_rating_system#Theory). Let's say two `Object of Comparison` compete, Object A and Object B. Object A has a rating of 1500, while Object B has a rating of 1000. We can calculate the expected score of a particular `Object of Comparison` to be

```

```

## APIs developed

### 1. GET /GetPlayers
This API is used to get the list of players
```
REQUEST:
GET /GetPlayers?<Parameter>

Parameter:
- roomId: <string>
- sorted: <boolean> [Optional]
- unfiltered: <boolean> [Optional]
```

```
RESPONSE: (when sorted=false and unfiltered = true, notice there is element with status:"DELETED")
[
  {"playerId":"0", "playerRating":985, "name":"Boeing 747", "image":"<random image link>", "status":"ACTIVE"},  
  {"playerId":"1", "playerRating":1105, "name":"Airbus A380", "image":"<random image link>", "status":"ACTIVE"}, 
  {"playerId":"2", "playerRating":1001, "name":"Airbus A350", "image":"<random image link>", "status":"ACTIVE"},
  {"playerId":"3", "playerRating":909, "name":"Boeing 737", "image":"<random image link>","status":"ACTIVE"},
  {"playerId":"4","playerRating":"1000","name":"Boeing 767","image":"<random image link>","status":"DELETED"}
]
```


### 2. POST /CreateNewRoom
```
REQUEST:
POST /CreateNewRoom

Request Body:
{

}
```

```
RESPONSE:
```

### 3. POST /EditRoom
```
REQUEST:
POST /EditRoom

Request Body:
{

}
```

```
RESPONSE:
```

### 4. POST /UpdateRanking
```
REQUEST:
POST /UpdateRanking

Request Body:
{

}
```

```
RESPONSE:
```

## Why do I decide to keep the deleted 
