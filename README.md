# This or That Application

## What exactly is this app doing?
This app enables users to create multiple comparison rooms where votes determine the rankings. Users can set up a room, list items or people for comparison, and invite others to vote. It is inspired by Mark Zuckerberg's [Facemash application](https://thesocialnetwork.fandom.com/wiki/Facemash).

## Functionalities currently supported
- A user can create a room and list the items or people they want others to vote on.
- A user can edit a room, insert new items or people (let's call it `Object of Comparison`), or delete existing items or people.
- A user can get a room, vote, and have the `rating score` updated on the backend.
- A user can see the leaderboard of that particular room, which contains a list of active `Object of Comparison` sorted in descending order of the `rating score`.

## What exactly is this `rating score`?
This `rating score` is [Elo-rating](https://en.wikipedia.org/wiki/Elo_rating_system#Theory). Let's say two `Object of Comparison` compete, Object A and Object B. Object A has a rating of 1500 ($R_A = 1500$), while Object B has a rating of 1000 ($R_B = 1000$). We can calculate the expected score of Object A (expressed as $E_A$) and the expected score of Object B ($E_B$) using below's formula:

$$ E_A = {1 \over 1 + 10 ^ {(R_B - R_A)/400}} $$

$$ E_B = {1 \over 1 + 10 ^ {(R_A - R_B)/400}} $$

Inserting $R_A = 1500$, $R_B = 1000$, we got $E_A = 0.948$ and $E_B = 0.052$ (Notice that $E_A + E_B = 1$). This means that, if the winning score is 1, Object A is expected to score 0.948 points, while Object B is expected to score only 0.052.

To calculate the new score of Object A ($R_A'$) and the new score of ($R_B'$), we can use this formula:

$$R_A' = R_A + K(S_A - E_A)$$ 

$$R_B' = R_B + K(S_B - E_B)$$

Where $S_A$ and $S_B$ are the scores of A and B respectively. If Object A wins, then $S_A = 1$ and $S_B = 0$. Likewise, if Object 2 wins, then $S_A = 0$ and $S_B = 1$. The adjustment factor, the $K$, is set to 32.

If Object A wins the game against Object B, plugging in the value for new rating, we got $R_A' = 1502$ and $R_B' = 998$ (difference in rating $\pm 2$). On the contrary, if Object B pulls a surprise win against Object A, we get $R_A' = 1470$ and $R_B' = 1030$ (difference in rating $\pm 30$). This happens since it is expected that Object A will win against Object B. If for some reason, Object B wins against Object A, the increase in rating will be significant to help "course-correct".

## APIs developed

### 1. GET /GetPlayers

<p align="center">
  <img src="https://github.com/mattboentoro/ThisOrThatDocumentation/blob/main/getPlayerDiagram.png" alt="GetPlayers Diagram"/>
</p>

This API is used to get the list of players. We have 3 parameters, `roomId` being the compulsory parameter, both `sorted` and `unfiltered` are optional. This API will trigger a Lambda function that will get the room details from MongoDB table. If the user sets `sorted` to be true (mainly used in the game scenario and leaderboard), the response will be sorted according to the `playerRating` value. If the user sets the `unfiltered` value to be true (mainly used in editing the room scenario), the response will have the items marked as deleted. Read more on why I decided to keep the soft delete approach below.

```
REQUEST:
GET /getPlayers?<Parameter>

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

<p align="center">
  <img src="https://github.com/mattboentoro/ThisOrThatDocumentation/blob/main/createNewRoom.png" alt="GetPlayers Diagram"/>
</p>

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

<p align="center">
  <img src="https://github.com/mattboentoro/ThisOrThatDocumentation/blob/main/editRoom.png" alt="GetPlayers Diagram"/>
</p>

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

#### Why do I decide to mark the deleted `Object of Comparison` (soft delete) rather than actually deleting the `Object of Comparison` (hard delete)?
Consider a scenario with two users, User A and User B. User A caches data for one round, which can cause problems if User B deletes an `Object of Comparison` in the room that User A is in. If this deletion occurs, User A's message request in the SQS queue could fail because the Lambda function will try to access the now-deleted `Object of Comparison` from the MongoDB.

To avoid this issue, we can mark the `Object of Comparison` as deleted instead of immediately removing them. This way, the `UpdateRating` Lambda function can still update User A’s SQS message to update the score, even though User B has deleted the `Object of Comparison`. In the following round, User A will receive the most recent data and will no longer see the deleted `Object of Comparison`.

### 4. POST /UpdateRanking

<p align="center">
  <img src="https://github.com/mattboentoro/ThisOrThatDocumentation/blob/main/updateRating.png" alt="GetPlayers Diagram"/>
</p>

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

#### Alternatives
- Alter the DB directly from POST API.
- Tradeoff: pros (cost, easy to setup), cons (not scalable)
