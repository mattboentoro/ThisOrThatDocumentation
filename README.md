# This or That Application

<p align="center"><img src="https://github.com/mattboentoro/ThisOrThatDocumentation/blob/main/pictures/this-or-that-pic-1.png" width="600" alt="GetPlayers Diagram"/></p>
<p align="center"><a href="https://choose-this-or-that-3f8c620e977f.herokuapp.com/">Demo Link</a></p>

## What exactly is this app doing?
This app enables users to create multiple comparison rooms where votes determine the rankings. Users can set up a room, list items or people for comparison, and invite others to vote. It is inspired by Mark Zuckerberg's [Facemash application](https://thesocialnetwork.fandom.com/wiki/Facemash).

## Core functionalities supported
1. Users can create a room and list the items or people they want others to vote on.
<p align="center">
  <img src="https://github.com/mattboentoro/ThisOrThatDocumentation/blob/main/pictures/createNewRoomFlow.png" alt="GetPlayers Diagram"/>
  <br/><br/>
  <img src="https://github.com/mattboentoro/ThisOrThatDocumentation/blob/main/pictures/this-or-that-pic-7.png" width="600" alt="GetPlayers Diagram"/>
</p>

2. Users can edit a room, insert new items or people (let's call it `Object of Comparison`), or delete existing items or people.
<p align="center">
  <img src="https://github.com/mattboentoro/ThisOrThatDocumentation/blob/main/pictures/editRoomFlow3.png" alt="GetPlayers Diagram"/>
  <br/><br/>
  <img src="https://github.com/mattboentoro/ThisOrThatDocumentation/blob/main/pictures/this-or-that-pic-4.png" width="600" alt="GetPlayers Diagram"/>
  <br/><br/>
  <img src="https://github.com/mattboentoro/ThisOrThatDocumentation/blob/main/pictures/this-or-that-pic-5.png" width="600" alt="GetPlayers Diagram"/>
  <br/><br/>
  <img src="https://github.com/mattboentoro/ThisOrThatDocumentation/blob/main/pictures/this-or-that-pic-6.png" width="600" alt="GetPlayers Diagram"/>
</p>

3. Users can get a room, vote, and have the `rating score` updated on the backend.
<p align="center">
  <img src="https://github.com/mattboentoro/ThisOrThatDocumentation/blob/main/pictures/playFlow.png" alt="GetPlayers Diagram"/>
  <br/><br/>
  <img src="https://github.com/mattboentoro/ThisOrThatDocumentation/blob/main/pictures/this-or-that-pic-2.png" width="600" alt="GetPlayers Diagram"/>
</p>

4. Users can see the leaderboard of that particular room, which contains a list of active `Object of Comparison` sorted in descending order of the `rating score`.
<p align="center">
  <img src="https://github.com/mattboentoro/ThisOrThatDocumentation/blob/main/pictures/leaderboardFlow.png" alt="GetPlayers Diagram"/>
  <br/><br/>
  <img src="https://github.com/mattboentoro/ThisOrThatDocumentation/blob/main/pictures/this-or-that-pic-3.png" width="600" alt="GetPlayers Diagram"/>
</p>


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

### 1. GET `/GetPlayers`

<p align="center">
  <img src="https://github.com/mattboentoro/ThisOrThatDocumentation/blob/main/pictures/getPlayerDiagram.png" alt="GetPlayers Diagram"/>
</p>

This API is used to get the list of `Object of Comparison`. We have 3 parameters, `roomId` being the compulsory parameter, both `sorted` and `unfiltered` are optional. This API will trigger a Lambda function that will get the room details from MongoDB table. If the user sets `sorted` to be true (mainly used in the game scenario and leaderboard), the response will be sorted according to the `playerRating` value. If the user sets the `unfiltered` value to be true (mainly used in editing the room scenario), the response will have all items, including items that are marked as deleted. Read more on why I decided to keep the soft delete approach below.

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

---

### 2. POST `/CreateNewRoom`

<p align="center">
  <img src="https://github.com/mattboentoro/ThisOrThatDocumentation/blob/main/pictures/createNewRoom.png" alt="GetPlayers Diagram"/>
</p>

This API creates a room with a specified `roomId` and a list of players. When a new room is created, every player starts with a score of 1000, which is managed by the front end. The front end also validates whether a room with the given `roomId` already exists. When the user enters a `roomId`, a request is sent to the /getPlayers API to retrieve the current list of `Object of Comparison`. If no such room exists, the API is then called.

```
REQUEST:
POST /CreateNewRoom

Request Body:
{
  roomId: "12349"
  players:
    [
      {"playerId":"<uuid>", "playerRating":1000, "name":"Boeing 747", "image":"<random image link>", "status":"ACTIVE"},  
      {"playerId":"<uuid>", "playerRating":1000, "name":"Airbus A380", "image":"<random image link>", "status":"ACTIVE"}, 
      {"playerId":"<uuid>", "playerRating":1000, "name":"Airbus A350", "image":"<random image link>", "status":"ACTIVE"},
    ]
}
```

```
RESPONSE:
Status Code: 200
{"acknowledged":true,"insertedId":"<random-id>"}
```

---

### 3. POST `/EditRoom`

<p align="center">
  <img src="https://github.com/mattboentoro/ThisOrThatDocumentation/blob/main/pictures/editRoom.png" alt="GetPlayers Diagram"/>
</p>

This API sets the content of the `roomId` to be `players`. Essentially, all the logic of setting `DELETED` status is handled in the front-end. The back-end will loop through the changes, and make the change according to the `changes` from the request.

```
REQUEST:
POST /EditRoom

Request Body:
{
  roomId: "12349"
  changes:
    [
      {
          "playerId": "3",
          "change": "DELETE"
      },
      {
          "playerId": "5",
          "change": "EDIT",
          "values": {
              "name": "Porsche Macan"
          }
      },
      {
          "playerId": "9",
          "change": "CREATE",
          "values": {
              "playerId": "<uuid>",
              "name": "BMW iX",
              "image": "https://hips.hearstapps.com/hmg-prod/images/2023-bmw-ix-m60-108-1653422436.jpg?crop=0.728xw:0.820xh;0.160xw,0.0264xh&resize=768:*",
              "playerRating": 1000,
              "status": "ACTIVE"
          }
      }
    ]
}
```

```
RESPONSE:
Status Code: 200
"Success"
```

#### Why do I decide to mark the deleted `Object of Comparison` (soft delete) rather than actually deleting the `Object of Comparison` (hard delete)?
Consider a scenario with two users, User A and User B. User A caches data for one round, which can cause problems if User B deletes an `Object of Comparison` in the room that User A is in. If this deletion occurs, User A's message request in the SQS queue could fail because the Lambda function will try to access the now-deleted `Object of Comparison` from the MongoDB.

To avoid this issue, we can mark the `Object of Comparison` as deleted instead of immediately removing them. This way, the `UpdateRating` Lambda function can still update User A’s SQS message to update the score, even though User B has deleted the `Object of Comparison`. In the following round, User A will receive the most recent data and will no longer see the deleted `Object of Comparison`.

#### Why am I just keeping the changes log and not overwriting everything?

Let's consider two users, User A and User B, who attempt to access the edit room functionality simultaneously. If the system simply overwrites values, the final value retained in the system will be from the user who submits their editRoom request last, resulting in the earlier submission being overwritten.

If we implement a change log system, we can handle these requests in parallel without losing any information. Each request can be recorded as a distinct entry in the change log, capturing all changes made by both users. This way, the system does not need to wait for one request to complete before processing the next one, thereby potentially saving time and improving efficiency. Additionally, using a change log allows for better tracking of changes and conflict resolution, ensuring that all user edits are preserved and can be merged or reviewed as needed.

---

### 4. POST `/UpdateRating`

<p align="center">
  <img src="https://github.com/mattboentoro/ThisOrThatDocumentation/blob/main/pictures/updateRating.png" alt="GetPlayers Diagram"/>
</p>

When the user submits the `/updateRanking` POST API call, API Gateway then triggers the `SendToSQSQueueFunction` Lambda function upon receiving the POST request. This Lambda function processes the initial request and sends a message to the SQS queue. The SQS queue acts as a buffer, decoupling the request processing from the response. Subsequently, Worker `UpdateRatingFunction` Lambda functions poll the SQS queue, process the messages, and update the ratings on MongoDB.

I opted to cache the `playerRating` for all `Object of Comparison` on the front end. This means the Worker `UpdateRatingFunction` Lambda function won't need to constantly query MongoDB for ratings, reducing the number of database reads. However, this also means that until the round ends, comparison objects will retain their ratings as they were when the user first loaded the game on the front end. Once all pairs have been compared, the front end will invoke the `/getPlayers` API, resetting the cache and providing the user with accurate information at the time of the API request submission.

#### `UpdateRating` Lambda Function (Worker Function) Logic

I've chosen to calculate the difference in `Object of Comparison`'s ratings and delegate the task of incrementing the MongoDB value to the SQS Worker `UpdateRatingFunction` Lambda. With users having a cache of player ratings, we can utilize it to compute the difference between old and new ratings for both players. This difference is then incremented to MongoDB by the SQS Worker `UpdateRatingFunction` Lambda.

Why opt for incrementing/decrementing the difference instead of directly updating to the new rating in the Worker `UpdateRatingFunction` Lambda? Consider this scenario: two identical SQS messages arrive simultaneously from different users playing at the same time.

```
{
  "roomId": "12345",
  "losingPlayerId": "1",
  "losingPlayerRating": 1500,
  "winningPlayerId": "2",
  "winningPlayerRating": 1000
},
{
  "roomId": "12345",
  "losingPlayerId": "1",
  "losingPlayerRating": 1500,
  "winningPlayerId": "2",
  "winningPlayerRating": 1000
}
```

According to the earlier Elo-Score calculation, `Object of Comparison` with ID `2` should gain an additional `30 + 30 = 60` points, resulting in a rating of `1060`. However, if we directly set the value (instead of using increment) from the Worker Lambda, we'll only obtain `1030`, which was the last rating value written by the last message from the queue (as values get overwritten). Since the calculation is independent for each message using the information provided from the JSON, using an increment method ensures that the score can be updated to reflect all votes. This approach enables us to still obtain a close-to-accurate rating from the cached values.


```
REQUEST:
POST /UpdateRanking

Request Body:
{
  roomId: "12345",
  losingPlayerId: "3",
  losingPlayerRating: 909,
  winningPlayerId: "1"
  winningPlayerRating: 1105
}
```

```
RESPONSE:
Status Code: 200
{"message":"Message sent to SQS","messageId":"<randomId>"}
```

#### Architectural Alternative 1: Direct Handling (Not Recommended)

The other approach is to handle the update rating on the lambda which is directly invoked by the `/updateRating` API.

The client sends a POST request to the `/updateRating` API. The API Gateway receives this request and directly triggers a Lambda function. This Lambda function immediately handles the request, performs the necessary processing to update the rating, and returns a response to the client.

**Pros**:
  - Simpler architecture with fewer components.
  - Lower latency with immediate processing.
  - Potential cost savings from fewer services being used.

**Cons**:
  - Scalability issues and potential bottlenecks under high load.
  - Less fault tolerance and resilience without SQS.
  - More complex load management within a single Lambda function.


##### Chosen Alternative 2: Using SQS Approach
**Pros**:
  - Better scalability and load handling due to decoupling.
  - Improved fault tolerance with message retry mechanisms.
  - Resilience to traffic spikes, as SQS can buffer requests.

**Cons**:
  - Added complexity with more components to manage.
  - Potentially higher latency due to asynchronous processing.
  - Additional costs associated with SQS and multiple Lambda invocations.

#### Verdict
Since I want to make the system scalable, I decided to use the SQS approach.

---

## How do we get all players combinations to be paired?
I used slightly tweaked (Round Robin)[https://github.com/tournament-js/roundrobin?tab=readme-ov-file ] algorithm to get a pairing list of two `Object of Comparison`, such that all `Object of Comparison` will get to meet one another on one round. This is done on the front end, after the user gets the room information from `/getRooms` API.

## Future works

- [x] <s>Use React-Router to allow direct access to all pages from the URL (like `<URL>/play/<roomId>`).</s>
- [ ] Support room deletion.
- [x] <s>Support edit individual `Object of Comparison` after added.</s>
- [x] <s>Validate the API calls on the front-end, and return error message accordingly.</s>
- [ ] Support [versioning](https://www.liquibase.com/resources/guides/database-version-control#:~:text=Database%20versioning%20is%20applying%20version,changes%20with%20precision%20and%20reliability.) for `updateRoom`.

## Helpful links that helped me during the project:

- https://github.com/tournament-js/roundrobin?tab=readme-ov-file 
- https://www.mongodb.com/docs/manual/reference/operator/update/inc/
- https://www.mongodb.com/community/forums/t/mongodb-update-a-value-in-array-of-object-of-array/208302
- https://www.mongodb.com/developer/products/atlas/serverless-development-lambda-atlas/
- https://sadam-bapunawar.medium.com/add-and-remove-form-fields-dynamically-using-react-and-react-hooks-3b033c3c0bf5
